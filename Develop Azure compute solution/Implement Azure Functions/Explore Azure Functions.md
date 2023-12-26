# Discover Azure Functions
We often build systems to react to a series of critical events. Whether you're building a web API, responding to database changes, processing IoT data streams, or even managing message queues - every application needs a way to run some code as these events occur.

> Azure Functions supports _triggers_, which are ways to start execution of your code, and _bindings_, which are ways to simplify coding for input and output data. There are other integration and automation services in Azure and they all can solve integration problems and automate business processes. They can all define input, actions, conditions, and output.

## Compare Azure Functions and Azure Logic Apps
Both Functions and Logic Apps are Azure Services that enable serverless workloads. Azure Functions is a serverless compute service, whereas Azure Logic Apps is a serverless workflow integration platform. Both can create complex _orchestrations_. An orchestration is a collection of functions or steps, called actions in Logic Apps, that are executed to accomplish a complex task.

For Azure Functions, you develop orchestrations by writing code and using the [Durable Functions extension](https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview). For Logic Apps, you create orchestrations by using a GUI or editing configuration files.

| |Azure Functions | Logic Apps|
|---|---|---|
|Development|Code-first (imperative)|Designer-first (declarative)|
|Connectivity|About a dozen built-in binding types, write code for custom bindings|Large collection of connectors, Enterprise Integration Pack for B2B scenarios, build custom connectors|
|Actions|Each activity is an Azure function; write code for activity functions|Large collection of ready-made actions|
|Monitoring|Azure Application Insights|Azure portal, Azure Monitor logs|
|Management|REST API, Visual Studio|Azure portal, REST API, PowerShell, Visual Studio|
|Execution context|Runs in Azure, or locally|Runs in Azure, locally, or on premises|

## Compare Azure Functions and WebJobs

| |Functions|WebJobs with WebJobs SDK|
|---|---|---|
|Serverless app model with automatic scaling|Yes|No|
|Develop and test in browser|Yes|No|
|Pay-per-use pricing|Yes|No|
|Integration with Logic Apps|Yes|No|
|Trigger events|Timer  <br>Azure Storage queues and blobs  <br>Azure Service Bus queues and topics  <br>Azure Cosmos DB  <br>Azure Event Hubs  <br>HTTP/WebHook (GitHub  <br>Slack)  <br>Azure Event Grid|Timer  <br>Azure Storage queues and blobs  <br>Azure Service Bus queues and topics  <br>Azure Cosmos DB  <br>Azure Event Hubs  <br>File system|

Azure Functions offers more developer productivity than Azure App Service WebJobs does. It also offers more options for programming languages, development environments, Azure service integration, and pricing. For most scenarios, it's the best choice.

# Compare Azure Functions hosting options
When you create a function app in Azure, you must choose a hosting plan for your app. There are three basic hosting plans available for Azure Functions: [Consumption plan](https://learn.microsoft.com/en-us/azure/azure-functions/consumption-plan), [Premium plan](https://learn.microsoft.com/en-us/azure/azure-functions/functions-premium-plan), and [Dedicated (App service) Dedicated plan](https://learn.microsoft.com/en-us/azure/azure-functions/dedicated-plan). All hosting plans are generally available (GA) on both Linux and Windows virtual machines.
Following is a summary of the benefits of the three main hosting plans for Functions:

|Plan|Benefits|
|---|---|
|**Consumption plan**|This is the default hosting plan. It scales automatically and you only pay for compute resources when your functions are running. Instances of the Functions host are dynamically added and removed based on the number of incoming events.|
|**Premium plan**|Automatically scales based on demand using pre-warmed workers, which run applications with no delay after being idle, runs on more powerful instances, and connects to virtual networks.|
|**Dedicated plan**|Run your functions within an App Service plan at regular App Service plan rates. Best for long-running scenarios where [Durable Functions](https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview) can't be used.|

There are two other hosting options, which provide the highest amount of control and isolation in which to run your function apps.

|Hosting option|Details|
|---|---|
|**ASE**|[App Service Environment (ASE)](https://learn.microsoft.com/en-us/azure/app-service/environment/intro) is an App Service feature that provides a fully isolated and dedicated environment for securely running App Service apps at high scale.|
|**Kubernetes** ([Direct](https://learn.microsoft.com/en-us/azure/azure-functions/functions-kubernetes-keda) or [Azure Arc](https://learn.microsoft.com/en-us/azure/app-service/overview-arc-integration))|Kubernetes provides a fully isolated and dedicated environment running on top of the Kubernetes platform.|

## Hosting plan and scaling

|Plan|Scale out|Max # instances|
|---|---|---|
|**Consumption plan**|Event driven. Scale out automatically, even during periods of high load. Azure Functions infrastructure scales CPU and memory resources by adding more instances of the Functions host, based on the number of incoming trigger events.|Windows: 200, Linux: 100|
|**Premium plan**|Event driven. Scale out automatically, even during periods of high load. Azure Functions infrastructure scales CPU and memory resources by adding more instances of the Functions host, based on the number of events that its functions are triggered on.|Windows: 100, Linux: 20-100|
|**Dedicated plan**|Manual/autoscale|10-20|
|**ASE**|Manual/autoscale|100|
|**Kubernetes**|Event-driven autoscale for Kubernetes clusters using KEDA.|Varies by cluster|

## Function App timeout duration

The `functionTimeout` property in the _host.json_ project file specifies the timeout duration for functions in a function app. This property applies specifically to function executions. After the trigger starts function execution, the function needs to return/respond within the timeout duration.

The following table shows the default and maximum values (in minutes) for specific plans:

|Plan|Default|Maximum|
|---|---|---|
|Consumption plan|5|10|
|Premium plan|302|Unlimited|
|Dedicated plan|302|Unlimited|

## Storage account requirements
On any plan, a function app requires a general Azure Storage account, which supports Azure Blob, Queue, Files, and Table storage. This is because Functions rely on Azure Storage for operations such as managing triggers and logging function executions, but some storage accounts don't support queues and tables.

The same storage account used by your function app can also be used by your triggers and bindings to store your application data. However, for storage-intensive operations, you should use a separate storage account.

# Scale Azure Functions
In the Consumption and Premium plans, Azure Functions scales CPU and memory resources by adding more instances of the Functions host. The number of instances is determined on the number of events that trigger a function.

Each instance of the Functions host in the Consumption plan is limited to 1.5 GB of memory and one CPU. An instance of the host is the entire function app, meaning all functions within a function app share resource within an instance and scale at the same time. Function apps that share the same Consumption plan scale independently. In the Premium plan, the plan size determines the available memory and CPU for all apps in that plan on that instance.

Function code files are stored on Azure Files shares on the function's main storage account. When you delete the main storage account of the function app, the function code files are deleted and can't be recovered.

## Run time and scaling

Azure Functions uses a component called the _scale controller_ to monitor the rate of events and determine whether to scale out or scale in. The scale controller uses heuristics for each trigger type. For example, when you're using an Azure Queue storage trigger, it scales based on the queue length and the age of the oldest queue message.

The unit of scale for Azure Functions is the function app. When the function app is scaled out, more resources are allocated to run multiple instances of the Azure Functions host. Conversely, as compute demand is reduced, the scale controller removes function host instances. The number of instances is eventually "scaled in" to zero when no functions are running within a function app.

## Scaling behaviors
Scaling can vary on many factors, and scale differently based on the trigger and language selected. There are a few intricacies of scaling behaviors to be aware of:

- **Maximum instances:** A single function app only scales out to a maximum of 200 instances. A single instance may process more than one message or request at a time though, so there isn't a set limit on number of concurrent executions.    
- **New instance rate:** For HTTP triggers, new instances are allocated, at most, once per second. For non-HTTP triggers, new instances are allocated, at most, once every 30 seconds.
## Limit scale-out
You may wish to restrict the maximum number of instances an app used to scale out. This is most common for cases where a downstream component like a database has limited throughput. By default, Consumption plan functions scale out to as many as 200 instances, and Premium plan functions scales out to as many as 100 instances. You can specify a lower maximum for a specific app by modifying the `functionAppScaleLimit` value. The `functionAppScaleLimit` can be set to `0` or `null` for unrestricted, or a valid value between `1` and the app maximum.



