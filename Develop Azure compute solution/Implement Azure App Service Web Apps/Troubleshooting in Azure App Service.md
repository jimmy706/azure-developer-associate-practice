#azure_app_service #troubleshooting
# Health check
![app service health check](https://learn.microsoft.com/en-us/azure/app-service/media/app-service-monitor-instances-health-check/health-check-diagram.png)

## What App Service does with health check?
- When given a path on your app, Health check pings this path on all instances of your App Service app at 1-minute intervals.
- If an instance doesn't respond with a status code between 200-299 (inclusive) after 10 requests, App Service determines it's unhealthy and removes it from the load balancer for this Web App. The required number of failed requests for an instance to be deemed unhealthy is configurable to a minimum of two requests.
- After removal, Health check continues to ping the unhealthy instance. If the instance begins to respond with a healthy status code (200-299), then the instance is returned to the load balancer.
- If an instance remains unhealthy for one hour, it's replaced with a new instance.
- When scaling out, App Service pings the Health check path to ensure new instances are ready.

## Enable healthcheck
1. To enable Health check, browse to the Azure portal and select your App Service app.
2. Under **Monitoring**, select **Health check**.
3. Select **Enable** and provide a valid URL path on your application, such as `/health` or `/api/health`.
4. Select **Save**.
![enable health check](https://learn.microsoft.com/en-us/azure/app-service/media/app-service-monitor-instances-health-check/azure-portal-navigation-health-check.png)

> Health check configuration changes restart your app. To minimize impact to production apps, we recommend [configuring staging slots](https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots) and swapping to production.
## Configure healthceck

|App setting name|Allowed values|Description|
|---|---|---|
|`WEBSITE_HEALTHCHECK_MAXPINGFAILURES`|2 - 10|The required number of failed requests for an instance to be deemed unhealthy and removed from the load balancer. For example, when set to `2`, your instances are removed after `2` failed pings. (Default value is `10`)|
|`WEBSITE_HEALTHCHECK_MAXUNHEALTHYWORKERPERCENT`|1 - 100|By default, no more than half of the instances will be excluded from the load balancer at one time to avoid overwhelming the remaining healthy instances. For example, if an App Service Plan is scaled to four instances and three are unhealthy, two are excluded. The other two instances (one healthy and one unhealthy) continue to receive requests. In the worst-case scenario where all instances are unhealthy, none are excluded.  <br>To override this behavior, set app setting to a value between `1` and `100`. A higher value means more unhealthy instances are removed (default value is `50`).|

## Healthcheck authentication

Health check integrates with App Service's [authentication and authorization features](https://learn.microsoft.com/en-us/azure/app-service/overview-authentication-authorization). No other settings are required if these security features are enabled.

If you're using your own authentication system, the Health check path must allow anonymous access. To secure the Health check endpoint, you should first use features such as [IP restrictions](https://learn.microsoft.com/en-us/azure/app-service/app-service-ip-restrictions#set-an-ip-address-based-rule), [client certificates](https://learn.microsoft.com/en-us/azure/app-service/app-service-ip-restrictions#set-an-ip-address-based-rule), or a Virtual Network to restrict application access. Once you have those features in-place, you can authenticate the health check request by inspecting the header, `x-ms-auth-internal-token`, and validating that it matches the SHA256 hash of the environment variable `WEBSITE_AUTH_ENCRYPTION_KEY`. If they match, then the health check request is valid and originating from App Service.

```python
from hashlib import sha256
import base64
import os

def header_matches_env_var(header_value):
    """
    Returns true if SHA256 of header_value matches WEBSITE_AUTH_ENCRYPTION_KEY.
    
    :param header_value: Value of the x-ms-auth-internal-token header.
    """
    
    env_var = os.getenv('WEBSITE_AUTH_ENCRYPTION_KEY')
    hash = base64.b64encode(sha256(env_var.encode('utf-8')).digest()).decode('utf-8')
    return hash == header_value
```

# Diagnostics logging
When you’re running a web application, you want to be prepared for any issues that may arise, from 500 errors to your users telling you that your site is down. App Service diagnostics is an intelligent and interactive experience to help you troubleshoot your app with no configuration required. If you do run into issues with your app, App Service diagnostics points out what’s wrong to guide you to the right information to more easily and quickly troubleshoot and resolve the issue.

## Enable diagnostics logging in Azure app service
To enable application logging for Linux apps or custom containers in the [Azure portal](https://portal.azure.com/), navigate to your app and select **App Service logs**.

In **Application logging**, select **File System**.

In **Quota (MB)**, specify the disk quota for the application logs. In **Retention Period (Days)**, set the number of days the logs should be retained.

When finished, select **Save**.

## Access the log files

For logs stored in the App Service file system, the easiest way is to download the ZIP file in the browser at:

- Linux/custom containers: `https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip`
- Windows apps: `https://<app-name>.scm.azurewebsites.net/api/dump`
In the App Service file system, these log files are the contents of the _/home/LogFiles_ directory.