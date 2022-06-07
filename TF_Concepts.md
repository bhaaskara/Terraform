# Terraform Intro

# What is IAC?
Infrastructure as Code (_IaC_) is the managing and provisioning of infrastructure through code instead of through manual processes.

# What is Terraform
- Terraform is a open source IaC (Infrastructure as code) tool.
- It uses a high level declarative style language called HashiCorp Configuration Language (HCL) for defining infrastructure in ‘simple for humans to read’ configuration files.
- Its a CLI tool and doesn't have GUI
- It supports Windows, Mac and Linux

# Terraform vs Ansible
- Ansible, Puppet, Chef and Saltstack have of focus on automating the installation and configuration of the software. Keeping the machine in compliance.
- Terraform can automate provisioning of the infrastructure itself.

# Pros and Cons of Terraform
## Pros
- Automation of the infrastructure provisioning
- Repeatability
- Make your infra auditable.
    Auditability - Tracking changes or reverting changes

# Types of code
## Declarative
- Will have config files in HCL (Hashicorp config language)

Ex: AWS cloud formation

## Imperative
- Its a procedural approach
- We will have script

examples are AWS CLI or Python boto

Difficult to do changes.

## Imperative vs Declarative
**Imperative** model you define exact steps to follow to get an end result.
**Declarative** model you define what the end result you want and the tool will takes care of implementation.

# SDLC in IAC
![[SDLC_IAC.png]](Images/SDLC_IAC.png)

# Terraform concepts
## Life cycle
Terraform template (code) is written in HCL language and stored as a configuration file with a .tf extension. HCL is a declarative language, which means our goal is just to provide the end state of the infrastructure and Terraform will figure out how to create it. Terraform can be used to create and manage infrastructure across all major cloud platforms. These platforms are referred to as ‘providers’ in Terraform jargon, and cover AWS, Google Cloud, Azure, Digital Ocean, Open Stack and many others.

![](Pasted%20image%2020220607231049.png)


## Terraform init
The `terraform init` command is used to initialize a working directory containing Terraform configuration files. This is the first command that should be run after writing a new Terraform configuration or cloning an existing one from version control. 
It is safe to run this command multiple times.

Initializes the backend
Initializes provider plugins. (upgrades the plugins if available)

## TF Stages
- Refresh - query infrastructure provider to get current state
- Plan - Create an execution plan
- Apply - Execute the plan
- Destroy - Destroy the infrastructure/resources.

## Terraform HCL (HashiCorp configuration language)

## TF Configuration files
TF config files will be in HCL (Hashicorp configuration language) format
will have an tf extension i.e example.tf

helloworld.tf
```tf
output "Greetings"{
value = "Hello world"
}
Provider "random" {}
```
```
> terraform init
> terraform plan
> terraform apply
```

## Variables
• Everything in one file is not great   
• Use variables to hide secrets   
    • You don't want the AWS credentials in your git repository   
• Use variables for elements that might change   
    • AMIs are different per region   
• Use variables to make it yourself easier to reuse terraform files
- Terraform variables were completely reworked for the terraform 0.12 release.
- You can now have more control over the variables, and have `for` and `for-each` loops which      
   where not possible with earlier versions.
- You don't have to specify the type in variables, but its recommended.

- Terraform's simple variable types
    - String
    - Number
    - Bool

- Terraform's complex types
    - List(type): [5,1,1,2]
       A list is always ordered.
    - Set(type)
       A "set" is like a list, but it doesn't keep the order you put it in, and can only
       contain unique values.
       A list that has [5, 1, 1, 2] becomes [1 ,2,5] in a set (when you output it, terraform will sort it).
    - Map(type): {"key" = "value"}
    - object
    - tuple

**Note:** Most used variables are String, number, bool, list and map. 

### Define a variable
#### Strings
vars.tf
```tf
variable "myvar"{
    type= string
    default= "hello world."
}
```

**Access the variables**
```sh
> terraform console # it validates the files in CWD.
> var.myvar
> "${var.myvar}" #Introduced in v0.12
```

#### Map
```tf
variable "mymap"{
    type= map(string)
    default= {
      key1 = "value1"
    }
}
```

Access map
```tf
> terraform console
> var.mymap
> var.mymap["key1"]
> "${var.mymap["key1"]}" #Introduced in v0.12
```

#### List
```tf
variable "mylist"{
    type= list
    default= [1,2,3]
}
```

Access map
```tf
> terraform console
> var.mylist
> var.mylist[0]
> "${var.mylist[0]}" #Introduced in v0.12
```

## vars.tf vs terraform.tfvars
`vars.tf` file is used to define a variable for your terraform configuration.
If you run `terraform plan or apply` with a variable defined but no default. Terraform will prompt you for the value.

vars.tf
```tf
variable "my_var" { default = "foo" }
```

Or there are three ways to set or override defaults, or in the absence of a default, set the value. These are:

-   `terraform.tfvars` files
-   Environment variables prefixed with `TF_VAR`, e.g `TF_VAR_my_var=bar`
-   Flags passed to the `terraform` command `terraform apply -var 'my_var=bar'`

`terraform.tfvars` is a default file name, if this is present in your root directory then terraform will load this file and apply the values. You can also specify a custom file name  
using the command line flag `terraform apply -var-file=./nics.tfvars`



## Functions
### Element
```tf
> terraform console
> element(var.mylise, 1)


```
### Slice
```
slice(var.mylist, 0,2)
```
