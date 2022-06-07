# TF Installation
Goto terraform.io
download the zip file for required version
extract and run the terraform.exe (windows) / terraform (Linux)

## Install Using vagrant
```
install virtual box
install vagrant
clone the repo ...
vagrant up
vagrant ssh-config
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
}

resource "aws_instance" "example" {  # example is the instance name
    ami = "ami-0d729a60"  #Get the ami id from aws console
    instance_type = "t3.micro"
}
```

version.tf
```terraform
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
```tf
provider "google" {
    credentials = file("account.json")


}
```