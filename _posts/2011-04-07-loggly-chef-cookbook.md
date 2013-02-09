---
layout: post
title: "Chef Cookbook for Loggly"
description: ""
category: 
tags: [loggly, open source, chef, ops]
---
{% include JB/setup %}

*(originally written for EA2D's engineering blog)*

As mentioned in a [previous post](http://eng.ea2d.com/loggly-python-an-open-source-library), we aggregate and store our logs using a service called  [Loggly](http://loggly.com).

Since we wrote a [library](https://github.com/mbabineau/loggly-python) for programmatically managing Loggly inputs and devices, it was only natural for us to integrate it with our [Chef](http://wiki.opscode.com/display/chef/Home) deployment.

We've written and open sourced a Chef cookbook for Loggly.

From the [README](https://github.com/mbabineau/loggly-cookbook#readme):
> Installs the loggly-python library and provides a definition for the configuration of Loggly logging.
> 
> More specifically, the loggly_conf definition will configure rsyslog to watch a log file and send its lines to a Loggly input. When first run, loggly_conf will create the input and authorize the server to publish events to that input.
> 
> Developed for and tested on Ubuntu 10.10 LTS

You can grab it from the Opscode site:

    $ knife cookbook site vendor loggly

Or from our GitHub repository:

[https://github.com/mbabineau/loggly-cookbook](https://github.com/mbabineau/loggly-cookbook)