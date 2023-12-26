# Explore Azure Function development

A **function** contains two important pieces - your code, which can be written in various languages, and some config, the _function.json_ file. For compiled languages, this config file is generated automatically from annotations in your code. For scripting languages, you must provide the config file yourself.

The _function.json_ file defines the function's trigger, bindings, and other configuration settings. Every function has one and only one trigger. The runtime uses this config file to determine the events to monitor and how to pass data into and return data from a function execution. Following is an example _function.json_ file.

```json
{
    "disabled":false,
    "bindings":[
        // ... bindings here
        {
            "type": "bindingType",
            "direction": "in",
            "name": "myParamName",
            // ... more depending on binding
        }
    ]
}
```

The `bindings` property is where you configure both triggers and bindings. Each binding shares a few common settings and some settings that are specific to a particular type of binding. Every binding requires the following settings:

|Property|Types|Comments|
|---|---|---|
|`type`|string|Name of binding. For example, `queueTrigger`.|
|`direction`|string|Indicates whether the binding is for receiving data into the function or sending data from the function. For example, `in` or `out`.|
|`name`|string|The name that is used for the bound data in the function. For example, `myQueue`.|

## Function App

A function app provides an execution context in Azure in which your functions run. As such, it's the unit of deployment and management for your functions. A function app is composed of one or more individual functions that are managed, deployed, and scaled together. All of the functions in a function app share the same pricing plan, deployment method, and runtime version. Think of a function app as a way to organize and collectively manage your functions.

# Create triggers and bindings

> Triggers cause a function to run. A trigger defines how a function is invoked and a function must have exactly one trigger. Triggers have associated data, which is often provided as the payload of the function. Binding to a function is a way of declaratively connecting another resource to the function; bindings may be connected as _input bindings_, _output bindings_, or both. Data from bindings is provided to the function as parameters.

You can mix and match different bindings to suit your needs. Bindings are optional and a function might have one or multiple input and/or output bindings.

Triggers and bindings let you avoid hardcoding access to other services. Your function receives data (for example, the content of a queue message) in function parameters. You send data (for example, to create a queue message) by using the return value of the function.

## Trigger and binding definitions

|Language|Triggers and bindings are configured by...|
|---|---|
|C# class library|decorating methods and parameters with C# attributes|
|Java|decorating methods and parameters with Java annotations|
|JavaScript/PowerShell/Python/TypeScript|updating _function.json_ schema|

## Binding direction
All triggers and bindings have a direction property in the _function.json_ file:

- For triggers, the direction is always `in`
- Input and output bindings use `in` and `out`
- Some bindings support a special direction `inout`. If you use `inout`, only the **Advanced editor** is available via the **Integrate** tab in the portal.

When you use attributes in a class library to configure triggers and bindings, the direction is provided in an attribute constructor or inferred from the parameter type.

## Azure Functions trigger and binding example
Suppose you want to write a new row to Azure Table storage whenever a new message appears in Azure Queue storage. This scenario can be implemented using an Azure Queue storage trigger and an Azure Table storage output binding.

```json
{
  "bindings": [
    {
      "type": "queueTrigger",
      "direction": "in",
      "name": "order",
      "queueName": "myqueue-items",
      "connection": "MY_STORAGE_ACCT_APP_SETTING"
    },
    {
      "type": "table",
      "direction": "out",
      "name": "$return",
      "tableName": "outTable",
      "connection": "MY_TABLE_STORAGE_ACCT_APP_SETTING"
    }
  ]
}
```

# Connect functions to Azure Service
Your function project references connection information by name from its configuration provider. It doesn't directly accept the connection details, allowing them to be changed across environments. For example, a trigger definition might include a `connection` property, but you can't set the connection string directly in a `function.json`. Instead, you would set `connection` to the name of an environment variable that contains the connection string.

The default configuration provider uses environment variables that are set in [Application Settings](https://learn.microsoft.com/en-us/azure/azure-functions/functions-how-to-use-azure-function-app-settings?tabs=portal#settings) when running in the Azure Functions service, or from the [local settings file](https://learn.microsoft.com/en-us/azure/azure-functions/functions-develop-local#local-settings-file) when developing locally.

## Configure an identity-based connection
Some connections in Azure Functions are configured to use an identity instead of a secret. Support depends on the extension using the connection. In some cases, a connection string may still be required in Functions even though the service to which you're connecting supports identity-based connections.
When hosted in the Azure Functions service, identity-based connections use a [managed identity](https://learn.microsoft.com/en-us/azure/app-service/overview-managed-identity?toc=/azure/azure-functions/toc.json). The system-assigned identity is used by default, although a user-assigned identity can be specified with the `credential` and `clientID` properties. When run in other contexts, such as local development, your developer identity is used instead.

### Grant permission to the identity

Whatever identity is being used must have permissions to perform the intended actions. This is typically done by assigning a role in Azure RBAC or specifying the identity in an access policy, depending on the service to which you're connecting.

# When to use Azure Functions?
Consider the following examples of how you could implement different functions.

|Example scenario|Trigger|Input binding|Output binding|
|---|---|---|---|
|A new queue message arrives which runs a function to write to another queue.|Queue*|_None_|Queue*|
|A scheduled job reads Blob Storage contents and creates a new Azure Cosmos DB document.|Timer|Blob Storage|Azure Cosmos DB|
|The Event Grid is used to read an image from Blob Storage and a document from Azure Cosmos DB to send an email.|Event Grid|Blob Storage and Azure Cosmos DB|SendGrid|
|A webhook that uses Microsoft Graph to update an Excel sheet.|HTTP|_None_|Microsoft Graph|
