Note: Much of this configuration is done from Visual Studio which handles partial WebJob orchestration for you

List all available regions
- az account list-locations --query [].name

Store env variable
- STORAGE_ACCOUNT_NAME=mslearnwebjobs$RANDOM

Create resource group
- az group create --name mslearn-webjobs --location centralus

Create storage account
- az storage account create \
    --name $STORAGE_ACCOUNT_NAME \
    --resource-group mslearn-webjobs

Get storage account connection string (and strong in env variable)
- STORAGE_ACCOUNT_CONNSTR=$(az storage account show-connection-string --name $STORAGE_ACCOUNT_NAME --query connectionString --output tsv)

After a Web App has been created on Azure you can retrieve the WEB_APP_ID (looks like a location) with the following command
- WEB_APP_ID=$(az webapp list --resource-group mslearn-webjobs --query [0].id --output tsv)

That Web App Id can then be used to configure the Web App from the CLI.  For instance setting the web app to be always on:
- az webapp config set --id $WEB_APP_ID --always-on true
  or setting the connection string to point towards your storage account
- az webapp config connection-string set --id $WEB_APP_ID --connection-string-type Custom --settings StorageAccount=$STORAGE_ACCOUNT_CONNSTR

If using the JobHost Object in Azure it supports multiple different trigger types:
    - A new Blob appears in a storage account.
    - A new item appears in a storage account queue.
    - An entity is added to an Azure Cosmos DB database.
    - An event occurs in a WebHook.
    - The schedule in the settings.json file triggers the WebJob.
  - Following are some example functions:
    - public static void ProcessQueueMessage([QueueTrigger("queue")] string message, ILogger logger)
        {
            logger.LogInformation(message);
        }
        ### Triggers on Queue
    - [FunctionName("BlobTriggerCSharp")]
        public static void Run([BlobTrigger("watchescontainer/{name}")] Stream watchInstructions, string name, ILogger log)
        {
            log.LogInformation($"A new blob was added.\n Name:{name} \n Size: {watchInstructions.Length}");
        }
        ### Triggers on Blob added to a storage account
    - [FunctionName("CosmosTrigger")]
        public static void Run([CosmosDBTrigger(
            databaseName: "ToDoItems",
            collectionName: "Items",
            ConnectionStringSetting = "CosmosDBConnection",
            LeaseCollectionName = "leases",
            CreateLeaseCollectionIfNotExists = true)]IReadOnlyList<Document> documents,
            TraceWriter log)
        {
            if (documents != null && documents.Count > 0)
            {
                log.Info($"Documents modified: {documents.Count}");
                log.Info($"First document Id: {documents[0].Id}");
            }
        }
        ### Triggers when data is added to an Azure Cosmos DB database
    - public static void ConfirmStockCheck([QueueTrigger("stockchecks")] string message, [Blob("confirmations/{id}")] out string output)
        {
            var timestamp = message.Split()[3];
            output = $"{timestamp} stock check successfully processed";
        }
        ### Looks at the stockchecks queue and processes messages there

The JobHost Object is dependent on 2 variables to operate:
    - AzureWebJobsDashboard & AzureWebJobsStorage
      - These can be configured directly in the XML file or the WebJob "App.config" or better can be configured as environment variables.

A special consideration when database names or other variables may change, use Bindings (these are placed in a file by the name of functions.json):
    - Example:
      - {
          "bindings": [
              {
              "type": "queueTrigger",
              "direction": "in",
              "name": "order",
              "queueName": "new-watches",
              "connection": "WATCHES_STORAGE_ACCT_APP_SETTING"
              },
              {
              "type": "table",
              "direction": "out",
              "name": "$return",
              "tableName": "Watches",
              "connection": "WATCHES_TABLE_STORAGE_ACCT_APP_SETTING"
              }
          ]
        }




VS Code Actions:
- A WebJob can be added to a project by right clicking on the project in the right pain and selecting Add > New Azure WebJob Project
- If there are no webjobs already configured this will create the webjob project and additionally add a "webjobs-list.json" file to the main WebApp project
