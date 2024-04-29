Usage | Command
:-- | :--
Terraform version | `Terraform -v`

Usage | Command
:-- | :--
Initialize | `terraform init`
Preview changes | `terraform plan`
save preview changes to a file | `terraform plan -out out.terraform`
Apply changes | `terraform apply`
Apply changes from an out file | `terraform apply out.terraform`
Destroy | `terraform destroy`

**Note**: Recommended to save changes to a file and then use that file with `terraform apply`

Terraform state unlock
`terraform force-unlock -force 1708007759844811`