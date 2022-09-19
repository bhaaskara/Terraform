# TF files and Dirs
| File/Dir | Usage |
| :-- | :-- |
| .terraform/plugins | contains plugin info
| .tf | all the configuratin files
|.tfstate | Current state of the Infrastructure

# TF commands
| Usage | Command |
| :-- | :-- |
| Initilize the backend/providers | `terraform init` |
| | `terraform get` |
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


# Get Iterative names like, vm-1,vm-2
```sh
variable "name_count" { default= ["server1","server2","server3"]}

resource "azurerm_virtual_machine" "myvm"{
    count= length(var.name_count)
    name= "vm-${count.index+1}""
    ...
}
```
