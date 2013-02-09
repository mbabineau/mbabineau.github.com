---
layout: post
title: "pingdom-python"
tagline: "An Open Source Library"
description: ""
category: 
tags: [api, monitoring, open source, pingdom, python, ops]
---
{% include JB/setup %}

[Pingdom](http://pingdom.com) is one of several monitoring tools we use at EA2D.  Besides alerting us when things go down, we query Pingdom's API to include check status in our dashboards.

The old Pingdom SOAP API was unwieldy and slow.  Fortunately, Pingdom [released](http://royal.pingdom.com/2011/03/22/new-pingdom-api-enters-public-beta/) a new, JSON-ified REST API that remedied the problems of its predecessor.

I've written a Python library for this new API and released it as open source.  For now, it supports only a subset of available resources, but the framework is there for others to be added easily.

[https://github.com/mbabineau/pingdom-python](https://github.com/mbabineau/pingdom-python)

### pingdom-python in action

Set up a Pingdom connection:

    >>> import pingdom
    >>> c = pingdom.PingdomConnection(PINGDOM_USERNAME, PINGDOM_PASSWORD)  # Same credentials you use for the Pingdom website

Create a new Pingdom check:

    >>> c.create_check('EA2D Website', 'ea2d.com', 'http')
    Check:EA2D Website
    
Get basic information about a Pingdom check:

    >>> check = c.get_all_checks(['EA2D Website'])[0]   # Expects a list, returns a list
    >>> check.id
    302632
    >>> check.status
    u'up'

Get more detailed information about a Pingdom check:

    >>> check = c.get_check(210702)  # Look up by check ID
    >>> check.lasterrortime
    1289482981

Delete a Pingdom check:

    >>> c.delete_check(302632)
    {u'message': u'Deletion of check was successful!'}

Go check it out on [GitHub](https://github.com/mbabineau/pingdom-python), or install it directly from PyPI:

    sudo easy_install pingdom

If you have any questions, drop me a line: [michael.babineau@gmail.com](mailto:michael.babineau@gmail.com).