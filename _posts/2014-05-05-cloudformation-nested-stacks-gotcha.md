---
layout: post
title: "List Params and Nested Stacks on CloudFormation"
description: ""
category: 
tags: [aws, cloudformation]
---

While creating a nested stack in CloudFormation, you may see a failure with this cryptic message:

`Value of property Parameters must be an object`

The full event will look something like this:

```json
        {
            "StackId": "arn:aws:cloudformation:us-west-2:760937633930:stack/my_nested_stack1/32499563-d49c-21e3-a914-302bfc8340a6", 
            "EventId": "MyNestedStack-CREATE_FAILED-1399325327000", 
            "ResourceStatus": "CREATE_FAILED", 
            "ResourceType": "AWS::CloudFormation::Stack", 
            "Timestamp": "2014-05-05T21:28:47Z", 
            "ResourceStatusReason": "Value of property Parameters must be an object", 
            "StackName": "my_nested_stack1", 
            "PhysicalResourceId": null, 
            "LogicalResourceId": "MyNestedStack"
        }
```

Not very helpful. And a Google search turned up nothing.

Chances are, you're trying to pass a list as a parameter for a child stack. `Parameter` objects can only accept Strings and Numbers, not lists.

You may be passing a list inadvertently with a `CommaDelimitedList` parameter in the parent:

```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  
  "Description" : "Launches a nested stack",
  
  "Parameters" : {
    "TemplateUrl" : {
      "Description" : "URL for S3-hosted CloudFormation template",
      "Type" : "String"
    },
    "FooList" : {
      "Description" : "List of Foos (subnets, availability zones, whatever)",
      "Type" : "CommaDelimitedList"
    }
  }

  "Resources" : {
    "MyNestedStack" : {
      "Type" : "AWS::CloudFormation::Stack",
      "Properties" : {
        "TemplateURL" : { "Ref" : "TemplateUrl" },
        "Parameters" : {
          "FooList" : { "Ref" : "FooList" }
        }
      }
    }
  }

}
```

The problem here is that CloudFormation has already converted `FooList` from a string to a list.

In this case, you can simply defer parsing by treating `FooList` as a String in the parent:

```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  
  "Description" : "Launches a nested stack",
  
  "Parameters" : {
    "TemplateUrl" : {
      "Description" : "URL for S3-hosted CloudFormation template",
      "Type" : "String"
    },
    "FooList" : {
      "Description" : "List of Foos (subnets, availability zones, whatever)",
      "Type" : "String"
    }
  }

  "Resources" : {
    "MyNestedStack" : {
      "Type" : "AWS::CloudFormation::Stack",
      "Properties" : {
        "TemplateURL" : { "Ref" : "TemplateUrl" },
        "Parameters" : {
          "FooList" : { "Ref" : "FooList" }
        }
      }
    }
  }

}
```

A more general solution is to assemble the list into a string with `Fn::Join` and pass that through instead:

```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  
  "Description" : "Launches a nested stack",
  
  "Parameters" : {
    "TemplateUrl" : {
      "Description" : "URL for S3-hosted CloudFormation template",
      "Type" : "String"
    },
    "FooList" : {
      "Description" : "List of Foos (subnets, availability zones, whatever)",
      "Type" : "CommaDelimitedList"
    }
  }

  "Resources" : {
    "MyNestedStack" : {
      "Type" : "AWS::CloudFormation::Stack",
      "Properties" : {
        "TemplateURL" : { "Ref" : "TemplateUrl" },
        "Parameters" : {
          "FooList" : {"Fn::Join" : [ ",", { "Ref" : "FooList" }] }
        }
      }
    }
  }

}
```

Hope this helps!