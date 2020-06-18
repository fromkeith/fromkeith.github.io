---
layout: post
title: "Using DynamoDB too early"
categories: mistakes
---
 
## Starting out
When I first started my company over 5 years ago I was a much more naive engineer. I took the opportunity and thrill of
learning new tech over the practically of finding what was the best fit. I mean, for the first time ever I got to pick
every tech aspect! GoLang was new, lets try that! MySql is old and boring, NoSQL is the new cool! That's what being your own
boss is all about!

While fun, that attitude did not lead to the best decision criteria. I still got a product up, running and usable single-handedly... but it wasn't the smoothest ride.

Of all those mistakes, choosing DynamoDB as my sole database is one I regret. It wasn't the end of the world mind you.
I can just imagine my life being a bit easier is all.

To start, at the time, I didn't feel like my decision making was equivalent to throwing darts against the wall. I had justifications
to back up my decision:

* it was _designed for scalable traffic_
* had fast response times
* it took care of partitions for me so I wouldn't have to worry as we grew
* its was much cheaper than paying the RDS monthly fee
* i didn't have to worry about confusing firewalls, vpcs, or managing user accounts

Overall DynamoDB felt like it would minimize my time on the boring (to me) dev ops side. Freeing me up for the fun coding!
Initially, I was correct. For my local development work, it was perfect. I could build out features easily and focus on
what the product needed to do.

## Release Time
Then it came time to release. My experience at Amazon told me I needed to run automated load tests. So I pulled out [JMeter](https://jmeter.apache.org/) and
setup some basic tests. For the first minute (or less) it worked. Then the 500s started. I pulled up the logs, and yep. It
was the database failing. DynamoDB had started throttling because my provisioned readers/writers were too low.

So that needed to be fixed! But this was before 2017. DynamoDB did not have AutoScaling. It did not have OnDemand. Only whatever you told it to provision did it provision.

If I remember correctly, my first approach was to look at the "Consumed Capacity" returned by DynamoDB. I would then try to
merge those together across calls on the same webserver and try to scale up if I hit the threshold. This seems pretty straight
forward. However, I quickly abandoned it.

1. Consumed Capacity documentation was confusing, and it only told me after I did something what the cost was.
2. If I ever wanted more than one webserver this method would not work.
3. Someone has got to have run into this before. I can't be the only one.

## Auto Sacling

After some quick research I managed to find [Dynamic DynamoDB](https://github.com/sebdah/dynamic-dynamodb). A python project focused specifically on automatically scaling DynamoDB. The testimonials I could find talked wonders of it keeping costs down for different businesses. So I set it up and got it working.

Then went back to my load tests. Steady... Boom, 500s. Turns out Dynamic DynamoDB was waiting for a 5 minute
threshold before scaling. That felt a bit long to me, so reduced it to 1 minute, re-run tests. Steady... Boom, 500s. Mmm. My server
logs showed the same error. DynamoDB metrics in CloudWatch showed the spike, but no scaling. The logs for Dynamic DynamoDB show it
properly adjusting. So it was scaling. CloudWatch metrics were just several delayed on provisioned amount. The scaling was also
happening just a bit too slow

I determined the main problem was that scaling from 1 reader to 20 in less than a minute was too much to ask.
This probably seems pretty obvious. However, I was trying to keep my monthly costs down. I now needed have all my scaling tables sit with padded readers/writers. Ones I knew I was not going to use for 90% of the day, but couldn't predict when in the day they would be used. So scheduling was out of the question from the get go.

So for the first few months or years that was the status quo. Dynamic DynamoDB running on it own server (before Docker, before T class instances). Tables sitting with extra resources so that if a spike happened there was room for reaction.

## Reflecting

For what its worth, the design did scale and I had very few problems** outside of the scaling issue. When AWS announced AutoSacling in 2017 I gave it a try. However, it was extremely slow. Useless slow. I can see how it would work if I had predictable usage and a high steady volume. However, when dealing with spiky traffic it was not up to the task. On Demand for DynamoDB was announced late 2018 and was useful. However, I mainly took advantage of it for those small tables that rarely see any steady traffic. It also helped keep my staging/development environment costs down.

If I were to go back to that initial decision, I probably would have stuck everything into an RDS MySQL. It would have saved some early personal cash and some unnecessary hurdles. But thats easy to say now, who knows what problems I would have encountered if I changed the past. I may have had more downtime when trying to scale to larger loads of traffic.
