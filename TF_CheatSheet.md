# TF files and Dirs
| File/Dir | Usage |
| :-- | :-- |
| .terraform/plugins | contains plugin info
| .tf | all the configuratin files
|.tfstate | Current state of the Infrastructure

# Installation and Configuration

# TF commands
| Usage | Command |
| :-- | :-- |
| Initilize the backend/providers | `terraform init` |
| Downloads the modules | `terraform get` |
|Compared the desired state (.tf files) with the current state (.tfstate) | `terraform plan` |
|Refreshes current state in terraform.tfstate file  | `terraform refresh`
|  | `terraform apply -auto-approve` |
|Delete the resources | `terraform destroy` |
| Backup tfstate before modifying | `terraform destroy -backup=<path>`
| Delete a specific resource | `terraform destroy -target="azurerm_resource_group.myrg1"`
| Format corections / auto format | `terraform fmt` |
| Validate the code | `terraform validate`
| List of resources in tfstate file | `terraform state list`
| show a resource details | `terraform state show`
| List/Show the entire terraform statefile | `terraform show`
| List providers | `terraform providers`

# List all the output values or attributes for a resource
- Refer to the attribute section under terraform documentation.
- In the output block of code mention only `aws_instance.myec2` to list all the attributes available.
   i.e dont mention any attribute in the output block. `aws-instance.myec2.eip`

# Get Iterative names like, vm-1,vm-2
```sh
variable "name_count" { default= ["server1","server2","server3"]}

resource "azurerm_virtual_machine" "myvm"{
    count= length(var.name_count)
    name= "vm-${count.index+1}""
    ...
}
```
