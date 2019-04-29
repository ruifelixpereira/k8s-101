#
# K8S Workshop
#

# Set environment variables
export LOCATION?=northeurope
export APP_NAME?=azure-vote
export RESOURCE_GROUP=MyK8s
export CLUSTER_NAME?=rfpk8s
export SUBSCRIPTION?=5dd549af-d55f-4c2c-ba14-8bd7699a59b3
export ACR_NAME?=rfp01
export ACR_RESOURCEGROUP?=containerreg

# Permanent local overrides
-include ../.env

# Get Service Principal for Kubernetes cluster
SERVICE_PRINCIPAL_ID=$(shell az aks show -g $(RESOURCE_GROUP) --name $(CLUSTER_NAME) -o json | jq -r ".servicePrincipalProfile.clientId")

# Set the CLI to use my subscription
default:
	az account set -s $(SUBSCRIPTION)
	az acr login --name $(ACR_NAME) --resource-group $(ACR_RESOURCEGROUP)

# dump resource groups
resources:
	az group list --output table

# Dump list of location IDs
locations:
	az account list-locations --output table

#
# Lab 01.
#
# Create a resource group and create the kubernetes cluster resources inside it
create:
	-az group create --name $(RESOURCE_GROUP) --location $(LOCATION) --output table
	az aks create \
		--resource-group $(RESOURCE_GROUP) \
		--name $(CLUSTER_NAME) \
		--node-count 2 \
		--node-vm-size Standard_A2_v2 \
		--generate-ssh-keys
#		--no-wait

install-cli:
	sudo az aks install-cli
	
validate:
	az aks show \
		--resource-group $(RESOURCE_GROUP) \
		--name $(CLUSTER_NAME)
	az aks get-credentials \
		--resource-group $(RESOURCE_GROUP) \
		--name $(CLUSTER_NAME)
	kubectl get nodes
	kubectl cluster-info		

# Destroy the entire resource group
destroy:
	az group delete \
		--name $(RESOURCE_GROUP) \
		--no-wait


# Add kubernetes service principal permissions to access ACR with Reader role
allow-access-to-acr:
	az role assignment create \
		--role=Reader \
		--scope=/subscriptions/$(SUBSCRIPTION)/resourceGroups/$(ACR_RESOURCEGROUP)/providers/Microsoft.ContainerRegistry/registries/$(ACR_NAME) \
		--assignee $(SERVICE_PRINCIPAL_ID)

check-access-to-acr:
	az role assignment list \
		--assignee $(SERVICE_PRINCIPAL_ID) --all  

#
# Lab 02.
#
# Deploy kubernetes app to the cluster
deploy-app:
	kubectl create -f azure-vote.yaml

# Explore the created app artifacts
get-deployments:
	kubectl get deployments

describe-deployments:
	kubectl describe deployments

get-replicasets:
	kubectl get rs

get-pods:
	kubectl get pods --show-labels

#
# Lab 03.
#
# Deploy kubernetes services to the cluster

create-public-service:
	kubectl create -f azure-vote-service-public.yaml

# Get demo app dashboard external IP address
app-public-ip:
	kubectl get service azure-vote-front --watch

create-private-service:
	kubectl create -f azure-vote-service-private.yaml

create-nodeport-service:
	kubectl create -f nodeport-service.yaml
	kubectl describe service azure-vote-front-node

new-cleanup:
#	kubectl delete service azure-vote-front-node
#	kubectl delete service azure-vote-front-internal
	kubectl delete deployment azure-vote-front
	kubectl delete deployment azure-vote-back
	kubectl delete service azure-vote-front
	kubectl delete service azure-vote-back

#
# Lab 01-01.
#
# Get versions
get-cluster-versions:
	az aks get-versions --resource-group $(RESOURCE_GROUP) --name $(CLUSTER_NAME) --location $(LOCATION) --output table

get-instrumentation-key:
	az resource show --resource-group $(NAMESPACE_PREFIX)-monitoring --name $(NAMESPACE_PREFIX) --resource-type "Microsoft.Insights/components" --query 'properties.InstrumentationKey' --output tsv

# Upgrade cluster (takes time, avoid it in demo)
upgrade-cluster-to-%:
	az aks upgrade --resource-group $(RESOURCE_GROUP) --name $(CLUSTER_NAME) --kubernetes-version $*

# Open the management console
browse-console:
	az aks browse --resource-group $(RESOURCE_GROUP) --name $(CLUSTER_NAME)

# Proxy to the management console
# Browse this URL: http://localhost:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/#!/overview?namespace=default
proxy:
	kubectl proxy --port 8001


# Scale number of frontend pods
scale-frontend:
	kubectl scale deployment azure-vote-front \
		--replicas=2

# Configure auto-scale
auto-scale:
	kubectl autoscale deployment azure-vote-front \
		--min=2 \
		--max=15 \
		--cpu-percent=80

# Scale the cluster (2 min - avoid it in the demo - it doesn't appear imediately in the Kubernetes admin dashboard)
scale-cluster:
	az aks scale \
		--resource-group=$(RESOURCE_GROUP) \
		--name=$(CLUSTER_NAME) \
		--node-count 3

#
# Lab 04.
#
create-namespace:
	kubectl get namespaces
	kubectl create namespace aznamespace

#
# Lab 05.
#
# Check storage classes
get-storageclasses:
	kubectl get storageclasses

# Create new storage class
create-storageclass:
	kubectl create -f sample-storageclass.yaml

# Create new persistent volume
create-pvc:
	kubectl create -f sample-pvc.yaml

# Get persistent volumes
get-pv:
	kubectl get pv

describe-pv:
	kubectl describe pv

test-pv:
	kubectl apply -f test-persistent-volumes.yaml

check-pv:
	kubectl get pod task-pv-pod
	kubectl exec -it task-pv-pod -- /bin/bash
