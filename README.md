# 🏛️ Azure Active Directory Domain Controller — Terraform Deployment Lab

![Terraform](https://img.shields.io/badge/Terraform-v1.3+-7B42BC?logo=terraform&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-AzureRM_3.x-0078D4?logo=microsoftazure&logoColor=white)
![Windows Server](https://img.shields.io/badge/Windows_Server-2022_Datacenter-0078D4?logo=windows&logoColor=white)
![Status](https://img.shields.io/badge/Lab_Status-Ready-brightgreen)

> Deploy a **Windows Server 2022 Domain Controller** on Azure — fully automated with Terraform and a Custom Script Extension. One `terraform apply` provisions all infrastructure and promotes the server to an Active Directory forest.

---

## 📌 Overview

This lab deploys a **Windows Server 2022 VM** on Azure and automatically configures it as an **Active Directory Domain Controller** using a Terraform `CustomScriptExtension`. The extension executes a PowerShell command that installs the AD DS role and promotes the server to a new forest — all in a single apply.

### What Gets Deployed

| Resource | Name Pattern |
|---|---|
| Resource Group | `rg-ad-<yourname>` |
| Virtual Network | `vnet-ad-<yourname>` |
| Subnet | `snet-ad` |
| Public IP (Static) | `pip-ad-<yourname>` |
| Network Security Group | `nsg-ad-<yourname>` (RDP open) |
| Network Interface | `nic-ad-<yourname>` |
| Windows Server 2022 VM | `vm-ad-<yourname>` |
| Custom Script Extension | Installs AD DS + DNS, promotes to new forest |

> ⚠️ **Post-deployment authentication:** The VM reboots automatically after AD DS installs. Use domain credentials after reboot — `CORP\adadmin` or `adadmin@corp.charles.com` — not the local account syntax.

---

## ✅ Prerequisites

Before starting, ensure the following are ready:

- [ ] [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) installed and authenticated (`az login`)
- [ ] [Terraform](https://developer.hashicorp.com/terraform/downloads) **v1.3+** installed
- [ ] Active Azure subscription with permissions to create resources
- [ ] A local directory to store Terraform files



To learn how to Install Terraform and connect it to your Azure susbscription, please check on: 

[Terraform installation and connection to Azure](https://github.com/smarcecd/Terraform-Automation---Azure-Active-Directory-Domain-Controller/blob/main/Terraform%20Install%20and%20Azure%20connection.md)

---

## 🗂️ Folder Structure

```bash
az-ad-vm/
├── main.tf            # All Azure resources + Custom Script Extension
├── variables.tf       # Input variable declarations
├── outputs.tf         # Public IP, domain name, admin user outputs
└── terraform.tfvars   # Your personal values (⚠️ do not commit to Git)
```

---

## 🚀 Part 1 — Scaffold the Project

Run this single command on PowerShell to create the project directory and all four Terraform files:

```powershell
mkdir -p ~/repos/az-ad-vm && cd ~/repos/az-ad-vm && \
  touch main.tf variables.tf outputs.tf terraform.tfvars
```
---

## 📄 Part 2 — Terraform Configuration Files

Defines all Azure resources and the Custom Script Extension that installs and configures AD DS.
Open each of the files on Visual Studio Code and paste the followin information:

**main.tf**

```bash
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "main" {
  name     = "rg-ad-${var.yourname}"
  location = var.location
  tags     = var.tags
}

resource "azurerm_virtual_network" "main" {
  name                = "vnet-ad-${var.yourname}"
  location            = var.location
  resource_group_name = azurerm_resource_group.main.name
  address_space       = ["10.0.0.0/16"]
  tags                = var.tags
}

resource "azurerm_subnet" "main" {
  name                 = "snet-ad"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_public_ip" "main" {
  name                = "pip-ad-${var.yourname}"
  location            = var.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
  tags                = var.tags
}

resource "azurerm_network_security_group" "main" {
  name                = "nsg-ad-${var.yourname}"
  location            = var.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "allow-rdp"
    priority                   = 1000
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "3389"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  tags = var.tags
}

resource "azurerm_network_interface" "main" {
  name                = "nic-ad-${var.yourname}"
  location            = var.location
  resource_group_name = azurerm_resource_group.main.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.main.id
    private_ip_address_allocation = "Static"
    private_ip_address            = "10.0.1.4"
    public_ip_address_id          = azurerm_public_ip.main.id
  }

  tags = var.tags
}

resource "azurerm_network_interface_security_group_association" "main" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.main.id
}

resource "azurerm_windows_virtual_machine" "main" {
  name                  = "vm-ad-${var.yourname}"
  computer_name         = "ad-${var.yourname}"
  location              = var.location
  resource_group_name   = azurerm_resource_group.main.name
  size                  = "Standard_D2s_v3"
  admin_username        = "adadmin"
  admin_password        = var.admin_password
  network_interface_ids = [azurerm_network_interface.main.id]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
    disk_size_gb         = 127
  }

  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2022-Datacenter"
    version   = "latest"
  }

  additional_unattend_content {
    content = "<AutoLogon><Password><Value>${var.admin_password}</Value></Password><Enabled>true</Enabled><LogonCount>1</LogonCount><Username>adadmin</Username></AutoLogon>"
    setting = "AutoLogon"
  }

  tags = var.tags
}

resource "azurerm_virtual_machine_extension" "ad_setup" {
  name                 = "install-ad-ds"
  virtual_machine_id   = azurerm_windows_virtual_machine.main.id
  publisher            = "Microsoft.Compute"
  type                 = "CustomScriptExtension"
  type_handler_version = "1.10"

  settings = jsonencode({
    commandToExecute = "powershell -ExecutionPolicy Unrestricted -Command \"Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools; Import-Module ADDSDeployment; Install-ADDSForest -DomainName '${var.domain_name}' -DomainNetbiosName '${var.domain_netbios}' -ForestMode 'WinThreshold' -DomainMode 'WinThreshold' -InstallDns:$true -SafeModeAdministratorPassword (ConvertTo-SecureString '${var.dsrm_password}' -AsPlainText -Force) -Force:$true\""
  })

  tags = var.tags
}
```

**variables.tf**

```bash

variable "yourname" {
  description = "Your name — used to make resource names unique."
  type        = string
}

variable "location" {
  description = "Azure region."
  type        = string
  default     = "eastus"
}

variable "admin_password" {
  description = "Local admin password for the VM."
  type        = string
  sensitive   = true
}

variable "dsrm_password" {
  description = "Directory Services Restore Mode password for AD DS."
  type        = string
  sensitive   = true
}

variable "domain_name" {
  description = "Fully qualified domain name (e.g. corp.example.com)."
  type        = string
  default     = "corp.example.com"
}

variable "domain_netbios" {
  description = "NetBIOS name for the domain (max 15 characters)."
  type        = string
  default     = "CORP"
}

variable "tags" {
  description = "Tags applied to all resources."
  type        = map(string)
  default = {
    project = "ad-lab"
  }
}


```

**terraform.tfvars**

⚠️ This is ONLY for lab pospurses ⚠️  Never commit this file to Git. Add *.tfvars to your .gitignore. Treat passwords as secrets.

```bash
yourname       = "sandy"
location       = "eastus"
admin_password = "YourPassword123!"
dsrm_password  = "YourDSRMPassword123!"
domain_name    = "corp.sandy.com"
domain_netbios = "CORP"

```
💡 DSRM Password: The Directory Services Restore Mode password is separate from the VM admin password. Store it in a secrets manager — it is only needed for AD recovery operations and cannot be retrieved after deployment.


**outputs.tf**

```bash
output "public_ip" {
  description = "Public IP — use this to RDP into the domain controller."
  value       = azurerm_public_ip.main.ip_address
}

output "domain_name" {
  description = "Active Directory domain name."
  value       = var.domain_name
}

output "admin_username" {
  description = "Local admin username."
  value       = "adadmin"
}


```

---

## ▶️ Part 3 — Deploy

Run from the ~/repos/az-ad-vm directory:

```bash
# 1 — Initialize providers and backend
terraform init

# 2 — Preview the execution plan
terraform plan

# 3 — Apply and deploy all resources
terraform apply

```

⏱️ Estimated time: terraform apply takes 5–8 minutes for the VM deployment, followed by 3–5 more minutes for the Custom Script Extension to install AD DS and trigger the automatic reboot.

---

## 🖥️ Part 4 — Connect via RDP

Retrieve the public IP once apply completes:

```bash
terraform output public_ip

```

Use one of the following credential formats to RDP in:

| Method | Username | When to Use |
|--------|--------|--------|
| Domain prefix | CORP\adadmin | ✅ Use this first — standard post-promotion login |
| UPN format | adadmin@corp.sandy.com | If domain prefix is not accepted |
| Local account | .\adadmin | Only if AD DS promotion failed |

⏳ Wait for reboot: The VM reboots automatically after AD DS installs. Wait 5–10 minutes after terraform apply completes before connecting. Early connections may result in a failed session or black screen.

---

## 🔍 Part 5 — Verify AD DS

Once connected via RDP, open PowerShell as Administrator and run:

```powershell
# Verify AD DS service is running
Get-Service NTDS | Select-Object Name, Status

# Confirm domain configuration
Get-ADDomain

# List all domain controllers
Get-ADDomainController -Filter *

# Verify DNS is resolving the domain
Resolve-DnsName corp.charles.com
```

All four commands should return without errors. Get-ADDomain will display the full domain configuration including forest and domain functional levels.

---

## 🔧 Troubleshooting

Check the extension provisioning status from your local machine:

```bash
az vm extension show \
  --resource-group rg-ad-charles \
  --vm-name vm-ad-charles \
  --name install-ad-ds \
  --query "provisioningState" \
  --output tsv

```

## Configure the Active Directory server

For the AD configuration, you can follow the guides below:

- 🔐 **Active Directory Server Lab**   https://github.com/smarcecd/active-directory-az-vm-lab


- ⚡ **AD PowerShell Automation Guide**  https://github.com/smarcecd/active-directory-az-vm-lab/blob/main/AD_POWERSHELL_GUIDE.md

---

## 🧹 Teardown

Destroy all resources when you are finished:

```bash
terraform destroy
```

This removes the resource group and everything inside it — VM, OS disk, NIC, public IP, NSG, VNet, and subnet.


## 🧪 Lab authored for educational purposes. Customize domain_name, yourname, and passwords before deploying. Always destroy lab resources when done to avoid unnecessary Azure charges.

