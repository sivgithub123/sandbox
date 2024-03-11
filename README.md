# sandbox

# Create a DNS Zone for custom Domain
az network dns zone create -g <GROUP_NAME> -n mydev.org


# Assign Variables
tenantid=$(az account show --subscription <SUBSCRIPTION_NAME> --query tenantId --output tsv)

subscriptionid=$(az account show --query id -o tsv)

UserClientId=$(az aks show --name <CLUSTER_NAME> --resource-group <RG> --query identityProfile.kubeletidentity.clientId -o tsv)

DNSID=$(az network dns zone show --name mydev.org --resource-group <RG> --query id -o tsv) # Assign managed identity of cluster’s node pools DNS Zone Contributor rights on to Custom Domain DNS zone.

az role assignment create --assignee $UserClientId --role 'DNS Zone Contributor' --scope $DNSID

# Create an ingress controller
## Create a namespace for ingress resources
kubectl create namespace <NAMESPACE_NAME>  # Name for a namespace for ingress
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update # Use Helm to deploy an NGINX ingress controller

helm install ingress-nginx ingress-nginx/ingress-nginx \
    --namespace <NAMESPACE_NAME> \
    --set controller.replicaCount=2

# Deploy ExternalDNS
## Add the Helm repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

~~helm install my-release oci://registry-1.docker.io/bitnamicharts/external-dns~~

## Use Helm to deploy an External DNS
helm install external-dns bitnami/external-dns --namespace <NAMESPACE_NAME> --set provider=azure --set txtOwnerId=<CLUSTER_NAME> --set policy=sync --set azure.resourceGroup=<GROUP_NAME> --set azure.tenantId=$tenantid --set azure.subscriptionId=$subscriptionid --set azure.useManagedIdentityExtension=true --set azure.userAssignedIdentityID=$UserClientId


#Install cert-manager
## Label the cert-manager namespace to disable resource validation
kubectl label namespace ingress-basic cert-manager.io/disable-validation=true
## Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io
## Update your local Helm chart repository cache
helm repo update# Install CRDs with kubectl
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.2/cert-manager.crds.yaml

## Install the cert-manager Helm chart
helm install cert-manager jetstack/cert-manager \
  --namespace ingress-basic \
  --version v1.14.2

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.2/cert-manager.yaml

kubectl get pods -n cert-manager

# kubectl apply 

kubectl apply -f aks-helloworld-one.yaml --namespace <NAMESPACE_NAME>

kubectl apply -f aks-helloworld-two.yaml --namespace <NAMESPACE_NAME>

kubectl apply -f ingress.yaml --namespace <NAMESPACE_NAME>
