---
layout: post
title: "Icinga-PagerDuty Integration via Chef"
description: ""
category: 
tags: [chef, pagerduty, monitoring, ops, icinga, nagios, open source]
---

I wrote a Chef recipe for enabling PagerDuty support in Icinga. With luck, it'll be merged into Marius Ducea's [icinga cookbook](https://github.com/mdxp/icinga-cookbook). The code is [here](https://github.com/mdxp/icinga-cookbook/pull/11).

### Usage
#### Configure PagerDuty
Add a service for Icinga:
1. Go to [https://your-domain.pagerduty.com/services/new](https://your-domain.pagerduty.com/services/new)
1. Set the service type to "Nagios"
1. Add the service

#### Configure your monitoring node
1. Add the `icinga::pagerduty` recipe to your role

        name "monitoring"
        description "Monitoring server"
        run_list(
          "recipe[icinga]",
          "recipe[icinga::pagerduty]"
        )    


1. Get the new PagerDuty service's API key
![api key](/img/pagerduty_service_api_key.png)

1. Copy it into your node attributes:

        default_attributes({
          :icinga => {
            :pagerduty => {
              :service_key => "318e318e318e318e318e318e318ead29cf"
            }
        })
    
1. Run chef-client

#### Watch alerts show up in PagerDuty

You should now see alerts like these:
![api key](/img/pagerduty_icinga_alert.png)

PagerDuty will automatically resolve these incidents as Icinga sends recovery notifications. More details [here](http://www.pagerduty.com/docs/nagios-integration-guide/).