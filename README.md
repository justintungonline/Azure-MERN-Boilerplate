# Azure-MERN-Boilerplate

A very basic boilerplate for an Azure ready MERN app

The tutorial for deploying this boilerplate can be found here:
<https://medium.com/@tuna.sogut/how-to-deploy-a-mern-stack-app-to-azure-via-continuous-integration-a3a551526e26?sk=0fc4fa9d7c7072ad7e95b94d7e5733e4>

Forked from: <https://github.com/Sogutt/Azure-MERN-Boilerplate>

## About This Repository

It builds on the tutorial by Tuna Sogut by:

- Describing two other options to deploy the MERN boilerplate application to an Azure Web App using:
  - [Local git push to remote](https://docs.microsoft.com/en-us/azure/app-service/deploy-local-git?tabs=cli), or
  - [Azure DevOps pipelines](https://azure.microsoft.com/en-ca/services/devops/pipelines/)
  - [Deploy and Run directly from a ZIP package](https://docs.microsoft.com/en-us/azure/app-service/deploy-run-package)
- Add Azure CLI method of creating a web app   

### Built With

- [MERN](https://www.mongodb.com/mern-stack)
- [Node JS](https://nodejs.org/en/)
- [Microsoft Azure](https://azure.microsoft.com/)

[![Gitpod ready-to-code](https://img.shields.io/badge/Gitpod-ready--to--code-blue?logo=gitpod)](https://gitpod.io/#https://github.com/justunsix/)

## Getting Started

### Prerequisites

- [Yarn](https://classic.yarnpkg.com/en/docs/install#windows-stable)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/) (optional if you want to create the Azure App Services using command line)

### Installation

This section is a summary of key points in the tutorial.

In the file `routes/new-index.js` change:

```js

// The connection string the database to use your connection. 
// You can create free one from MongoDB Atlas (https://www.mongodb.com/cloud/atlas) 
// then create a database and initial collection and add a document which the application retrieves.
var url = "mongodb+srv://<username>:<password>@<cluster>-vgz77.azure.mongodb.net/test?retryWrites=true&w=majority";
...
// Update the database name to name of your database.
  var dbo = db.db("<database>");
```

#### Build and Run Application

```sh
cd <repository directory>
npm install
# Optionally fix vulnerabilities with npm audit fix

cd client
yarn install
npm run build

# Run the application to test it locally
cd ..
npm run start
```

In the `client` directory, run `npm run build` to build whenever you have made changes to the React app.

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

Alternatively, follow instructions on how to us the GitHub [webapps-deploy](https://github.com/Azure/webapps-deploy) node.js action which will use a workflow `yaml` like the following. 
- In this repository is a sample [`.github/workflows/node.js-webapp-on-azure.yml`](https://github.com/justunsix/Azure-MERN-Boilerplate/blob/master/.github/workflows/node.js-webapp-on-azure.yml)

```yaml
name: Deploy Node.js to Azure Web App

on:
  [push]

# CONFIGURATION
# For help, go to https://github.com/Azure/Actions
#
# 1. Set up the following secrets in your repository:
#   AZURE_WEBAPP_PUBLISH_PROFILE
#
# 2. Change these variables for your configuration:
env:
  AZURE_WEBAPP_NAME: Azure-MERN-Boilerplate-github    # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '12.x'                # set this to the node version to use

jobs:
  build-and-deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest
    environment: dev
    steps:
    - uses: actions/checkout@master
    - name: Use Node.js ${{ env.NODE_VERSION }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ env.NODE_VERSION }}
    - name: npm install, build, and test
      run: |
        # Build and test the project, then
        # deploy to Azure Web App.
        npm install
        cd client
        yarn install
        npm run build        
    - name: 'Deploy to Azure WebApp'
      uses: azure/webapps-deploy@v2
      with: 
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
        
  # For more information on GitHub Actions for Azure, refer to https://github.com/Azure/Actions
  # For more samples to get started with GitHub Action workflows to deploy to Azure, refer to https://github.com/Azure/actions-workflow-samples
```

### Using local git to push to Azure remote git which deploys the application

These steps can be done manually or automated.

```sh
# In a separate terminal, set up user-level deployment credentials with Azure CLI
az webapp deployment user set --user-name <username> --password <password>
# If an error like: Operation returned an invalid status 'Conflict'
# occurs, it may mean the username is taken or password is not complex enough.
# Choose a different username and more complex password

# Get new remote git URL from the Azure Web App deployment setting or with:
az webapp deployment source config-local-git --name Azure-MERN-Boilerplate --resource-group myResourceGroup --query url --output tsv

# Add remote URL to git repo
git remote add azure <insert URL you got from the previous command>

# Push you code to Azure and enter your password from the deployment user set command previously when asked. 
# Instead of 'main', you may want to choose your branch to push to Azure. 
# master is required after your local branch to ensure it pushes to the 'master' branch read for Azure deployment
git push azure main:master
```

### Using Azure DevOps pipelines to deploy to Azure

1. Create a [new Azure DevOps project](https://dev.azure.com).
2. Go to the Repos menu
3. Generate the git credentials and save your username and password
4. Use the push to this repository option and push from your local git repository to Azure DevOps repo
5. Enter your password from the credentials when asked
6. Go to Pipelines and create a new one using Node JS, Express template
7. When asked, login to Azure and select your Azure Web App with the pipeline. Behind the scenes, this step https://docs.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure?view=azure-devops to your Azure subscription which will allow the pipeline to interact with the Azure resources in the subscription. For production pipelines, it is recommendated to use a service principal (a managed identity) instead of a personal identity.
8. Azure DevOps will generate a `pipelines.yaml` like the one below.
9. Verify or Update the node version and build commands to reflect the application. These lines were changed from the auto-generated `yaml`: 
  1. `versionSpec: '12.x'` 
  2. `runtimeStack: 'NODE|12-lts'` 
  3. `script: |` section with the application specific build.

In this repository is a sample [`azure-pipelines.yml`](https://github.com/justunsix/Azure-MERN-Boilerplate/blob/master/azure-pipelines.yml).

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

### Run from Zip

Follow [Run your app in Azure App Service directly from a ZIP package](https://docs.microsoft.com/en-us/azure/app-service/deploy-run-package) which mounts the zip file directly to the *wwwroot* directory. Here are [different options to deploy code from a ZIP or WAR file](https://docs.microsoft.com/en-us/azure/app-service/deploy-zip).

There are several benefits to running directly from a package:

- Eliminates file lock conflicts between deployment and runtime.
- Ensures only full-deployed apps are running at any time.
- Can be deployed to a production app (with restart).
- Improves the performance of Azure Resource Manager deployments.
- May reduce cold-start times, particularly for JavaScript functions with large npm package trees.
- By default, the deployment engine assumes that a ZIP file is ready to run as-is and doesn't run any build automation which reduces automation time if work is already done prior to ZIP creation. To enable the same build automation as in a Git deployment, set the SCM_DO_BUILD_DURING_DEPLOYMENT app setting to true.

## Usage - Running Locally

Use `npm run start` in the root of the repository.

## License

Distributed under the MIT License

## Contact

[Justin Tung on GitHub](https://github.com/justunsix/)

## Acknowledgements

- [Best README template](https://github.com/othneildrew/Best-README-Template/blob/master/README.md)
