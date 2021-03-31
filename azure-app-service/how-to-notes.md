Azure App Service is a great way to get up and running quickly with a Web App backed by built in CI/CD and testing capabilities.

1. From the marketplace search for "Web App" and select the Microsoft provided Web App Resource
    - This will create a resource group and components for your web app.  Additionally this supports containers!

2. If you are configuring something basic from the Cloud Shell you can:
    - Navigate through the file structure using familiar commands
        - If its a node envrionment cd, mkdir, touch all work.
    - To initialize a new node project you can
        - Run npm init -y to create the package.json file necessary for configuration
    - Files can be edited directly in browser by running the command: <code .> this will open an interactive text editor in the Cloud Shell
        - Using Ctrl + S in the code window will save the file edits
        - Using Ctrl + Q will exit the code editor

3. Automated Deployment Options (of interest):
    - Azure DevOps
    - GitHub

4. Manual Deployment Options (of interest):
    - Git
    - az webapp up (CLI Command)
        - This command requires multiple pieces of information to deploy.  Here is are some required variables from the guide (stored in ENV variables)
            - APPNAME=$(az webapp list --query [0].name --output tsv)
            - APPRG=$(az webapp list --query [0].resourceGroup --output tsv)
            - APPPLAN=$(az appservice plan list --query [0].name --output tsv)
            - APPSKU=$(az appservice plan list --query [0].sku.name --output tsv)
            - APPLOCATION=$(az appservice plan list --query [0].location --output tsv)
        - Once this information has been gathered the app can be launched with the following command (referencing the ENV variables)
            - az webapp up --name $APPNAME --resource-group $APPRG --plan $APPPLAN --sku $APPSKU --location "$APPLOCATION"