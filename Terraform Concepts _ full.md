Till vid 90 refer to the TF_concepts file.
...

# Terraform state
## Terraform remote state
### Cross-project collaboration using remote state
The `terraform_remote_state` data source retrieves the root module output values from some   
other Terraform configuration, using the latest state snapshot from the remote backend.

![](Pasted%20image%2020230625133743.png)

Ref: [terraform-beginner-to-advanced-resource/Section 5 - Remote State Management/remote-states at master · bhaaskara/terraform-beginner-to-advanced-resource · GitHub](https://github.com/bhaaskara/terraform-beginner-to-advanced-resource/tree/master/Section%205%20-%20Remote%20State%20Management/remote-states)

### Implementing Remote states connections
1. Create a project with output values and remote-backend
    ![](Pasted%20image%2020230625140158.png)
2. Reference output values from different project 
    ![](Pasted%20image%2020230625140303.png)
## Terraform Import

# Security
Never mention the access key or Secret key in the code, i.e provider block.

## Provider configuration - Resources in multiple regions
**Using single AWS account**
use `alias` while defining the provider block.
```yaml
provider "aws" {
  region     =  "us-west-1"
}

provider "aws" {
  alias      =  "aws02"
  region     =  "ap-south-1"
}
```
in resource block refer to the region with alias name.
```yaml
resource "aws_eip" "myeip" {
  vpc = "true"
}

resource "aws_eip" "myeip01" {
  vpc = "true"
  provider = "aws.aws02"
}
```
Ref; [terraform-beginner-to-advanced-resource/Section 6 - Security Primer](https://github.com/bhaaskara/terraform-beginner-to-advanced-resource/tree/master/Section%206%20-%20Security%20Primer)

**Using multiple AWS accounts**
- use profile in provider definition block
- use AWS CLI to configure multiple profiles.
```
provider "aws" {
  region     =  "us-west-1"
}

provider "aws" {
  alias      =  "aws02"
  region     =  "ap-south-1"
  profile    =  "account02"
}
```

## Sensitive parameter
when `sensitive = true` is set, the variable value is not shown in the terraform output.
But the variable value is not encrypted in the state file.
```yaml
locals {
  db_password = {
    admin = "password"
  }
}

output "db_password" {
  value = local.db_password
  sensitive   = true
}
```