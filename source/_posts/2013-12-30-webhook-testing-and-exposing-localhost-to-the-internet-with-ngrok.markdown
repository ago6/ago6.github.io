---
layout: post
title: "Local Webhook Testing and Exposing Localhost to the Internet with ngrok"
date: 2013-12-30 21:49
comments: true
categories: tech
---

Recently I was working on incorporating some [Stripe](http://Stripe.com) webhooks, but quite naturally I wanted to test it and iterate locally. Having to write some code, commit, merge it to master, deploy to production, and then finally test, is just way too long of a feedback loop.

I wanted to be able to expose my localhost to the internet so I can test the webhooks locally, and I found the perfect tool in [ngrok](https://ngrok.com)!

<!-- more -->

Ngrok is a simple application written in Go. You run it locally and it creates a tunnel to https://some-random-subdomain.ngrok.com. The application is a single binary with absolutely no run-time dependencies. It is hilariously easy to set up. In fact, we're going to set it up for you right now (these are mac instructions);

    # Download ngrok
    $ curl 'https://dl.ngrok.com/darwin_amd64/ngrok.zip' > ~/Downloads/ngrok.zip

    # Unzip it
    $ unzip ~/Downloads/ngrok.zip

    # Create a tunnel from localhost:3000 to the interwebs
    $ ./ngrok 3000

That took 6, maybe 7 seconds to do, and our locally running server is now being served up to the web and is publicly accessible. Amazing.

While running it, you'll see a status screen like the following.

    ngrok                                                           (Ctrl+C to quit)

    Tunnel Status                 online
    Version                       1.6/1.5
    Forwarding                    http://kl23j.ngrok.com -> 127.0.0.1:3000
    Forwarding                    https://kl23j.ngrok.com -> 127.0.0.1:3000
    Web Interface                 127.0.0.1:4040
    # Conn                        0
    Avg Conn Time                 0.00ms

Now if your sole goal was to get your local server exposed to the web so it can be hit with webhooks from external services, just specify your ngrok url as the endpoint with your webhooking service and you're done. Also, on this screen you'll see a live list of incoming HTTP requests. Pretty snazzy.

The snazziness does not end there however. Something that you may notice in this status screen is that you get a web interface at 127.0.0.1:4040. What's there you may ask? Well hopefully at this point you're running this locally so you can check it out first-hand, but in short it gives you a wealth of information including all requests, response time, the actual responses provided, and *the ability to replay requests*. Where do the surprises end?

Oh, but what if you don't want a randomly assigned subdomain each time? Want to be able to specify a specific url/subdomain so your external services can always go to the same spot instead of always changing it to whatever randomly assigned subdomain ngrok gives you? Do you have another 15 to 20 seconds to invest in this fantastic little tool? Read on friend;

If you register with ngrok.com [here](https://ngrok.com/signup), which is *completely free*, you get some additional functionality such as being able to specify specific subdomains (like `ngrok -subdomain=adrian-likes-waffles 3000` will create a tunnel from https://adrian-likes-waffles.ngrok.com, assuming it is currently available.), TCP tunneling, password protection, etc.. If you do decide you are willing to pay, there are even further features as well.

Pretty great tool/service in such a tight package. Within a few minutes of playing with it I moved the binary to `/usr/local/bin/ngrok` so I always have access to it, making it a part of my regular tool belt. I hope you find it just as useful and it makes its way to your toolbelt as well.

