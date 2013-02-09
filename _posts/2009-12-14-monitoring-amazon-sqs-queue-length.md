---
layout: post
title: "Monitoring Amazon SQS Queue Length"
description: ""
category: 
tags: [aws, sqs, monitoring, python, boto, nagios, icinga]
---
{% include JB/setup %}

At ShareThis, we use Amazon SQS for a number of core product features, most notably the delivery of email shares. When you use our widget to share something via email, our system creates a database record and logs a message in a queue. Queued items are asynchronously processed by our message sending service.

We monitor this sending service in a variety of ways, but as any monitoring expert will tell you, it’s tough to anticipate every means by which something can break. Fortunately for us, most of our potential sending issues share a common symptom: the queue backs up.

Amazon exposes SQS queue length through their API, but as there were no tools available for monitoring it, I wrote one in Python and made it available on GitHub: [check_sqs_queue](https://github.com/mbabineau/check_sqs_queue).

check_sqs_queue can be run as a stand-alone script, emailing alert recipients directly through a configured SMTP server, or as a Nagios plugin. To run it, you must have the [boto](http://code.google.com/p/boto/) library installed, and you must have `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` defined in `boto.cfg` or a specified config file.

Usage:

    check_sqs_queue.py -q <queue name> [-w <warning threshold>] -c <critical threshold> [-n <recipient(s)>] [-f <config file]

    Options:
    -f FILE, --config=FILE
                configuration file
    -q QUEUE, --queue=QUEUE
                Amazon SQS queue name (name only, not the URL)
    -w WARN, --warning=WARN
                warning threshold
    -c CRIT, --critical=CRIT
                critical threshold
    -n RECIPIENT(s), --notify=RECIPIENT(s)
                comma-separated list of email addresses to notify

By default, check_sqs_queue uses the boto config file:

    $ cat /etc/boto.cfg
    [Credentials]
    aws_access_key_id = 123456790ABCDEFGHIJ
    aws_secret_access_key = 0987654321ZXYWVUTSRQPO123456789

Let’s see it in action:

    $ check_sqs_queue.py -q test_queue -c 50
    Queue OK: "test_queue" contains 17 messages
    
    $ check_sqs_queue.py -q test_queue -c 10
    Queue CRITICAL: "test_queue" contains 17 messages
    
    $ check_sqs_queue.py -q test_queue -w 10 -c 50
    Queue WARNING: "test_queue" contains 17 messages
    
By putting SMTP credentials into a specified config file, the script can alert a list of email recipients:

    $ cat check_sqs_queue.conf
    [AWS]
    aws_access_key_id = 123456790ABCDEFGHIJ
    aws_secret_access_key = 0987654321ZXYWVUTSRQPO123456789
    
    [SMTP]
    smtp_server = smtp.gmail.com
    smtp_port = 587
    smtp_user = user@example.com
    smtp_password = cleverpassword
    
    $ check_sqs_queue.py -f check_sqs_queue.conf -q test_queue -w 5 \
    -c 10 -n mike@example.com,joe@example.com,dan@example.com
    Queue CRITICAL: "test_queue" contains 17 messages

The resulting email:

    Date: Mon, 14 Dec 2009 15:52:44 -0800 (PST)  
    From: user@example.com  
    Subject: Queue CRITICAL: "test_queue" contains 17 messages  
    To: mike@example.com
    
    "test_queue" contains 17 messages

So go ahead and give it a try. If you run into any issues, please feel free to drop me a line: [michael.babineau@gmail.com](mailto:michael.babineau@gmail.com).