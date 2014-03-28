---
layout: post
title: "Multi Region Gotcha on Elastic Beanstalk"
description: ""
category: 
tags: [aws, elastic beanstalk]
---
{% include JB/setup %}

It's not in the current Elastic Beanstalk documentation, but you can't create a new application version from an S3 file hosted in a different region. Attempts to do so will return this error:
{% highlight console %}
$ aws elasticbeanstalk create-application-version --region ap-southeast-1 --application-name myapp --version-label myversion --source-bundle '{"S3Bucket":"mybuilds", "S3Key":"myapp-myversion.war"}'
{
    "Errors": [
        {
            "Message": "Unable to download from S3 location (Bucket: mybuilds  Key: myapp-myversion.war). Reason: Moved Permanently", 
            "Code": "InvalidParameterCombination", 
            "Type": "Sender"
        }
    ], 
    "ApplicationVersion": {}, 
    "ResponseMetadata": {
        "RequestId": "bfaf70b6-0aae-11e3-ae62-0d8638135266"
    }
}
{% endhighlight %}


If you want to create an application version in multiple regions, you'll need a location-constrained bucket for each region. It's a good pattern to include the region in the bucket name:
{% highlight console %}
$ aws s3 create-bucket --bucket mybuilds-ap-southeast-1 --create-bucket-configuration '{"LocationConstraint":"ap-southeast-1"}'
$ aws s3 create-bucket --bucket mybuilds-eu-west-1 --create-bucket-configuration '{"LocationConstraint":"eu-west-1"}'
$ aws s3 create-bucket --bucket mybuilds-sa-east-1 --create-bucket-configuration '{"LocationConstraint":"sa-east-1"}'
...
{% endhighlight %}

Now, just use the regional bucket for each `create-application-version` call:
{% highlight console %}
$ aws elasticbeanstalk create-application-version --region ap-southeast-1 --application-name myapp --version-label myversion --source-bundle '{"S3Bucket":"mybuilds-ap-southeast-1", "S3Key":"myapp-myversion.war"}'
{
    "ApplicationVersion": {
        "ApplicationName": "myapp", 
        "VersionLabel": "myversion", 
        "SourceBundle": {
            "S3Bucket": "mybuilds-ap-southeast-1", 
            "S3Key": "myapp-myversion.war"
        }, 
        "DateUpdated": "2013-08-21T22:12:32.738Z", 
        "DateCreated": "2013-08-21T22:12:32.738Z"
    }
}
{% endhighlight %}