```
 configures a remote back-end to enable remote state management using Terraform
```
# TF Installation
Goto terraform.io
download the zip file for required version
extract and run the terraform.exe (windows) / terraform (Linux)

## Windows
If you download Terraform for the Windows operating system:

1.  Find the install package, which is bundled as a zip file.
2.  Copy files from the zip to a local directory such as C:\terraform. It's the Terraform `PATH`, so make sure that the Terraform binary is available on the `PATH`.
3.  To set the PATH environment variable, run the command **set PATH=%PATH%;C:\terraform**, or point to wherever you've placed the Terraform executable.
4.  Open an administrator command window at `C:\Terraform` and run the command **Terraform** to verify the installation. You should view the terraform help output.

### Setup TF extension on VS code
![](Pasted%20image%2020220728225514.png)
OR
![](Pasted%20image%2020220611150526.png)


## Linux
1.  Download Terraform using the following command:
  ```
   wget https://releases.hashicorp.com/terraform/0.xx.x/terraform_0.xx.x_linux_amd64.zip
  ```

2.  Install Unzip using the command:
    ```
    sudo apt-get install unzip
    
    ```   

3.  Unzip and set the path using the command:
    ```
    unzip terraform_0.11.1_linux_amd64.zip
    sudo mv terraform /usr/local/bin/
    
    ```
   
4.  Verify the installation by running the command `Terraform`. Verify that the Terraform helps output displays.

## Install Using vagrant
```
install virtual box
install vagrant
clone the repo ...
vagrant up
vagrant ssh-config
```


# TF with Azure

## Authenticating Terraform with Azure
Terraform supports several different methods for authenticating to Azure. You can use:

-   The Azure CLI
-   A Managed Service Identity (MSI)
-   A service principal and a client certificate
-   A service principal and a client secret

When running Terraform as part of a continuous integration pipeline, you can use either an Azure service principal or MSI to authenticate.

To configure Terraform to use your Azure Active Directory (Azure AD) service principal, set the following environment variables:

-   ARM_SUBSCRIPTION_ID
-   ARM_CLIENT_ID
-   ARM_CLIENT_SECRET
-   ARM_TENANT_ID
-   ARM_ENVIRONMENT

The Azure Terraform modules then use these variables. You can also set the environment if you work with an Azure cloud other than an Azure public cloud.

Use the following sample shell script to set these variables:

```sh
#!/bin/sh
echo "Setting environment variables for Terraform"
export ARM_SUBSCRIPTION_ID=your_subscription_id
export ARM_CLIENT_ID=your_appId
export ARM_CLIENT_SECRET=your_password
export ARM_TENANT_ID=your_tenant_id

# Not needed for public, required for usgovernment, german, china
export ARM_ENVIRONMENT=public
```

## Azure CLI
`az login`
```shell
$ az account list

[
  {
    "cloudName": "AzureCloud",
    "id": "00000000-0000-0000-0000-000000000000", #Subscription ID
    "isDefault": true,
    "name": "PAYG Subscription",
    "state": "Enabled",
    "tenantId": "00000000-0000-0000-0000-000000000000",
    "user": {
      "name": "user@example.com",
      "type": "user"
    }
  }
]


```


## Provider setup
```sh
# Terraform version 0.13 or later
# We strongly recommend using the required_providers block to set the
# Azure Provider source and version being used
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=3.0.0"
    }
  }
}

# Configure the Microsoft Azure Provider
provider "azurerm" {
    features {}
}
```

To use specific subscription
```sh
# Configure the Microsoft Azure Provider
provider "azurerm" {
    features {}
    subscription_id = "00000000-0000-0000-0000-000000000000" #In case of multiple subscriptions
}
```

## Storage account setup
stoarge.tf
```sh
variable "storage_account_name" {
    default="appstore50001"
}
 
variable "resource_group_name" {
    default="terraform_grp"
}
 
provider "azurerm"{
version = "=2.0"
subscription_id = "20c6eec9-2d80-4700-b0f6-4fde579a8783"
tenant_id       = "5f5f1c90-abac-4ebe-88d7-0f3d121f967e"
features {}
}
 
resource "azurerm_resource_group" "grp" {
  name     = var.resource_group_name
  location = "North Europe"
}
 
resource "azurerm_storage_account" "store" {
  name                     = var.storage_account_name
  resource_group_name      = azurerm_resource_group.grp.name
  location                 = azurerm_resource_group.grp.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

```
terraform init #Initialise the backend
terraform plan -out storage.tfplan # Do a dry run
terraform apply "storage.tfplan" 
```

Note: Generating a random id to create a unique Storage account name.
```sh
resource "random_id" "storage_account" {
  byte_length = 8
}

resource "azurerm_storage_account" "testsa" {
  name                = "tfsta${lower(random_id.storage_account.hex)}"
  resource_group_name = azurerm_resource_group.testrg.name

  location     = "westus"
  account_type = "Standard_GRS"

  tags {
    environment = "staging"
  }
}
```

## Azure VM
vm.tf
```sh
variable "storage_account_name" {
    default="appstore50001" 
}
 
variable "network_name" {
    default="staging"
}
 
variable "vm_name" {
    default="stagingvm"
}
 
provider "azurerm"{
version = "=2.0"
subscription_id = "20c6eec9-2d80-4700-b0f6-4fde579a8783"
tenant_id       = "5f5f1c90-abac-4ebe-88d7-0f3d121f967e"
features {}
}
 
resource "azurerm_virtual_network" "staging" {
  name                = var.network_name
  address_space       = ["10.0.0.0/16"]
  location            = "North Europe"
  resource_group_name = "terraform_grp"
}
 
resource "azurerm_subnet" "default" {
  name                 = "default"
  resource_group_name  = "terraform_grp"
  virtual_network_name = azurerm_virtual_network.staging.name
  address_prefix     = "10.0.0.0/24"
}
 
resource "azurerm_network_interface" "interface" {
  name                = "default-interface"
  location            = "North Europe"
  resource_group_name = "terraform_grp"
 
  ip_configuration {
    name                          = "interfaceconfiguration"
    subnet_id                     = azurerm_subnet.default.id
    private_ip_address_allocation = "Dynamic"
  }
}
 
resource "azurerm_virtual_machine" "vm" {
  name                  = var.vm_name
  location              = "North Europe"
  resource_group_name   = "terraform_grp"
  network_interface_ids = [azurerm_network_interface.interface.id]
  vm_size               = "Standard_DS1_v2"
 
  storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
  storage_os_disk {
    name              = "osdisk1"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }
  os_profile {
    computer_name  = "stagingvm"
    admin_username = "demousr"
    admin_password = "AzurePortal@123"
  }
  os_profile_linux_config {
    disable_password_authentication = false
  }  
}
```

## Azure key vault
machine.tf
```sh
variable "storage_account_name" {
    default="appstore50001"
}
 
variable "network_name" {
    default="staging"
}
 
variable "vm_name" {
    default="stagingvm"
}
 
provider "azurerm"{
version = "=2.0"
subscription_id = "20c6eec9-2d80-4700-b0f6-4fde579a8783"
tenant_id       = "5f5f1c90-abac-4ebe-88d7-0f3d121f967e"
features {}
}
 
data "azurerm_key_vault" "keyvault" {
  name                = "appvault10001"
  resource_group_name = "newgrp1"
}
 
data "azurerm_key_vault_secret" "vmsecret" {
  name         = "vmpassword"
  key_vault_id = data.azurerm_key_vault.keyvault.id
}
 
resource "azurerm_virtual_network" "staging" {
  name                = var.network_name
  address_space       = ["10.0.0.0/16"]
  location            = "North Europe"
  resource_group_name = "terraform_grp"
}
 
resource "azurerm_subnet" "default" {
  name                 = "default"
  resource_group_name  = "terraform_grp"
  virtual_network_name = azurerm_virtual_network.staging.name
  address_prefix     = "10.0.0.0/24"
}
 
resource "azurerm_network_interface" "interface" {
  name                = "default-interface"
  location            = "North Europe"
  resource_group_name = "terraform_grp"
 
  ip_configuration {
    name                          = "interfaceconfiguration"
    subnet_id                     = azurerm_subnet.default.id
    private_ip_address_allocation = "Dynamic"
  }
}
 
resource "azurerm_virtual_machine" "vm" {
  name                  = var.vm_name
  location              = "North Europe"
  resource_group_name   = "terraform_grp"
  network_interface_ids = [azurerm_network_interface.interface.id]
  vm_size               = "Standard_DS1_v2"
 
  storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
  storage_os_disk {
    name              = "osdisk1"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }
  os_profile {
    computer_name  = "stagingvm"
    admin_username = "demousr"
    admin_password = data.azurerm_key_vault_secret.vmsecret.value
  }
  os_profile_linux_config {
    disable_password_authentication = false
  }  
}
```

# TF with AWS
## Setup
```
1. Open AWS Account
2. Create IAM admin user
```

1. Open AWS account
    sign up on aws.amazon.com
2. Create IAM admin user
    Create user and grant only programmatic access.
    Assign `AdministratorAccess` policy
    Download ur secret access key

## Spin up an EC2 instance
```
1. Create terraform file to spin up t2.micro ins ance
2. Run terraform apply
```

ec2_instance.tf
```tf
provider "aws" {
    access_key = "ACCESS_KEY_HERE"
    secret_key = "SECRET_KEY_HERE"
    region = "us-east-1"
    version = "~>=3.2.0" #Format- major.minor.patch
                         #~> will allow patch upgrades
}

resource "aws_instance" "example" {  # example is the instance name
    ami = "ami-0d729a60"  #Get the ami id from aws console
    instance_type = "t3.micro"
}
```

version.tf
```tf
terraform {

    required_version = ">= 0.12"

}
```

Provision the Infra
```sh
terraform init # when ever new dir is created, init has to be run
terraform plan # preview of the changes
terraform apply # apply the changes
```

# Launching GKE with Terraform
## Pre reqs
gcloud
terraform

`gcloud --version`
`gcloud auth login #Logon to gcloud`
`gcloud projects list`

`terraform -version`

## Procedure
have variable.tf created with variables

Create google project on windows cmd
```
export PROJECT_ID=doc-$(date +%Y%m%d%%H%M%S)
gcloud project create $PROJECT_ID
gcloud projects list
```
Create IAM service
```
gcloud iam service-accounts \
       create devops-catalog \
       --project $PROJECT_ID \
       --display-name devops-catalog
```
List and note down the email
`gcloud iam service-accounts list --project $PROJECT_ID`

Create keys for the IAM service account
```
gcloud iam service-accounts \
    keys create account.json \
    --iam-account devops-catalog@$PROJECT_ID.iam.gserviceaccount.com \
    --project $PROJECT_ID
```
list the keys
```
gcloud iam service-accounts \
    keys list \
    --iam-account devops-catalog@$PROJECT_ID.iam.gserviceaccount.com \
    --project $PROJECT_ID
```
Assign owner permission # **dont assign owner permission in prod**
```
gcloud projects add-iam-policy-binding $PROJECT_ID \
     --member serviceAccount:devops-catalog@$PROJECT_ID.iam.gserviceaccount.com \
     --role roles/owner
```
`export TF_VAR_project_id=$PROJECT_ID`

### Define the providers
provider.tf
```sh
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=3.0.0"
    }

    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

# Configure the Microsoft Azure Provider
provider "azurerm" {
  features {}
}
# Configure the AWS Provider
provider "aws" {
    access_key = "ACCESS_KEY_HERE"
    secret_key = "SECRET_KEY_HERE"
    region = "us-east-1"
}
```

## Exercises
1. create an ec2 resource and attach 5 EBS volumes to it using dynamic block. 