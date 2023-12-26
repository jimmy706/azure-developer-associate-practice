#azure_app_service

# App Service common knowledge

### What is Azure App Service?
Azure App Service is HTTP-based service for hosting web-application, web APIs or mobile back ends. Azure App Service supports multiple programming language like C#, Java, Python, GO and JavaScript...

### What is App Service deployment slots?
One app can have multiple environments and you can separate deployment environment using deployment slots. Deployment slots are live apps with their own host names. App content and configurations elements can be swapped between two deployment slots, including the production slot.

### What are limitation of application deploy in Linux?
- Not support shared pricing tier
- Azure portal only shows features that are works in Linux app
- The disk latency is higher and require more variable 

### What are available plans in AZ App Service?
- **Shared computing:** Free and shared, the two base tiers, runs an app on the same Azure VM as other App Service apps, including apps of other customers. These tiers allocate CPU quotas to each app that runs on the shared resources, and the resources can't scale out.
- **Dedicated computing:** The **Basic**, **Standard**, **Premium**, **PremiumV2**, and **PremiumV3** tiers run apps on dedicated Azure VMs. Only apps in the same App Service plan share the same compute resources. The higher the tier, the more VM instances are available to you for scale-out.
- **Isolated:** **Isolated** and **IsolatedV2** tiers run dedicated VMs on dedicated AZ Virtual Networks. It provides network isolation on top of compute isolation to your apps. It provides the **maximum scale-out capabilities.**
### Which plan in AZ App Service give maximum scale-out?
The isolated plan

### How can one app be scaled using App Service?
- An app runs on all the VM instances configured in the App Service plan.
- If multiple apps are in the same App Service plan, they all share the same VM instances.
- If you have multiple deployment slots for an app, all deployment slots also run on the same VM instances.
- If you enable diagnostic logs, perform backups, or run WebJobs, they also use CPU cycles and memory on these VM instances.
### Which of the following plans that are available for scale-out?
- Shared-computing
- Free-tier
- Dedicated computing ✅
- Isolated computing ✅

### In which case an app need isolation resource?
- Region different
- The app is resource-intensive
- You want to scale the app independently

### Your company has a web app named WebApp1.  You use the WebJobs SDK to design a triggered App Service background task that automatically invokes a function in the code every time new data is received in a queue.  You are preparing to configure the service processes a queue data item.  Which of the following is the service you should use?

WebJobs



### You are developing an e-Commerce Web App.  You want to use Azure Key Vault to ensure that sign-ins to the e-Commerce Web App are secured by using Azure App Service authentication and Azure Active  Directory (AAD).  What should you do on the e-Commerce Web App?

Enable Managed Service Identity (MSI).



### What automate deployment options do App Service support?
- **Azure DevOps Services:** You can push your code to Azure DevOps Services, build your code in the cloud, run the tests, generate a release from the code, and finally, push your code to an Azure Web App.
- **Github:** Azure supports automatically deploy directly with Github. When connect to Github repository and Azure, any changes you push to deployment branches will automatically be deployed for you.
- **Bitbucket:** Similar to Github.
### Which manual deployment options supported by App Service?
- **Git**: App Service web apps feature a Git URL that you can add as a remote repository. Pushing to the remote repository deploys your app.
- **CLI**: `webapp up` is a feature of the `az` command-line interface that packages your app and deploys it. Unlike other deployment methods, `az webapp up` can create a new App Service web app for you if you haven't already created one.
- **Zip deploy**: Use `curl` or a similar HTTP utility to send a ZIP of your application files to App Service.
- **FTP/S**: FTP or FTPS is a traditional way of pushing your code to many hosting environments, including App Service.




### What is built-in authentication for App Service?
- Azure App Service allows you to integrate various auth capabilities into your web app or API without implementing them yourself.
- It’s built directly into the platform and doesn’t require any particular language, SDK, security expertise, or code.
- You can integrate with multiple login providers. For example, Microsoft Entra ID, Facebook, Google, Twitter.
### Which identity providers do App Service support for authentication?
- Microsoft
- Google
- Facebook
- Twitter
- OpenID Connector
- Github
### How authentication and authorization work in App Service?
When enable authentication & authorization module in App Service, it will handle every incoming HTTP requests by providing:
- Authenticate users and clients with specific identity providers
- Validate, stores and refresh Oauth token
- Manage authentication session
- Inject identity information into HTTP headers

### How many ways can you deploy your App Service with virtual network?
1.  The multitenant public service hosts App Service plans in the Free, Shared, Basic, Standard, Premium, PremiumV2, and PremiumV3 pricing SKUs.
2. Single-tenant App Service Environment (ASE) hosts Isolated SKU App Service plans directly in your Azure virtual network.
### What is outbound traffic and which features it have in App Service?
Outbound refer to traffic that go outside Azure resources.
Two of the most popular features of outbound in App Service are:
- Hybrid connection:  Provides a secure and easy-to-configure way to access resources hosted in an on-premises network from Azure services.
- Virtual network integration: Connect Azure Resources through VPN or ExpressRoute
### What is inbound traffic and which features it have in App Service?
Inbound refer to traffics that coming to Azure resources. Normally, inbound traffics are restricted to network security rules and only allow access by certain policies such as whitelist addresses, opened port numbers, protocols...
Two of the most popular features of inbound in App Service are:
- Service endpoint: Allows Azure services to access resources in a virtual network over a private endpoint, reducing exposure to the internet.
- Private endpoint: Enables secure access to Azure services over a private connection, eliminating the need for public IP addresses and firewall rules.

# Auto-scaling in App Service
### What is auto-scaling?
Autoscaling is a cloud system or process that adjusts available resources based on the current demand. Autoscaling performs scaling _in and out_, as opposed to scaling _up and down_.
### How auto-scaling work?
Autoscaling in Azure App Service monitors the resource metrics of a web app as it runs. It detects situations where other resources are required to handle an increasing workload, and ensures those resources are available before the system becomes overloaded. Autoscaling responds to changes in the environment by adding or removing web servers and balancing the load between them. Autoscaling doesn't have any effect on the CPU power, memory, or storage capacity of the web servers powering the app, it only changes the number of web servers.

### What are rules on auto-scaling?
Autoscaling makes its decisions based on rules that you define. A rule specifies the threshold for a metric, and triggers an autoscale event when this threshold is crossed. Autoscaling can also deallocate resources when the workload has diminished.