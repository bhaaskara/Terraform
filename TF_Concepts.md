# Terraform Concepts
# What is Terraform
Terraform is a declarative IAC (Infrastructure as code) tool.
Its a CLI tool and doesn't have GUI
It supports windows, Mac and Linux

# Pros and Cons of Terraform
## Pros
Repeatability
Auditability - Tracking changes or reverting changes
...

# Types of code
## Declarative
will have config files in HCL (Hashicorp config language)

Ex: AWS cloud formation
## Imperative
its a procedural approach
we will have script

examples are AWS CLI or Python boto

Difficult to do changes.

# SDLC in IAC
![[SDLC_IAC.png]](Images/SDLC_IAC.png)

# TF Configuration files
TF config files will be in HCL (Hashicorp configuration language) format
will have an tf extension i.e example.tf

```tf
output "Greetings"{
value = "Hello world"
}
Provider "random" {}
```
