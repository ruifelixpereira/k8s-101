#
# K8S Workshop - 201
#

# Set environment variables
export LOCATION?=northeurope
export APP_NAME?=azure-vote
export RESOURCE_GROUP=xxxxx
export CLUSTER_NAME?=xxxxx
export SUBSCRIPTION?=xxxxxxx
export SERVICE_PRINCIPAL_APP_ID?=xxxxxxx
export SERVICE_PRINCIPAL_SECRET?=xxxxxxxx

# Permanent local overrides
-include ./.env

# Set the CLI to use my subscription
default:
	az account set -s $(SUBSCRIPTION)

# dump resource groups
resources:
	az group list --output table

# Dump list of location IDs
locations:
	az account list-locations --output table

#
# Lab 00. Setup
#
# Deploy kubernetes app to the cluster
deploy-app:
	kubectl create -f sample-app/azure-vote.yaml

#
# Lab 01. Volumes Managed
#
# Check storage classes
get-storageclasses:
	kubectl get storageclasses

# Create new storage class
create-storageclass:
	kubectl create -f volumes/sample-storageclass.yaml

# Create new persistent volume
create-pvc:
	kubectl create -f volumes/sample-pvc.yaml

# Get persistent volumes
get-pv:
	kubectl get pv

describe-pv:
	kubectl describe pv

test-pv:
	kubectl apply -f volumes/test-persistent-volumes.yaml

check-pv:
	kubectl get pod task-pv-pod
	kubectl exec -it task-pv-pod -- /bin/bash

#
# Lab 02. Volumes Unmanaged (static)
#
create-namespace:
	kubectl get namespaces
	kubectl create namespace aznamespace

#
# Lab 03. RBAC
#
provision-rbac:
	AKS_ID=$(az aks show --resource-group myResourceGroup --name myAKSCluster --query id -o tsv)
	APPDEV_ID=$(az ad group create --display-name appdev --mail-nickname appdev --query objectId -o tsv)
	az role assignment create \
		--assignee $(APPDEV_ID) \
		--role "Azure Kubernetes Service Cluster User Role" \
  		--scope $(AKS_ID)
	AKSDEV_ID=$(az ad user create --display-name "AKS Dev" --user-principal-name aksdev@contoso.com --password P@ssw0rd1 --query objectId -o tsv)
	az ad group member add --group appdev --member-id $(AKSDEV_ID)

# Colect final output Id and use it to replace groupObjectId in file rolebinding-dev-namespace.yaml
aks-rbac-admin:
	az aks get-credentials --resource-group myResourceGroup --name myAKSCluster –admin
	kubectl create namespace dev
	kubectl apply -f rbac/role-dev-namespace.yaml
	az ad group show --group appdev --query objectId -o tsv

# After running the previous command and customizing file rolebinding-dev-namespace.yaml
aks-rbac-admin-2:
	kubectl apply -f rolebinding-dev-namespace.yaml

test-with-user:
	az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --overwrite-existing

#
# Lab 04. Config Map
#
# Create new config map
create-configmap:
	kubectl create -f configmap/myconfigmap.yaml

# Create Pod using configmap
create-pod-configmap:
	kubectl create -f configmap/pod-using-configmap.yaml

#
# Lab 05. Secrets
#
# Create new secret
create-secret:
	kubectl create -f secret/secret.yaml

# Create Pod using secret:
create-pod-secret:
	kubectl create -f secret/pod-using-secret.yaml


#
# Lab 06. Operators
#
# Deploy sample operator
op-deploy:


#
# Lab 07. Helm
#
# create helm package
helm-create:

# deploy helm package
helm-deploy:


#
# Debug pod for testing and curl internal IPs
#
test-pod:
	kubectl run -i --tty --rm debug --image=radial/busyboxplus --restart=Never -- sh

