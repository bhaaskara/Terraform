# Terraform Intro

## What is IAC?
Infrastructure as Code (_IaC_) is the managing and provisioning of infrastructure through code instead of through manual processes.

## What is Terraform
- Terraform is a open source IaC (Infrastructure as code) tool.
- It uses a high level declarative style language called HashiCorp Configuration Language (HCL) for defining infrastructure in ‘simple for humans to read’ configuration files.
- Its a CLI tool and doesn't have GUI
- It supports Windows, Mac and Linux

The Terraform language is declarative, describing an intended goal rather than the steps to reach that goal. The ordering of blocks and the files they are organized into are generally not significant; Terraform only considers implicit and explicit relationships between resources when determining an order of operations.

## Terraform vs Ansible
- Ansible, Puppet, Chef and Saltstack have of focus on automating the installation and configuration of the software. Keeping the machine in compliance.
- Terraform can automate provisioning of the infrastructure itself.

## Pros and Cons of Terraform
## Pros
- Automation of the infrastructure provisioning
- Repeatability
- Make your infra auditable.
    Auditability - Tracking changes or reverting changes

## Types of code
### Declarative
- Will have config files in HCL (Hashicorp config language)

Ex: AWS cloud formation

### Imperative
- Its a procedural approach
- We will have script

examples are AWS CLI or Python boto
Difficult to do changes.

## Imperative vs Declarative
**Imperative** model you define exact steps to follow to get an end result.
**Declarative** model you define what the end result you want and the tool will takes care of implementation.

## SDLC in IAC
![[SDLC_IAC.png]](Images/SDLC_IAC.png)

# Terraform Terminology
## Provider
Terraform relies on plugins called "providers" to interact with cloud providers, SaaS providers, and other APIs.

Terraform configurations must declare which providers they require so that Terraform can install and use them. Additionally, some providers require configuration (like endpoint URLs or cloud regions) before they can be used.



# Terraform concepts
## Providers
Terraform relies on plugins called "providers" to interact with cloud providers, SaaS providers, and other APIs.

Terraform configurations must declare which providers they require so that Terraform can install and use them. Additionally, some providers require configuration (like endpoint URLs or cloud regions) before they can be used.

https://registry.terraform.io

### What Providers Do
Each provider adds a set of [resource types](https://www.terraform.io/language/resources) and/or [data sources](https://www.terraform.io/language/data-sources) that Terraform can manage.
Every resource type is implemented by a provider; without providers, Terraform can't manage any kind of infrastructure.

### Where Providers Come From
Providers are distributed separately from Terraform itself, and each provider has its own release cadence and version numbers.

The [Terraform Registry](https://registry.terraform.io/browse/providers) is the main directory of publicly available Terraform providers, and hosts providers for most major infrastructure platforms.

Terraform template (code) is written in HCL language and stored as a configuration file with a .tf extension. HCL is a declarative language, which means our goal is just to provide the end state of the infrastructure and Terraform will figure out how to create it. Terraform can be used to create and manage infrastructure across all major cloud platforms. These platforms are referred to as ‘providers’ in Terraform jargon, and cover AWS, Google Cloud, Azure, Digital Ocean, Open Stack and many others.

### Provider version
| Version number | Description |
| :-- | :-- |
| >=1.0 | Great than or equal to the version |
| <=1.0 | Less than or equal to the version |
| ~>2.0 | any version in the 2.x range |
| >=2.0,<=2.3 | any version between 2.0 and 2.3 

### Define Terraform Providers

### Multiple providers
provider.tf
```sh
provider "aws" {
    region = "us-east-1"
    access_key = "acess key"
    secret_key = "secret key"
    version = "~>2.0"
}

provider "aws" {
    region = "us-west-1"
    access_key = "acess key"
    secret_key = "secret key"
    version = "~>2.0"
    alias = "test2"
}

```
resource.tf
```sh
resource "aws_instance" "web" {
    ami = "ami-id"
    instance_type = t2.micro
    
    provider = aws.test2 # when provider is not mentioned it takes default
}
```

**Note:** alias is required only when more than one provider is defined.
          alias is not required for default provider

### Multiple AWS accounts
...
- Update the multiple accounts on aws cli using - `aws configure`
- Mention the profile in provider block
```sh
provider "aws" {
    region = "us-east-1"
    access_key = "acess key"
    secret_key = "secret key"
    version = "~>2.0"
}

provider "aws" {
    region = "us-west-1"
    #access_key = "acess key"
    #secret_key = "secret key"
    profile = "user2"
    version = "~>2.0"
    alias = "test2"
}
```

```sh
resource "aws_instance" "web" {
    ami = "ami-id"
    instance_type = t2.micro
    
    provider = aws.test2 # when provider is not mentioned it takes default
}
```

### Terraform Settings Block
-   We primarily define the below 3 items in Terraform Settings Block
    -   Terraform Version
    -   Terraform Providers
        -   [Azure RM Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest)
        -   [Azure AD Provider](https://registry.terraform.io/providers/hashicorp/azuread/latest)
        -   [Random Provider](https://registry.terraform.io/providers/hashicorp/random/latest)
        -   Append with **/docs** for above 3 links to get their equivalent documentation
    -   Terraform State Storage Backend
-   Create a file **01-main.tf** and create terraform providers

```yml
# Terraform Settings Block (https://www.terraform.io/docs/configuration/terraform.html)
terraform {
  # Use a recent version of Terraform
  required_version = ">= 0.13"

  # Map providers to thier sources, required in Terraform 13+
  required_providers {
    # Azure Resource Manager 2.x (Base Azure RM Module)
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 2.0"
    }
    # Azure Active Directory 1.x (required for AKS and Azure AD Integration)
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 1.0"
    }
    # Random 3.x (Required to generate random names for Log Analytics Workspace)
    random = {
      source  = "hashicorp/random"
      version = "~> 3.0"
    }
  }
# Configure Terraform remote State Storage
    backend "azurerm" {
    resource_group_name   = "terraform-storage-rg"
    storage_account_name  = "terraformstatexlrwdrzs"
    container_name        = "prodtfstate"
    key                   = "terraform.tfstate"
  }
}

# Intialize the Azurerm provider
# This block is required for azurerm 2.x
provider azurerm {
  # v2.x azurerm requires "features" block
  features {}
}
```

## Life cycle
![](Pasted%20image%2020220607231049.png)

### Init
- Initializes the backend
- Downloads/Updates the plugins
- Downloads/Updates the providers

The Terraform binary contains the basic functionality and everything else is downloaded as and when required. The `terraform init` step analyses the code, figures out the provider and downloads all the plugins (code) needed by the provider (here, it’s AWS). Provider plugins are responsible for interacting over APIs provided by the cloud platforms using the corresponding CLI tools. They are responsible for the life cycle of the resource, i.e., create, read, update and delete. Figure below shows the checking and downloading of the provider ‘aws’ plugins after scanning the configuration file.

![](Pasted%20image%2020220618135155.png)

### Plan
The ‘terraform plan’ is a dry run for our changes. It builds the topology of all the resources and services needed and, in parallel, handles the creation of dependent and non-dependent resources. It efficiently analyses the previously running state and resources, using a resource graph to calculate the required modifications. It provides the flexibility of validating and scanning infrastructure resources before provisioning, which otherwise would have been risky.

**Refresh** - `Terraform refresh` will update the terraform.tfstate file with the existing cloud (remote) configuration. But Terraform only tracks the resources created by terraform.
If you need to add any existing resources to terraform, use `terraform import`.

changes in resources, with ‘+’ indicating addition and ‘-’ indicating deletion.

![](Pasted%20image%2020220618135403.png)

### Validate
Administrators can validate and approve significant changes produced in ‘terraform planning’s’ dry run.

### Apply
The ‘terraform apply’ executes the exact same provisioning plan defined in the last step, after being reviewed. Terraform transforms configuration files (.tf) based on appropriate API calls to cloud provider(s) automating resource creation (using the provider’s CLI) seamlessly.

### Destroy
After resources are created, there may be a need to terminate them. As Terraform tracks all the resources, terminating them is also simple. All that is needed is to run ‘terraform destroy’. Again, Terraform will evaluate the changes and execute them after you give permission.


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
will have an tf extension i.e `example.tf`

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

## Comments
The Terraform language supports three different syntaxes for comments:

-   [`#`](https://www.terraform.io/language/syntax/configuration#) begins a single-line comment, ending at the end of the line.
-   [`//`](https://www.terraform.io/language/syntax/configuration#-1) also begins a single-line comment, as an alternative to `#`.
-   [`/*`](https://www.terraform.io/language/syntax/configuration#-2) and `*/` are start and end delimiters for a comment that might span over multiple lines.

The `#` single-line comment style is the default comment style and should be used in most cases. Automatic configuration formatting tools (`terraform fmt`) may automatically transform `//` comments into `#` comments.

## Variables
• We can parameterize our deployments using Terraform Input Variables.
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

### Different options available to pass input variables
-   vars.tf or variables.tf 
	- Define and assign a default value.
-   arguments during runtime (-var)
-   arguments during runtime (-var-file terraform.tfvars) with a file containing variables

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
> terraform console # it validates the files in CMD.
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

# List of strings

variable "string_list" {"string1","string2"."string3" }

```

Access list
```tf
> terraform console
> var.mylist
> var.mylist[0]
> "${var.mylist[0]}" #Introduced in v0.12
```

### vars.tf vs terraform.tfvars
`vars.tf` file is used to define a variable for your terraform configuration.
If you run `terraform plan or apply` with a variable defined but no default. Terraform will prompt you for the value.

vars.tf
```tf
variable "my_var" { default = "foo" }
```

Or there are three ways to set or override defaults, or in the absence of a default, set the value. These are:
-   `terraform.tfvars` files or `.auto.tfvars`
-   Environment variables prefixed with `TF_VAR`, e.g `TF_VAR_my_var=bar`
-   Flags passed to the `terraform` command `terraform apply -var 'my_var=bar'`

`terraform.tfvars` is a default file name, if this is present in your root directory then terraform will load this file and apply the values. You can also specify a custom file name  
using the command line flag `terraform apply -var-file=./nics.tfvars`

### Variable assignment
1. Environment variables - `export TF_VAR_<var_name>=<value>` # in Linux
2. From file: `terraform.tfvars`
    ```
    <varname> = <value>
    ```
**Note:** `terraform.tfvars` is the default file name, if you want to use a different name, you need to parse it.
`terraform plan -var-file="<file.tfvars>"`

3. Command line arguments
`terraform plan -var="<var_name>=<value>" -var="<var_name2>=<value2>"`

4. Default values: `vars.tf`
```
    #This file defines the variables and assigns a default value
    variable "<var_name>" {
        default = "value"
        #type = string #in TF 0.13 and later type is not required
    }
```

Precedence...

## Output
-   Output values are like the return values of a Terraform module
-   Output values are a way to expose some of that information to the user of your module.
-   A child module can use outputs to expose a subset of its resource attributes to a parent module
-   A root module can use outputs to print certain values in the CLI output after running terraform apply

```
Provider "random" {}

output "Greetings"{
value = "Hello world"
}
```

```sh
# Output a variable value
output "source_dest_ckech" {
    value = aws_instance.web.source_dest_check

}
```

To export a value or to reference a value remotely or on a different resource tf file, then first output the value in output block of a tf file, and refer to it in data block in another tf file.

`terraform output` command will shows the output block values.

## Count
```sh
resorce "aws_instance" "testvm" {
    ami = "ami_id"
    instance_type = "t2.micro"
    count = 5
    tags {
        name = "vm.${count.index}"
    }
}
```

## Data
Data block is useful in fetching data from different resources (tfstate file) or remotely from S3 buckets or dynamically from cloud provider. (i.e AWS or Azure)



### Referencing values from dirrent resources tfstate file
![](Pasted%20image%2020220624203804.png)
Here lambda functions ARN (should be in output block)is refenced by cloudwatch(data block). 


## Locals
```sh
locals {
    common_tags= {
        company = "mycomp"
        project = "tfpoc"
    }
}

resource "aws_instance" "myvm1" {
    ami = "ami-id"
    instance_type = "t2.micro"
    tags = local.common_tags
}

resource "aws_instance" "myvm2" {
    ami = "ami-id"
    instance_type = "t2.small"
    tags = local.common_tags
}

```


## Functions
**Note:** user defined functions are not allowed in Terraform.

### Element
```tf
> terraform console
> element(var.mylist, 1)
```

### Lookup
```
lookup(map, key, default)

> lookup({a="ay", b="bee"}, "a", "what?")
> ay
```

### Slice
`slice(var.mylist, 0,2)`

### Length
`length(var.string_list)`

```
count = ${length(var.string_list)}
```

### Join
### Depends on
`depends_on = ["azurerm_resource_group.main"]`

### zipmap
```sh
resource "aws_iam_user" "username" {
    count = 4
    name = "iamuser.${count.index}"
}

output "userARN" {
    value = "aws_iam_user.username[*].arn"
}

output "userName" {
    value = "aws_iam_user.username[*].name"
}

## Here the output blocks will not have relation between arn and name
## Zipmap would give combined/mapped output

output "combined" {
    value = zipmap(aws_iam_user.username[*].name,aws_iam_user.username[*].arn)

}
```

### file

## IF / Conditional statement
`vmsize = ${var.environmet == "production" ? var.machine_type_prod : var.machine_type_dev }`

`count = var.project == "Atos" ? 3 : 1`

## Data source

## Formatting
`terraform fmt`

## Validate the configuration file
`terraform validate`

## Load order and semantics

## Dynamic blocks

vars.tf
```sh
variable "port" {
    type = list
    default = ["22","8443","3389","443"]
}

variable "eport" {
    type = list
    default = {"22","23","9443"}
}
```
dynamicsg.tf
```sh
resource "aws_security_group" "dynamicsg" {
    name = "DynamicSG"
    description = "Allow TLS inbound traffic"

    dynamic "ingress" {
        for_each = var.port
        content {
            from_port   = ingress.value 
            to_port     = ingress.value
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]  
        }
    }
    dynamic "egress" {
        for_each = var.eport
        content {
            
        }
    }
}
```

**Iterator**
```sh
resource "aws_security_group" "dynamicsg" {
    name = "DynamicSG"
    description = "Allow TLS inbound traffic"

    dynamic "ingress" {
        for_each = var.port
        iterator =iport
        content {
            from_port   = iport.value 
            to_port     = iport.value
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]  
        }
    }
    dynamic "egress" {
        for_each = var.eport
        content {
            
        }
    }
}
```

## Debugging
`export TF_LOG=TRACE` # TF_LOG can be TRACE, DEBUG, INFO, WARN, or ERROR
`export TF_LOG_PATH=trace.txt`

## Taint a resource
`terraform taint aws_insatnce.demoec2`
Tainted resource will be recreated next time when you run `terraform apply`

- list tainted resources ?
- how to untaint a resource ?

## Splat expression
A _splat expression_ provides a more concise way to express a common operation that could otherwise be performed with a `for` expression.

If `var.list` is a list of objects that all have an attribute `id`, then a list of the ids could be produced with the following `for` expression:

```hcl
[for o in var.list : o.id]
```
This is equivalent to the following _splat expression:_

```hcl
var.list[*].id
```
The special `[*]` symbol iterates over all of the elements of the list given to its left and accesses from each one the attribute name given on its right.

```hcl
resource "aws_instance" "demoec2" {
    ami = "amiid"
    instance_type = "t2.micro"
    count = var.project == "Atos" ? 5 : 1
    tags  = {
        name = "demoec2"
    }

    output "type" {
        value = aws.instance.demoec2[*].instance_type
    }
}
```

## Save terraform plan to a file
`terraform plan -out=testplan`
`terraform apply testplan`
**Note:** plan output file testplan is not human readable.

- how to preview a terraform plan file ?

## Terraform graph
```
1. Generate a dot file using terraform graph > graphout.dot
2. use graphviz utility to convert the dot file into graph
3. cat graphout.dot | dot -Tsvg > graph.svg
4. open the graph file with browser
```

## Terraform version and Provider version
```sh
terraform {
    required_version = "< 0.12"
    required_providers {
        aws = "~> 2.0"
    }
}
```
this can be mentioned in provider.tf or version.tf
`terraform state replace provider`

## tfswitch

## Dealing with large infrastructure
`terraform plan -refresh=false`  # To avoid API limits exhaust
`terraform plan -target=aws_instance.myec2`

## Provisioners
### Local provisioners
```sh
resource "aws_instance" "ec2test" {
    ami = "ami-id"
    instance_type = "t2.micro"
    tags = {
        name = "Testproject"
    }
    provisioner "local-exec" {
        command = "echo ${self.private_ip} >> pribate_ips.txt"
        on_failure = continue
    }
}
```
local provisioner runs only when the resource is created, and it wont run when resource is modified.
local resource runs on the machine where TF is installed or run from.
provisioner creation time failure cause the resource to be tainted and recreated during next time `terraform apply`.

### Remote provisioners
```sh
resource "aws_instance" "ec2test" {
    ami = "ami-id"
    instance_type = "t2.micro"
    tags = {
        name = "Testproject"
    }
    provisioner "remote-exec" {
        inline = [
            "puppet apply",
            "consul join${aws_instance.ec2test.private_ip}",
        ]
        connection {
            type = "ssh"
            host = "${var.host}"
            user = "ec2-user"
            passowrd = "${var.root_password}"
        }
}
```
- it runs on remote machine
- it works with ssh and winRM

### Destroy-time provisioner
```sh
resource "aws_instance" "ec2test" {
    ami = "ami-id"
    instance_type = "t2.micro"
    tags = {
        name = "Testproject"
    }
    provisioner "local-exec" {
        command = "echo ${self.private_ip} >> pribate_ips.txt"
        on_failure = continue
        when = destroy
    }
}
```

- how to run a provisioner every time ?

## Modules
**DRY principle - Do not repeat yourself**

Get the community built modules from registry.terrafor.io

### Module source from Git
```sh
module "module_local_name" {
    source = "git::<github SSH/HTTPS path>"
}
```
Reference a branch or tag
```sh
module "module_local_name" {
    source = "git::<github SSH/HTTPS path>?ref=<v0.0.11>/<branchname>"
}
```
## Workspaces
```
terraform workspace list
terraform workspace new <name>
terraform workspace select <name>
terraform workspace delete
terraform workspace show - shows current work space
```

### .gitignore
https://github.com/github/gitignore/blob/main/Terraform.gitignore

## Null Resource
![](Pasted%20image%2020220728132023.png)

...

# TF Remote state management using Backend
- With remote state management the statefile will be saved on the backend
- Local tfstate file will not be created/saved locally. 
## Azure
![](Pasted%20image%2020220728120358.png)
## AWS
```
1. Create a bucket on S3 and provide nessesary permissions
2. Define the backend in terraform block
3. Do a terraform init
```

```
terraform {
    backend "s3" {
        bucket = "my-state" #Bucket name on AWS
        key    = "prod/terraform.state" #it creates the dir prod, if not exists
        region = "us-east-1"    
    }
}
```


# Workspaces

# TF Import
Terraform also supports bringing existing infrastructure under its management. To do so, you can use the `import` command to migrate resources into your Terraform state file. The `import` command does not currently generate the configuration for the imported resource, so you must write the corresponding configuration block to map the imported resource to it.

Bringing existing infrastructure under Terraform's control involves five steps:

1.  Identify the existing infrastructure you will import.
2.  Import infrastructure into your Terraform state.
3.  Write Terraform configuration that matches that infrastructure.
4.  Review the Terraform plan to ensure the configuration matches the expected state and infrastructure.
5.  Apply the configuration to update your Terraform state.

**Warning:** Importing infrastructure manipulates Terraform state in ways that could leave existing Terraform projects in an invalid state. Make a backup of your `terraform.tfstate` file and `.terraform` directory before using Terraform import on a real Terraform project, and store them securely.

https://learn.hashicorp.com/tutorials/terraform/state-import
