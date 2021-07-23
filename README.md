# Azure-MERN-Boilerplate

A very basic boilerplate for an Azure ready MERN app

The tutorial for deploying this boilerplate can be found here:
<https://medium.com/@tuna.sogut/how-to-deploy-a-mern-stack-app-to-azure-via-continuous-integration-a3a551526e26?sk=0fc4fa9d7c7072ad7e95b94d7e5733e4>

## About The Project

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

In the file `routes/new-index.js` change:

```js

// The connection string the database to use your connection. You can create free one from MongoDB Atlas (https://www.mongodb.com/cloud/atlas) and create a database and initial collection.
var url = "mongodb+srv://<username>:<password>@<cluster>-vgz77.azure.mongodb.net/test?retryWrites=true&w=majority";
...
// Update the database name to name of your database.
  var dbo = db.db("<database>");
```

```sh
cd <repository directory>
npm install
# Optionally fix vulnerabilities with npm audit fix

cd client
yarn install
npm run build
```

In the `client` directory, run `npm run build` to build whenever you have made changes.

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

Follow the rest of [the tutorial regarding deployment to Azure](https://medium.com/@tuna.sogut/how-to-deploy-a-mern-stack-app-to-azure-via-continuous-integration-a3a551526e26?sk=0fc4fa9d7c7072ad7e95b94d7e5733e4).

## Usage

## Roadmap

- Azure App Service deployment instructions

## License

Distributed under the MIT License

## Contact

[Justin Tung on GitHub](https://github.com/justintungonline/)

## Acknowledgements

- [Best README template](https://github.com/othneildrew/Best-README-Template/blob/master/README.md)
