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

# used for static persistence volumes lab
export AKS_PERS_STORAGE_ACCOUNT_NAME?=xxxxxxxx
export STORAGE_KEY?=xxxxxxx

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
	kubectl create -f lab00-sample-app/azure-vote-complete.yaml

#
# Lab 01. Volumes Managed
#
# Check storage classes
get-storageclasses:
	kubectl get storageclasses

# Create new storage class
create-storageclass:
	kubectl create -f lab01-volumes-dynamic/sample-storageclass.yaml

# Create new persistent volume
create-pvc:
	kubectl create -f lab01-volumes-dynamic/sample-pvc.yaml

# Get persistent volumes
get-pv:
	kubectl get pv

describe-pv:
	kubectl describe pv

test-pv:
	kubectl apply -f lab01-volumes-dynamic/test-persistent-volumes.yaml

check-pv:
	kubectl get pod task-pv-pod
	kubectl exec -it task-pv-pod -- /bin/bash

#
# Lab 02. Volumes Unmanaged (static)
#

# Ensure that first you create Azure Disk and Azure File Share, 
# collect the URIs and customize the the yamls files:
# - lab02-volumes-static/azure-disk-pod.yaml
# - lab02-volumes-static/azure-files-pod.yaml
# with the proper URIs and share name.

create-static-disk:
	kubectl apply -f lab02-volumes-static/azure-disk-pod.yaml

config-storage-account:
	kubectl create secret generic azure-storage-secret \
		--from-literal=azurestorageaccountname=$(AKS_PERS_STORAGE_ACCOUNT_NAME) \
		--from-literal=azurestorageaccountkey=$(STORAGE_KEY)

create-static-file:
	kubectl apply -f lab02-volumes-static/azure-files-pod.yaml

#
# Lab 03. RBAC
#
provision-rbac:
	AKS_ID=$(az aks show --resource-group $(RESOURCE_GROUP) --name $(CLUSTER_NAME) --query id -o tsv)
	APPDEV_ID=$(az ad group create --display-name appdev --mail-nickname appdev --query objectId -o tsv)
	az role assignment create \
		--assignee $(APPDEV_ID) \
		--role "Azure Kubernetes Service Cluster User Role" \
  		--scope $(AKS_ID)
	AKSDEV_ID=$(az ad user create --display-name "AKS Dev" --user-principal-name aksdev@contoso.com --password P@ssw0rd1 --query objectId -o tsv)
	az ad group member add --group appdev --member-id $(AKSDEV_ID)

# Colect final output Id and use it to replace groupObjectId in file rolebinding-dev-namespace.yaml
aks-rbac-admin:
	az aks get-credentials --resource-group $(RESOURCE_GROUP) --name $(CLUSTER_NAME) –admin
	kubectl create namespace dev
	kubectl apply -f lab03-rbac/role-dev-namespace.yaml
	az ad group show --group appdev --query objectId -o tsv

# After running the previous command and customizing file rolebinding-dev-namespace.yaml
aks-rbac-admin-2:
	kubectl apply -f lab03-rbac/rolebinding-dev-namespace.yaml

test-with-user:
	az aks get-credentials --resource-group $(RESOURCE_GROUP) --name $(CLUSTER_NAME) --overwrite-existing

#
# Lab 04. Config Map
#
# Create new config map
create-configmap:
	kubectl create -f lab04-configmap/myconfigmap.yaml

# Create Pod using configmap
create-pod-configmap:
	kubectl create -f lab04-configmap/pod-using-configmap.yaml

#
# Lab 05. Secrets
#
# Create new secret
create-secret:
	kubectl create -f lab05-secret/secret.yaml

# Create Pod using secret:
create-pod-secret:
	kubectl create -f lab05-secret/pod-using-secret.yaml


#
# Lab 06. Operators
#
# CR and Operator deploy
ctrl-deploy:
	kubectl apply -f lab06-operator/tomcat-crd-definition.yaml
	kubectl apply -f lab06-operator/service_account.yaml
	kubectl apply -f lab06-operator/role.yaml
	kubectl apply -f lab06-operator/role_binding.yaml
	kubectl apply -f lab06-operator/operator.yaml

# Deploy a tomcat cluster
tomcat-deploy:
	kubectl apply -f lab06-operator/tomcat-cr-instance.yaml

# Check what was created
tomcat-validate:
	kubectl get tomcat
	kubectl get pod
	kubectl get svc

# check and open browser with http://<tomcat-service-ip>/sample

#
# Lab 07. Helm
#
# create helm package/chart
helm-create:
	helm create webfrontend

# deploy/install helm package/chart
helm-install:
	helm install webfrontend webfrontend/

# validate installed chart and deployed resources
helm-validate:
	helm list
	kubectl get all

# uninstall helm package/chart
helm-uninstall:
	helm uninstall webfrontend


#
# Debug pod for testing and curl internal IPs
#
test-pod:
	kubectl run -i --tty --rm debug --image=radial/busyboxplus --restart=Never -- sh

