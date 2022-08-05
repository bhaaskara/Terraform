# AWS Resources
## AWS Instance ID
```sh
resource "aws_instance" "web" {
    ami = "<ami_id>"
    instance_type = "t2.micro"
}

resource "aws_ami_from_instance" "testami" {
    name = "TFtraining"
    source_instance_id = "aws_instance.web.id" 
} 
```

