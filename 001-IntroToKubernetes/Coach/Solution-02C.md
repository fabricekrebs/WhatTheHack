# Challenge 02 - Path C: The Azure Container Registry - Coach's Guide 

[< Previous Solution](./Solution-01.md) - **[Home](./README.md)** - [Next Solution >](./Solution-03.md)

## Challenge 1 & 2 Path C

This is **PATH C**: Use this path if your students understand docker, don't care about building images , and/or have environments issues that would prevent them from building containers locally. In this path, your students will will create an Azure Container Registry, and then will import the images to their ACR.

## Notes & Guidance

#### Importing the Application
If the students get stuck, point them to the ACR documentation:

- https://docs.microsoft.com/en-us/azure/container-registry/container-registry-import-images

### Step 0: Set up variables
```bash
RESOURCE_GROUP="rg-westeurope-akstraining-01"
ACR_NAME="acrakstraining01.azurecr.io"
LOCATION="westeurope"
```

### Step 1: Create the Azure Container Registry

First, create a resource group (if not already created) and the ACR:

```bash
# Create ACR with Standard SKU
az acr create --resource-group $RESOURCE_GROUP --name $ACR_NAME --sku Standard

# Enable admin user (optional, for basic authentication)
az acr update --name $ACR_NAME --admin-enabled true

# Get admin credentials (if admin user is enabled)
az acr credential show --name $ACR_NAME
```

**Alternative: Create a token-based user for more granular access control:**

```bash
# Create a token for user 'myuser' with content read/write permissions
az acr token create --name myuser --registry $ACR_NAME --scope-map _repositories_admin
```

### Step 2: Pull images from Docker Hub
```bash
sudo docker pull whatthehackmsft/content-api:latest
sudo docker pull whatthehackmsft/content-web:latest
```

### Step 3: Tag images for ACR
```bash
sudo docker image tag whatthehackmsft/content-api:latest $ACR_NAME/whatthehackmsft/content-api:latest
sudo docker image tag whatthehackmsft/content-web:latest $ACR_NAME/whatthehackmsft/content-web:latest
```

### Step 4: Login to ACR
```bash
sudo az acr login -n $ACR_NAME
```

### Step 5: Push images to ACR
```bash
ACR_NAME="acrakstraining01.azurecr.io"
docker image push $ACR_NAME/whatthehackmsft/content-api:latest
docker image push $ACR_NAME/whatthehackmsft/content-web:latest
```

### Verify images
```bash
az acr repository list --name $ACR_NAME
docker images
```

**Note:** If running commands requiring Docker daemon access, you may need to use `sudo` or add your user to the docker group.


