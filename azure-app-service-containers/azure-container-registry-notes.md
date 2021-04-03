# Create a Resource Group

# Create a Container Registry
- az acr create --name <myregistry> --resource-group <mygroup> --sku standard --admin-enabled true
  - Note: admin-enabled true should not be used in general, it is better to handle permissions through Azure AD

# Build an image from a Container Registry
- az acr build --file <Dockerfile> --registry <myregistry> --image <myimage> .

# Enable continuous integration from github -> docker build -> app deploy
CLI
1. Create a task that will build the updated web-app
   - az acr task create --registry <container_registry_name> --name buildwebapp --image webimage --context https://github.com/MicrosoftDocs/mslearn-deploy-run-container-app-service.git --file Dockerfile --git-access-token <access_token>

Portal
1. Navigate to the deployed web app & open the Deployment Center
2. In the Deployment Center enable "Continuous Deployment"
3. This will automatically create a Webhook in the source Container Registry (Can view this from the container registry in the Portal)
- Now when this container is updated with a new build it will automatically be deployed



azappservicelearntw123

azappservicelearnctw123