# sandbox

# Create a DNS Zone for custom Domain
az network dns zone create -g <GROUP_NAME> -n mydev.org


# Assign Variables
tenantid=$(az account show --subscription <SUBSCRIPTION_NAME> --query tenantId --output tsv)
subscriptionid=$(az account show --query id -o tsv)
UserClientId=$(az aks show --name <CLUSTER_NAME> --resource-group <RG> --query identityProfile.kubeletidentity.clientId -o tsv)
DNSID=$(az network dns zone show --name mydev.org --resource-group <RG> --query id -o tsv)# Assign managed identity of cluster’s node pools DNS Zone Contributor rights on to Custom Domain DNS zone.

az role assignment create --assignee $UserClientId --role 'DNS Zone Contributor' --scope $DNSID
