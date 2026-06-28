<img width="671" height="208" alt="terraform update" src="https://github.com/user-attachments/assets/a3e72c51-c55f-4b7b-a760-37cfb9cba92c" />
# 🖥️ Terraform installation and connection to Azure

Terraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp that lets you define, provision, and manage cloud resources using declarative configuration files. 
Terraform connects to Microsoft Azure through the AzureRM provider, which authenticates in this case using an interactive Azure CLI login. With authentication in place, Terraform can communicate directly with Azure APIs to create, update, and destroy resources in a safe, repeatable, and version-controlled way.

In this section you will learn how to install Terraform and Azure CLI in a Widows Machine and how to connect it to you Azure susbcription.


---

## Prerequisites

Before you begin, ensure you have the following:

- An active **Azure Subscription**
- Permissions to create resources and assign roles (`Owner` or `Contributor` + `User Access Administrator`)

---

## Installation

**Install Terraform on Windows machine using PowerShell command**

<img width="671" height="208" alt="terraform update" src="https://github.com/user-attachments/assets/7aa0102c-38a7-431c-bfc6-d2df618c3403" />


- If you want to install the latest version, Open a PowerShell window as admin and type: 

```powershell

winget install -e --id Hashicorp.Terraform

```

- If you want to install and specific version:

```powershell

 winget install -e --id Hashicorp.Terraform --version 1.12.2

```

- If you want to uninstall it:

```powershell 

Open a PowerShell window as admin and type: winget uninstall -e --id Hashicorp.Terraform
```


**Install Azure CLI on Windows machine using PowerShell command**

Open a PowerShell window as admin and type:

<img width="951" height="215" alt="install Azure CLI" src="https://github.com/user-attachments/assets/2e9c9ae3-b03f-4d9a-8523-18942b2f43a5" />


```powershell 
winget install Microsoft.AzureCLI
```


---


## Authenticate the Azure Account

In the terminal, PowerShell or Visual Studio Code, window type:

```powershell 
az login

```

It will to launch a window and you should select your subscription.

Copy the subscription ID.

```powershell 
az account set --subscription "<your-subscription-id>"
```


Confirm the active subscription

```powershell 
az account show
```

---

