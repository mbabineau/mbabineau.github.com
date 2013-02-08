---
layout: post
title: "Continuous Deployment of Ops Configs"
description: "what"
category: 
tags: [configuration management, continuous deployment, devops, git, jenkins, ops]
---
{% include JB/setup %}

A core tenant of DevOps is the notion of "Infrastructure as Code." Provisioning and deployment should be done programmatically, potentially with a configuration management (CM) system such as Chef or Puppet.

Jesse Robbins describes the goal well:
> “Enable the reconstruction of the business from nothing but a source code repository, an application data backup, and bare metal resources”

While CM tools facilitate this reconstruction, there are some tricks for getting the most of your implementation.  Below are the key points from a [lightning talk](http://www.slideshare.net/mbabineau/continuous-deployment-for-ops-who-use-chef-and-git) I gave on this at the [last ArchCamp](http://www.meetup.com/ArchCamp/events/16419122/).

### The wrong way
Using the configuration management server as the system of record.  A Chef example is creating and modifying roles directly on the Chef server:

    $ knife node create myrole
    $ knife node edit myrole

This decreases visibility and introduces unnecessary risk.  Losing the Chef server is now a major event.  Yes, you can mitigate this risk with regular backups, but you still lack visibility into changes.

### A better approach
Place your CM data into source control and treat that as your system of record.  Changes are committed to source control, then deployed to the CM server.  Under Jesse's description, this buckets CM configs as source code rather than application data.

Going back to the Chef example, your workflow would instead look like:

    $ git commit -am "added newfeature to myrole"
    $ knife role from file roles/myrole.json

This gives you an auditable history of every operational config change.  

### Deploying via git

But what if instead of deploying Chef changes via knife, you did so using git?  Changes to multiple roles or cookbooks could be pushed simultaneously, and with one command.

In other words, this:

    $ git commit -am "added newfeature to mycookbook, integrated it with myrole, and added supporting data to mydatabag"
    $ knife cookbook upload mycookbook
    $ knife role from file roles/myrole.json
    $ knife data bag from file mydatabag newitem

Would become:

    $ git commit -am "added newfeature to mycookbook, integrated it with myrole, and added supporting data to mydatabag"
    $ git push origin master

This type of deployment can be implemented via a process that monitors the git repo for changes and deploys any changes to the Chef server.

A simplified version of the upload code is:
    
    #!bash
    for cookbook in $(git_diff("cookbooks")); do
        knife cookbook upload $cookbook
    done

    for role in $(git_diff("roles")); do
        knife role from file $role
    done

    for bag in $(git_diff("databags")); do
        for item in $(git_diff("items", $bag)); do
            knife data bag from file $bag $item
        done
    done

By putting this on a continuous integration server (Jenkins, BuildBot, etc.) and detecting repository changes through polling or post-commit hooks, you implement continuous deployment of your operational configs.

That wasn't so bad, was it?

Don't forget to [check out the slides](http://www.slideshare.net/mbabineau/continuous-deployment-for-ops-who-use-chef-and-git).