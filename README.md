# Hands on demo: creating your own containerized application

In this demo, we will cover following steps:

  1) Prepare your application for AKS
  2) Create an Azure Container Registry instance
  3) Create an Azure Kubernetes Cluster
  4) Deploy the containerized application

Tip: create an empty nodepad with the following variables

    Resource group name:
    Acr name:
    AKS cluster name:

## Step 1: Prepare your application for AKS

Download Git: https://git-scm.com/download/win

### Get application code

  Before jumping into tutorial, open the Command Prompt and create a new folder using following command:

    mkdir msftemergentdemo

  1) Use git to clone the sample application to your env
     
    git clone https://github.com/Azure-Samples/aks-store-demo.git

  2) Change into the cloned directory
     
    cd aks-store-demo

  3) Review the Docker Compose file (docker-compose-quickstart.yml)

    code .

### Create container images and run application

  1) Create the container image using the docker compose command

    docker compose -f docker-compose-quickstart.yml up -d

  2) View the created images

    docker images

  3) View the running containers

    docker ps

  To check if your application is running properly, navigate to http://localhost:8080 in a local web browser.

  ## Step 2: Create an Azure Container Registry instance

  ### Create an Azure Container Registry

  1) Create a resource group

    az group create --name <insert name> --location westeurope

  2) Create Azure Container Registry instance

    az acr create --resource-group <take the same name as above> --name <insert name> --sku basic

  ### Build and push container images to registry created above

  1) Build and push the images to your ACR

    az acr build -g <resource group name> --registry <acr name chosen above> --image aks-store-demo/product-service:latest ./src/product-service/
  -> 
    
    az acr build -g <resource group name> --registry <acr name chosen above> --image aks-store-demo/order-service:latest ./src/order-service/
    
  ->
  
    az acr build -g <resource group name> --registry <acr name chosen above> --image aks-store-demo/store-front:latest ./src/store-front/

  In case of an error, try the following:
    
    az acr build -g <resource group name> --registry <acr name chosen above> --image product-service:latest ./src/product-service/
    az acr build -g <resource group name> --registry <acr name chosen above> --image order-service:latest ./src/order-service/
    az acr build -g <resource group name> --registry <acr name chosen above> --image store-front:latest ./src/store-front/
    
  2) View and list the image in your ACR

    az acr repository list --name <acr name set in step 2> --output table

  Example output:

    Result
    ----------------
    aks-store-demo/product-service
    aks-store-demo/order-service
    aks-store-demo/store-front

## Step 3: Create an Azure Kubernetes Cluster

### Install the Kubernetes CLI

    az aks install-cli

### Create an AKS cluster

    az aks create --resource-group <resource group name> --name <insert name> --node-count 2 --generate-ssh-keys --attach-acr <ACR name>

### Connect to cluster using kubectl

  1) Configure kubectl to connect to your Kubernetes cluster using the az aks get-credentials command.

    az aks get-credentials --resource-group <Resource group name> --name <AKS cluster name>

  2) Verify connection

    kubectl get nodes

  Example output:

    NAME                                STATUS   ROLES   AGE   VERSION
    aks-nodepool1-19366578-vmss000002   Ready    agent   47h   v1.25.6
    aks-nodepool1-19366578-vmss000003   Ready    agent   47h   v1.25.6

## Step 4: Deploy the containerized application

### Update the manifest file

  1) Get your login server address

    az acr list --resource-group <name of the resource group> --query "[].{acrLoginServer:loginServer}" --output table

  2) Go to the file called: aks-store-quickstart.yaml

    code .

  4) Update the image property for the containers by replacing ghcr.io/azure-samples with your ACR login server name

    Example:

    containers:
    ...
    - name: order-service
      image: <acrName>.azurecr.io/aks-store-demo/order-service:latest
    ...
    - name: product-service
      image: <acrName>.azurecr.io/aks-store-demo/product-service:latest
    ...
    - name: store-front
      image: <acrName>.azurecr.io/aks-store-demo/store-front:latest
    ...

  4) Save *CTRL + S* and close the file using following command

  ### Run the application

  1) Deploy the application

    kubectl apply -f aks-store-quickstart.yaml

  2) Check if deployment is succesfull

    kubectl get pods

### Test the application 

#### Option 1: Command Line
    
  1) Monitor progress

    kubectl get service store-front --watch

  2) When the EXTERNAL-IP address changes from pending to an actual public IP address, use CTRL-C to stop the kubectl watch process.

    Example:

    store-front   LoadBalancer   10.0.34.242   52.179.23.131   80:30676/TCP   67s

  4) View the application in action by opening a web browser to the external IP address of your service.

#### Option 2: Azure Portal

  1) Open your Resource Group on the Azure portal
  2) Navigate to the Kubernetes service for your cluster
  3) Select Services and Ingress under Kubernetes Resources
  4) Copy the External IP shown in the column for store-front
  5) Paste the IP into your browser and visit your store page

### Throubleshooting

"Code is not recognized as an internal or external command":https://stackoverflow.com/questions/46638944/code-is-not-recognized-as-an-internal-or-external-command


    
