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


- If you want to install the latest version, Open a PowerShell window as admin and type: 

```powershell

winget install -e --id Hashicorp.Terraform

```

<img width="671" height="208" alt="terraform update" src="https://github.com/user-attachments/assets/932a327a-9da7-4b1b-bc7f-ff5e9e80701e" />




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

```powershell 
winget install Microsoft.AzureCLI
```


<img width="951" height="215" alt="install Azure CLI" src="https://github.com/user-attachments/assets/c023d977-59cf-433f-831b-02ef9da87de3" />



---


## Authenticate the Azure Account

In the terminal, PowerShell or Visual Studio Code, window type:


```powershell 
az login

```


It will to launch a window and you should select your subscription.


<img width="796" height="456" alt="select account" src="https://github.com/user-attachments/assets/72d2edba-f8d4-4273-bf0c-2d8fe7ed2a1a" />



Copy the subscription ID.


```powershell 
az account set --subscription "<your-subscription-id>"
```


Confirm the active subscription

```powershell 
az account show
```

---

