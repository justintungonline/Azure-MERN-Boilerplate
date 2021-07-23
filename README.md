# Azure-MERN-Boilerplate

A very basic boilerplate for an Azure ready MERN app

The tutorial for deploying this boilerplate can be found here:
<https://medium.com/@tuna.sogut/how-to-deploy-a-mern-stack-app-to-azure-via-continuous-integration-a3a551526e26?sk=0fc4fa9d7c7072ad7e95b94d7e5733e4>

## About This Repository

It builds on the tutorial by Tuna Sogut by:

- Describing two other options to deploy the MERN boilerplate application to an Azure Web App using:
  - [Local git push to remote](https://docs.microsoft.com/en-us/azure/app-service/deploy-local-git?tabs=cli), or
  - [Azure DevOps pipelines](https://azure.microsoft.com/en-ca/services/devops/pipelines/)
- Add Azure CLI method of creating a web app   

### Built With

- [MERN](https://www.mongodb.com/mern-stack)
- [Node JS](https://nodejs.org/en/)
- [Microsoft Azure](https://azure.microsoft.com/)

[![Gitpod ready-to-code](https://img.shields.io/badge/Gitpod-ready--to--code-blue?logo=gitpod)](https://gitpod.io/#https://github.com/justintungonline/)

## Getting Started

### Prerequisites

- [Yarn](https://classic.yarnpkg.com/en/docs/install#windows-stable)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/) (optional if you want to create the Azure App Services using command line)

### Installation

This section is a summary of key points in the tutorial.

In the file `routes/new-index.js` change:

```js

// The connection string the database to use your connection. You can create free one from MongoDB Atlas (https://www.mongodb.com/cloud/atlas) and create a database and initial collection.
var url = "mongodb+srv://<username>:<password>@<cluster>-vgz77.azure.mongodb.net/test?retryWrites=true&w=majority";
...
// Update the database name to name of your database.
  var dbo = db.db("<database>");
```

#### Build Application

```sh
cd <repository directory>
npm install
# Optionally fix vulnerabilities with npm audit fix

cd client
yarn install
npm run build
```

In the `client` directory, run `npm run build` to build whenever you have made changes.

#### Set up new Azure Web App

Set up a new Azure Web App using Node 12.0 LTS. Follow [the Microsoft tutorial](https://docs.microsoft.com/en-us/azure/app-service/quickstart-nodejs) or using Azure CLI:

```sh
# Log into Azure
az login

# Create resource group and set as default values to avoid specifying them each time later
az group create --name myResourceGroup --location westus
az config set defaults.group=myResourceGroup defaults.location=westus

# Create an App Service plan using Linux in free tier.
az appservice plan create --name Azure-MERN-Boilerplate --resource-group myResourceGroup --sku FREE --is-linux

# Create and deploy web app service with Azure CLI command
# Get run times using command: az webapp list-runtimes --linux
az webapp create --name Azure-MERN-Boilerplate --resource-group myResourceGroup --plan Azure-MERN-Boilerplate --runtime "NODE|12-lts"
```

The tutorial points out several important items in `server.js` that are required for running in Azure:

```js
// Get port from environment and do not hard code it
app.set('port', process.env.PORT || 5000);
console.log("++++++++++++++++" + app.get('port'));

...
// Front end is served as a static React build. “npm run build” is run in the client directory 
// This enables Express to serve up the build, which is how it serves the frontend on Azure.
app.use(express.static('./client/build'));

...
// point GET/ route in server.js to the index.html in build.
app.get("*", (req, res) => {
   res.sendFile(path.resolve(__dirname, "client", "build",     
   "index.html"));
});

```

## Usage: How to Deploy to Azure

### Using GitHub Integration

If your code is in GitHub, this option is easiest.

Follow the rest of [the tutorial (3. Setting up Continuous Integration)](https://medium.com/@tuna.sogut/how-to-deploy-a-mern-stack-app-to-azure-via-continuous-integration-a3a551526e26?sk=0fc4fa9d7c7072ad7e95b94d7e5733e4) for deployment to Azure using GitHub integration and GitHub workflows.

#### Use local git to push to Azure remote git which deploys the application

```sh
# In a separate terminal, set up user-level deployment credentials with Azure CLI
az webapp deployment user set --user-name <username> --password <password>
# If an error like: Operation returned an invalid status 'Conflict'
# occurs, it may mean the username is taken or password is not complex enough.
# Choose a different username and more complex password

# For "--deployment-local-git", get new remote git URL from the Azure Web App deployment setting or with:
az webapp deployment source config-local-git --name Azure-MERN-Boilerplate --resource-group myResourceGroup --query url --output tsv

# Add remote URL to git repo
git remote add azure <insert URL you got from the previous command>

# For "--deployment-local-git",
# Push you code to Azure and enter your password from the deployment user set command previously when asked. 
# Instead of 'main', you may want to choose your branch to push to Azure. 
# master is required after your local branch to ensure it pushes to the 'master' branch read for Azure deployment
git push azure main:master

# View (tail) logs
az webapp log tail --name Azure-MERN-Boilerplate
```

#### Using Azure DevOps pipelines to deploy to Azure

1. Create a [new Azure DevOps project](https://dev.azure.com).
2. Go to the Repos menu
3. Generate the git credentials and save your username and password
4. Use the push to this repository option and push from your local git repository to Azure DevOps repo
5. Enter your password from the credentials when asked
6. Go to Pipelines and create a new one using Node JS, Express template
7. When asked, login to Azure and select your Azure Web App with the pipeline.
8. Adjust the node version and build commands like the following `pipelines.yaml` below. These were changed from the auto-generated yaml: `versionSpec: '12.x'` and `runtimeStack: 'NODE|12-lts'` and the `script: |` section with the application specific build.

```yaml
# Node.js Express Web App to Linux on Azure
# Build a Node.js Express app and deploy it to Azure as a Linux web app.
# Add steps that analyze code, save build artifacts, deploy, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/javascript

trigger:
- main

variables:

  # Azure Resource Manager connection created during pipeline creation
  azureSubscription: 'fb7f0ac5-01b4-4c6a-9427-6c95079a39ff'

  # Web app name
  webAppName: 'Azure-MERN-Boilerplate'

  # Environment name
  environmentName: 'Azure-MERN-Boilerplate'

  # Agent VM image name
  vmImageName: 'ubuntu-latest'

stages:
- stage: Build
  displayName: Build stage
  jobs:
  - job: Build
    displayName: Build
    pool:
      vmImage: $(vmImageName)

    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '12.x'
      displayName: 'Install Node.js'

    - script: |
        npm install
        npm run build --if-present
        npm run test --if-present
        cd client
        yarn install
        npm run build
      displayName: 'npm install, build and test'
    # Create archive file
    - task: ArchiveFiles@2
      displayName: 'Archive files'
      inputs:
        rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
        includeRootFolder: false
        archiveType: zip
        archiveFile: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
        replaceExistingArchive: true

    - upload: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
      artifact: drop

# Deploy archive file to Azure Web App
- stage: Deploy
  displayName: Deploy stage
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: Deploy
    displayName: Deploy
    environment: $(environmentName)
    pool:
      vmImage: $(vmImageName)
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            displayName: 'Azure Web App Deploy: Azure-MERN-Boilerplate'
            inputs:
              azureSubscription: $(azureSubscription)
              appType: webAppLinux
              appName: $(webAppName)
              runtimeStack: 'NODE|12-lts'
              package: $(Pipeline.Workspace)/drop/$(Build.BuildId).zip
              startUpCommand: 'npm run start'
```

## License

Distributed under the MIT License

## Contact

[Justin Tung on GitHub](https://github.com/justintungonline/)

## Acknowledgements

- [Best README template](https://github.com/othneildrew/Best-README-Template/blob/master/README.md)
