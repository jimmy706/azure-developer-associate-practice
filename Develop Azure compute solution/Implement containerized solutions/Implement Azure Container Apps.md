# Explore Azure Container App
Azure Container Apps enables you to run microservices and containerized applications on a serverless platform that runs on top of Azure Kubernetes Service. Common uses of Azure Container Apps include:

- Deploying API endpoints
- Hosting background processing applications
- Handling event-driven processing
- Running microservices

Applications built on Azure Container Apps can dynamically scale based on: HTTP traffic, event-driven processing, CPU or memory load, and any [KEDA-supported scaler](https://keda.sh/docs/scalers/).

## Compare Azure Container Apps with Azure Container Instances and Azure App Service
### Container Apps vs ACI
**Azure Container Instances (ACI) provides a single pod of Hyper-V isolated containers on demand.** **It can be thought of as a lower-level "building block" option compared to Container Apps**. Concepts like scale, load balancing, and certificates are not provided with ACI containers.

### Container Apps vs App Service
Azure App Service is a fully managed platform that provides hosting for web applications including websites and web APIs. It is optimized for web applications. Container Apps is more focused on containerization which you have more control over your running application.

|Feature|Azure Container App|Azure Container Instances|Azure App Service|
|---|---|---|---|
|Container Orchestration|Yes|No|No|
|Scaling|Automatic|Manual|Automatic|
|Load Balancing|Yes|No|Yes|
|Custom Domain|Yes|No|Yes|
|SSL/TLS|Yes|No|Yes|
|Custom Docker Image|Yes|Yes|Limited|
|Continuous Deployment|Yes|No|Yes|
|Monitoring and Logs|Yes|Limited|Yes|
|Environment Variables|Yes|Limited|Yes|
|Integration with Azure VNET|Yes|Limited|Yes|
|Persistent Storage|Yes|Limited|Yes|
|Service Plan|Consumption-based|Pay-as-you-go|Various Options|
|Price Model|Pay-per-use|Pay-per-use|Pay-per-use|\


## What Azure Container App can do?

- Run multiple container revisions and manage the container app's application lifecycle.
- Autoscale your apps based on any KEDA-supported scale trigger. Most applications can scale to zero. (Applications that scale on CPU or memory load can't scale to zero.)
- Enable HTTPS ingress without having to manage other Azure infrastructure.
- Split traffic across multiple versions of an application for Blue/Green deployments and A/B testing scenarios.
- Use internal ingress and service discovery for secure internal-only endpoints with built-in DNS-based service discovery.
- Build microservices with [Dapr](https://docs.dapr.io/concepts/overview/) and access its rich set of APIs.
- Run containers from any registry, public or private, including Docker Hub and Azure Container Registry (ACR).
- Use the Azure CLI extension, Azure portal or ARM templates to manage your applications.
- Provide an existing virtual network when creating an environment for your container apps.
- Securely manage secrets directly in your application.
- Monitor logs using Azure Log Analytics.

## Azure Container App Environment
Individual container apps are deployed to a single Container Apps environment, which acts as a secure boundary around groups of container apps. Container Apps in the same environment are deployed in the same virtual network and write logs to the same Log Analytics workspace. You may provide an existing virtual network when you create an environment.

Reasons to deploy container apps to the same environment include situations when you need to:

- Manage related services
- Deploy different applications to the same virtual network
- Instrument [Dapr](https://docs.dapr.io/concepts/overview/) applications that communicate via the Dapr service invocation API
- Have applications to share the same Dapr configuration
- Have applications share the same log analytics workspace

Reasons to deploy container apps to different environments include situations when you want to ensure:

- Two applications never share the same compute resources
- Two Dapr applications can't communicate via the Dapr service invocation API
## Microservice with Azure Container App

Microservice architectures allow you to independently develop, upgrade, version, and scale core areas of functionality in an overall system. Azure Container Apps provides the foundation for deploying microservices featuring:

- Independent scaling, versioning, and upgrades
- Service discovery
- Native [Dapr](https://docs.dapr.io/concepts/overview/) integration

## Dapr Integration
When you implement a system composed of microservices, function calls are spread across the network. To support the distributed nature of microservices, you need to account for failures, retries, and timeouts. While Container Apps features the building blocks for running microservices, use of Dapr provides an even richer microservices programming model. Dapr includes features like observability, pub/sub, and service-to-service invocation with mutual TLS, retries, and more.

# Explore containers in Azure Container Apps

Azure Container Apps manages the details of Kubernetes and container orchestration for you. Containers in Azure Container Apps can use any runtime, programming language, or development stack of your choice.

## Configuration
The following code is an example of the `containers` array in the `properties.template` section of a container app resource template. The excerpt shows some of the available configuration options when setting up a container when using Azure Resource Manager (ARM) templates. Changes to the template ARM configuration section trigger a new container app revision.

```json
"containers": [
  {
       "name": "main",
       "image": "[parameters('container_image')]",
    "env": [
      {
        "name": "HTTP_PORT",
        "value": "80"
      },
      {
        "name": "SECRET_VAL",
        "secretRef": "mysecret"
      }
    ],
    "resources": {
      "cpu": 0.5,
      "memory": "1Gi"
    },
    "volumeMounts": [
      {
        "mountPath": "/myfiles",
        "volumeName": "azure-files-volume"
      }
    ]
    "probes":[
        {
            "type":"liveness",
            "httpGet":{
            "path":"/health",
            "port":8080,
            "httpHeaders":[
                {
                    "name":"Custom-Header",
                    "value":"liveness probe"
                }]
            },
            "initialDelaySeconds":7,
            "periodSeconds":3
// file is truncated for brevity
```

## Multiple Containers
You can define multiple containers in a single container app to implement the [sidecar pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/sidecar). The containers in a container app share hard disk and network resources and experience the same application lifecycle.

Examples of sidecar containers include:

- An agent that reads logs from the primary app container on a shared volume and forwards them to a logging service.
- A background process that refreshes a cache used by the primary app container in a shared volume.

> Running multiple containers in a single container app is an advanced use case. In most situations where you want to run multiple containers, such as when implementing a microservice architecture, deploy each service as a separate container app.

## Container registries
You can deploy images hosted on private registries by providing credentials in the Container Apps configuration.

To use a container registry, you define the required fields in registries array in the `properties.configuration` section of the container app resource template. The `passwordSecretRef` field identifies the name of the secret in the secrets array name where you defined the password.

```json
{
  ...
  "registries": [{
    "server": "docker.io",
    "username": "my-registry-user-name",
    "passwordSecretRef": "my-password-secret-name"
  }]
}
```

## Limitations
Azure Container Apps has the following limitations:

- **Privileged containers**: Azure Container Apps can't run privileged containers. If your program attempts to run a process that requires root access, the application inside the container experiences a runtime error.
- **Operating system**: Linux-based (`linux/amd64`) container images are required.

# Authentication and Authorization in Azure Container Apps
Azure Container Apps provides built-in authentication and authorization features to secure your external ingress-enabled container app with minimal or no code. The built-in authentication feature for Container Apps can save you time and effort by providing out-of-the-box authentication with federated identity providers, allowing you to focus on the rest of your application.

- Azure Container Apps provides access to various built-in authentication providers.
- The built-in auth features don’t require any particular language, SDK, security expertise, or even any code that you have to write.

This feature should only be used with HTTPS. Ensure `allowInsecure` is disabled on your container app's ingress configuration. You can configure your container app for authentication with or without restricting access to your site content and APIs.

- To restrict app access only to authenticated users, set its _Restrict access_ setting to **Require authentication**.
- To authenticate but not restrict access, set its _Restrict access_ setting to **Allow unauthenticated** access.
## Identity providers
Container Apps uses federated identity, in which a third-party identity provider manages the user identities and authentication flow for you. The following identity providers are available by default:

|Provider|Sign-in endpoint|How-To guidance|
|---|---|---|
|Microsoft Identity Platform|`/.auth/login/aad`|[Microsoft Identity Platform](https://learn.microsoft.com/en-us/azure/container-apps/authentication-azure-active-directory)|
|Facebook|`/.auth/login/facebook`|[Facebook](https://learn.microsoft.com/en-us/azure/container-apps/authentication-facebook)|
|GitHub|`/.auth/login/github`|[GitHub](https://learn.microsoft.com/en-us/azure/container-apps/authentication-github)|
|Google|`/.auth/login/google`|[Google](https://learn.microsoft.com/en-us/azure/container-apps/authentication-google)|
|Twitter|`/.auth/login/twitter`|[Twitter](https://learn.microsoft.com/en-us/azure/container-apps/authentication-twitter)|
|Any OpenID Connect provider|`/.auth/login/<providerName>`|[OpenID Connect](https://learn.microsoft.com/en-us/azure/container-apps/authentication-openid)|

## Feature Architecture
The authentication and authorization middleware component is a feature of the platform that runs as *a sidecar container on each replica in your application*. When enabled, every incoming HTTP request passes through the security layer before being handled by your application.

![container apps authentication](https://learn.microsoft.com/en-us/training/wwl-azure/implement-azure-container-apps/media/container-apps-authorization-architecture.png)

The platform middleware handles several things for your app:

- Authenticates users and clients with the specified identity provider(s)
- Manages the authenticated session
- Injects identity information into HTTP request headers

The authentication and authorization module runs in a separate container, isolated from your application code. As the security container doesn't run in-process, no direct integration with specific language frameworks is possible. However, relevant information your app needs is provided in request headers.

## Authentication Flow
- **Without provider SDK** (server-directed flow or server flow): The application delegates federated sign-in to Container Apps. Delegation is typically the case with browser apps, which presents the provider's sign-in page to the user.
    
- **With provider SDK** (client-directed flow or client flow): The application signs users in to the provider manually and then submits the authentication token to Container Apps for validation. This approach is typical for browser-less apps that don't present the provider's sign-in page to the user. An example is a native mobile app that signs users in using the provider's SDK.

# Manage revisions and secrets in Azure Container Apps
Azure Container Apps implements container app versioning by creating revisions. A revision is an immutable snapshot of a container app version. You can use revisions to release a new version of your app, or quickly revert to an earlier version of your app. New revisions are created when you update your application with [revision-scope changes](https://learn.microsoft.com/en-us/azure/container-apps/revisions#revision-scope-changes). You can also update your container app based on a specific revision.

You can control which revisions are active, and the external traffic that is routed to each active revision. Revision names are used to identify a revision, and in the revision's URL. You can customize the revision name by setting the revision suffix.

By default, Container Apps creates a unique revision name with a suffix consisting of a semi-random string of alphanumeric characters. For example, for a container app named _album-api_, setting the revision suffix name to _1st-revision_ would create a revision with the name _album-api--1st-revision_. You can set the revision suffix in the ARM template, through the Azure CLI `az containerapp create` and `az containerapp update` commands, or when creating a revision via the Azure portal.

## Updating a container app
With the `az containerapp update` command you can modify environment variables, compute resources, scale parameters, and deploy a different image. If your container app update includes [revision-scope changes](https://learn.microsoft.com/en-us/azure/container-apps/revisions#revision-scope-changes), a new revision is generated.

```bash
az containerapp update \
  --name <APPLICATION_NAME> \
  --resource-group <RESOURCE_GROUP_NAME> \
  --image <IMAGE_NAME>
```

## Manage secrets in Azure Container Apps

Azure Container Apps allows your application to securely store sensitive configuration values. Once secrets are defined at the application level, secured values are available to container apps. Specifically, you can reference secured values inside scale rules.

- Secrets are scoped to an application, outside of any specific revision of an application.
- Adding, removing, or changing secrets doesn't generate new revisions.
- Each application revision can reference one or more secrets.
- Multiple revisions can reference the same secret(s).

An updated or deleted secret doesn't automatically affect existing revisions in your app. When a secret is updated or deleted, you can respond to changes in one of two ways:

1. Deploy a new revision.
2. Restart an existing revision.

Before you delete a secret, deploy a new revision that no longer references the old secret. Then deactivate all revisions that reference the secret.

> Container Apps doesn't support Azure Key Vault integration. Instead, enable managed identity in the container app and use the Key Vault SDK in your app to access secrets.

### Defining secrets
When you create a container app, secrets are defined using the `--secrets` parameter.

- The parameter accepts a space-delimited set of name/value pairs.
- Each pair is delimited by an equals sign (`=`).

In the example below a connection string to a queue storage account is declared in the `--secrets` parameter. The value for queue-connection-string comes from an environment variable named `$CONNECTION_STRING`.

```bash
az containerapp create \
  --resource-group "my-resource-group" \
  --name queuereader \
  --environment "my-environment-name" \
  --image demos/queuereader:v1 \
  --secrets "queue-connection-string=$CONNECTION_STRING"
```

After declaring secrets at the application level, you can reference them in environment variables when you create a new revision in your container app. When an environment variable references a secret, its value is populated with the value defined in the secret. To reference a secret in an environment variable in the Azure CLI, set its value to `secretref:`, followed by the name of the secret.

The following example shows an application that declares a connection string at the application level. This connection is referenced in a container environment variable.

```bash
az containerapp create \
  --resource-group "my-resource-group" \
  --name myQueueApp \
  --environment "my-environment-name" \
  --image demos/myQueueApp:v1 \
  --secrets "queue-connection-string=$CONNECTIONSTRING" \
  --env-vars "QueueName=myqueue" "ConnectionString=secretref:queue-connection-string"
```