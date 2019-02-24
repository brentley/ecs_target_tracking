First we start up a baseline of traffic:
```
while true; do curl -so /dev/null http://farga-Publi-VS8RO4CLAF8R-216733506.us-east-1.elb.amazonaws.com/ ; sleep 1 ; done &                  

```
Our service is holding at the minimum level of 3 tasks.


After establishing that baseline, we add some load:
```
while true; do ab -l -c 9 -t 60 http://farga-Publi-VS8RO4CLAF8R-216733506.us-east-1.elb.amazonaws.com/ ; sleep 1; done
```
This command is Apache Bench. We generate 9 concurrent requests (`-c 9`), as 
rapidly as can be filled, for 60 seconds (`-t 60`), and ignore length variances (`-l`).
We loop this repeatedly, so we can see statistics every 60 seconds.

We see in the CloudWatch graph that traffic ramps up over the maximum threshold, 
and stays there, triggering an alarm status.

![cloudwatch ramp up](images/cloudwatch_ramp_up.png)

We see the scale out to meet the traffic increase:

![ecs events scale out 1](images/ecs_events_scale_out_1.png)

Our CloudWatch graph now shows the traffic per target back down under the alarm threshold.

![cloudwatch traffic green](images/cloudwatch_traffic_green.png)

Scaling out works well. Let's see how scaling in works.

First, let's stop our traffic load. `Ctrl-C` the ab loop (twice to cancel the loop).

We take a more conservative approach to scaling in, not triggering any action until
traffic has been under the threshold for 15 consecutive minutes. After that, Autoscaling
will drain and stop one container at a time until we reach the per-target threshold, or
the minimum task count.

![ecs events scale in 1](images/ecs_events_scale_in_1.png)
![ecs events scale in 2](images/ecs_events_scale_in_1.png)

If we look at our baseline request graph, we see that requests are much lower than our 
scaling threshold, but we maintain the minimum number of tasks, for redundancy.

Now let's check the request graph, and we see a spike in incoming requests, going above our scaling threshold. This triggers
an alarm, and a scale up of our service to meet the demand.

We continue to scale up until our requests per task falls below our established limit. 
(logs of ecs scaling events)

When traffic subsides, our service will begin to scale back down (slowly and conservatively)