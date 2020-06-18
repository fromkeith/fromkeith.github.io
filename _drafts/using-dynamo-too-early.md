---
layout: post
title: "Using DynamoDB too early"
categories: mistakes
---

## Starting out
When I first started my company over 5 years ago I was a much more nieve engineer. I took the opportunity and thrill of
learning new tech over the practically of finding what was the best fit. I mean, for the first time ever I got to pick
every tech aspect! Golang was new, lets try that! Mysql is old and boring, NoSQL is the new cool! Not the best decision
criteria. I still got it all to work and launch... but it wasn't the smoothest ride.

Of all those mistakes, choosing DynamoDB as my sole database is one I regret. It wans't the end of the world mind you.
I can just imagine my life being a bit easier is all.

To start, at the time, I didn't feel like my decision making was equivalent to throwing darts against the wall. I had justifications
to back up my decision:

* it was _designed for scalable traffic_
* had fast response times
* it took care of paritions for me so I wouldn't have to worry as we grew
* its was much cheaper than paying the RDS monthly fee
* i didn't have to worry about confusing firewalls, vpcs, or managing user accounts

Overall DynamoDB felt like it would minimize my time on the boring (to me) managment side. Freeing me up for the fun coding!
Initially, I was correct. For my local development work, it was just fine. I could build out features easily and focus on
what the product needed.

## Release Tiem
Then it came time to release. My experience at Amazon told me I needed to run automated load tests. So I pulled out JMeter and
setup some basic tests. And for the first minute (or less) it worked. Then the 500s started. I pulled up the logs, and yep. It
was the database failing. DynamoDB had started throttling because my provisioned readers/writers were too low. So that needed
to be fixed.

This was before 2017. AWS did not have AutoScaling. It did not have OnDemand. Only whatever you told it to provision did it provision.

If I remember correctly, my first approach was to look at the "Consumed Capacity" returned by DynamoDB. I would then try to
merge those together across calls on the same webserver and try to scale up if I hit the threshold. This seems pretty straight
forward. However, I quickily abandoned it. 1) Consumed Capacity documentation was confusing as hell. 2) If I ever wanted
more than 1 webserver this method would not work. 3) Someone has got to have run into this before.

## Auto Sacling

After some quick research I managed to find DynamicDynamoDB. A python project focused specifically on scaling DynamoDB, with some great testimonails on it saving money and doing wonder. I set it
up, get it working. Then went back to my load tests. Steady... Boom, 500s. Turns out DynamicDynamoDB was waiting for the 5 minute
threshold before scaling. That felt a bit long to me, so reduce to 1 minute, re-run tests. Steady... Boom, 500s. Mmm. It was scaling
but it still wasn't fast enough. Faster... but still not fast enough.

The main problem being that it is super hard to scale from 1 reader to 20 in less than a minute.
This probably seems pretty obvious. However, I was trying to keep my monthly costs down! I now needed to sit with all my tables,
that needed to quickily scale, with padded readers/writers. Ones I knew I was not going to use for 90% of the day. I also couldn't
predict when that 10% of time was during the day. So scheduling was out of the question from the get go.

## Reflecting
So for the first few months or years that was the status quo. Dynamic DynamoDB running on it own server (before Docker, before T class instnaces). Tables sitting with extra resources so that if a spike happened there was room for reaction.

For what its worth, the design did scale and I had very few problems outside of the scaling issue. When AWS announced AutoSacling in 2017 I gave it a try. However, it was extremely slow. Useless slow. I can see how it would work if I had predicatable useage and high steady volume. However dealing with spikes in traffic it could not help. On Demand that was announced late 2018 was useful, but only for those small tables that rarely see any steady traffic. It also helped keep my staging/developement environment costs down.

If I were to go back to that initial decision, I probably would have stuck everything into an RDS MySQL. It would have saved some early personal cash and some unessary hurdles. But thats easy to say now, who knows what problems I would have encountered if I changed the past.
