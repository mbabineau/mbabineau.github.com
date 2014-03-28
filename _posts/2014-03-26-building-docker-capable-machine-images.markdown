---
layout: post
title: "Building Docker-Capable Machine Images"
description: ""
category: 
tags: [docker, packer, ec2, ami]
---

{% include JB/setup %}

Docker allows you to create lightweight and portable containers that encapsulate any application. Your app and its runtime environment are packaged together. Starting your app requires only Docker and your container.

Docker installation is simple, but takes a non-trivial amount of time to complete. Baking Docker into your machine image has the desired effect of minimizing provisioning time, but image creation is typically a hassle.

Enter Packer. Packer simplifies the creation of machine images for EC2, Digital Ocean, Vagrant, and many other virtual environments. By defining a basic Packer template, creating Docker-capable images can be done with a single command.

Here is our Packer template:
{% highlight json %}
{
  "variables": {
    "docker_version": "0.9.1"
  },

  "builders": [
    {
      "type": "amazon-ebs",
      "region": "us-west-2",
      "source_ami": "ami-c8bed2f8",
      "instance_type": "m1.small",
      "ssh_username": "ubuntu",
      "ami_name": "ubuntu-12.04-docker-{{isotime | clean_ami_name}}",
      "tags": {
        "Release": "12.04 LTS"
      }
    }
  ],

  "provisioners": [
    {
      "type": "shell",
      "inline": [
        "# Source: http://docs.docker.io/en/latest/installation/ubuntulinux/#ubuntu-precise"
        "sudo apt-get update",
        "sudo apt-get install -y linux-image-generic-lts-raring linux-headers-generic-lts-raring",
        "sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9",
        "sudo sh -c 'echo deb http://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list'",
        "sudo apt-get update",
        "sudo apt-get -y install lxc-docker={% raw %}{{user `docker_version`}}{% endraw %}"
      ]
    }
  ]
}
{% endhighlight %}

We take a base Ubuntu 12.04 LTS image and install Docker on it (as per the [official guide](http://docs.docker.io/en/latest/installation/ubuntulinux/#ubuntu-precise)) using the script defined in `provisioners`. Other provisioners are supported: you could swap the shell script out for (or append) Chef, Ansible, or another [supported provisioner](http://www.packer.io/docs/templates/provisioners.html).

This template will create an EC2 AMI. To create other images, simply replace/append the builder with one for [another provider](http://www.packer.io/docs/templates/builders.html).

Note we specify the Docker version in a variable. [Variables](http://www.packer.io/docs/templates/user-variables.html) can be used throughout the template with ```{% raw %}{{user `var_name`}}{% endraw %}```.

Before we build our image on EC2, we'll need to export our AWS credentials as environment variables:
{% highlight console %}
$ export AWS_ACCESS_KEY_ID="your_access_key"
$ export AWS_SECRET_ACCESS_KEY="your_secret_key"
{% endhighlight %}

To kick off the build, we invoke `packer build`:
{% highlight console %}
$ packer build foo.json
amazon-ebs output will be in this color.

==> amazon-ebs: Creating temporary keypair: packer 5234c1f7-b8df-871e-9617-61982c79fe01
==> amazon-ebs: Creating temporary security group for this instance...
==> amazon-ebs: Authorizing SSH access on the temporary security group...
==> amazon-ebs: Launching a source AWS instance...
    amazon-ebs: Instance ID: i-bef803c2
==> amazon-ebs: Waiting for instance (i-bef803c2) to become ready...
==> amazon-ebs: Waiting for SSH to become available...
==> amazon-ebs: Connected to SSH!
==> amazon-ebs: Provisioning with shell script: /var/folders/pk/v470lvv12bl_7f3r041ccbc40000gn/T/packer-shell287742457
    amazon-ebs: Get:1 http://security.ubuntu.com precise-security Release.gpg [198 B]
    amazon-ebs: Hit http://archive.ubuntu.com precise Release.gpg
    amazon-ebs: Get:2 http://security.ubuntu.com precise-security Release [49.6 kB]
[... CUT ...]
    amazon-ebs: docker start/running, process 11064
    amazon-ebs: Setting up lxc-docker (0.9.1) ...
    amazon-ebs: Processing triggers for libc-bin ...
    amazon-ebs: ldconfig deferred processing now taking place
==> amazon-ebs: Stopping the source instance...
==> amazon-ebs: Waiting for the instance to stop...
==> amazon-ebs: Creating the AMI: ubuntu-12.04-docker-2014-03-28T00-27-35Z
    amazon-ebs: AMI: ami-db237fec
==> amazon-ebs: Waiting for AMI to become ready...
==> amazon-ebs: Adding tags to AMI (ami-db237fec)...
    amazon-ebs: Adding tag: "Release": "12.04 LTS"
==> amazon-ebs: Terminating the source AWS instance...
==> amazon-ebs: Deleting temporary security group...
==> amazon-ebs: Deleting temporary keypair...
Build 'amazon-ebs' finished.

==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs: AMIs were created:

us-west-2: ami-db237fec
{% endhighlight %}

For EC2, you can copy the AMI to other regions using the AWS Console or [awscli](https://github.com/aws/aws-cli):
{% highlight console %}
$ aws ec2 copy-image --source-image-id ami-db237fec --source-region us-west-2 --region us-east-1
{
    "ImageId": "ami-ca232495"
}
{% endhighlight %}

Typical build times are ~5-15min, but this could be improved by using a newer release of Ubuntu. (Docker requires a newer kernel than is shipped with Ubuntu 12.04). Cross-region copy times are quick, typically under a minute.

With your new AMI, you should now be able to provision new Docker hosts in just a minute or two.