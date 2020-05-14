#
# K8S Workshop - 101
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
# Lab 01.
#
# Create a resource group and create the kubernetes cluster resources inside it
create:
	-az group create --name $(RESOURCE_GROUP) --location $(LOCATION) --output table
	az aks create \
		--resource-group $(RESOURCE_GROUP) \
		--name $(CLUSTER_NAME) \
		--node-count 2 \
		--node-vm-size Standard_D4s_v3 \
		--service-principal $(SERVICE_PRINCIPAL_APP_ID) \
		--client-secret $(SERVICE_PRINCIPAL_SECRET) \
		--generate-ssh-keys
#		--no-wait

# Only run this if not using Azure cloud Shell
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

#
# Lab 01 (optional).
#
# Get available ersions
get-available-versions:
	az aks get-versions --location $(LOCATION) --output table

# Upgrade cluster (takes time, avoid it in demo)
upgrade-cluster-to-%:
	az aks upgrade --resource-group $(RESOURCE_GROUP) --name $(CLUSTER_NAME) --kubernetes-version $*

# Setup k8s console permission with rbac. You only need to run this once.
prep-console-perms:
	kubectl create clusterrolebinding kubernetes-dashboard -n kube-system \
		--clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard

# Open the management console
browse-console:
	az aks browse --resource-group $(RESOURCE_GROUP) --name $(CLUSTER_NAME)

# Scale the cluster (it might take a couple of minutes)
scale-cluster:
	az aks scale \
		--resource-group=$(RESOURCE_GROUP) \
		--name=$(CLUSTER_NAME) \
		--node-count 3

#
# Lab 02.
#
# Deploy kubernetes app to the cluster
deploy-app:
	kubectl create -f lab02-04-sample-app/azure-vote.yaml

# Explore the created app artifacts
get-deployments:
	kubectl get deployments

describe-deployments:
	kubectl describe deployments

get-replicasets:
	kubectl get rs

get-pods:
	kubectl get pods -o wide

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

#
# Lab 03. Namespaces
#
create-namespace:
	kubectl get namespaces
	kubectl create namespace hellons


#
# Lab 04.
#
# Deploy kubernetes services to the cluster

create-public-service:
	kubectl create -f lab02-04-sample-app/azure-vote-service-public.yaml

# Get demo app dashboard external IP address
app-public-ip:
	kubectl get service azure-vote-front --watch

create-private-service:
	kubectl create -f lab02-04-sample-app/azure-vote-service-private.yaml

create-nodeport-service:
	kubectl create -f lab02-04-sample-app/nodeport-service.yaml
	kubectl describe service azure-vote-front-node

new-cleanup:
	kubectl delete service azure-vote-front-node
	kubectl delete service azure-vote-front-internal
	kubectl delete deployment azure-vote-front
	kubectl delete deployment azure-vote-back
	kubectl delete service azure-vote-front
	kubectl delete service azure-vote-back


#
# Lab 05b. Setup Ingress Controller
#
setup-ingress-controller:
	# Create a namespace for your ingress resources
	kubectl create namespace ingress-basic

	# Add the official stable repo
	helm repo add stable https://kubernetes-charts.storage.googleapis.com/

	# Use Helm to deploy an NGINX ingress controller
	helm install nginx stable/nginx-ingress \
    		--namespace ingress-basic \
    		--set controller.replicaCount=2 \
    		--set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    		--set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux

check-ingress-controller:
	kubectl get service -l app=nginx-ingress --namespace ingress-basic

#
# Lab 05a. Create Ingress distribution rules
#
create-ingress-sample-apps:
	kubectl create -f lab05-ingress/apple.yaml 
	kubectl create -f lab05-ingress/banana.yaml

define-ingress-rules:
	kubectl create -f lab05-ingress/app-ingress.yaml


#
# Debug pod for testing and curl internal IPs
#
test-pod:
	kubectl run -i --tty --rm debug --image=radial/busyboxplus --restart=Never -- sh

