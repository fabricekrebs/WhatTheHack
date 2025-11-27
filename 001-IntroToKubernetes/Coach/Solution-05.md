# Challenge 05 - Scaling and High Availability - Coach's Guide 

[< Previous Solution](./Solution-04.md) - **[Home](./README.md)** - [Next Solution >](./Solution-06.md)

## Notes & Guidance

- In the YAML file, they will have to update the **spec.replicas** value. They can use this command to edit the deployment resource:
	- `kubectl edit deployment content-web`
	- Alternatively (and perhaps preferably!), they can edit the yaml directly in the Azure portal.  
- They can watch cluster events by using the following command:
	- `kubectl get events --sort-by='{.lastTimestamp}' --watch`
- The error they will encounter is that there aren’t enough CPUs in the cluster to support the number of replicas they want to scale to.
- The three fixes to address resource constraints are:
	- Use the Azure portal or CLI to add more nodes to the node pool of the AKS cluster.
	- Use the cluster autoscaler to automatically add more nodes to the node pool of the cluster as resources are needed.
    	- Once the challenge is complete, show the team how easy it is to enable the cluster autoscaler  (Portal -> Cluster -> Node Pools -> Scale, then select 'Autoscale')
	- Change the deployment and reduce the needed CPU number from “0.5” to “0.125” (500m to 125m).
		- In production environment, consider a discussion with the application owner/architect before reducing any resources.
		- **NOTE** In this Challenge, if the last option doesn't work, delete the old deployment and reapply it. Kubernetes deploys new pods before tearing down old ones and if we are out of resources, no new pods will be deployed.
- **NOTE:** In case they do **NOT** get an error and are able to scale up, check how many nodes they have in their cluster and the size of the node VMs. Over provisioned clusters will not fail.
	- If a team doesn’t get a failure, just have them double the number of Web and API app instances.  

## Videos

### Challenge 5 Solution

[![Challenge 5 solution](../Images/WthVideoCover.jpg)](https://youtu.be/97P6pT4Dlk4 "Challenge 5 Solution")




## Solution

### Step 1: Set up variables

```bash
RESOURCE_GROUP="rg-aks-training"
CLUSTER_NAME="akstraining01"
```

### Step 2: Scale the web application by editing the deployment

**Option A: Edit the deployment file and reapply**

Edit your `content-web.yml` file and change the `replicas` value:

```yaml
spec:
  replicas: 4  # Change this value to scale up
```

Then apply the changes:

```bash
kubectl apply -f content-web.yml
```

**Option B: Use kubectl scale command**

```bash
kubectl scale deployment content-web --replicas=4
```

**Option C: Edit the deployment directly**

```bash
kubectl edit deployment content-web
```

### Step 3: Scale the API application

```bash
kubectl scale deployment content-api --replicas=4
```

### Step 4: Watch the scaling events

```bash
# Watch pod status in real-time
kubectl get pods --watch

# Watch cluster events
kubectl get events --sort-by='{.lastTimestamp}' --watch

# Check current replica status
kubectl get deployments
```

### Step 5: Troubleshooting - If pods fail to schedule due to insufficient resources

If you encounter errors about insufficient CPU or memory, you have three options:

**Option A: Manually scale the node pool**

```bash
# Scale up the node pool (e.g., from 3 to 5 nodes)
az aks nodepool scale --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME --name nodepool1 --node-count 5

# Scale down the node pool (for testing resource constraints)
az aks nodepool scale --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME --name nodepool1 --node-count 1
```

**Option B: Reduce CPU/Memory requests in the deployment**

Edit your deployment YAML and reduce resource requests (e.g., from 500m to 125m):

```yaml
spec:
  containers:
  - name: content-api
    resources:
      requests:
        cpu: "125m"  # Reduced from 500m
        memory: "128Mi"
```

Then apply the changes:

```bash
kubectl apply -f content-api.yml
```

**Note:** If reducing resources doesn't work, delete the old deployment first:

```bash
kubectl delete deployment content-api
kubectl apply -f content-api.yml
```

### Step 6: Verify scaling

```bash
# Check deployment status
kubectl get deployments

# Check pod distribution across nodes
kubectl get pods -o wide

# Check node resource usage
kubectl top nodes

# Check pod resource usage
kubectl top pods
```
