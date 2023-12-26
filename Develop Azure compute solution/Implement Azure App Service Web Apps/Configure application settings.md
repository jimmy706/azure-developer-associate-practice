In App Service, app settings are variables passed as environment variables to the application code. For Linux apps and custom containers, App Service passes app settings to the container using the `--env` flag to set the environment variable in the container.
You can access application settings in your App Service by selecting **Configuration > Application Settings**.
![](https://learn.microsoft.com/en-us/training/wwl-azure/configure-web-app-settings/media/configure-app-settings.png) 

> Application settings always encrypted when stored (encrypted-at-rest)

## Editing application settings in bulk
To add or edit app settings in bulk, select the **Advanced** edit button. When finished, select **Update**. App settings have the following JSON formatting:

```
[
  {
    "name": "<key-1>",
    "value": "<value-1>",
    "slotSetting": false
  },
  {
    "name": "<key-2>",
    "value": "<value-2>",
    "slotSetting": false
  },
  ...
]
```
## App Service general settings
Some options you can do on **Configuration > General settings** section:
- **Stack settings**: The software stack to run the app, including the language and SDK versions. For Linux apps and custom container apps, you can also set an optional start-up command or file.
- **Platform settings**: Lets you configure settings for the hosting platform, including:
    - **Bitness**: 32-bit or 64-bit.
    - **WebSocket protocol**: For ASP.NET SignalR or socket.io, for example.
    - **Always On**: Keep the app loaded even when there's no traffic. By default, **Always On** isn't enabled and the app is unloaded after 20 minutes without any incoming requests. It's required for continuous WebJobs or for WebJobs that are triggered using a CRON expression.
    - **Managed pipeline version**: The IIS pipeline mode. Set it to **Classic** if you have a legacy app that requires an older version of IIS.
    - **HTTP version**: Set to 2.0 to enable support for HTTPS/2 protocol.
    - **ARR affinity**: In a multi-instance deployment, ensure that the client is routed to the same instance for the life of the session. You can set this option to **Off** for stateless applications.
- **Debugging**: Enable remote debugging for ASP.NET, ASP.NET Core, or Node.js apps. This option turns off automatically after 48 hours.
- **Incoming client certificates**: require client certificates in mutual authentication. TLS mutual authentication is used to restrict access to your app by enabling different types of authentication for it.
## App Service Path Mapping
In the **Configuration > Path mappings** section you can configure handler mappings, and virtual application and directory mappings. The **Path mappings** page displays different options based on the OS type.

### Windows apps (uncontainerized)

For Windows apps, you can customize the IIS handler mappings and virtual applications and directories.

Handler mappings let you add custom script processors to handle requests for specific file extensions. To add a custom handler, select **New handler**. Configure the handler as follows:

- **Extension**: The file extension you want to handle, such as *_.php_ or _handler.fcgi_.
- **Script processor**: The absolute path of the script processor. Requests to files that match the file extension are processed by the script processor. Use the path `D:\home\site\wwwroot` to refer to your app's root directory.
- **Arguments**: Optional command-line arguments for the script processor.

Each app has the default root path (`/`) mapped to `D:\home\site\wwwroot`, where your code is deployed by default. If your app root is in a different folder, or if your repository has more than one application, you can edit or add virtual applications and directories.

You can configure virtual applications and directories by specifying each virtual directory and its corresponding physical path relative to the website root (`D:\home`). To mark a virtual directory as a web application, clear the **Directory** check box.

### Linux and containerized apps

You can add custom storage for your containerized app. Containerized apps include all Linux apps and also the Windows and Linux custom containers running on App Service. Select **New Azure Storage Mount** and configure your custom storage as follows:

- **Name**: The display name.
- **Configuration options**: Basic or Advanced.
- **Storage accounts**: The storage account with the container you want.
- **Storage type**: **Azure Blobs** or **Azure Files**. Windows container apps only support Azure Files.
- **Storage container**: For basic configuration, the container you want.
- **Share name**: For advanced configuration, the file share name.
- **Access key**: For advanced configuration, the access key.
- **Mount path**: The absolute path in your container to mount the custom storage.
## Enable diagnostic logging
The following table shows the types of logging, the platforms supported, and where the logs can be stored and located for accessing the information.

|Type|Platform|Location|Description|
|---|---|---|---|
|Application logging|Windows, Linux|App Service file system and/or Azure Storage blobs|Logs messages generated by your application code. The messages are generated by the web framework you choose, or from your application code directly using the standard logging pattern of your language. Each message is assigned one of the following categories: **Critical**, **Error**, **Warning**, **Info**, **Debug**, and **Trace**.|
|Web server logging|Windows|App Service file system or Azure Storage blobs|Raw HTTP request data in the W3C extended log file format. Each log message includes data like the HTTP method, resource URI, client IP, client port, user agent, response code, and so on.|
|Detailed error logging|Windows|App Service file system|Copies of the _.html_ error pages that would have been sent to the client browser. For security reasons, detailed error pages shouldn't be sent to clients in production, but App Service can save the error page each time an application error occurs that has HTTP code 400 or greater.|
|Failed request tracing|Windows|App Service file system|Detailed tracing information on failed requests, including a trace of the IIS components used to process the request and the time taken in each component. One folder is generated for each failed request, which contains the XML log file, and the XSL stylesheet to view the log file with.|
|Deployment logging|Windows, Linux|App Service file system|Helps determine why a deployment failed. Deployment logging happens automatically and there are no configurable settings for deployment logging.|
### Enable application logging (Windows)
1. To enable application logging for Windows apps in the Azure portal, navigate to your app and select **App Service logs**.
2. Select **On** for either **Application Logging (Filesystem)** or **Application Logging (Blob)**, or both. The **Filesystem** option is for temporary debugging purposes, and turns itself off in 12 hours. The **Blob** option is for long-term logging, and needs a blob storage container to write logs to.
3. You can also set the **Level** of details included in the log as shown in the following table.
4. When finished, select **Save**.
### Enable application logging (Linux/container)
1. In **App Service logs** set the **Application logging** option to **File System**.
2. In **Quota (MB)**, specify the disk quota for the application logs. In **Retention Period (Days)**, set the number of days the logs should be retained.
3. When finished, select **Save**.
### Stream logs
Before you stream logs in real time, enable the log type that you want. Any information written to files ending in .txt, .log, or .htm that are stored in the `/LogFiles` directory (`d:/home/logfiles`) is streamed by App Service.
- Azure portal - To stream logs in the Azure portal, navigate to your app and select **Log stream**.
    
- Azure CLI - To stream logs live in Cloud Shell, use the following command:
    ```
    az webapp log tail --name appname --resource-group myResourceGroup
    ```
- Local console - To stream logs in the local console, install Azure CLI and sign in to your account. Once signed in, follow the instructions shown for Azure CLI.
## Configure security certificate
Azure App Service has tools that let you create, upload, or import a private certificate or a public certificate into App Service.

A certificate uploaded into an app is stored in a deployment unit that is bound to the app service plan's resource group and region combination (internally called a _webspace_). This makes the certificate accessible to other apps in the same resource group and region combination.

The table below details the options you have for adding certificates in App Service:

|Option|Description|
|---|---|
|Create a free App Service managed certificate|A private certificate that's free of charge and easy to use if you just need to secure your custom domain in App Service.|
|Purchase an App Service certificate|A private certificate that's managed by Azure. It combines the simplicity of automated certificate management and the flexibility of renewal and export options.|
|Import a certificate from Key Vault|Useful if you use Azure Key Vault to manage your certificates.|
|Upload a private certificate|If you already have a private certificate from a third-party provider, you can upload it.|
|Upload a public certificate|Public certificates aren't used to secure custom domains, but you can load them into your code if you need them to access remote resources.|
