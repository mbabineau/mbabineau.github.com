---
layout: post
title: "Introducing Cloudviz"
description: ""
category: 
tags: [aws, cloudwatch, cloudviz, open source, ops, monitoring]
---
{% include JB/setup %}

*(originally posted on Bizo's dev blog [here](http://dev.bizo.com/2010/03/introducing-cloudviz.html))*

[Amazon CloudWatch](http://aws.amazon.com/cloudwatch/) exposes a variety of useful metrics for EC2 instances, Elastic Load Balancers, and more. Unfortunately, it is tedious to query directly and the results can be difficult to interpret.

Like most operational metrics, CloudWatch data provides the most insight when graphed. While there are existing tools to graph CloudWatch data, they are only available as part of a proprietary suite or service and, generally, they sacrifice customization and flexibility for ease-of-use.

Here at Bizo, we wanted to incorporate CloudWatch data into operational dashboards. Nothing we found was flexible enough to meet our needs, so we decided to write our own. We are now releasing it to for all to use.

I'm pleased to introduce [cloudviz](http://github.com/mbabineau/cloudviz), an open source tool for creating embeddable CloudWatch graphs.

Specifically, cloudviz is a data source that exposes CloudWatch data for graphing by [Google Chart Tools](https://developers.google.com/chart/). It's written in Python using Google's [Data Source library](https://developers.google.com/chart/interactive/docs/dev/gviz_api_lib) and Mitch Garnaat's excellent AWS interface, [boto](http://code.google.com/p/boto).

With cloudviz, it's easy to create graphs like these:
![example host cpu](/img/cloudviz-example-hosts-cpu.png)
![example elb request count](/img/cloudviz-example-elb-requestcount.png)

I encourage you to check out the project on GitHub [here](http://github.com/mbabineau/cloudviz). There's a fairly detailed README and plenty of examples, but feel free to drop me a line if you have any questions, [michael.babineau@gmail.com](michael.babineau@gmail.com).

Happy graphing!