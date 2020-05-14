# Static File Share Volumes in Azure K8S

Reference: https://docs.microsoft.com/en-us/azure/aks/azure-files-volume

## Create a File Share

Use the `create-file-share.sh` script for this.

## Create a K8S secret

This secret is to store the storage account credentials.

```shell
kubectl create secret generic azure-secret --from-literal=azurestorageaccountname=$AKS_PERS_STORAGE_ACCOUNT_NAME --from-literal=azurestorageaccountkey=$STORAGE_KEY
```

## Update pod yaml

Just ensure if the `shareName` property in the `azure-files-pod.yaml` matches the share name created on Azure Files.

## Deploy the new pod

```shell
kubectl apply -f azure-files-pod.yaml
```

## Test the new pod

```shell
# access persistent volume (write something, destroy the pod and repeat the test again)
kubectl exec -it mypod-files-tst -- /bin/bash
```
