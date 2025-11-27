# Challenge 03 - Introduction To Kubernetes - Coach's Guide 

[< Previous Solution](./Solution-01.md) - **[Home](./README.md)** - [Next Solution >](./Solution-04.md)

## Notes & Guidance

- Remind teams that kubectl can be installed through the CLI, but don’t give away the answer:

## Solution

### Step 1: Install kubectl (if not already installed)

```bash
az aks install-cli
```

### Step 2: Set up variables

```bash
RESOURCE_GROUP="rg-westeurope-akstraining-01"
CLUSTER_NAME="akstraining01"
ACR_NAME="acrakstraining01.azurecr.io"
LOCATION="westeurope"
```

### Step 3: Create the AKS cluster

```bash
az aks create \
  --resource-group $RESOURCE_GROUP \
  --name $CLUSTER_NAME \
  --node-count 3 \
  --network-plugin azure \
  --network-plugin-mode overlay \
  --enable-managed-identity \
  --node-vm-size Standard_D2ls_v5 \
  --zones 1 2 3 \
  --location $LOCATION \
  --attach-acr $ACR_NAME \
  --generate-ssh-keys
```

**Note:** The `--attach-acr` option requires Owner or Azure account administrator role. If you don't have sufficient permissions, the ACR can be attached after cluster creation using `az aks update`.

### Step 4: Get credentials to connect kubectl to the cluster

```bash
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```

### Step 5: Verify cluster status and configuration

```bash
# Show cluster details
az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --output table

# List the cluster with key information
az aks list --resource-group $RESOURCE_GROUP --output table

# Get the Kubernetes version
az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query "kubernetesVersion" --output tsv

# Check node count and VM size
az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query "agentPoolProfiles[0].[count,vmSize]" --output tsv

# Verify nodes are running
kubectl get nodes

# Show nodes with availability zones
kubectl get nodes -o custom-columns=NAME:'{.metadata.name}',REGION:'{.metadata.labels.topology\.kubernetes\.io/region}',ZONE:'{metadata.labels.topology\.kubernetes\.io/zone}'

# Get cluster status and health
kubectl get nodes -o wide

# Check system pods are running
kubectl get pods --all-namespaces
```

### Optional: Attach ACR after cluster creation

If you couldn't attach the ACR during cluster creation:

```bash
az aks update --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --attach-acr $ACR_NAME
```

### Optional: Open AKS workloads in Azure Portal

```bash
az aks browse --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```
