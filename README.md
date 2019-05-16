# Managed OpenShift
https://github.com/Azure/OpenShift
https://docs.microsoft.com/en-us/azure/aks/aad-integration

create app id and create permissions the same as for aks
```
az ad app create --display-name $ARO_CLUSTER_NAME --key-type Password --password $ARO_AAD_SECRET --identifier-uris $ARO_AAD_REPLY_URL --reply-urls $ARO_AAD_REPLY_URL
```

Define variables
```
ARO_RG_NAME=ARO-AUSTEAST-RG
ARO_CLUSTER_NAME=ARO311
LOCATION=australiaeast
ARO_AAD_SECRET=supersecret
ARO_AAD_ID=<aadclientid>
ARO_AAD_TENANT=<aadtenant>
ARO_AAD_REPLY_URL=https://$ARO_CLUSTER_NAME.$LOCATION.cloudapp.azure.com/oauth2callback/Azure%20AD
ARO_FQDN=$ARO_CLUSTER_NAME.$LOCATION.cloudapp.azure.com
ARO_ADMIN_GROUP_ID=<admingroupobjectid>
```

## create cluster via cli
```
az group create --name $ARO_RG_NAME --location $LOCATION

az openshift create --resource-group $ARO_RG_NAME --name $ARO_CLUSTER_NAME -l $LOCATION --fqdn $ARO_FQDN --aad-client-app-id $ARO_AAD_ID --aad-client-app-secret $ARO_AAD_SECRET --aad-tenant-id $ARO_AAD_TENANT

open https://ARO311.westeurope.cloudapp.azure.com

oc new-app openshift/ruby:25~https://github.com/denniszielke/ruby-ex 
```

## create via arm
```
az group create -n $ARO_RG_NAME -l $LOCATION

az group deployment create \
    --name openshiftmanaged \
    --resource-group $ARO_RG_NAME \
    --template-file "arm/openshift_template.json" \
    --parameters "arm/openshift_parameters.json" \
    --parameters "resourceName=$ARO_CLUSTER_NAME" \
        "location=$LOCATION" \
        "fqdn=$ARO_FQDN" \
        "servicePrincipalClientId=$ARO_AAD_ID" \
        "servicePrincipalClientSecret=$ARO_AAD_SECRET" \
        "tenantId=$ARO_AAD_TENANT" \
        "customerAdminGroupId=$ARO_ADMIN_GROUP_ID"
```


## cluster operations
```
az openshift scale --resource-group $ARO_RG_NAME --name $ARO_CLUSTER_NAME --compute-count 5

az openshift delete --resource-group $ARO_RG_NAME --name $ARO_CLUSTER_NAME 
```