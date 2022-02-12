# deploy-container-app-to-aks
Deployin a multi-container application to Azure Kubernetes Services


![image](https://user-images.githubusercontent.com/44494776/153155303-9392fb1c-dd9d-40a7-8a90-330208093dab.png)


STEPS:

1. Create an Azure Container Registry (ACR), AKS and Azure Sql Server
    - ACR -> used to store the docker images privately
    - AKS -> docker images are deployed to pods running inside aks
    - SQL Server -> host database
3. Provision the Azure DevOps Team Project with a .NET Core application using the Azure Devops Demo Generator tool
4. Configure application and database deployment, using Continuous Deployment (CD) in the Azure DevOPS
    - build pipelines to get image, build image and push the imame
    - release pipelines to deploy db and docker image to AKS
5. Initiate the build to automatically deploy the application


SETTING UP ENVIRONMENT

1. Create Resource group: 
az group create --name aks-container-application --location westeurope
2. Create AKS:
az aks create --resource-group aks-container-application --name mhc-webapp --enable-addons monitoring --kubernetes-version 1.22.3 --generate-ssh-keys --location westeurope
3. Create ACR
az acr create --resource-group aks-container-application --name cgwebapp --sku Standard --location westeurope
4. Authenticate with ACR from AKS: 
az acr create --resource-group aks-container-application --name cgwebapp --sku Standard --location westeurope
5. Create Azure Sql Server and Database
az sql server create -l westeurope -g aks-container-application -n akssqlserver2604 -u sqladmin -p <password>
    ![image](https://user-images.githubusercontent.com/44494776/153701429-cb57c78b-47f3-4f17-ad64-d84b10b6a0ed.png)
    
6. Click on set server firewall on sql server and enable "Allow Azure services" option
    ![image](https://user-images.githubusercontent.com/44494776/153701481-c67ab264-f254-4f30-bd23-50373f63ca83.png)

    
    
 
    
 CONFIGURE BUILD PIPELINE
    
 1. Navigate to Azure DevOps Lab -> Pipelines and Create new pipeline with below tasks:
 2. Replace tokens in appsettings.json with sql credentials. All variables will be set in variables
 "ConnectionStrings": {
    "DefaultConnection": "Server=tcp:__SQLserver__,1433;Initial Catalog=mhcdb;Persist Security Info=False;User ID=__SQLuser__;Password=__SQLpassword__;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;".
![image](https://user-images.githubusercontent.com/44494776/153701870-861ec37a-2824-4629-a0fc-c9be0b6c8fd1.png)

    
 3. Replace tokens in yaml file -> In variables I set the acr name to pull image from container registry
 4.  Tasks for run, build and push docker image to container image
 5. Publish all files to Artifact
    
    
![image](https://user-images.githubusercontent.com/44494776/153701882-e01945f6-e167-462a-845f-88894ef94e35.png)

    

    
 
CONFIGURE RELEASE PIPELINE
    
1. Go to pipelines -> releases. Seletct Dev Stage and click edit to add tasks:
    ![image](https://user-images.githubusercontent.com/44494776/153704258-e2bc8212-4003-4341-b4a7-33121eac945a.png)

2. In the Dev environment, under the DB deployment phase, select Azure Resource Manager from the drop down for Azure Service Connection Type, update the Azure Subscription value from the dropdown for Execute Azure SQL: DacpacTask task.
    ![image](https://user-images.githubusercontent.com/44494776/153704289-d4c77eec-7e22-4231-ac0d-f83911603e5a.png)

 3. In the AKS deployment phase, select Create Deployments & Services in AKS task.
    Update the Azure Subscription, Resource Group and Kubernetes cluster from the dropdown. Expand the Secrets section and update the parameters for Azure subscription and Azure container registry from the dropdown.
Repeat similar steps for Update image in AKS task.
    ![image](https://user-images.githubusercontent.com/44494776/153704339-569f3747-717a-4fd6-b9d8-117a2c5df2a8.png)
    

    
DEPLOY APPLICATION TO AKS
    
1. The build will generate and push the docker image to ACR. After the build is completed, you will see the build summary. To view the generated images navigate to the Azure Portal, select the Azure Container Registry and navigate to the Repositories.
    
![image](https://user-images.githubusercontent.com/44494776/153704608-66bddf89-4573-4b24-8025-4a3e987a935f.png)

2.  Select the Releases tab in the Pipelines section and double-click on the latest release. Select In progress link to see the live logs and release summary.
    ![image](https://user-images.githubusercontent.com/44494776/153704631-c5f192c1-a1fe-4a77-a0f3-5521e2bc6ede.png)
    
    
CHECK THE RUNNING APP
 1. Launch the Azure Cloud Shell and run the below commands to see the pods running in AKS:
    
    ![image](https://user-images.githubusercontent.com/44494776/153704686-5d4aeaef-e702-46fa-b28f-6ec13ce9c314.png)

 
2. To access the application, run the below command. If you see that External-IP is pending, wait for sometime until an IP is assigned.
![image](https://user-images.githubusercontent.com/44494776/153704699-45ac7bcc-db7b-4262-b925-19a3d0ccf496.png)
    
3. Copy the External-IP and paste it in the browser and press the Enter button to launch the application.
![image](https://user-images.githubusercontent.com/44494776/153704711-bef64e6d-0340-401a-ac7f-49e9088c2133.png)
