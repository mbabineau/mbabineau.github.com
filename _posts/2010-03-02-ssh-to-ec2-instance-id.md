---
layout: post
title: "SSH to EC2 Instance ID"
description: ""
category: 
tags: []
---
{% include JB/setup %}

I often find myself looking up EC2 nodes by instance ID so I can grab the external DNS name and SSH in. Fed up with the extra “ec2-describe-instance , copy, paste” layer, I threw together a function (basically a fancy alias) to SSH into an EC2 instance referenced by ID.

Assuming you’re on Mac OS X / Linux, just put this somewhere in `~/.profile`, reload your terminal, and you’re good to go.  Alternatively, you can use the [shell script](https://gist.github.com/mbabineau/319882#file_ssh_instance.sh) version.

Update (3/5/10): Added region support

{% render_gist https://gist.github.com/mbabineau/319882/raw/4116128eb09ebc293bfc111941dd1091671036b9/ssh-instance-function.sh bash %}

And the [script version](https://gist.github.com/mbabineau/319882/raw/3cb3f1ea64e4e752e033ca472c04ee547fa042c3/ssh-instance.sh)
