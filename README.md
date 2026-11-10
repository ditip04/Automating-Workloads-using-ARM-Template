Automating Workloads Using Azure ARM Templates
Azure Solution Design Document
________________________________________
1. Project Overview
Client Corporation wants to adopt Infrastructure as Code (IaC) practices by implementing Azure Resource Manager (ARM) templates.
The operations team currently wants to move away from manual Azure resource creation and establish an automated, repeatable deployment model.
The objective of Project Ent is to create reusable ARM templates that provision the complete Azure infrastructure required for hosting the company's image-based application platform.
The solution must provide:
•	Automated infrastructure deployment.
•	Reusable ARM templates.
•	Secure application connectivity.
•	Private access to Azure Storage.
•	Controlled operational permissions.
________________________________________
2. Business Requirements
Client Corporation delivers image-based content to a global audience.
The application platform requires:
•	Virtual Machines to host application workloads.
•	Azure Storage Account to store image content.
•	Secure communication between application servers and storage.
•	A repeatable deployment process.
________________________________________
3. Solution Objectives
The solution must achieve the following:
Infrastructure Automation
Azure resources must be deployed using ARM templates instead of manual portal deployment.
Benefits:
•	Consistent deployments.
•	Reduced manual errors.
•	Faster provisioning.
•	Reusable infrastructure definition.
________________________________________
Secure Storage Communication
The application running on the VM must communicate with Azure Storage through Azure internal networking.
The design must avoid:
•	Public internet access.
•	Public Storage endpoints.
The required traffic flow:
Application VM

      |

      |

Azure Virtual Network

      |

      |

Private Endpoint

      |

      |

Azure Storage Account
________________________________________
Access Control
The operations team should only have permissions required to deploy Ent resources.
The implementation must follow:
•	Principle of least privilege.
•	Role-Based Access Control (RBAC).
•	Separation of duties.
________________________________________
4. Azure Architecture Overview
                         Users

                          |

                          |

                 Application Workload

                          |

                          |

                 Azure Virtual Network

                          |

        -------------------------------------

        |                                   |

    VM Subnet                    Private Endpoint Subnet

        |                                   |

        |                                   |

 Virtual Machine                 Private Endpoint

                                            |

                                            |

                                  Azure Storage Account

                                            |

                                            |

                                  Private DNS Zone

________________________________________
5. Resource Group Design
All resources are deployed inside:
RG-Ent
Purpose:
•	Provides resource management boundary.
•	Enables RBAC assignment.
•	Simplifies deployment lifecycle.
________________________________________
6. Network Architecture Design
Virtual Network
Resource:
Microsoft.Network/virtualNetworks
Example:
Rand-VNet
Purpose:
Provides private communication between Azure resources.
Example address space:
10.0.0.0/16
________________________________________
7. Subnet Design
VM Subnet
Purpose:
Hosts application virtual machines.
Example:
VMSubnet

10.0.1.0/24
Contains:
•	Virtual Machine NICs.
________________________________________
Private Endpoint Subnet
Purpose:
Hosts private connectivity endpoints.
Example:
PrivateEndpointSubnet

10.0.2.0/24
Contains:
•	Azure Private Endpoint.
________________________________________
8. Virtual Machine Architecture
Resource:
Microsoft.Compute/virtualMachines
Purpose:
Hosts the application workload used by Client.
The VM deployment includes:
Virtual Machine
Responsibilities:
•	Runs application services.
•	Connects to Azure Storage.
•	Operates inside Azure Virtual Network.
________________________________________
Network Interface
Resource:
Microsoft.Network/networkInterfaces
Responsibilities:
•	Provides VM network connectivity.
•	Connects VM to subnet.
•	Provides private IP communication.
________________________________________
Network Security Group
Resource:
Microsoft.Network/networkSecurityGroups
Responsibilities:
•	Controls inbound and outbound traffic.
•	Protects VM network access.
________________________________________
9. Storage Account Architecture
Resource:
Microsoft.Storage/storageAccounts
Purpose:
Stores image-based content for global application delivery.
________________________________________
Storage Security Requirements
The Storage Account must:
•	Disable public network access.
•	Allow only private connectivity.
•	Support Private Endpoint access.
•	Use secure communication.
Final design:
Public Internet

       X

       |

Azure Storage Account

       |

Private Endpoint

       |

Azure Virtual Network
________________________________________
10. Private Endpoint Design
Resource:
Microsoft.Network/privateEndpoints
Purpose:
Provides private connectivity from the application VM to Azure Storage.
The Private Endpoint assigns a private IP address inside the VNet.
Communication flow:
Virtual Machine

      |

      |

Private IP Address

      |

      |

Private Endpoint

      |

      |

Azure Storage
Benefits:
•	Removes public exposure.
•	Uses Azure backbone network.
•	Supports secure PaaS connectivity.
________________________________________
11. Private DNS Design
Resource:
Microsoft.Network/privateDnsZones
DNS Zone:
privatelink.blob.core.windows.net
Purpose:
Ensures that Storage Account DNS resolution points to the private endpoint.
Without Private DNS:
Storage URL

      |

Public Endpoint
With Private DNS:
Storage URL

      |

Private Endpoint IP
________________________________________
12. ARM Template Design
The solution uses a modular ARM template structure.
Each infrastructure component is separated into an individual template.
________________________________________
ARM Template Structure
Ent-ARM

├── azuredeploy.json

├── azuredeploy.parameters.json

|

├── network.json

├── network.parameters.json

|

├── storage.json

├── storage.parameters.json

|

├── vm.json

├── vm.parameters.json

|

├── privateendpoint.json

├── privateendpoint.parameters.json

|

├── dns.json

├── dns.parameters.json

________________________________________
13. Template Responsibilities
Master Template
File:
azuredeploy.json
Responsibilities:
•	Controls complete deployment.
•	Calls child templates.
•	Passes parameters.
•	Maintains deployment order.
________________________________________
Network Template
File:
network.json
Creates:
•	Virtual Network.
•	VM subnet.
•	Private Endpoint subnet.
________________________________________
Storage Template
File:
storage.json
Creates:
•	Storage Account.
•	Storage security configuration.
________________________________________
VM Template
File:
vm.json
Creates:
•	Virtual Machine.
•	Network Interface.
•	Network Security Group.
•	Required compute resources.
________________________________________
Private Endpoint Template
File:
privateendpoint.json
Creates:
•	Private Endpoint.
•	Storage private connection.
________________________________________
DNS Template
File:
dns.json
Creates:
•	Private DNS Zone.
•	DNS network link.
________________________________________
14. Deployment Sequence
The deployment order is:
1. Network Deployment

        |

        |

2. Storage Account Deployment

        |

        |

3. Virtual Machine Deployment

        |

        |

4. Private Endpoint Deployment

        |

        |

5. Private DNS Deployment

Reason:
•	VM requires networking.
•	Private Endpoint requires Storage Account.
•	DNS requires private connectivity configuration.
________________________________________
15. Security and Least Privilege Design
The operations team requires controlled deployment access.
The recommended approach is creating a custom RBAC role.
________________________________________
Custom Role Design
Role:
Ent Deployment Operator
Allowed actions:
Microsoft.Resources/deployments/*

Microsoft.Network/*

Microsoft.Storage/*

Microsoft.Compute/*
Restricted actions:
Microsoft.Authorization/*
________________________________________
Reason for Restriction
The operations team can:
•	Deploy infrastructure.
•	Update resources.
•	Manage application components.
The operations team cannot:
•	Create new roles.
•	Assign permissions.
•	Modify access control.
This prevents privilege escalation.
________________________________________
16. Azure Governance
Azure Policy should be used to enforce:
•	Approved VM sizes.
•	Approved Azure regions.
•	Storage security standards.
•	Compliance requirements.
Deployment process:
ARM Template

      |

      |

Azure Resource Manager

      |

      |

Azure Policy Validation

      |

      |

Resource Deployment
________________________________________
17. Azure Well-Architected Framework Alignment
Pillar	Implementation
Security	Private Endpoint, RBAC, Azure Policy
Reliability	Repeatable ARM deployments
Operational Excellence	Infrastructure as Code
Performance Efficiency	Azure Virtual Network
Cost Optimization	Parameterized deployments
Governance	RBAC and Policy
________________________________________
18. Final Solution Outcome
The Ent ARM template solution provides:
•	Automated Azure infrastructure deployment.
•	Reusable infrastructure templates.
•	Secure VM-to-Storage communication.
•	Private Storage access without public exposure.
•	Controlled operational permissions.
•	Governance through Azure RBAC and Azure Policy.
The solution demonstrates key AZ-305 Solution Architect capabilities:
•	Designing secure Azure architectures.
•	Implementing Infrastructure as Code.
•	Designing private connectivity.
•	Applying governance and security controls.
________________________________________
Conclusion
Project Ent successfully implements an automated Azure infrastructure deployment model using ARM templates.
The architecture provides a secure, scalable, and reusable platform for Client Corporation while ensuring that application workloads communicate with Azure Storage securely through Azure internal networking.

