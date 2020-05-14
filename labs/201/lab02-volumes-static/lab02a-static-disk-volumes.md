# Static Disk Volumes in Azure K8S

Reference: https://docs.microsoft.com/en-us/azure/aks/azure-disk-volume

## Create a Managed disk

```shell
az disk create \
  --resource-group rfpk8s-volumes \
  --name myAKSDisk \
  --size-gb 20 \
  --query id --output tsv
```

## Check proper access

The Service Principal used by the AKS cluster must have `Contributor` access to this resource or to the resource group where it resides. If not, just assign it before proceeding to the next steps.

## Update pod yaml

Collect the the previous output resource id (e.g., like this `/subscriptions/7dd549af-d55f-4c2c-ba14-8bd788a59b3/resourceGroups/rfpk8s-volumes/providers/Microsoft.Compute/disks/myAKSDisk`) and update the `diskURI` property in the `azure-disk-pod.yaml` file.

## Deploy the new pod

```shell
kubectl apply -f azure-disk-pod.yaml
```

## Test the new pod

```shell
# access persistent volume (write something, destroy the pod and repeat the test again)
kubectl exec -it mypod-disk-tst -- /bin/bash
```
