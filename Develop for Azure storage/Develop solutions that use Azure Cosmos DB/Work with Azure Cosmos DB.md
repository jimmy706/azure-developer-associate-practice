# Azure Cosmos DB for NoSQL Client and Python
## Object Model
Azure Cosmos DB has a specific object model used to create and access resources. The Azure Cosmos DB creates resources in a hierarchy that consists of accounts, databases, containers, and items.
![Azure Cosmos DB Object Model](https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/includes/media/object-model/resource-hierarchy.svg)

You'll use the following Python classes to interact with these resources:

- [`CosmosClient`](https://learn.microsoft.com/en-us/python/api/azure-cosmos/azure.cosmos.cosmos_client.cosmosclient) - This class provides a client-side logical representation for the Azure Cosmos DB service. The client object is used to configure and execute requests against the service.
- [`DatabaseProxy`](https://learn.microsoft.com/en-us/python/api/azure-cosmos/azure.cosmos.database.databaseproxy) - This class is a reference to a database that may, or may not, exist in the service yet. The database is validated server-side when you attempt to access it or perform an operation against it.
- [`ContainerProxy`](https://learn.microsoft.com/en-us/python/api/azure-cosmos/azure.cosmos.container.containerproxy) - This class is a reference to a container that also may not exist in the service yet. The container is validated server-side when you attempt to work with it.

## Create Azure Cosmos Db using Python SDK

### Authentication with `DefaultAzureCredential`
```python
# app.py
import os
import json
import asyncio

from azure.cosmos.aio import CosmosClient
from azure.identity.aio import DefaultAzureCredential

endpoint = os.environ["COSMOS_ENDPOINT"] # Define Cosmos db endpoint
DATABASE_NAME = "cosmicworks" # Create database name
CONTAINER_NAME = "products" # Create container name

# Create new client instance using CosmosClient and DefaultAzureCredential
client = async with CosmosClient(url=ENDPOINT, credential=credential) as client:
    database = client.get_database_client(DATABASE_NAME)
    print("Database\t", database.id)
```

### Create database

```python
database = await client.create_database_if_not_exists(id=DATABASE_NAME)
print("Database\t", database.id)
```

### Create container
```python
key_path = PartitionKey(path="/categoryId")
container = await database.create_container_if_not_exists(
    id=CONTAINER_NAME, partition_key=key_path, offer_throughput=400
)
print("Container\t", container.id)
```

### Create items
```python
new_item = {
    "id": "70b63682-b93a-4c77-aad2-65501347265f",
    "categoryId": "61dba35b-4f02-45c5-b648-c6badc0cbd79",
    "categoryName": "gear-surf-surfboards",
    "name": "Yamba Surfboard",
    "quantity": 12,
    "sale": False,
}
await container.create_item(new_item)
```
# Created stored procedures
Azure Cosmos DB provides language-integrated, transactional execution of JavaScript that lets you write **stored procedures**, **triggers**, and **user-defined functions (UDFs)**. To call a stored procedure, trigger, or user-defined function, you need to register it. For more information, see [How to work with stored procedures, triggers, user-defined functions in Azure Cosmos DB](https://learn.microsoft.com/en-us/azure/cosmos-db/sql/how-to-use-stored-procedures-triggers-udfs).

## Writing stored procedures

Stored procedures can create, update, read, query, and delete items inside an Azure Cosmos container. Stored procedures are registered per collection, and can operate on any document or an attachment present in that collection.

```javascript
var helloWorldStoredProc = {
    id: "helloWorld",
    serverScript: function () {
        var context = getContext();
        var response = context.getResponse();

        response.setBody("Hello, World");
    }
}
```

The context object provides access to all operations that can be performed in Azure Cosmos DB, and access to the request and response objects. In this case, you use the response object to set the body of the response to be sent back to the client.

## Create items using stored procedure
When you create an item by using stored procedure, it's inserted into the Azure Cosmos container and an ID for the newly created item is returned. Creating an item is an asynchronous operation and depends on the JavaScript callback functions. The callback function has two parameters:

- The error object in case the operation fails
- A return value

Inside the callback, you can either handle the exception or throw an error. In case a callback isn't provided and there's an error, the Azure Cosmos DB runtime throws an error.

The stored procedure also includes a parameter to set the description, it's a boolean value. When the parameter is set to true and the description is missing, the stored procedure throws an exception. Otherwise, the rest of the stored procedure continues to run.

```javascript
var createDocumentStoredProc = {
    id: "createMyDocument",
    body: function createMyDocument(documentToCreate) {
        var context = getContext();
        var collection = context.getCollection();
        var accepted = collection.createDocument(collection.getSelfLink(),
              documentToCreate,
              function (err, documentCreated) {
                  if (err) throw new Error('Error' + err.message);
                  context.getResponse().setBody(documentCreated.id)
              });
        if (!accepted) return;
    }
}
```

## Arrays as input parameters for stored procedure
When defining a stored procedure in the Azure portal, input parameters are always sent as a string to the stored procedure. Even if you pass an array of strings as an input, the array is converted to string and sent to the stored procedure. To work around this, you can define a function within your stored procedure to parse the string as an array. The following code shows how to parse a string input parameter as an array:
```javascript
function sample(arr) {
    if (typeof arr === "string") arr = JSON.parse(arr);

    arr.forEach(function(a) {
        // do something here
        console.log(a);
    });
}
```

## Bounded execution
All Azure Cosmos DB operations must complete within a limited amount of time. Stored procedures have a limited amount of time to run on the server. All collection functions return a Boolean value that represents whether that operation completes or not
## Transactions within stored procedures
You can implement transactions on items within a container by using a stored procedure. JavaScript functions can implement a continuation-based model to batch or resume execution. The continuation value can be any value of your choice and your applications can then use this value to resume a transaction from a new starting point. The diagram below depicts how the transaction continuation model can be used to repeat a server-side function until the function finishes its entire processing workload.

![transactions within stored procedures](https://learn.microsoft.com/en-us/training/wwl-azure/work-with-cosmos-db/media/transaction-continuation-model.png)

