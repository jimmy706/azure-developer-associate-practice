#azure_app_service #autoscaling

> **Reference:** https://learn.microsoft.com/en-us/training/modules/scale-apps-app-service/

## Examine Autos scale factors
Autoscaling can be triggered according to a schedule, or by assessing whether the system is running short on resources. For example, autoscaling could be triggered if CPU utilization grows, memory occupancy increases, the number of incoming requests to a service appears to be surging, or some combination of factors.
### What is auto-scaling?
Autoscaling is a cloud system or process that adjusts available resources based on the current demand. Autoscaling performs scaling _in and out_, as opposed to scaling _up and down_.

Autoscaling in Azure App Service monitors the resource metrics of a web app as it runs. It detects situations where other resources are required to handle an increasing workload, and ensures those resources are available before the system becomes overloaded.

Autoscaling responds to changes in the environment by adding or removing web servers and balancing the load between them. Autoscaling doesn't have any effect on the CPU power, memory, or storage capacity of the web servers powering the app, it only changes the number of web servers.

### Auto-scaling rules
Autoscaling makes its decisions based on rules that you define. A rule specifies the threshold for a metric, and triggers an autoscale event when this threshold is crossed. Autoscaling can also deallocate resources when the workload has diminished.

Define your autoscaling rules carefully. For example, a Denial of Service attack will likely result in a large-scale influx of incoming traffic. Trying to handle a surge in requests caused by a DoS attack would be fruitless and expensive. These requests aren't genuine, and should be discarded rather than processed. A better solution is to implement detection and filtering of requests that occur during such an attack before they reach your service.

### When should you consider autoscaling?
Autoscaling isn't the best approach to handling long-term growth. You might have a web app that starts with a few users, but increases in popularity over time. Autoscaling has an overhead associated with monitoring resources and determining whether to trigger a scaling event. In this scenario, if you can anticipate the rate of growth, manually scaling the system over time may be a more cost effective approach.

The number of instances of a service is also a factor. You might expect to run only a few instances of a service most of the time. However, in this situation, your service is susceptible to downtime or lack of availability whether autoscaling is enabled or not. The fewer the number of instances initially, the less capacity you have to handle an increasing workload while autoscaling spins up more instances.

## Identify autoscaling factors
Autoscaling allows you to specify which web app should be scaled in which condition. The goal of effective scaling is to ensure resources are available even on large volumes and effectively manage cost when demand drops.
### Autoscaling and App Service plan
Autoscaling is part of App Service Plan, when the web app scale out, Azure create instances of hardware to serve the traffic. You can also prevent unwanted autoscaling price rising by limit number of instances created. Autoscaling cannot create more instances than the limit.

### Auto scaling condition
Azure provides 2 options for autoscaling:
- Scale based on metric like diskspace availability, number of HTTP requests await.
- Scale to specific instance count according scheduled. Like, you can specify time of the day or day of week for autoscaling and also you can arrange the end date for system to scale backs in

Scaling to a specific instance count only enables you to scale out to a defined number of 
instances. If you need to scale out incrementally, you can combine metric and schedule-based autoscaling in the same autoscale condition. So, you could arrange for the system to scale out if the number of HTTP requests exceeds some threshold, but only between certain hours of the day.

You can create multiple autoscale conditions to handle different schedules and metrics. Azure autoscales your service when any of these conditions apply. An App Service Plan also has a default condition that is used if none of the other conditions are applicable. This condition is always active and doesn't have a schedule.

### Metric for autoscale rules
Autoscaling by metric requires you to define one or more rule. Azure will monitor these rules and response scaling application when the threshold is crossed. The metrics you can monitor for a web app are:

- **CPU Percentage**. This metric is an indication of the CPU utilization across all instances. A high value shows that instances are becoming CPU-bound, which could cause delays in processing client requests.
- **Memory Percentage**. This metric captures the memory occupancy of the application across all instances. A high value indicates that free memory could be running low, and could cause one or more instances to fail.
- **Disk Queue Length**. This metric is a measure of the number of outstanding I/O requests across all instances. A high value means that disk contention could be occurring.
- **Http Queue Length**. This metric shows how many client requests are waiting for processing by the web app. If this number is large, client requests might fail with HTTP 408 (Timeout) errors.
- **Data In**. This metric is the number of bytes received across all instances.
- **Data Out**. This metric is the number of bytes sent by all instances.
### How an autoscaling-rule analyze metrics?
In the first step, an autoscale rule aggregates the values retrieved for a metric for all instances across a period of time known as the _time grain_. Each metric has its own intrinsic time grain, but in most cases this period is 1 minute. The aggregated value is known as the _time aggregation_. The options available are _Average_, _Minimum_, _Maximum_, _Sum_, _Last_, and _Count_.

An interval of one minute is a short interval in which to determine whether any change in metric is long-lasting enough to make autoscaling worthwhile. So, an autoscale rule performs a second step that performs a further aggregation of the value calculated by the _time aggregation_ over a longer, user-specified period, known as the _Duration_. The minimum _Duration_ is 5 minutes. If the _Duration_ is set to 10 minutes for example, the autoscale rule aggregates the 10 values calculated for the _time grain_.

The aggregation calculation for the _Duration_ can be different from the _time grain_. For example, if the _time aggregation_ is _Average_ and the statistic gathered is _CPU Percentage_ across a one-minute _time grain_, each minute the average CPU percentage utilization across all instances for that minute is calculated. If the _time grain statistic_ is set to _Maximum_, and the _Duration_ of the rule is set to 10 minutes, the maximum of the 10 average values for the CPU percentage utilization is to determine whether the rule threshold has been crossed.
### Autoscaling actions
An autoscale action uses an operator (such as _less than_, _greater than_, _equal to_, and so on) to determine how to react to the threshold. Scale-out actions typically use the _greater than_ operator to compare the metric value to the threshold. Scale-in actions tend to compare the metric value to the threshold with the _less than_ operator. An autoscale action can also set the instance count to a specific level, rather than incrementing or decrementing the number available.

>An autoscale action has a _cool down_ period, specified in minutes. During this interval, the scale rule won't be triggered again. This is to allow the system to stabilize between autoscale events. Remember that it takes time to start up or shut down instances, and so any metrics gathered might not show any significant changes for several minutes. The minimum cool down period is five minutes.

### Pairing autoscale rules
You should plan for scaling-in when a workload decreases. Consider defining autoscale rules in pairs in the same autoscale condition. One autoscale rule should indicate how to scale the system out when a metric exceeds an upper threshold. Then other rule should define how to scale the system back in again when the same metric drops below a lower threshold.

### Combining autoscale rules
A single autoscale condition can contain several autoscale rules (for example, a scale-out rule and the corresponding scale-in rule). However, the autoscale rules in an autoscale condition don't have to be directly related. You could define the following four rules in the same autoscale condition:

- If the HTTP queue length exceeds 10, scale out by 1
- If the CPU utilization exceeds 70%, scale out by 1
- If the HTTP queue length is zero, scale in by 1
- If the CPU utilization drops below 50%, scale in by 1

> When determining whether to scale out, the autoscale action is performed if **any** of the scale-out rules are met (HTTP queue length exceeds 10 **or** CPU utilization exceeds 70%). When scaling in, the autoscale action runs **only if all** of the scale-in rules are met (HTTP queue length drops to zero **and** CPU utilization falls below 50%). If you need to scale in if only one of the scale-in rules are met, you must define the rules in separate autoscale conditions.

## Enable autoscaling in app service
To get started with autoscaling navigate to your App Service plan in the Azure portal and select **Scale out (App Service plan)** in the **Settings** group in the left navigation pane.

## Autoscaling best practices
### Autoscaling concepts
- An autoscale setting scales instances horizontally, which is _out_ by increasing the instances and _in_ by decreasing the number of instances. An autoscale setting has a maximum, minimum, and default value of instances.
    
- An autoscale job always reads the associated metric to scale by, checking if it has crossed the configured threshold for scale-out or scale-in.
    
- All thresholds are calculated at an instance level. For example, "scale out by one instance when average CPU > 80% when instance count is 2", means to scale out when the average CPU across all instances is greater than 80%.
    
- All autoscale successes and failures are logged to the Activity Log. You can then configure an activity log alert so that you can be notified via email, SMS, or webhooks whenever there's activity.
### Best practices
- Ensure the maximum and minimum values are different and have an adequate margin between them
- Choice appropriate statistics for diagnostic metric
- Pick the threshold carefully
- Consider scaling when multiple rules are configured in the profile