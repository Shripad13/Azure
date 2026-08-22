# Azure
# Why organization prefers Azure over AWS Cloud?
The answer is not that Azure is better than AWS. Organizations choose the cloud platform that best fits their business, existing investments, technical requirements, and cost model.

Organizations choose Azure over AWS primarily because of their existing Microsoft ecosystem and business strategy rather than because one cloud is universally better. 
Companies already using Windows Server, Active Directory, SQL Server, and Microsoft 365 benefit from Azure's seamless integration, unified identity through Microsoft Entra ID, and licensing advantages like Azure Hybrid Benefit. 

AWS remains an excellent choice, particularly for cloud-native organizations and startups, so the decision usually depends on technical requirements, existing investments, compliance needs, and overall cost rather than one platform being inherently superior.

# Why do some organizations still prefer AWS?
AWS remains the market leader for many reasons:
Largest range of cloud services
Highly mature cloud platform
Broad third-party ecosystem
Strong support for cloud-native architectures
Excellent innovation in areas like serverless, analytics, AI, and databases


# Custom Data - 
When we some tools needs to be installed on VM during setting up of VM, Can be useful when we do Auto Scaling.

# User Data - 
It will keep running on VM all the time, even after VM creation.

# Azure Storage Account-
1. Blob/ Containers
2. File
3. Table
4. Queue

# Azure Command Line Interface -
Commands Used Almost Daily by DevOps Engineers

| Azure CLI Command               | Purpose                                  |
| ------------------------------- | ---------------------------------------- |
| `az login`                      | Authenticate to Azure                    |
| `az account show`               | Check current subscription               |
| `az account set --subscription` | Switch subscriptions                     |
| `az group list`                 | List resource groups                     |
| `az vm list`                    | List virtual machines                    |
| `az vm start/stop/restart`      | Manage VMs                               |
| `az storage account list`       | List storage accounts                    |
| `az storage blob upload`        | Upload files to Blob Storage             |
| `az storage blob generate-sas`  | Generate temporary access links          |
| `az aks get-credentials`        | Connect to an AKS cluster                |
| `az acr login`                  | Authenticate to Azure Container Registry |
| `az acr repository list`        | List container repositories              |
| `az webapp restart`             | Restart Azure App Service                |
| `az keyvault secret show`       | Retrieve secrets securely                |
| `az monitor activity-log list`  | Troubleshoot Azure activities            |
| `az network nsg list`           | View Network Security Groups             |
| `az identity list`              | Manage Managed Identities                |
| `az pipelines run`              | Trigger Azure DevOps pipelines           |



# Azure DevOps 
Platform which have collection of cloud services used by Devops Engineers will improve the SDLC process & Release cycle time.

Azure Boards: Plan and track tasks. Use agile boards and lists to see who does what.
Azure Repos: Store your code safely, version cotrolled. Use Git to save changes and work together.
Azure Pipelines: Build and test code automatically. Send updates to the cloud or servers.
Azure Test Plans: Check code quality. Run tests by hand or use smart tools.
Azure Artifacts: Share code packages with your team.


# Azure Micorsoft Entra ID (IAM)-
Who is a human? → User

How do I manage permissions for many humans? → Group

Who is an application/automation? → Service Principal

Who is an Azure resource that needs an identity? → Managed Identity

What can that identity do? → Role / Azure RBAC


Microsoft specifically recommends **managed identities** for service-to-service communication where Azure resources support Entra authentication, because Azure manages the credentials and you don't have to store or rotate secrets.

Example: VM → Blob Storage

Created VM
Craeted Blob Storage and in Access Control >> Grant Access to this resource 
Added the Role then Selected members to access this container.

Verification - ssh to VM and Fetch the access token first then using curl command accessed the file
1. Fetch the access token
access_token=$(curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fstorage.azure.com%2F' -H Metadata:true | jq -r '.access_token')

2. Access the blob from Virtual Machine
curl "https://$storage_account_name.blob.core.windows.net/$container_name/$blob_name" -H "x-ms-version: 2017-11-09" -H "Authorization: Bearer $access_token"


# Azure Kubernetes Service AKS Vs Self Managed K8s Service
In how many ways we can create a AKS Cluster - 
1. On-Premise servers - 5 servers - 3 server for control plane and 2 server for Data Plane
2. On Azure Cloud VM's - 5VM's - 3VM for control plane and 2VM for Data Plane
3. AKS - Node pool - Azure Managed K8s

Option 1 & 2 are Self Managed k8s service.

# Choose on-prem Kubernetes when:
You have strong on-prem requirements.
Data must remain in your datacenter.
You need very specific infrastructure/control.
You already have a mature Kubernetes platform team.
“Maximum control and customization, but maximum operational overhead—we own servers, OS, Kubernetes control plane, networking, storage, HA and upgrades.”

# Choose Kubernetes on Azure VMs when:
You specifically need to control the Kubernetes control plane.
You have an existing self-managed Kubernetes architecture.
You have specialized requirements that AKS doesn't satisfy.
“Azure removes the physical infrastructure burden, but Kubernetes is still self-managed, so we own the control plane, upgrades, networking, storage and operational complexity.”


# Choose AKS when:
You're building a new Kubernetes workload on Azure.
You want less infrastructure management.
You want strong Azure integration.
You want Entra ID, Azure RBAC, Azure networking, Azure Monitor, ACR, Key Vault, etc.
You want your team focused more on applications than maintaining Kubernetes control-plane infrastructure.
“AKS gives us managed Kubernetes control-plane operations and strong Azure integration, so we focus mainly on worker nodes, workloads and application reliability.”


If multiple Nodes then go for EFS PVC
If single Node then go for EBS PVC
You can also mention default storage class for pvc

AKS Node pool will have virtual machine scale set service
Image repository - Container registry

# Monitoring

L6 - Other Azure components - VM, Blob, etc - Use Azure Monitor
L5 - Application Performance Monitoring - Use Azure Application Insights in Monitor service
L4 - Pods, containers  - Using Prometheus/ Manged Prometheus
L3 - EKS Control plane - Use Azure Monitor
L2 - Nodes, Node Pools - Using Prometheus/ Manged Prometheus
L1 - Network Components - Using Network watcher Service 


## Azure Key vault
Using CSI Driver Pod gets secret from Keyvault
1. Create a resource group and create a AKS Cluster
2. Enable addons such as azure-keyvault-secrets-provider and oidc-cluster

--enable -addons azure-keyvault-secrets-provider --enable-oidc-issuer --enable-workload-identity

secret store CSI driver is a special kind of 
CSI driver gives capability to pod for communicating external solutions

Create a Managed Idenitity, Allow pod to talk to instance of Azure Vault key 
Integrate service account of pod with managed identity using this managed identity pod will access the secrets in key vault


[ AKS Pod ] ---> Uses a Kubernetes Service Account 
                    │
                    ▼ (Authenticated via Microsoft Entra Workload ID)
[ Secrets Store CSI Driver ] ---> Fetches secrets from ---> [ Azure Key Vault ]
                    │
                    ▼ (Mounts secret securely)
[ Pod File System (/mnt/secrets) ] 

1. The Pod is assigned a standard Kubernetes Service Account.
2. Microsoft Entra Workload ID maps that Kubernetes service account directly to an Azure Managed Identity.
3. The CSI Driver uses this identity to securely authenticate against Azure Key Vault, pulls the keys defined in your SecretProviderClass configuration, and maps them directly into your container.

4. Create a Secret store CSI
5. Install relevant provider - Azure vault 
6. when one resource wants to talk to other resource then we create Managed Identity
7. Create a Service account and integrate with Managed Identity

OIDC Issuer URL required to connect Service Account of AKS pod with Manged Identity

We will assign the Secret provider Class to the pod and within pod it should access object


# Why Terraform when you have ARM template, UI, CLI, Bicep, SDK?
1. Platform agnostics - supports multi-cloud   
2. Reusability
3. Version controlled
4. 