 # Creating the AKS Cluster using Azure CLI
 https://github.com/iam-veeramalla/Azure-zero-to-hero/tree/main/Day-20
# Errors-
1. Exception Details:      (MissingSubscriptionRegistration) The subscription is not registered to use namespace 'Microsoft.KeyVault'. See https://aka.ms/rps-not-found for how to register subscriptions.
If any subscription missing then register with below command

Run - $ az provider register --namespace Microsoft.KeyVault

  
  $ az group delete --name keyvault-demo 

  $ az account list --output table
  $ az account show --output table
  $ az account set --subscription "Subscription2"

 $ az group create --name keyvault-demo1 --location eastus

 $   az aks create \
    --name keyvault-demo-cluster \
    --resource-group keyvault-demo1 \
    --node-count 1 \
    --node-vm-size Standard_D2ds_v7 \
    --enable-addons azure-keyvault-secrets-provider \
    --enable-oidc-issuer \
    --enable-workload-identity \
    --generate-ssh-keys

 Verification-
 $ az aks show \
  --name keyvault-demo-cluster \
  --resource-group keyvault-demo1 \
  --query "agentPoolProfiles[].{Name:name,VMSize:vmSize,Count:count}" \
  --output table    

  $ az aks get-credentials --resource-group keyvault-demo1 --name keyvault-demo-cluster


  $ kubectl config view
  $ kubectl config current-context             ----> get the Cluster name

  $ kubectl get pods -n kube-system -l 'app in (secrets-store-csi-driver,secrets-store-provider-azure)' -o wide 

# Create the key vault

  $ az keyvault create -n aks-demo-shri -g keyvault-demo1 -l eastus --enable-rbac-authorization

# Connect your Azure ID to the Azure Key Vault Secrets Store CSI Driver
1. Configure workload identity

export SUBSCRIPTION_ID=fed0b121-49e9-44e0-be8b-471575df14e8
export RESOURCE_GROUP=keyvault-demo1
export UAMI=azurekeyvaultsecretsprovider-keyvault-demo-cluster
export KEYVAULT_NAME=aks-demo-shri
export CLUSTER_NAME=keyvault-demo-cluster

az account set --subscription $SUBSCRIPTION_ID

2. Create a managed identity

$ az identity create --name $UAMI --resource-group $RESOURCE_GROUP

export USER_ASSIGNED_CLIENT_ID="$(az identity show -g $RESOURCE_GROUP --name $UAMI --query 'clientId' -o tsv)"
export IDENTITY_TENANT=$(az aks show --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP --query identity.tenantId -o tsv)

3. Create a role assignment that grants the workload ID access the key vault

$ export KEYVAULT_SCOPE=$(az keyvault show --name $KEYVAULT_NAME --query id -o tsv)

 $ az role assignment create --role "Key Vault Administrator" --assignee $USER_ASSIGNED_CLIENT_ID --scope $KEYVAULT_SCOPE

4. Get the AKS cluster OIDC Issuer URL
 $ export AKS_OIDC_ISSUER="$(az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query "oidcIssuerProfile.issuerUrl" -o tsv)"
echo $AKS_OIDC_ISSUER 

5. Create the service account for the pod
 $ export SERVICE_ACCOUNT_NAME="workload-identity-sa"
 $ export SERVICE_ACCOUNT_NAMESPACE="default" 

 cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    azure.workload.identity/client-id: ${USER_ASSIGNED_CLIENT_ID}
  name: ${SERVICE_ACCOUNT_NAME}
  namespace: ${SERVICE_ACCOUNT_NAMESPACE}
EOF

 $ kubectl get sa

 6. Setup Federation

 $ export FEDERATED_IDENTITY_NAME="aksfederatedidentity" 

 $ az identity federated-credential create --name $FEDERATED_IDENTITY_NAME --identity-name $UAMI --resource-group $RESOURCE_GROUP --issuer ${AKS_OIDC_ISSUER} --subject system:serviceaccount:${SERVICE_ACCOUNT_NAMESPACE}:${SERVICE_ACCOUNT_NAME}

 7. Create the Secret Provider Class
# This is a SecretProviderClass example using workload identity to access your key vault
cat <<EOF | kubectl apply -f -
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-kvname-wi # needs to be unique per namespace
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    clientID: "${USER_ASSIGNED_CLIENT_ID}" # Setting this to use workload identity
    keyvaultName: ${KEYVAULT_NAME}       # Set to the name of your key vault
    cloudName: ""                         # [OPTIONAL for Azure] if not provided, the Azure environment defaults to AzurePublicCloud
    objects:  |
      array:
        - |
          objectName: secret1             # Set to the name of your secret
          objectType: secret              # object types: secret, key, or cert
          objectVersion: ""               # [OPTIONAL] object versions, default to latest if empty
        - |
          objectName: key1                # Set to the name of your key
          objectType: key
          objectVersion: ""
    tenantId: "${IDENTITY_TENANT}"        # The tenant ID of the key vault
EOF

# Verify Keyvault AKS Integration
1. Create a sample pod to mount the secrets

 $ kubectl get pods


2. List the contents of the volume
 $ kubectl exec busybox-secrets-store-inline-wi -- ls /mnt/secrets-store/

 3. Verify the contents in the file
 $ kubectl exec busybox-secrets-store-inline-wi -- cat /mnt/secrets-store/secret1
 $ kubectl exec busybox-secrets-store-inline-wi -- cat /mnt/secrets-store/key1

 # Delete the AKS Cluster

  $ az group delete --name keyvault-demo 



# Most common causes of CrashLoopBackOff error are grouped below by their underlying trigger:  
error occurs when a Kubernetes pod fails to start properly, crashes, and then repeatedly attempts to restart in a loop. 


1. Application and Code Issues

• Runtime Errors: The application throws an unhandled exception or panic immediately upon startup (e.g., a missing syntax element or unhandled null pointer).
• Missing Dependencies: The application requires specific packages, libraries, or external binaries that were not included in the container image.
• Database/API Unavailability: The code is written to exit/fail immediately if it cannot connect to its database or an external dependency upon initialization, rather than waiting or retrying gracefully. 


2. Configuration and Environment Faults

• Missing Environment Variables: The application expects certain config variables (like credentials or API keys) that were not injected into the pod specification.
• Incorrect File Paths: The application tries to read a configuration file or certificate from a path that does not exist inside the container.
• Secret or ConfigMap Mistakes: The pod references a Secret or ConfigMap that has not been created yet or has a typo in its name.


3. Container and Image Misconfigurations

• Wrong Entrypoint or Command: The command or args defined in the Kubernetes YAML are incorrect, making the container attempt to run a non-existent file or use invalid flags.
• Missing Command-Line Arguments: The container starts but immediately exits because it wasn’t provided with required startup parameters.
• Short-lived Containers: The container successfully runs a brief script (like an setup script) and exits normally with an exit code of 0. Because Kubernetes expects a deployment container to run indefinitely, it views any exit as a failure and restarts it. (Note: For short tasks, a Kubernetes Job should be used instead of a Deployment).


4. Resource Overload

• Out of Memory (OOMKilled): The container tries to use more memory than its defined Kubernetes limits.memory. The Linux kernel terminates the process, causing the pod to crash.
• CPU Throttling/Starvation: While rare for CPU to cause an outright crash, severe resource starvation can cause startup scripts to time out and fail.


5. Health Check Failures

• Failed Liveness Probes: The pod starts up, but its livenessProbe fails repeatedly (due to a wrong port, tight timeout, or slow application startup). Kubernetes assumes the pod is dead and kills it, triggering the crash loop.



# Troubleshooting Checklist

 $ kubectl describe pod <pod-name>
 $ kubectl logs <pod-name> --previous

# What Challenges have you faced when working with AKS?

Working with AKS can be challenging due to its complexity.
