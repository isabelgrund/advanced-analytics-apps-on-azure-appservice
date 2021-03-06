# advanced-analytics-apps-on-azure-appservice
Instructions on how to deploy containerized Data Analytics on Azure App Service.

# Overview
![analytics-docker-appservice-overview](/images/overview.png)

# Required Tools
Please make sure you have the following tools installed on your machine.  
* Code Editor: Visual Studio Code
https://code.visualstudio.com/ 
* git source-code management
https://git-scm.com/
* azure command line interface 'az'
https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest (Azure CLI) 
* Docker for Windows
https://docs.docker.com/docker-for-windows/install/

# Create an Azure DevOps project
We will use Azure DevOps to host the source-code and secondly utilize Azure Pipelines to automate build and deployment tasks.

* login at https://dev.azure.com using your corporate credentials.
* Pleasse refere to https://docs.microsoft.com/en-us/azure/devops/organizations/projects/create-project?view=azure-devops for further guidance.

# Clone the repository
In the project please proceed to "Repos".  
Follow the instructions to clone the repository to your local computer.

# Download the example project for Bokeh
Please proceed to the following URL.    
**Please just perform the ZIP download and extract the content in the previously cloned git repository**
https://github.com/dariustehrani/bokeh-on-docker

# Test the Dockerfile
* Open Powershell (in Visual Studio Code use STRG+ö on a german keyboard)
* CD into the project folder
* run ````docker build -t bokeh-on-docker:latest .````
* test the image by running ````docker run -p 8080:8080 -it bokeh-on-docker:latest````
* Open a browser and point it to http://localhost:8080
You should see a simple bokeh demo app.

**Did this work for you?**  
Excellent. Within your project directory perform:  
````git add . ````  
````git commit . -m "your commit message"````  
````git push````  
Visit the repository view in Azure DevOps to check if your code has arrived there.

# Azure Container Registry (azurecr.io)
Typically we would not want to build and persist Docker images locally but have Azure DevOps manage this for us. Let's create a Container Registry. You can do this in the portal or using the following Azure CLI commands:
````az account show````
If you are not logged in, run:  
````az login````

Create a resource group:
````az group create -l westeurope -n bokehondockerYOURNAME```` 

Create your azure container registry
````az acr create -n bokehYOURNAME -g bokehondockerYOURNAME --sku Standard --admin-enabled````

azure container registry docker login
````az acr login -name bokehYOURNAME````  

(optional) show the current username and password
````az acr credential show -n bokehyourname````

# Azure Pipeline Setup

### Create a new service principal (SP)
The SP will be used by Azure DevOps to connect to your Azure subscription and manage resources on your behalf.
You need to replace the values with your subscription ID and Resource Group Name.  
````az account show````  
````az ad sp create-for-rbac -n "bokehondocker" --role contributor --scopes /subscriptions/{SubID}/resourceGroups/{ResourceGroup1}````

# Setup Azure DevOps Service Connections

## AzureRM Service connection
Insert the credentials you receive here:
* Proceed to your Azure DevOps project.
* Click "Project settings" -> "Service connections" -> "New service connection" -> Select "Azure Resource Manager".
* Define a meaningful name. Select  
"Scope:Subscription"  
"Subscription: YOURSUBSCRIPTION"  
"Resource Group: NAMEOFRGYOUCREATED"  
* Click "use the automated version of the service connection dialog."
* Insert the AppID from the shell output into "Service principal client id" . 
* Insert the key into "Service principal client ID".
* Click "Verify connection".  
* Make sure you tick the "Allow all pipeline to use this connection" box.

## Azure Container Registry Service connection:
* Click "New servic connection" and select "Docker Registry".
* Select "Azure Container Registry" and fill out the remaining information.
* Take note of your service connection names.

# Bring up the Azure DevOps Pipeline
* Click on "Pipelines" -> "Builds"
* Click "+ New"
* Select "Azure Repos Git"
* Azure DevOps shall detect an existing azure-pipelines.yml in the repository.
* You will need to customize the service connections accordingly. Leave the rest as is.
* Click "Run"
* Proceed to the "Pipelines" overview and click on the pipeline you created.

# Shiny R on Microft Open R (ADVANCED USERS ONLY)
As building the R Packages take a lot of time, the following example separates the creation of the runtime container from the actual Shiny app container. This will save 18 Minutes on average per deployment. Secondly you might want to set up a scheduled trigger, e.g. on a weekly basis so that your base images always includes latest updates and patches.

### Microsoft R Open Shiny BASE Image
Build a CI/CD Pipeline for the base Image. 
https://github.com/dariustehrani/mro-shiny-base

### Microsoft R Open Shiny APP Image
The following example will build shiny app container. You will need to modify the FROM entry to reference your azure container registry.
````FROM YOUR.azurecr.io/mro-shiny-base:latest````

https://github.com/dariustehrani/mro-on-docker
