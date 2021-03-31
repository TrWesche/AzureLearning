# Building a Docker Container on Azure
1. Create a resource group to operate in (if your target resource group does not already exist):
    - az group create --name learn-deploy-acr-rg --location <choose-a-location>
        - Replace "<choose-a-location>" with the target region.  For example eastus
2. Create a container registry which will house your container
    - You can save this in an environment variable like so:
        - ACR_NAME=howtocicdinazuretw
    - Create the container registry in the target resource group (here learn-deploy-acr-rg)
        - az acr create --resource-group learn-deploy-acr-rg --name $ACR_NAME --sku Premium
3. Create the Dockerfile to build from
4. Run the build command
    - az acr build --registry $ACR_NAME --image helloacrtasks:v1 .
5. Verify the image was built as expected
    - az acr repository list --name $ACR_NAME --output table
6. (Note: Not a Permanent Access Solution) Enable registry admin account
    - az acr update -n $ACR_NAME --admin-enabled true
        - Access should be handled via Azure Active Directory Identities at a later time, but the admin account allows for faster and easier setup.
7. Retrieve Admin Account credentials
    - az acr credential show --name $ACR_NAME
8. Deploy a container instance
    - az container create \
    --resource-group learn-deploy-acr-rg \
    --name acr-tasks \
    --image $ACR_NAME.azurecr.io/helloacrtasks:v1 \
    --registry-login-server $ACR_NAME.azurecr.io \
    --ip-address Public \
    --location <location> \
    --registry-username [username] \
    --registry-password [password]
        - location: Replace with location provided when creating the container registry
        - username: Replace with username from the previous step
        - password: Replace with password from previous step

    - ex. az container create \
    --resource-group learn-deploy-acr-rg \
    --name acr-tasks \
    --image $ACR_NAME.azurecr.io/helloacrtasks:v1 \
    --registry-login-server $ACR_NAME.azurecr.io \
    --ip-address Public \
    --location eastus \
    --registry-username howtocicdinazuretw \
    --registry-password JNevwvOhH/pH8RiAozMTsp21W5rb8FiH
9. Retrieve the IP address of the deployed container instance (This IP is also shared upon creation complete)
    - az container show --resource-group  learn-deploy-acr-rg --name acr-tasks --query ipAddress.ip --output table