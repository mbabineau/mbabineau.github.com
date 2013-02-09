---
layout: post
title: "loggly-python"
tagline: "An Open Source Library"
description: ""
category: 
tags: [loggly, python, open source, ops, logging]
---
{% include JB/setup %}

*(originally written for EA2D's engineering blog)*

For centralized logging, we use a service called [Loggly](http://loggly.com).  We forward our logs to Loggly and aggregate them by application and environment.  This gives us a handy web interface for viewing logs across all servers within an application group, and provides some great tools for search, comparison, and alerting.

We send events to Loggly using syslog over TCP, and these events are bucketed based on destination port.  For security, Loggly locks down each port to a list of authorized IP addresses.

Since we make heavy use of EC2 [Auto Scaling Groups](http://aws.amazon.com/autoscaling/), we simply can not maintain this authorized IP list manually.  Additionally, we're constantly launching new applications and environments, so our list of buckets (Loggly "inputs") is in constant flux.

Fortunately, Loggly has exposed a set of administration [APIs](http://wiki.loggly.com/apidocumentation) for managing inputs and authorized devices.  Since no library was available, we ended up writing one ourselves (in Python) and releasing it as open source.

### Getting the library

You can find it on GitHub:

[https://github.com/mbabineau/loggly-python](https://github.com/mbabineau/loggly-python)

Or install it from PyPI:

    sudo easy_install loggly

### Batteries-included

This package includes scripts for managing inputs and devices.  To use them, simply set up your credentials:

    export LOGGLY_USERNAME='someuser'
    export LOGGLY_PASSWORD='somepassword'
    export LOGGLY_DOMAIN='somesubdomain.loggly.com'

Create an input:

    $ loggly-create-input -i testinput -s syslogtcp
    Creating input "testinput" of type "syslogtcp"
    Input:testinput2

Add a device to an input:

    $ loggly-add-device -i testinput -d 192.168.1.1
    Adding device "192.168.1.1" to input "testinput"
    Device:192.168.1.1

Delete a device:

    $ loggly-remove-device -d 192.168.1.1
    Removing device "192.168.1.1" from all inputs

Delete an input:

    $ loggly-delete-input -i testinput
    Deleting input testinput

Enjoy!