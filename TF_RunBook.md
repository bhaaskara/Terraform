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

## Provider setup
```sh
# Terraform version 0.13 or later


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