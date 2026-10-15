

1. By default, any virtual machine (VM) in an Azure VNet has outbound access to the internet. 
2. Azure automatically creates a system route (0.0.0.0/0) that routes internet-bound traffic directly to its infrastructure, without requiring you to deploy an "Internet Gateway" resource.

3. To allow traffic into a VNet from the internet, Azure uses explicit public IP attachments rather than an IGW routing rule. 
   You achieve this by:
• Attaching a Public IP address directly to a VM's Network Interface (NIC).
• Placing the VM behind a public Azure Load Balancer or Application Gateway.

4. Azure's NAT Alternatives (For Outbound-Only Control)
If you want to mimic the secure, outbound-only design of an AWS private subnet (which uses an IGW + NAT Gateway in AWS), Azure provides:
• Azure NAT Gateway: The recommended service to give subnets outbound-only internet connectivity using static public IPs.
• Azure Firewall: A fully managed firewall service that secures outbound VNet traffic.





                    ┌──────────────────────┐
                    │    USERS / INTERNET  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   AZURE FRONT DOOR   │
                    │ Global LB + WAF + TLS│
                    └──────────┬───────────┘
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
                 ▼                           ▼
       ┌──────────────────┐        ┌──────────────────┐
       │   PRIMARY REGION │        │    DR REGION     │
       │     Region 1     │        │     Region 2     │
       └────────┬─────────┘        └────────┬─────────┘
                │                           │
                ▼                           ▼
       ┌──────────────────┐        ┌──────────────────┐
       │      VNET        │        │      VNET        │
       └────────┬─────────┘        └────────┬─────────┘
                │                           │
                ▼                           ▼
       ┌──────────────────┐        ┌──────────────────┐
       │ Application      │        │ Application      │
       │ Gateway / WAF     │        │ Gateway / WAF     │
       └────────┬─────────┘        └────────┬─────────┘
                │                           │
                ▼                           ▼
       ┌──────────────────┐        ┌──────────────────┐
       │    AKS Cluster   │        │    AKS Cluster   │
       │                  │        │                  │
       │ ┌────┐ ┌────┐    │        │ ┌────┐ ┌────┐    │
       │ │Pod │ │Pod │    │        │ │Pod │ │Pod │    │
       │ └────┘ └────┘    │        │ └────┘ └────┘    │
       └────────┬─────────┘        └────────┬─────────┘
                │                           │
                ▼                           ▼
       ┌──────────────────┐       ┌──────────────────┐
       │ Primary Database │──────►│ Secondary DB     │
       │                  │ Replication │             │
       └──────────────────┘       └──────────────────┘

                    ┌────────────────────────────┐
                    │      OBSERVABILITY        │
                    │                            │
                    │ Azure Monitor              │
                    │ Application Insights       │
                    │ Log Analytics              │
                    └────────────────────────────┘

                    ┌────────────────────────────┐
                    │       AUTOMATION            │
                    │                            │
                    │ Terraform                  │
                    │ Azure DevOps / GitHub      │
                    │ Actions CI/CD              │
                    └────────────────────────────┘