#azure_app_service 

> **Reference:** https://learn.microsoft.com/en-us/training/modules/introduction-to-azure-app-service/
# Examine Azure App Service
> Azure App Service is HTTP-based service for hosting web-application, web APIs or mobile back ends. Azure App Service supports multiple programming language like C#, Java, Python, GO and JavaScript...
## Auto-scale support
Azure App Service support for the ability to auto-scale by up/down or in/out. Scaling out/in is the ability to increase, or decrease, the number of machine instances that are running your web app. While scaling up/down is the ability to increase or decrease number of cores or amount of RAM of cloud resource.
## CI/CD support
The Azure portal provides out-of-the-box continuous integration and deployment with Azure DevOps Services, GitHub, Bitbucket, FTP, or a local Git repository on your development machine. Connect your web app with any of the above sources and App Service will do the rest for you by auto-syncing code and any future changes on the code into the web app.
## Deployment slots
One app can have multiple environments and you can separate deployment environment using deployment slots. Deployment slots are live apps with their own host names. App content and configurations elements can be swapped between two deployment slots, including the production slot.
## App Service in Linux
In some case, your application stacks need to deploy to native Linux, App Service supports you to also host the application on Linux VM.
### Limitations of App Service in Linux
- Not support shared pricing tier
- Azure portal only shows features that are works in Linux app
- The disk latency is higher and require more variable 
# Azure App Service plans
In App Service, an app always runs in an _App Service plan_. An App Service plan defines a set of compute resources for a web app to run. One or more apps can be configured to run on the same computing resources (or in the same App Service plan).
When an App Service plan in specific region is created, a set of computing resources for that plan are also created in the same region. Each App Service plan defines:
- OS (Linux/Windows)
- Region (West US, East US...)
- Number of VM instances
- Size of VM instances (Small, Medium, Large)
- Pricing Tier
The pricing tier divided into few categories:
- **Shared computing:** Free and shared, the two base tiers, runs an app on the same Azure VM as other App Service apps, including apps of other customers. These tiers allocate CPU quotas to each app that runs on the shared resources, and the resources can't scale out.
- **Dedicated computing:** The **Basic**, **Standard**, **Premium**, **PremiumV2**, and **PremiumV3** tiers run apps on dedicated Azure VMs. Only apps in the same App Service plan share the same compute resources. The higher the tier, the more VM instances are available to you for scale-out.
- **Isolated:** **Isolated** and **IsolatedV2** tiers run dedicated VMs on dedicated AZ Virtual Networks. It provides network isolation on top of compute isolation to your apps. It provides the **maximum scale-out capabilities.**
## How applications run and scale in App Service?
In the **Free** and **Shared** tiers, an app receives CPU minutes on a shared VM instance and can't scale out. In other tiers, an app runs and scales as follows:

- An app runs on all the VM instances configured in the App Service plan.
- If multiple apps are in the same App Service plan, they all share the same VM instances.
- If you have multiple deployment slots for an app, all deployment slots also run on the same VM instances.
- If you enable diagnostic logs, perform backups, or run WebJobs, they also use CPU cycles and memory on these VM instances.

In some case, you need to have an application to have a dedicated, isolation plan:
- The app is resource-intensive.
- You want to scale the app independently from the other apps in the existing plan.
- The app needs resource in a different geographical region.
# Deploy to App Service
App Service supports both automated and manual deployment.
## Automate deployment
> Continuous deployment is the process of pushing new changes in a fast and repeatable pattern with minimal effort for developers

Azure supports automate deployment options like:
- **Azure DevOps Services:** You can push your code to Azure DevOps Services, build your code in the cloud, run the tests, generate a release from the code, and finally, push your code to an Azure Web App.
- **Github:** Azure supports automatically deploy directly with Github. When connect to Github repository and Azure, any changes you push to deployment branches will automatically be deployed for you.
- **Bitbucket:** Similar to Github.
## Manual Deployment
Azure supports few options for manually deploy your applications:
- **Git**: App Service web apps feature a Git URL that you can add as a remote repository. Pushing to the remote repository deploys your app.
- **CLI**: `webapp up` is a feature of the `az` command-line interface that packages your app and deploys it. Unlike other deployment methods, `az webapp up` can create a new App Service web app for you if you haven't already created one.
- **Zip deploy**: Use `curl` or a similar HTTP utility to send a ZIP of your application files to App Service.
- **FTP/S**: FTP or FTPS is a traditional way of pushing your code to many hosting environments, including App Service.
## Deployment slots
Whenever possible, use deployment slots when deploying a new production build. When using a Standard App Service Plan tier or better, you can deploy your app to a staging environment and then swap your staging and production slots. The swap operation warms up the necessary worker instances to match your production scale, thus eliminating downtime.
# Authentication and Authorization in Azure App Service
You can add build-in Azure Authentication and Authorization with minimal set up for your application.
## Why using build-in authentication?
The built-in authentication feature for App Service and Azure Functions can save you time and effort by providing out-of-the-box authentication with federated identity providers, allowing you to focus on the rest of your application.

- Azure App Service allows you to integrate various auth capabilities into your web app or API without implementing them yourself.
- It’s built directly into the platform and doesn’t require any particular language, SDK, security expertise, or code.
- You can integrate with multiple login providers. For example, Azure AD, Facebook, Google, Twitter.
By default, Azure supports following identity providers:

|Provider|Sign-in endpoint|How-To guidance|
|---|---|---|
|Microsoft Identity Platform|`/.auth/login/aad`|[App Service Microsoft Identity Platform login](https://learn.microsoft.com/en-us/azure/app-service/configure-authentication-provider-aad)|
|Facebook|`/.auth/login/facebook`|[App Service Facebook login](https://learn.microsoft.com/en-us/azure/app-service/configure-authentication-provider-facebook)|
|Google|`/.auth/login/google`|[App Service Google login](https://learn.microsoft.com/en-us/azure/app-service/configure-authentication-provider-google)|
|Twitter|`/.auth/login/twitter`|[App Service Twitter login](https://learn.microsoft.com/en-us/azure/app-service/configure-authentication-provider-twitter)|
|Any OpenID Connect provider|`/.auth/login/<providerName>`|[App Service OpenID Connect login](https://learn.microsoft.com/en-us/azure/app-service/configure-authentication-provider-openid-connect)|
|GitHub|`/.auth/login/github`|[App Service GitHub login](https://learn.microsoft.com/en-us/azure/app-service/configure-authentication-provider-github)|

## How authentication and authorization work in your Azure Application?
The authentication and authorization module runs in the same sandbox as your application code. When it's enabled, every incoming HTTP request passes through it before being handled by your application code. This module handles several things for your app:

- Authenticates users and clients with the specified identity provider(s)
- Validates, stores, and refreshes OAuth tokens issued by the configured identity provider(s)
- Manages the authenticated session
- Injects identity information into HTTP request headers

The module runs separately from your application code and can be configured using Azure Resource Manager settings or using a configuration file. No SDKs, specific programming languages, or changes to your application code are required.
### Authentication flow
The authentication flow can be little difference depending on using provider's SDK or not:

- **Without provider SDK:** The application delegates federated sign-in to App Service. This is typically the case with browser apps, which can present the provider's login page to the user. The server code manages the sign-in process, so it's also called _server-directed flow_ or _server flow_.
- **With provider SDK:** The application signs users in to the provider manually and then submits the authentication token to App Service for validation. This is typically the case with browser-less apps, which can't present the provider's sign-in page to the user. The application code manages the sign-in process, so it's also called _client-directed flow_ or _client flow_. This applies to REST APIs, Azure Functions, JavaScript browser clients, and native mobile apps that sign users in using the provider's SDK.
The following table shows the steps of the authentication flow.

|Step|Without provider SDK|With provider SDK|
|---|---|---|
|Sign user in|Redirects client to `/.auth/login/<provider>`.|Client code signs user in directly with provider's SDK and receives an authentication token. For information, see the provider's documentation.|
|Post-authentication|Provider redirects client to `/.auth/login/<provider>/callback`.|Client code posts token from provider to `/.auth/login/<provider>` for validation.|
|Establish authenticated session|App Service adds authenticated cookie to response.|App Service returns its own authentication token to client code.|
|Serve authenticated content|Client includes authentication cookie in subsequent requests (automatically handled by browser).|Client code presents authentication token in `X-ZUMO-AUTH` header (automatically handled by Mobile Apps client SDKs).|

## Authorization behavior
In the Azure portal, you can configure App Service with many behaviors when an incoming request isn't authenticated.

- **Allow unauthenticated requests:** This option defers authorization of unauthenticated traffic to your application code. For authenticated requests, App Service also passes along authentication information in the HTTP headers. This option provides more flexibility in handling anonymous requests. It lets you present multiple sign-in providers to your users.
    
- **Require authentication:** This option rejects any unauthenticated traffic to your application. This rejection can be a redirect action to one of the configured identity providers. In these cases, a browser client is redirected to `/.auth/login/<provider>` for the provider you choose. If the anonymous request comes from a native mobile app, the returned response is an `HTTP 401 Unauthorized`. You can also configure the rejection to be an `HTTP 401 Unauthorized` or `HTTP 403 Forbidden` for all requests.
# App Service Networking feature
There are two way you can deploy your application with Azure virtual network: 
1.  The multitenant public service hosts App Service plans in the Free, Shared, Basic, Standard, Premium, PremiumV2, and PremiumV3 pricing SKUs.
2. Single-tenant App Service Environment (ASE) hosts Isolated SKU App Service plans directly in your Azure virtual network.
## Multi-tenant App Service networking features
Azure App Service is distributed system.  The roles that handle incoming HTTP or HTTPS requests are called _front ends_. The roles that host the customer workload are called _workers_. All the roles in an App Service deployment exist in a multi-tenant network. Because there are many different customers in the same App Service scale unit, you can't connect the App Service network directly to your network.

Instead of connecting the networks, you need features to handle the various aspects of application communication. The features that handle requests to your app can't be used to solve problems when you're making calls from your app. Likewise, the features that solve problems for calls from your app can't be used to solve problems to your app.

|Inbound features|Outbound features|
|---|---|
|App-assigned address|Hybrid Connections|
|Access restrictions|Gateway-required virtual network integration|
|Service endpoints|Virtual network integration|
|Private endpoints|

## Networking behavior
Azure App Service scale units support many customers in each deployment. The Free and Shared SKU plans host customer workloads on multitenant workers. The Basic and higher plans host customer workloads that are dedicated to only one App Service plan. If you have a Standard App Service plan, all the apps in that plan run on the same worker. If you scale out the worker, all the apps in that App Service plan are replicated on a new worker for each instance in your App Service plan.
### Outbound addresses
The worker VMs are broken down in large part by the App Service plans. The Free, Shared, Basic, Standard, and Premium plans all use the same worker VM type. The PremiumV2 plan uses another VM type. PremiumV3 uses yet another VM type. When you change the VM family, you get a different set of outbound addresses.

There are many addresses that are used for outbound calls. The outbound addresses used by your app for making outbound calls are listed in the properties for your app. These addresses are shared by all the apps running on the same worker VM family in the App Service deployment. If you want to see all the addresses that your app might use in a scale unit, there's a property called `possibleOutboundIpAddresses` that lists them.