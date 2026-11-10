# Automating Azure Infrastructure Using ARM Templates

## 📌 Project Overview

This project demonstrates the implementation of **Infrastructure as Code (IaC) using Azure Resource Manager (ARM) templates** to automate the deployment of cloud infrastructure for Client.

The solution uses a modular ARM template architecture to provision networking, virtual machines, Azure Storage, Private Endpoints and Private DNS configurations.

It also implements secure VM-to-Storage communication through Azure's private network, eliminating public internet exposure for storage access.

The project was completed as part of the **Microsoft Azure AZ-305: Designing Microsoft Azure Infrastructure Solutions** training.

## 🎯 Project Objectives

- Implement Infrastructure as Code using Azure ARM templates.
- Automate the deployment of Azure infrastructure.
- Create reusable and modular ARM templates.
- Deploy Azure Virtual Networks and subnets.
- Provision Azure Virtual Machines to host application workloads.
- Deploy Azure Storage Accounts for image-based content.
- Establish secure VM-to-Storage communication using Private Endpoints.
- Configure Private DNS for internal name resolution.
- Implement custom Azure RBAC roles following the principle of least privilege.
- Apply Azure governance using Azure Policy.

## 🏢 Business Scenario

Client delivers image-based content to a global audience and wants to adopt Infrastructure as Code practices to improve its cloud infrastructure management.

The organisation currently wants to move away from manual resource provisioning and establish an automated, consistent and reusable deployment model.

The proposed solution must support the following requirements:

**Infrastructure Automation**
- Deploy Azure resources using ARM templates.
- Create reusable infrastructure definitions.
- Reduce manual configuration errors.
- Support consistent deployments across environments.

**Secure Storage Connectivity**
- Host application workloads on Azure Virtual Machines.
- Store image-based content in Azure Storage.
- Establish private communication between application servers and Storage.
- Prevent public internet access to the Storage Account.

**Access Control and Governance**
- Provide controlled deployment permissions to the operations team.
- Implement custom Azure RBAC roles.
- Prevent unauthorised access control modifications.
- Enforce infrastructure standards using Azure Policy.

## 🏗️ Solution Architecture

The solution consists of the following Azure components:

| Azure Component | Purpose |
|---|---|
| Azure Resource Group | Organises and manages project resources |
| Azure Virtual Network | Provides private network connectivity |
| VM Subnet | Hosts application virtual machines |
| Private Endpoint Subnet | Hosts private connectivity endpoints |
| Azure Virtual Machine | Runs application workloads |
| Network Interface | Provides VM network connectivity |
| Network Security Group | Controls network traffic |
| Azure Storage Account | Stores image-based application content |
| Azure Private Endpoint | Provides private connectivity to Storage |
| Azure Private DNS Zone | Resolves Storage DNS names to private IP addresses |
| Azure ARM Templates | Automates infrastructure deployment |
| Azure RBAC | Controls operational permissions |
| Azure Policy | Enforces governance and compliance |

### Architecture Diagram

```text
                    Users
                      |
                      |
              Application Workload
                      |
                      |
              Azure Virtual Network
                      |
          ┌───────────┴────────────┐
          |                        |
       VM Subnet           Private Endpoint Subnet
          |                        |
          |                        |
   Virtual Machine          Private Endpoint
          |                        |
          |                        |
          └────────────┐           |
                       |           |
                       └─────┬─────┘
                             |
                     Azure Storage Account
                             |
                             |
                      Private DNS Zone
                privatelink.blob.core.windows.net
```

**Storage Communication Flow:**

Application VM → Azure Virtual Network → Private Endpoint → Azure Storage Account

Private DNS ensures that Storage Account DNS resolution points to the private endpoint rather than the public endpoint.

## 📁 Azure Resource Group

All project resources are deployed within a dedicated resource group.

| Configuration | Value |
|---|---|
| Resource Group | RG-RandEnt |
| Purpose | Centralised resource management |
| Access Control | Resource group-level RBAC |
| Deployment Method | ARM Templates |

The resource group provides a logical management boundary, simplifies deployment lifecycle management and enables scoped access control.

## 🌐 Network Architecture

### Virtual Network Configuration

**Virtual Network: Rand-VNet**

| Configuration | Value |
|---|---|
| Resource Type | Microsoft.Network/virtualNetworks |
| VNet Name | Rand-VNet |
| Address Space | 10.0.0.0/16 |

The virtual network provides private connectivity between application workloads and Azure services.

### Subnet Design

Two subnets were designed to separate application workloads from private connectivity resources.

**1. VM Subnet**

| Configuration | Value |
|---|---|
| Subnet Name | VMSubnet |
| Address Range | 10.0.1.0/24 |
| Purpose | Hosts application virtual machines |

**2. Private Endpoint Subnet**

| Configuration | Value |
|---|---|
| Subnet Name | PrivateEndpointSubnet |
| Address Range | 10.0.2.0/24 |
| Purpose | Hosts Azure Private Endpoints |

This subnet separation provides logical network segmentation and supports secure communication between application and storage resources.

## 💻 Virtual Machine Architecture

**Azure Resource:** `Microsoft.Compute/virtualMachines`

The virtual machine hosts the application workload required by client.

### Virtual Machine Responsibilities

- Runs application services.
- Communicates with Azure Storage.
- Operates within the Azure Virtual Network.
- Accesses Storage through the Private Endpoint.

### Supporting Network Resources

**Network Interface (NIC)**

Resource: `Microsoft.Network/networkInterfaces`

- Connects the virtual machine to the designated subnet.
- Provides private IP connectivity.
- Enables communication within the Azure Virtual Network.

**Network Security Group (NSG)**

Resource: `Microsoft.Network/networkSecurityGroups`

- Controls inbound and outbound network traffic.
- Protects virtual machine network access.
- Enforces network security rules.

## 💾 Azure Storage Account Architecture

**Azure Resource:** `Microsoft.Storage/storageAccounts`

The Storage Account is used to store image-based content required by the application platform.

### Storage Security Requirements

The Storage Account is designed to:

- Disable public network access.
- Allow connectivity through private networking.
- Support Azure Private Endpoint integration.
- Use secure communication between application and storage resources.

**Storage Connectivity Design**

```text
        Public Internet
              |
              X
              |
      Azure Storage Account
              |
              |
       Private Endpoint
              |
              |
      Azure Virtual Network
              |
              |
       Application VM
```

This design ensures that application-to-Storage communication takes place through private connectivity rather than the public internet.

## 🔐 Azure Private Endpoint Configuration

**Azure Resource:** `Microsoft.Network/privateEndpoints`

Azure Private Endpoint provides private connectivity between the application VM and Azure Storage.

A private IP address from the designated virtual network is assigned to the endpoint.

### Communication Flow

```text
      Virtual Machine
             |
             |
       Private IP Address
             |
             |
      Private Endpoint
             |
             |
     Azure Storage Account
```

### Key Benefits

- Removes public network exposure for Storage access.
- Uses Azure's private networking infrastructure.
- Provides secure connectivity to Azure PaaS services.
- Enables application workloads to access Storage using private IP addressing.

## 🌍 Private DNS Configuration

**Azure Resource:** `Microsoft.Network/privateDnsZones`

### Private DNS Zone

`privatelink.blob.core.windows.net`

Private DNS ensures that the Storage Account hostname resolves to the private IP address associated with the Private Endpoint.

**Without Private DNS**

```text
Storage Account URL
        |
        |
  Public Endpoint
```

**With Private DNS**

```text
Storage Account URL
        |
        |
   Private DNS
        |
        |
 Private Endpoint IP
```

### DNS Configuration Responsibilities

- Create the Private DNS Zone.
- Configure the DNS network link.
- Associate the private DNS configuration with the Private Endpoint.
- Enable internal DNS resolution for Storage connectivity.

This allows application workloads to access Azure Storage through its private endpoint using the normal Storage Account hostname.

## 📄 ARM Template Architecture

The solution uses a **modular ARM template structure**, separating infrastructure components into individual templates.

This approach improves reusability, maintainability and deployment management.

### GitHub Repository Structure

```text
RandEnt-ARM/
│
├── README.md
│
├── azuredeploy.json
├── azuredeploy.parameters.json
│
├── network.json
├── network.parameters.json
│
├── storage.json
├── storage.parameters.json
│
├── vm.json
├── vm.parameters.json
│
├── privateendpoint.json
├── privateendpoint.parameters.json
│
├── dns.json
├── dns.parameters.json
│
└── Screenshots/
```

### ARM Template Responsibilities

| Template | Responsibility |
|---|---|
| azuredeploy.json | Master template controlling the complete deployment |
| azuredeploy.parameters.json | Parameters for the master deployment |
| network.json | Creates VNet and subnets |
| network.parameters.json | Network configuration parameters |
| storage.json | Creates Storage Account and security configuration |
| storage.parameters.json | Storage deployment parameters |
| vm.json | Creates VM, NIC, NSG and required compute resources |
| vm.parameters.json | Virtual machine deployment parameters |
| privateendpoint.json | Creates Private Endpoint and Storage private connection |
| privateendpoint.parameters.json | Private Endpoint parameters |
| dns.json | Creates Private DNS Zone and network link |
| dns.parameters.json | DNS deployment parameters |

### Master Template – azuredeploy.json

The master template acts as the central deployment controller.

**Responsibilities:**
- Coordinates the complete infrastructure deployment.
- Calls individual child templates.
- Passes parameters between templates.
- Maintains the required deployment sequence.

### Network Template – network.json

Creates:
- Azure Virtual Network.
- VM Subnet.
- Private Endpoint Subnet.

### Storage Template – storage.json

Creates:
- Azure Storage Account.
- Storage security configuration.

### VM Template – vm.json

Creates:
- Azure Virtual Machine.
- Network Interface.
- Network Security Group.
- Required compute resources.

### Private Endpoint Template – privateendpoint.json

Creates:
- Azure Private Endpoint.
- Private connectivity to Azure Storage.

### DNS Template – dns.json

Creates:
- Azure Private DNS Zone.
- DNS network link.

## 🚀 Deployment Sequence

The ARM templates follow a structured deployment sequence to ensure that resource dependencies are satisfied.

```text
1. Network Deployment
         |
         ▼
2. Storage Account Deployment
         |
         ▼
3. Virtual Machine Deployment
         |
         ▼
4. Private Endpoint Deployment
         |
         ▼
5. Private DNS Deployment
```

### Deployment Dependencies

- Network infrastructure must exist before deploying virtual machines.
- The Storage Account must be created before configuring its Private Endpoint.
- Private DNS configuration is required for private Storage name resolution.

This deployment structure ensures that infrastructure components are provisioned in the appropriate order.

## 🛡️ Security and Least Privilege Design

The operations team requires controlled permissions to deploy and manage infrastructure.

A custom Azure RBAC role named **RandEnt Deployment Operator** was designed to provide the required operational permissions while restricting access control management.

### Custom RBAC Role

**Role Name:** `RandEnt Deployment Operator`

**Allowed Actions**

```json
{
  "actions": [
    "Microsoft.Resources/deployments/*",
    "Microsoft.Network/*",
    "Microsoft.Storage/*",
    "Microsoft.Compute/*"
  ]
}
```

**Restricted Actions**

```json
{
  "notActions": [
    "Microsoft.Authorization/*"
  ]
}
```

### Role Capabilities

The operations team can:

- Deploy Azure infrastructure.
- Update Azure resources.
- Manage networking components.
- Manage Storage resources.
- Manage virtual machines.
- Execute ARM template deployments.

The operations team cannot:

- Create or modify Azure RBAC roles.
- Assign permissions to other users.
- Modify access control configurations.
- Manage Azure authorization settings.

This approach follows the principle of least privilege and reduces the risk of privilege escalation.

## 🏛️ Azure Governance and Policy

Azure Policy is used as a governance mechanism to enforce organisational infrastructure standards.

### Policy Requirements

The proposed governance controls include:

- Approved virtual machine sizes.
- Approved Azure deployment regions.
- Storage security standards.
- Organisational compliance requirements.

### Governance Workflow

```text
       ARM Template
             |
             ▼
    Azure Resource Manager
             |
             ▼
      Azure Policy
       Validation
             |
             ▼
    Resource Deployment
```

Azure Policy helps ensure that deployed infrastructure follows the organisation's governance and compliance requirements.

## 🏆 Azure Well-Architected Framework Alignment

| Pillar | Implementation |
|---|---|
| Security | Private Endpoint, RBAC and Azure Policy |
| Reliability | Repeatable ARM deployments |
| Operational Excellence | Infrastructure as Code |
| Performance Efficiency | Azure Virtual Network |
| Cost Optimization | Parameterised deployments |
| Governance | RBAC and Azure Policy |

## 📊 Project Outcome

The RandEnt project demonstrates the design of an automated and secure Azure infrastructure deployment model using ARM templates.

The implementation addresses the organisation's requirements through:

- Automated Azure infrastructure provisioning.
- Reusable and modular ARM templates.
- Secure VM-to-Storage communication.
- Private Storage access without public network exposure.
- Private DNS-based name resolution.
- Controlled operational permissions through custom RBAC.
- Infrastructure governance using Azure Policy.

## 🧠 Technical Skills Demonstrated

- Microsoft Azure
- Azure Solution Architecture
- Azure Resource Manager (ARM) Templates
- Infrastructure as Code (IaC)
- JSON Template Development
- Azure Virtual Networks
- Azure Subnets
- Azure Virtual Machines
- Azure Storage Accounts
- Azure Private Endpoints
- Azure Private DNS Zones
- Network Security Groups
- Azure Role-Based Access Control (RBAC)
- Custom Role Definitions
- Azure Policy
- Cloud Security
- Infrastructure Automation
- Cloud Governance

## 🏁 Conclusion

Project successfully demonstrates the implementation of an automated Azure infrastructure deployment model using modular ARM templates.

The solution provides a reusable infrastructure architecture that supports application workloads, private Storage connectivity, secure networking and controlled operational access.

By combining ARM templates, Azure Private Endpoints, Private DNS, RBAC and Azure Policy, the project demonstrates practical skills in Infrastructure as Code, secure cloud architecture, infrastructure automation and Azure governance.

This implementation provides a foundation for designing and managing secure, scalable and maintainable enterprise cloud environments.
