# Azure Container Instance
## Create a resource group
az group create --name learn-deploy-aci-rg --location eastus

## Create an environment variable "DNS_NAME_LABEL" with a random integer appended to the value
DNS_NAME_LABEL=aci-demo-$RANDOM

## Print out the value assigned to the environment variable in the CLI
echo "$DNS_NAME_LABEL"

## Create a new container instance in the "learn-deploy-aci-rg" resource group based on the listed image utilizing an enviornment variable for the dns
az container create \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer \
  --image microsoft/aci-helloworld \
  --ports 80 \
  --dns-name-label $DNS_NAME_LABEL \
  --location eastus

## Display the full qualified domain name of the deployed container (allows you to access the container via the web)
az container show \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer \
  --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
  --out table

## Deploy a container utilizing different restart policies.  This sample container runs through once and self terminates.
- "restart-policy" - 3 Options
    - Always (Best used for web apps)
    - OnFailure (Good use = single run containers which will self terminate & have a method to handle errors in previous runs)
    - Never
az container create \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer-restart-demo \
  --image microsoft/aci-wordcount:latest \
  --restart-policy OnFailure \
  --location eastus

## Loads the state of the target container (could be running, terminated, etc.)
az container show \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer-restart-demo \
  --query containers[0].instanceView.currentState.state

## Displays the logs generated by a certain container
az container logs \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer-restart-demo

# Azure Cosmos DB
## Create Environment Variable for the Cosmos DB Name & echo resulting value
COSMOS_DB_NAME=aci-cosmos-db-$RANDOM
echo "$COSMOS_DB_NAME"

## Create a Cosmos DB Instance
COSMOS_DB_ENDPOINT=$(az cosmosdb create \
  --resource-group learn-deploy-aci-rg \
  --name $COSMOS_DB_NAME \
  --query documentEndpoint \
  --output tsv)

## Get the Azure Cosmos DB connection key and store it in a Bash variable named COSMOS_DB_MASTERKEY
COSMOS_DB_MASTERKEY=$(az cosmosdb keys list \
  --resource-group learn-deploy-aci-rg \
  --name $COSMOS_DB_NAME \
  --query primaryMasterKey \
  --output tsv)


# Connecting Azure Container Instance & Cosmos DB
## Create a new container and using the previously captured environment values point the container to the cosmos db
- The environment-variables argument passes the specified variables defined afterwards into the container.
    - Note: The container needs to be configured to watch for these variables in order for this to work
az container create \
  --resource-group learn-deploy-aci-rg \
  --name aci-demo \
  --image microsoft/azure-vote-front:cosmosdb \
  --ip-address Public \
  --location eastus \
  --environment-variables \
    COSMOS_DB_ENDPOINT=$COSMOS_DB_ENDPOINT \
    COSMOS_DB_MASTERKEY=$COSMOS_DB_MASTERKEY


## Show the IP address of the created container based on the name
az container show \
  --resource-group learn-deploy-aci-rg \
  --name aci-demo \
  --query ipAddress.ip \
  --output tsv

## Displays the environment variables in the target container
az container show \
  --resource-group learn-deploy-aci-rg \
  --name aci-demo \
  --query containers[0].environmentVariables

## Creates a new container however the environment variables are secured (i.e. not stored in plain text)
- This uses the "secure-environment-variables" attribute instead of the standard "environment-variables" attribute
    - The environment variable Names created with this will still appear when you run the container show command however the values will be nulled out
az container create \
  --resource-group learn-deploy-aci-rg \
  --name aci-demo-secure \
  --image microsoft/azure-vote-front:cosmosdb \
  --ip-address Public \
  --location eastus \
  --secure-environment-variables \
    COSMOS_DB_ENDPOINT=$COSMOS_DB_ENDPOINT \
    COSMOS_DB_MASTERKEY=$COSMOS_DB_MASTERKEY

## Show the IP address of the created container based on the name
az container show \
  --resource-group learn-deploy-aci-rg \
  --name aci-demo-secure \
  --query ipAddress.ip \
  --output tsv

## Displays the environment variables in the target container
az container show \
  --resource-group learn-deploy-aci-rg \
  --name aci-demo-secure \
  --query containers[0].environmentVariables


# Azure File Share / Storage Account
## Create Environment Variable for the storage name & echo resulting value
STORAGE_ACCOUNT_NAME=mystorageaccount$RANDOM
echo "$STORAGE_ACCOUNT_NAME"


## Create a storage account with the name defined by the environment variable
az storage account create \
  --resource-group learn-deploy-aci-rg \
  --name $STORAGE_ACCOUNT_NAME \
  --sku Standard_LRS \
  --location eastus


## Save the storage account connection string into a new environment variable
export AZURE_STORAGE_CONNECTION_STRING=$(az storage account show-connection-string \
  --resource-group learn-deploy-aci-rg \
  --name $STORAGE_ACCOUNT_NAME \
  --output tsv)


## Create a file share in the storage account
az storage share create --name aci-share-demo


## Retrieve the storage account key needed to access to storage account
- Note: To mount a file share you need 3 values
    - Storage Account Name (In this example it is saved in the environment variable STORAGE_ACCOUNT_NAME)
    - Share Name: This was defined in the previous step as "aci-share-demo"
    - Storage Account Access Key: Retrieve below
STORAGE_KEY=$(az storage account keys list \
  --resource-group learn-deploy-aci-rg \
  --account-name $STORAGE_ACCOUNT_NAME \
  --query "[0].value" \
  --output tsv)


# Container Instance with File Share Mounting
## Create a container which mounts a file share
- The example below mounts /aci/logs/ to the file share
    - This command did not work from the local CLI, had to run on the azure cloud shell to get it to work.  Need to look into this to understand the reason why.
az container create \
  --resource-group learn-deploy-aci-rg \
  --name aci-demo-files \
  --image microsoft/aci-hellofiles \
  --location eastus \
  --ports 80 \
  --ip-address Public \
  --azure-file-volume-account-name $STORAGE_ACCOUNT_NAME \
  --azure-file-volume-account-key $STORAGE_KEY \
  --azure-file-volume-share-name aci-share-demo \
  --azure-file-volume-mount-path /aci/logs/

## Determine the IP address of the file share
az container show \
  --resource-group learn-deploy-aci-rg \
  --name aci-demo-files \
  --query ipAddress.ip \
  --output tsv


## View the files saved to your file share
az storage file list -s aci-share-demo -o table


## Download a file stored in your file share based on the name (filename)
az storage file download -s aci-share-demo -p <filename>


# Troubleshooting Containers
## Create a basic container
az container create \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer \
  --image microsoft/sample-aks-helloworld \
  --ports 80 \
  --ip-address Public \
  --location eastus

## View output logs from container's running application
az container logs \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer

## View container logs & events and keep a live connection to a container's running application
az container attach \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer

## Open a CLI within your target container
- Note: Did not work in local shell.  Worked in Azure Online CLI
az container exec \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer \
  --exec-command /bin/sh

## Get the ID of the target container & store
CONTAINER_ID=$(az container show \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer \
  --query id \
  --output tsv)

## Get the CPU usage information for the target container (stored in environment variable)
- Note: Did not work in local shell.  Worked in Azure Online CLI
az monitor metrics list \
  --resource $CONTAINER_ID \
  --metric CPUUsage \
  --output table

## Get the Memory usage information for the target container (stored in environment variable)
- Note: Did not work in local shell.  Worked in Azure Online CLI
az monitor metrics list \
  --resource $CONTAINER_ID \
  --metric MemoryUsage \
  --output table