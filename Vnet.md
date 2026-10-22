

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


Core VNet & High Availability Components


1. Network Container & Segmentation

• Virtual Network (VNet): Act as the foundational isolation boundary in Azure. For high availability, a single VNet is configured to cross-span multiple Availability Zones within the region. Simplilearn.com +1
• Subnets: Subnets logically segment your multi-tier application (e.g., Ingress, App, and Data tiers). In modern Azure design, a single subnet natively stretches across multiple Availability Zones, allowing you to deploy redundant resources in different zones under the exact same subnet prefix. 



2. Traffic Routing & Load Balancing (HA Ingress)

• Azure Application Gateway (with WAF): Functions as a Layer 7 load balancer that handles public HTTPS traffic. It is deployed in Zone-Redundant mode, meaning Azure automatically provisions and manages gateway instances across multiple zones simultaneously. If Zone 1 goes offline, instances in Zone 2 continue serving traffic smoothly. 

Azure Internal Load Balancer (ILB): Operates at Layer 4 to handle internal network traffic securely between tiers (such as passing requests from Web VMs to Application VMs). Configuring it with standard SKU ensures zone redundancy, dynamically distributing workloads across zone-separated instances


3. Compute Redundancy

• Virtual Machine Scale Sets (VMSS) / Zone-Spanning VMs: Instead of keeping all compute nodes inside one data center, VMs are distributed evenly across Availability Zones 1, 2, and 3. Placing at least two virtual machines in separate Availability Zones guarantees a higher uptime Service Level Agreement (SLA) up to 99.99%



4. Security & Traffic Control

• Network Security Groups (NSGs): Provide granular stateful packet filtering at the subnet or individual network interface card (NIC) level. NSGs enforce the rule of least privilege (e.g., blocking the Data Tier subnet from receiving direct public internet access). CloudThat +1


• User Defined Routes (UDRs) & Route Tables: Override default system pathways to redirect outbound or east-west traffic through a centralized inspection appliance like Azure Firewall for advanced threat protection and logging. 



5. High Availability Data Tier

• Data Tier Subnet: Hosts database clusters or platform data solutions. Utilizing configurations like SQL Server Always On Availability Groups ensures that data is synchronously replicated from the Primary database in Zone 1 to a Standby replica in Zone 2, making failover nearly instantaneous if a zone failure strikes. 