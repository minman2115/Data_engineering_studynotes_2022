.

Data_Engineering_TIL(20220119)

[학습자료]

테라폼 워크샵을 참가해서 공부한 내용을 정리했습니다.

[강의자료](https://hashicorp.github.io/field-workshops-terraform/slides/korean/aws/terraform-oss/#1)

[학습내용]

```console
# 테라폼 버전확인
root@workstation:~/hashicat-aws# terraform version
Terraform v1.0.7
on linux_amd64

Your version of Terraform is out of date! The latest version
is 1.1.3. You can update by downloading from https://www.terraform.io/downloads.html

# 테라폼 help 명령어
root@workstation:~/hashicat-aws# terraform --help
Usage: terraform [global options] <subcommand> [args]

The available commands for execution are listed below.
The primary workflow commands are given first, followed by
less common or more advanced commands.

Main commands:
  init          Prepare your working directory for other commands
  validate      Check whether the configuration is valid
  plan          Show changes required by the current configuration
  apply         Create or update infrastructure
  destroy       Destroy previously-created infrastructure

All other commands:
  console       Try Terraform expressions at an interactive command prompt
  fmt           Reformat your configuration in the standard style
  force-unlock  Release a stuck lock on the current workspace
  get           Install or upgrade remote Terraform modules
  graph         Generate a Graphviz graph of the steps in an operation
  import        Associate existing infrastructure with a Terraform resource
  login         Obtain and save credentials for a remote host
  logout        Remove locally-stored credentials for a remote host
  output        Show output values from your root module
  providers     Show the providers required for this configuration
  refresh       Update the state to match remote systems
  show          Show the current state or a saved plan
  state         Advanced state management
  taint         Mark a resource instance as not fully functional
  test          Experimental support for module integration testing
  untaint       Remove the 'tainted' state from a resource instance
  version       Show the current Terraform version
  workspace     Workspace management

Global options (use these before the subcommand, if any):
  -chdir=DIR    Switch to a different working directory before executing the
                given subcommand.
  -help         Show this help output, or the help for a specified subcommand.
  -version      An alias for the "version" subcommand.

# 기본적으로 실습 가상머신에 AWS 계정정보가 잡혀있음
root@workstation:~/hashicat-aws# echo $AWS_ACCESS_KEY_ID
xxxxxxxxxxxxxxx

root@workstation:~/hashicat-aws# echo $AWS_SECRET_ACCESS_KEY
yyyyyyyyyyyyyy

# tf 파일 확인
root@workstation:~/hashicat-aws# ls *.tf
main.tf  outputs.tf  variables.tf

# main.tf 내용 확인
root@workstation:~/hashicat-aws# cat main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "=3.42.0"
    }
  }
}

provider "aws" {
  region  = var.region
}

resource "aws_vpc" "hashicat" {
  cidr_block           = var.address_space
  enable_dns_hostnames = true

  tags = {
    name = "${var.prefix}-vpc-${var.region}"
  }

}

# resource "aws_subnet" "hashicat" {
#   vpc_id     = aws_vpc.hashicat.id
#   cidr_block = var.subnet_prefix

#   tags = {
#     name = "${var.prefix}-subnet"
#   }
# }

# resource "aws_security_group" "hashicat" {
#   name = "${var.prefix}-security-group"

#   vpc_id = aws_vpc.hashicat.id

#   ingress {
#     from_port   = 22
#     to_port     = 22
#     protocol    = "tcp"
#     cidr_blocks = ["0.0.0.0/0"]
#   }

#   ingress {
#     from_port   = 80
#     to_port     = 80
#     protocol    = "tcp"
#     cidr_blocks = ["0.0.0.0/0"]
#   }

#   ingress {
#     from_port   = 443
#     to_port     = 443
#     protocol    = "tcp"
#     cidr_blocks = ["0.0.0.0/0"]
#   }

#   egress {
#     from_port       = 0
#     to_port         = 0
#     protocol        = "-1"
#     cidr_blocks     = ["0.0.0.0/0"]
#     prefix_list_ids = []
#   }

#   tags = {
#     Name = "${var.prefix}-security-group"
#   }
# }

# resource "aws_internet_gateway" "hashicat" {
#   vpc_id = aws_vpc.hashicat.id

#   tags = {
#     Name = "${var.prefix}-internet-gateway"
#   }
# }

# resource "aws_route_table" "hashicat" {
#   vpc_id = aws_vpc.hashicat.id

#   route {
#     cidr_block = "0.0.0.0/0"
#     gateway_id = aws_internet_gateway.hashicat.id
#   }
# }

# resource "aws_route_table_association" "hashicat" {
#   subnet_id      = aws_subnet.hashicat.id
#   route_table_id = aws_route_table.hashicat.id
# }

# data "aws_ami" "ubuntu" {
#   most_recent = true

#   filter {
#     name = "name"
#     #values = ["ubuntu/images/hvm-ssd/ubuntu-disco-19.04-amd64-server-*"]
#     values = ["ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*"]
#   }

#   filter {
#     name   = "virtualization-type"
#     values = ["hvm"]
#   }

#   owners = ["099720109477"] # Canonical
# }

# resource "aws_eip" "hashicat" {
#   instance = aws_instance.hashicat.id
#   vpc      = true
# }

# resource "aws_eip_association" "hashicat" {
#   instance_id   = aws_instance.hashicat.id
#   allocation_id = aws_eip.hashicat.id
# }

# resource "aws_instance" "hashicat" {
#   ami                         = data.aws_ami.ubuntu.id
#   instance_type               = var.instance_type
#   key_name                    = aws_key_pair.hashicat.key_name
#   associate_public_ip_address = true
#   subnet_id                   = aws_subnet.hashicat.id
#   vpc_security_group_ids      = [aws_security_group.hashicat.id]

#   tags = {
#     Name = "${var.prefix}-hashicat-instance"
#   }
# }

# # We're using a little trick here so we can run the provisioner without
# # destroying the VM. Do not do this in production.

# # If you need ongoing management (Day N) of your virtual machines a tool such
# # as Chef or Puppet is a better choice. These tools track the state of
# # individual files and can keep them in the correct configuration.

# # Here we do the following steps:
# # Sync everything in files/ to the remote VM.
# # Set up some environment variables for our script.
# # Add execute permissions to our scripts.
# # Run the deploy_app.sh script.
# resource "null_resource" "configure-cat-app" {
#   depends_on = [aws_eip_association.hashicat]

#   triggers = {
#     build_number = timestamp()
#   }

#   provisioner "file" {
#     source      = "files/"
#     destination = "/home/ubuntu/"

#     connection {
#       type        = "ssh"
#       user        = "ubuntu"
#       private_key = tls_private_key.hashicat.private_key_pem
#       host        = aws_eip.hashicat.public_ip
#     }
#   }

#   provisioner "remote-exec" {
#     inline = [
#       "sudo apt -y update",
#       "sleep 15",
#       "sudo apt -y update",
#       "sudo apt -y install apache2",
#       "sudo systemctl start apache2",
#       "sudo chown -R ubuntu:ubuntu /var/www/html",
#       "chmod +x *.sh",
#       "PLACEHOLDER=${var.placeholder} WIDTH=${var.width} HEIGHT=${var.height} PREFIX=${var.prefix} ./deploy_app.sh",
#     ]

#     connection {
#       type        = "ssh"
#       user        = "ubuntu"
#       private_key = tls_private_key.hashicat.private_key_pem
#       host        = aws_eip.hashicat.public_ip
#     }
#   }
# }

# resource "tls_private_key" "hashicat" {
#   algorithm = "RSA"
# }

# locals {
#   private_key_filename = "${var.prefix}-ssh-key.pem"
# }

# resource "aws_key_pair" "hashicat" {
#   key_name   = local.private_key_filename
#   public_key = tls_private_key.hashicat.public_key_openssh
# }

# Terraform Init - Install the Providers
# We have downloaded some Terraform code for the HashiCat application. 
# We'll be using this source code for the rest of the track.
# Before we can do anything with Terraform we need to initialize our workspace. 
# The 'terraform init' command scans your Terraform code, identifies any providers that are needed, and downloads them.
root@workstation:~/hashicat-aws# terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "3.42.0"...
- Installing hashicorp/aws v3.42.0...
- Installed hashicorp/aws v3.42.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

# 아래의 경로에 레지스트리가 생성됨
# Run the following command to verify that the aws provider was installed under the ".terraform" directory
# This hidden directory is where all modules and plugins are stored.
root@workstation:~/hashicat-aws# ls .terraform/providers/registry.terraform.io/hashicorp
aws

root@workstation:~/hashicat-aws# pwd
/root/hashicat-aws

root@workstation:~/hashicat-aws# ll
total 72
drwxr-xr-x 7 root root  4096 Jan 19 02:20 ./
drwx------ 8 root root  4096 Jan 19 02:21 ../
drwxr-xr-x 2 root root  4096 Jan 19 02:04 .circleci/
drwxr-xr-x 8 root root  4096 Jan 19 02:04 .git/
-rw-r--r-- 1 root root   177 Jan 19 02:04 .gitignore
drwxr-xr-x 3 root root  4096 Jan 19 02:20 .terraform/
-rw-r--r-- 1 root root  1106 Jan 19 02:20 .terraform.lock.hcl
-rw-r--r-- 1 root root 11357 Jan 19 02:04 LICENSE
-rw-r--r-- 1 root root   244 Jan 19 02:04 README.md
drwxr-xr-x 2 root root  4096 Jan 19 02:04 exercises/
drwxr-xr-x 2 root root  4096 Jan 19 02:04 files/
-rw-r--r-- 1 root root  4708 Jan 19 02:04 main.tf
-rw-r--r-- 1 root root    96 Jan 19 02:04 outputs.tf
-rw-r--r-- 1 root root   182 Jan 19 02:04 terraform.tfvars
-rw-r--r-- 1 root root  1578 Jan 19 02:04 variables.tf

# main tf 파일 유효성 검사
# Terraform Validate - Test Your Code
# Terraform comes with a built-in subcommand called validate. 
# This is useful when you want to do a quick syntax check of your code to make sure it parses correctly.
# Edit the main.tf file and remove the double quotes between aws_vpc and hashicat on line 14 of the file, keeping the space that was between them. Save the file.
# Now put the double quotes back in line 14, save the file, and run the validate command again. This time you should pass the validation test.
# terraform validate is most often used in automated CI/CD test pipelines. It allows you to quickly catch errors in your code before any other steps are taken.
root@workstation:~/hashicat-aws# terraform validate
Success! The configuration is valid.

# 테라폼 플랜을 실행하면 아래와 같이 prefix value에 대해서 변수를 임의로 입력하라고 나온다. 그냥 이름으로 입력한다.
root@workstation:~/hashicat-aws# terraform plan
var.prefix
  This prefix will be included in the name of most resources.

  Enter a value: minsupark

# minsupark 입력후 아래와 같이 전시됨
# Terraform Plan - Dry Run Mode
# When you run this command Terraform will prompt you to enter the prefix variable.
# Enter a short string of lower-case letters and/or numbers. 
# We recommend that you use your first and last name.
# The prefix will become part of the name for our VPC, subnet, EC2 instance and other resources.
root@workstation:~/hashicat-aws# terraform plan
var.prefix
  This prefix will be included in the name of most resources.

  Enter a value: minsupark


Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_vpc.hashicat will be created
  + resource "aws_vpc" "hashicat" {
      + arn                              = (known after apply)
      + assign_generated_ipv6_cidr_block = false
      + cidr_block                       = "10.0.0.0/16"
      + default_network_acl_id           = (known after apply)
      + default_route_table_id           = (known after apply)
      + default_security_group_id        = (known after apply)
      + dhcp_options_id                  = (known after apply)
      + enable_classiclink               = (known after apply)
      + enable_classiclink_dns_support   = (known after apply)
      + enable_dns_hostnames             = true
      + enable_dns_support               = true
      + id                               = (known after apply)
      + instance_tenancy                 = "default"
      + ipv6_association_id              = (known after apply)
      + ipv6_cidr_block                  = (known after apply)
      + main_route_table_id              = (known after apply)
      + owner_id                         = (known after apply)
      + tags                             = {
          + "name" = "minsupark-vpc-us-east-1"
        }
      + tags_all                         = {
          + "name" = "minsupark-vpc-us-east-1"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take
exactly these actions if you run "terraform apply" now.

# 또는 아래와 같이 terraform.tfvars 에서 변수를 선언해두는 것도 방법이다.
root@workstation:~/hashicat-aws# cat terraform.tfvars
# Rename or copy this file to terraform.tfvars
# Prefix must be all lowercase letters, digits, and hyphens.
# Make sure it is at least 5 characters long.

#prefix = "yourname"

root@workstation:~/hashicat-aws# vim terraform.tfvars
#prefix = "yourname" 을 prefix = "minsupark"로 수정

root@workstation:~/hashicat-aws# cat terraform.tfvars
# Rename or copy this file to terraform.tfvars
# Prefix must be all lowercase letters, digits, and hyphens.
# Make sure it is at least 5 characters long.

prefix = "minsupark"

# 그리고 테라폼 플랜을 실행하면 변수선언한게 잘 반영된것을 확인할 수 있다.
root@workstation:~/hashicat-aws# terraform plan

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_vpc.hashicat will be created
  + resource "aws_vpc" "hashicat" {
      + arn                              = (known after apply)
      + assign_generated_ipv6_cidr_block = false
      + cidr_block                       = "10.0.0.0/16"
      + default_network_acl_id           = (known after apply)
      + default_route_table_id           = (known after apply)
      + default_security_group_id        = (known after apply)
      + dhcp_options_id                  = (known after apply)
      + enable_classiclink               = (known after apply)
      + enable_classiclink_dns_support   = (known after apply)
      + enable_dns_hostnames             = true
      + enable_dns_support               = true
      + id                               = (known after apply)
      + instance_tenancy                 = "default"
      + ipv6_association_id              = (known after apply)
      + ipv6_cidr_block                  = (known after apply)
      + main_route_table_id              = (known after apply)
      + owner_id                         = (known after apply)
      + tags                             = {
          + "name" = "minsupark-vpc-us-east-1"
        }
      + tags_all                         = {
          + "name" = "minsupark-vpc-us-east-1"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take
exactly these actions if you run "terraform apply" now.

# 이번에는 region 변수를 변경해보자.
root@workstation:~/hashicat-aws# cat terraform.tfvars
# Rename or copy this file to terraform.tfvars
# Prefix must be all lowercase letters, digits, and hyphens.
# Make sure it is at least 5 characters long.

prefix = "minsupark"

root@workstation:~/hashicat-aws# vim terraform.tfvars
가장하단에 region = "us-west-1" 문구를 추가

root@workstation:~/hashicat-aws# cat terraform.tfvars
# Rename or copy this file to terraform.tfvars
# Prefix must be all lowercase letters, digits, and hyphens.
# Make sure it is at least 5 characters long.

prefix = "minsupark"
region = "us-west-1"

# 리전 변수가 us-east-1에서 변경적용한 us-west-1으로 설정된 것을 확인할 수 있다.
root@workstation:~/hashicat-aws# terraform plan

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_vpc.hashicat will be created
  + resource "aws_vpc" "hashicat" {
      + arn                              = (known after apply)
      + assign_generated_ipv6_cidr_block = false
      + cidr_block                       = "10.0.0.0/16"
      + default_network_acl_id           = (known after apply)
      + default_route_table_id           = (known after apply)
      + default_security_group_id        = (known after apply)
      + dhcp_options_id                  = (known after apply)
      + enable_classiclink               = (known after apply)
      + enable_classiclink_dns_support   = (known after apply)
      + enable_dns_hostnames             = true
      + enable_dns_support               = true
      + id                               = (known after apply)
      + instance_tenancy                 = "default"
      + ipv6_association_id              = (known after apply)
      + ipv6_cidr_block                  = (known after apply)
      + main_route_table_id              = (known after apply)
      + owner_id                         = (known after apply)
      + tags                             = {
          + "name" = "minsupark-vpc-us-west-1"
        }
      + tags_all                         = {
          + "name" = "minsupark-vpc-us-west-1"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take
exactly these actions if you run "terraform apply" now.

root@workstation:~/hashicat-aws# terraform graph
digraph {
        compound = "true"
        newrank = "true"
        subgraph "root" {
                "[root] aws_vpc.hashicat (expand)" [label = "aws_vpc.hashicat", shape = "box"]
                "[root] provider[\"registry.terraform.io/hashicorp/aws\"]" [label = "provider[\"registry.terraform.io/hashicorp/aws\"]", shape = "diamond"]
                "[root] var.address_space" [label = "var.address_space", shape = "note"]
                "[root] var.admin_username" [label = "var.admin_username", shape = "note"]
                "[root] var.height" [label = "var.height", shape = "note"]
                "[root] var.instance_type" [label = "var.instance_type", shape = "note"]
                "[root] var.placeholder" [label = "var.placeholder", shape = "note"]
                "[root] var.prefix" [label = "var.prefix", shape = "note"]
                "[root] var.region" [label = "var.region", shape = "note"]
                "[root] var.subnet_prefix" [label = "var.subnet_prefix", shape = "note"]
                "[root] var.width" [label = "var.width", shape = "note"]
                "[root] aws_vpc.hashicat (expand)" -> "[root] provider[\"registry.terraform.io/hashicorp/aws\"]"
                "[root] aws_vpc.hashicat (expand)" -> "[root] var.address_space"
                "[root] aws_vpc.hashicat (expand)" -> "[root] var.prefix"
                "[root] meta.count-boundary (EachMode fixup)" -> "[root] aws_vpc.hashicat (expand)"
                "[root] meta.count-boundary (EachMode fixup)" -> "[root] var.admin_username"
                "[root] meta.count-boundary (EachMode fixup)" -> "[root] var.height"
                "[root] meta.count-boundary (EachMode fixup)" -> "[root] var.instance_type"
                "[root] meta.count-boundary (EachMode fixup)" -> "[root] var.placeholder"
                "[root] meta.count-boundary (EachMode fixup)" -> "[root] var.subnet_prefix"
                "[root] meta.count-boundary (EachMode fixup)" -> "[root] var.width"
                "[root] provider[\"registry.terraform.io/hashicorp/aws\"] (close)" -> "[root]aws_vpc.hashicat (expand)"
                "[root] provider[\"registry.terraform.io/hashicorp/aws\"]" -> "[root] var.region"
                "[root] root" -> "[root] meta.count-boundary (EachMode fixup)"
                "[root] root" -> "[root] provider[\"registry.terraform.io/hashicorp/aws\"] (close)"
        }
}

# blast-radius 를 실행하면 그래프뷰로 그림으로 볼 수 있다.
root@workstation:~/hashicat-aws# blast-radius --serve .
 * Serving Flask app 'blastradius.server.server' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://10.132.1.84:5000/ (Press CTRL+C to quit)
```

<img width="730" alt="1" src="https://user-images.githubusercontent.com/41605276/150125709-df60bdd8-19ab-4c6d-9373-87eb4a48ab29.png">

```console
root@workstation:~/hashicat-aws# terraform plan

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_vpc.hashicat will be created
  + resource "aws_vpc" "hashicat" {
      + arn                              = (known after apply)
      + assign_generated_ipv6_cidr_block = false
      + cidr_block                       = "10.0.0.0/16"
      + default_network_acl_id           = (known after apply)
      + default_route_table_id           = (known after apply)
      + default_security_group_id        = (known after apply)
      + dhcp_options_id                  = (known after apply)
      + enable_classiclink               = (known after apply)
      + enable_classiclink_dns_support   = (known after apply)
      + enable_dns_hostnames             = true
      + enable_dns_support               = true
      + id                               = (known after apply)
      + instance_tenancy                 = "default"
      + ipv6_association_id              = (known after apply)
      + ipv6_cidr_block                  = (known after apply)
      + main_route_table_id              = (known after apply)
      + owner_id                         = (known after apply)
      + tags                             = {
          + "name" = "minsupark-vpc-us-west-1"
        }
      + tags_all                         = {
          + "name" = "minsupark-vpc-us-west-1"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take
exactly these actions if you run "terraform apply" now.

root@workstation:~/hashicat-aws# terraform apply

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_vpc.hashicat will be created
  + resource "aws_vpc" "hashicat" {
      + arn                              = (known after apply)
      + assign_generated_ipv6_cidr_block = false
      + cidr_block                       = "10.0.0.0/16"
      + default_network_acl_id           = (known after apply)
      + default_route_table_id           = (known after apply)
      + default_security_group_id        = (known after apply)
      + dhcp_options_id                  = (known after apply)
      + enable_classiclink               = (known after apply)
      + enable_classiclink_dns_support   = (known after apply)
      + enable_dns_hostnames             = true
      + enable_dns_support               = true
      + id                               = (known after apply)
      + instance_tenancy                 = "default"
      + ipv6_association_id              = (known after apply)
      + ipv6_cidr_block                  = (known after apply)
      + main_route_table_id              = (known after apply)
      + owner_id                         = (known after apply)
      + tags                             = {
          + "name" = "minsupark-vpc-us-west-1"
        }
      + tags_all                         = {
          + "name" = "minsupark-vpc-us-west-1"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_vpc.hashicat: Creating...
aws_vpc.hashicat: Still creating... [10s elapsed]
aws_vpc.hashicat: Creation complete after 19s [id=vpc-0b2ad272619c62ae9]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

# Test and Repair
# Try running terraform plan again and see what happens
# Since your VPC has already been built, Terraform will report that there are no changes required.
# This is normal and expected.
root@workstation:~/hashicat-aws# terraform plan
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no
differences, so no changes are needed.

# Now try running another apply
# Terraform checks each resource to ensure it is in the proper state. 
# It will not re-create your VPC if it is already provisioned correctly.
root@workstation:~/hashicat-aws# terraform apply
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no
differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

# 프리픽스 변수를 다시 바꿔서 진행해보자.
# Change Your Prefix
# Edit the terraform.tfvars file to change your prefix. 
# You could simply add a number to the end if you like. Or change it to something entirely new.
# Save the terraform.tfvars file and run your apply command again.
root@workstation:~/hashicat-aws# cat terraform.tfvars
# Rename or copy this file to terraform.tfvars
# Prefix must be all lowercase letters, digits, and hyphens.
# Make sure it is at least 5 characters long.

prefix = "minsupark"
region = "us-west-1"

root@workstation:~/hashicat-aws# vim terraform.tfvars
프리픽스 변수를 minsupark 에서 minman으로 변경

root@workstation:~/hashicat-aws# cat terraform.tfvars
# Rename or copy this file to terraform.tfvars
# Prefix must be all lowercase letters, digits, and hyphens.
# Make sure it is at least 5 characters long.

prefix = "minman"
region = "us-west-1"

# 아래와 같이 테그 네임이 변경된 것을 확인할 수 있다.
root@workstation:~/hashicat-aws# terraform apply
aws_vpc.hashicat: Refreshing state... [id=vpc-0b6ede93432252507]

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # aws_vpc.hashicat will be updated in-place
  ~ resource "aws_vpc" "hashicat" {
        id                               = "vpc-0b6ede93432252507"
      ~ tags                             = {
          ~ "name" = "minsupark-vpc-us-west-1" -> "minsupark2-vpc-us-west-1"
        }
      ~ tags_all                         = {
          ~ "name" = "minsupark-vpc-us-west-1" -> "minsupark2-vpc-us-west-1"
        }
        # (14 unchanged attributes hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_vpc.hashicat: Modifying... [id=vpc-0b6ede93432252507]
aws_vpc.hashicat: Modifications complete after 6s [id=vpc-0b6ede93432252507]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.




# Add a Tag to Your VPC
# Several resource types support AWS's tags. 
# Read the Terraform documentation for the aws_vpc resource to learn about the tags argument and its format.
# Terraform Docs: the aws_vpc resource - Click Here
# Add a tag to your VPC resource in the main.tf file. 
# The name of the tag should be environment and the value should be Production. 
# (The tag is case-sensitive. Make sure you use a capital P.)
# Hint: Read the examples carefully! 
# Unlike the other resource arguments you've seen, the value of the tags argument must be a map (a {key = "value"} data structure). 
# Additionally, you must use = after tags and after environment. 
# For more about Terraform's data types, see the Terraform language documentation.
# Re-run terraform apply.
# What happens?
root@workstation:~/hashicat-aws# vim main.tf
...

resource "aws_vpc" "hashicat" {
  cidr_block           = var.address_space
  enable_dns_hostnames = true

  tags = {
    name = "${var.prefix}-vpc-${var.region}"
    environment = "Production"            <---- 요거를 추가
  }

}

...

root@workstation:~/hashicat-aws# terraform apply
aws_vpc.hashicat: Refreshing state... [id=vpc-0b6ede93432252507]

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # aws_vpc.hashicat will be updated in-place
  ~ resource "aws_vpc" "hashicat" {
        id                               = "vpc-0b6ede93432252507"
      ~ tags                             = {
          + "environment" = "Production"
            # (1 unchanged element hidden)
        }
      ~ tags_all                         = {
          + "environment" = "Production"
            # (1 unchanged element hidden)
        }
        # (14 unchanged attributes hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_vpc.hashicat: Modifying... [id=vpc-0b6ede93432252507]
aws_vpc.hashicat: Modifications complete after 7s [id=vpc-0b6ede93432252507]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.




#Add a Virtual Network
# Open the main.tf file again and uncomment the next resource block in the file. 
# The type of resource is aws_subnet and it is named hashicat.
#Uncomment the code by removing the # characters from the beginning of each line. Be sure to save the file.
# Now run terraform apply again. Observe the results.
#Look at the vpc_id parameter inside the aws_subnet resource. 
#See how it points back at the first resource in the file? The subnet resource inherits settings from the VPC.
#Terraform can map out the complex web of dependencies between hundreds of interconnected resources.

root@workstation:~/hashicat-aws# vim main.tf
# 아래 내용과 같이 모든 주석을 해제해준다.
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "=3.42.0"
    }
  }
}

provider "aws" {
  region  = var.region
}

resource "aws_vpc" "hashicat" {
  cidr_block           = var.address_space
  enable_dns_hostnames = true

  tags = {
    name = "${var.prefix}-vpc-${var.region}"
    environment = "Production"
  }

}

 resource "aws_subnet" "hashicat" {
   vpc_id     = aws_vpc.hashicat.id
   cidr_block = var.subnet_prefix

   tags = {
     name = "${var.prefix}-subnet"
   }
 }

 resource "aws_security_group" "hashicat" {
   name = "${var.prefix}-security-group"

   vpc_id = aws_vpc.hashicat.id

   ingress {
     from_port   = 22
     to_port     = 22
     protocol    = "tcp"
     cidr_blocks = ["0.0.0.0/0"]
   }

   ingress {
     from_port   = 80
     to_port     = 80
     protocol    = "tcp"
     cidr_blocks = ["0.0.0.0/0"]
   }

   ingress {
     from_port   = 443
     to_port     = 443
     protocol    = "tcp"
     cidr_blocks = ["0.0.0.0/0"]
   }

   egress {
     from_port       = 0
     to_port         = 0
     protocol        = "-1"
     cidr_blocks     = ["0.0.0.0/0"]
     prefix_list_ids = []
   }

   tags = {
     Name = "${var.prefix}-security-group"
   }
 }

 resource "aws_internet_gateway" "hashicat" {
   vpc_id = aws_vpc.hashicat.id

   tags = {
     Name = "${var.prefix}-internet-gateway"
   }
 }

 resource "aws_route_table" "hashicat" {
   vpc_id = aws_vpc.hashicat.id

   route {
     cidr_block = "0.0.0.0/0"
     gateway_id = aws_internet_gateway.hashicat.id
   }
 }

 resource "aws_route_table_association" "hashicat" {
   subnet_id      = aws_subnet.hashicat.id
   route_table_id = aws_route_table.hashicat.id
 }

 data "aws_ami" "ubuntu" {
   most_recent = true

   filter {
     name = "name"
     #values = ["ubuntu/images/hvm-ssd/ubuntu-disco-19.04-amd64-server-*"]
     values = ["ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*"]
   }

   filter {
     name   = "virtualization-type"
     values = ["hvm"]
   }

   owners = ["099720109477"] # Canonical
 }

 resource "aws_eip" "hashicat" {
   instance = aws_instance.hashicat.id
   vpc      = true
 }

 resource "aws_eip_association" "hashicat" {
   instance_id   = aws_instance.hashicat.id
   allocation_id = aws_eip.hashicat.id
 }

 resource "aws_instance" "hashicat" {
   ami                         = data.aws_ami.ubuntu.id
   instance_type               = var.instance_type
   key_name                    = aws_key_pair.hashicat.key_name
   associate_public_ip_address = true
   subnet_id                   = aws_subnet.hashicat.id
   vpc_security_group_ids      = [aws_security_group.hashicat.id]

   tags = {
     Name = "${var.prefix}-hashicat-instance"
   }
 }

# # We're using a little trick here so we can run the provisioner without
# # destroying the VM. Do not do this in production.

# # If you need ongoing management (Day N) of your virtual machines a tool such
# # as Chef or Puppet is a better choice. These tools track the state of
# # individual files and can keep them in the correct configuration.

# # Here we do the following steps:
# # Sync everything in files/ to the remote VM.
# # Set up some environment variables for our script.
# # Add execute permissions to our scripts.
# # Run the deploy_app.sh script.
 resource "null_resource" "configure-cat-app" {
   depends_on = [aws_eip_association.hashicat]

   triggers = {
     build_number = timestamp()
   }

   provisioner "file" {
     source      = "files/"
     destination = "/home/ubuntu/"

     connection {
       type        = "ssh"
       user        = "ubuntu"
       private_key = tls_private_key.hashicat.private_key_pem
       host        = aws_eip.hashicat.public_ip
     }
   }

   provisioner "remote-exec" {
     inline = [
       "sudo apt -y update",
       "sleep 15",
       "sudo apt -y update",
       "sudo apt -y install apache2",
       "sudo systemctl start apache2",
       "sudo chown -R ubuntu:ubuntu /var/www/html",
       "chmod +x *.sh",
       "PLACEHOLDER=${var.placeholder} WIDTH=${var.width} HEIGHT=${var.height} PREFIX=${var.prefix} ./deploy_app.sh",
     ]

     connection {
       type        = "ssh"
       user        = "ubuntu"
       private_key = tls_private_key.hashicat.private_key_pem
       host        = aws_eip.hashicat.public_ip
     }
   }
 }

 resource "tls_private_key" "hashicat" {
   algorithm = "RSA"
 }

 locals {
   private_key_filename = "${var.prefix}-ssh-key.pem"
 }

 resource "aws_key_pair" "hashicat" {
   key_name   = local.private_key_filename
   public_key = tls_private_key.hashicat.public_key_openssh
 }



# main.tf 변경후에 다시 terraform apply를 해보자
root@workstation:~/hashicat-aws# terraform init

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file
- Finding latest version of hashicorp/null...
- Finding latest version of hashicorp/tls...
- Installing hashicorp/null v3.1.0...
- Installed hashicorp/null v3.1.0 (signed by HashiCorp)
- Installing hashicorp/tls v3.1.0...
- Installed hashicorp/tls v3.1.0 (signed by HashiCorp)
- Using previously-installed hashicorp/aws v3.42.0

Terraform has made some changes to the provider dependency selections recorded
in the .terraform.lock.hcl file. Review those changes and commit them to your
version control system if they represent changes you intended to make.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.




root@workstation:~/hashicat-aws# terraform plan
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_eip.hashicat will be created
  + resource "aws_eip" "hashicat" {
      + allocation_id        = (known after apply)
      + association_id       = (known after apply)
      + carrier_ip           = (known after apply)
      + customer_owned_ip    = (known after apply)
      + domain               = (known after apply)
      + id                   = (known after apply)
      + instance             = (known after apply)
      + network_border_group = (known after apply)
      + network_interface    = (known after apply)
      + private_dns          = (known after apply)
      + private_ip           = (known after apply)
      + public_dns           = (known after apply)
      + public_ip            = (known after apply)
      + public_ipv4_pool     = (known after apply)
      + tags_all             = (known after apply)
      + vpc                  = true
    }

  # aws_eip_association.hashicat will be created
  + resource "aws_eip_association" "hashicat" {
      + allocation_id        = (known after apply)
      + id                   = (known after apply)
      + instance_id          = (known after apply)
      + network_interface_id = (known after apply)
      + private_ip_address   = (known after apply)
      + public_ip            = (known after apply)
    }

  # aws_instance.hashicat will be created
  + resource "aws_instance" "hashicat" {
      + ami                                  = "ami-00753573f3369dd7c"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = true
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = "minman-ssh-key.pem"
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Name" = "minman-hashicat-instance"
        }
      + tags_all                             = {
          + "Name" = "minman-hashicat-instance"
        }
      + tenancy                              = (known after apply)
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification {
          + capacity_reservation_preference = (known after apply)

          + capacity_reservation_target {
              + capacity_reservation_id = (known after apply)
            }
        }

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + snapshot_id           = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + enclave_options {
          + enabled = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_interface_id  = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }
    }

  # aws_internet_gateway.hashicat will be created
  + resource "aws_internet_gateway" "hashicat" {
      + arn      = (known after apply)
      + id       = (known after apply)
      + owner_id = (known after apply)
      + tags     = {
          + "Name" = "minman-internet-gateway"
        }
      + tags_all = {
          + "Name" = "minman-internet-gateway"
        }
      + vpc_id   = "vpc-0b2ad272619c62ae9"
    }

  # aws_key_pair.hashicat will be created
  + resource "aws_key_pair" "hashicat" {
      + arn         = (known after apply)
      + fingerprint = (known after apply)
      + id          = (known after apply)
      + key_name    = "minman-ssh-key.pem"
      + key_pair_id = (known after apply)
      + public_key  = (known after apply)
      + tags_all    = (known after apply)
    }

  # aws_route_table.hashicat will be created
  + resource "aws_route_table" "hashicat" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = [
          + {
              + carrier_gateway_id         = ""
              + cidr_block                 = "0.0.0.0/0"
              + destination_prefix_list_id = ""
              + egress_only_gateway_id     = ""
              + gateway_id                 = (known after apply)
              + instance_id                = ""
              + ipv6_cidr_block            = ""
              + local_gateway_id           = ""
              + nat_gateway_id             = ""
              + network_interface_id       = ""
              + transit_gateway_id         = ""
              + vpc_endpoint_id            = ""
              + vpc_peering_connection_id  = ""
            },
        ]
      + tags_all         = (known after apply)
      + vpc_id           = "vpc-0b2ad272619c62ae9"
    }

  # aws_route_table_association.hashicat will be created
  + resource "aws_route_table_association" "hashicat" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # aws_security_group.hashicat will be created
  + resource "aws_security_group" "hashicat" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 0
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "-1"
              + security_groups  = []
              + self             = false
              + to_port          = 0
            },
        ]
      + id                     = (known after apply)
      + ingress                = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 22
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 22
            },
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 443
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 443
            },
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 80
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 80
            },
        ]
      + name                   = "minman-security-group"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags                   = {
          + "Name" = "minman-security-group"
        }
      + tags_all               = {
          + "Name" = "minman-security-group"
        }
      + vpc_id                 = "vpc-0b2ad272619c62ae9"
    }

  # aws_subnet.hashicat will be created
  + resource "aws_subnet" "hashicat" {
      + arn                             = (known after apply)
      + assign_ipv6_address_on_creation = false
      + availability_zone               = (known after apply)
      + availability_zone_id            = (known after apply)
      + cidr_block                      = "10.0.10.0/24"
      + id                              = (known after apply)
      + ipv6_cidr_block_association_id  = (known after apply)
      + map_public_ip_on_launch         = false
      + owner_id                        = (known after apply)
      + tags                            = {
          + "name" = "minman-subnet"
        }
      + tags_all                        = {
          + "name" = "minman-subnet"
        }
      + vpc_id                          = "vpc-0b2ad272619c62ae9"
    }

  # null_resource.configure-cat-app will be created
  + resource "null_resource" "configure-cat-app" {
      + id       = (known after apply)
      + triggers = (known after apply)
    }

  # tls_private_key.hashicat will be created
  + resource "tls_private_key" "hashicat" {
      + algorithm                  = "RSA"
      + ecdsa_curve                = "P224"
      + id                         = (known after apply)
      + private_key_pem            = (sensitive value)
      + public_key_fingerprint_md5 = (known after apply)
      + public_key_openssh         = (known after apply)
      + public_key_pem             = (known after apply)
      + rsa_bits                   = 2048
    }

Plan: 11 to add, 0 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee
to take exactly these actions if you run "terraform apply" now.



root@workstation:~/hashicat-aws# terraform apply
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_eip.hashicat will be created
  + resource "aws_eip" "hashicat" {
      + allocation_id        = (known after apply)
      + association_id       = (known after apply)
      + carrier_ip           = (known after apply)
      + customer_owned_ip    = (known after apply)
      + domain               = (known after apply)
      + id                   = (known after apply)
      + instance             = (known after apply)
      + network_border_group = (known after apply)
      + network_interface    = (known after apply)
      + private_dns          = (known after apply)
      + private_ip           = (known after apply)
      + public_dns           = (known after apply)
      + public_ip            = (known after apply)
      + public_ipv4_pool     = (known after apply)
      + tags_all             = (known after apply)
      + vpc                  = true
    }

  # aws_eip_association.hashicat will be created
  + resource "aws_eip_association" "hashicat" {
      + allocation_id        = (known after apply)
      + id                   = (known after apply)
      + instance_id          = (known after apply)
      + network_interface_id = (known after apply)
      + private_ip_address   = (known after apply)
      + public_ip            = (known after apply)
    }

  # aws_instance.hashicat will be created
  + resource "aws_instance" "hashicat" {
      + ami                                  = "ami-00753573f3369dd7c"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = true
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = "minman-ssh-key.pem"
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Name" = "minman-hashicat-instance"
        }
      + tags_all                             = {
          + "Name" = "minman-hashicat-instance"
        }
      + tenancy                              = (known after apply)
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification {
          + capacity_reservation_preference = (known after apply)

          + capacity_reservation_target {
              + capacity_reservation_id = (known after apply)
            }
        }

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + snapshot_id           = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + enclave_options {
          + enabled = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_interface_id  = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }
    }

  # aws_internet_gateway.hashicat will be created
  + resource "aws_internet_gateway" "hashicat" {
      + arn      = (known after apply)
      + id       = (known after apply)
      + owner_id = (known after apply)
      + tags     = {
          + "Name" = "minman-internet-gateway"
        }
      + tags_all = {
          + "Name" = "minman-internet-gateway"
        }
      + vpc_id   = "vpc-0b2ad272619c62ae9"
    }

  # aws_key_pair.hashicat will be created
  + resource "aws_key_pair" "hashicat" {
      + arn         = (known after apply)
      + fingerprint = (known after apply)
      + id          = (known after apply)
      + key_name    = "minman-ssh-key.pem"
      + key_pair_id = (known after apply)
      + public_key  = (known after apply)
      + tags_all    = (known after apply)
    }

  # aws_route_table.hashicat will be created
  + resource "aws_route_table" "hashicat" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = [
          + {
              + carrier_gateway_id         = ""
              + cidr_block                 = "0.0.0.0/0"
              + destination_prefix_list_id = ""
              + egress_only_gateway_id     = ""
              + gateway_id                 = (known after apply)
              + instance_id                = ""
              + ipv6_cidr_block            = ""
              + local_gateway_id           = ""
              + nat_gateway_id             = ""
              + network_interface_id       = ""
              + transit_gateway_id         = ""
              + vpc_endpoint_id            = ""
              + vpc_peering_connection_id  = ""
            },
        ]
      + tags_all         = (known after apply)
      + vpc_id           = "vpc-0b2ad272619c62ae9"
    }

  # aws_route_table_association.hashicat will be created
  + resource "aws_route_table_association" "hashicat" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # aws_security_group.hashicat will be created
  + resource "aws_security_group" "hashicat" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 0
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "-1"
              + security_groups  = []
              + self             = false
              + to_port          = 0
            },
        ]
      + id                     = (known after apply)
      + ingress                = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 22
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 22
            },
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 443
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 443
            },
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 80
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 80
            },
        ]
      + name                   = "minman-security-group"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags                   = {
          + "Name" = "minman-security-group"
        }
      + tags_all               = {
          + "Name" = "minman-security-group"
        }
      + vpc_id                 = "vpc-0b2ad272619c62ae9"
    }

  # aws_subnet.hashicat will be created
  + resource "aws_subnet" "hashicat" {
      + arn                             = (known after apply)
      + assign_ipv6_address_on_creation = false
      + availability_zone               = (known after apply)
      + availability_zone_id            = (known after apply)
      + cidr_block                      = "10.0.10.0/24"
      + id                              = (known after apply)
      + ipv6_cidr_block_association_id  = (known after apply)
      + map_public_ip_on_launch         = false
      + owner_id                        = (known after apply)
      + tags                            = {
          + "name" = "minman-subnet"
        }
      + tags_all                        = {
          + "name" = "minman-subnet"
        }
      + vpc_id                          = "vpc-0b2ad272619c62ae9"
    }

  # null_resource.configure-cat-app will be created
  + resource "null_resource" "configure-cat-app" {
      + id       = (known after apply)
      + triggers = (known after apply)
    }

  # tls_private_key.hashicat will be created
  + resource "tls_private_key" "hashicat" {
      + algorithm                  = "RSA"
      + ecdsa_curve                = "P224"
      + id                         = (known after apply)
      + private_key_pem            = (sensitive value)
      + public_key_fingerprint_md5 = (known after apply)
      + public_key_openssh         = (known after apply)
      + public_key_pem             = (known after apply)
      + rsa_bits                   = 2048
    }

Plan: 11 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

tls_private_key.hashicat: Creating...
tls_private_key.hashicat: Creation complete after 1s [id=976fc16e7f3ba91c3f6e699d88ae61418dd3af20]
aws_key_pair.hashicat: Creating...
aws_security_group.hashicat: Creating...
aws_internet_gateway.hashicat: Creating...
aws_subnet.hashicat: Creating...
aws_key_pair.hashicat: Creation complete after 1s [id=minman-ssh-key.pem]
aws_subnet.hashicat: Creation complete after 2s [id=subnet-03be28a3358e39958]
aws_internet_gateway.hashicat: Creation complete after 3s [id=igw-0c5db75b6e4598e1a]
aws_route_table.hashicat: Creating...
aws_route_table.hashicat: Creation complete after 3s [id=rtb-05d15616067d71b7b]
aws_route_table_association.hashicat: Creating...
aws_security_group.hashicat: Creation complete after 6s [id=sg-0512901f8d650cc19]
aws_instance.hashicat: Creating...
aws_route_table_association.hashicat: Creation complete after 1s [id=rtbassoc-07db145cb4684c3f5]
aws_instance.hashicat: Still creating... [10s elapsed]
aws_instance.hashicat: Still creating... [20s elapsed]
aws_instance.hashicat: Still creating... [30s elapsed]
aws_instance.hashicat: Creation complete after 38s [id=i-02291afd8a0a4cfc1]
aws_eip.hashicat: Creating...

...

ull_resource.configure-cat-app (remote-exec): Enabling conf serve-cgi-bin.
null_resource.configure-cat-app (remote-exec): Enabling site 000-default.
null_resource.configure-cat-app (remote-exec): Created symlink /etc/systemd/system/multi-user.target.wants/apache2.service → /lib/systemd/system/apache2.service.
null_resource.configure-cat-app (remote-exec): Created symlink /etc/systemd/system/multi-user.target.wants/apache-htcacheclean.service → /lib/systemd/system/apache-htcacheclean.service.
Progress: [ 98%] [#################.] e-exec): Processing triggers for libc-bin (2.27-3ubuntu1.4) ...
null_resource.configure-cat-app (remote-exec): Processing triggers for systemd (237-3ubuntu10.53) ...
null_resource.configure-cat-app (remote-exec): Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
null_resource.configure-cat-app (remote-exec): Processing triggers for ufw (0.36-0ubuntu0.18.04.2) ...
null_resource.configure-cat-app (remote-exec): Processing triggers for ureadahead (0.100.0-21) ...

null_resource.configure-cat-app: Still creating... [50s elapsed]
null_resource.configure-cat-app (remote-exec): Script complete.
null_resource.configure-cat-app: Creation complete after 51s [id=6642248633071939512]

Apply complete! Resources: 11 added, 0 changed, 0 destroyed.





# Complete the Build
# We've uncommented all the rest of the Terraform code in the main.tf file for you. Run a terraform plan to see what will be built
root@workstation:~/hashicat-aws# terraform plan
tls_private_key.hashicat: Refreshing state... [id=976fc16e7f3ba91c3f6e699d88ae61418dd3af20]
aws_key_pair.hashicat: Refreshing state... [id=minman-ssh-key.pem]
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]
aws_subnet.hashicat: Refreshing state... [id=subnet-03be28a3358e39958]
aws_security_group.hashicat: Refreshing state... [id=sg-0512901f8d650cc19]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-0c5db75b6e4598e1a]
aws_route_table.hashicat: Refreshing state... [id=rtb-05d15616067d71b7b]
aws_instance.hashicat: Refreshing state... [id=i-02291afd8a0a4cfc1]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-07db145cb4684c3f5]
aws_eip.hashicat: Refreshing state... [id=eipalloc-09b8550bcd02d005d]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-0e4db23a59f475b5c]
null_resource.configure-cat-app: Refreshing state... [id=6642248633071939512]

Note: Objects have changed outside of Terraform

Terraform detected the following changes made outside of Terraform since the last "terraform
apply":

  # aws_key_pair.hashicat has been changed
  ~ resource "aws_key_pair" "hashicat" {
        id          = "minman-ssh-key.pem"
      + tags        = {}
        # (6 unchanged attributes hidden)
    }
  # aws_instance.hashicat has been changed
  ~ resource "aws_instance" "hashicat" {
        id                                   = "i-02291afd8a0a4cfc1"
      ~ public_dns                           = "ec2-54-183-42-20.us-west-1.compute.amazonaws.com" -> "ec2-54-219-4-25.us-west-1.compute.amazonaws.com"
      ~ public_ip                            = "54.183.42.20" -> "54.219.4.25"
        tags                                 = {
            "Name" = "minman-hashicat-instance"
        }
        # (27 unchanged attributes hidden)





        # (5 unchanged blocks hidden)
    }
  # aws_route_table.hashicat has been changed
  ~ resource "aws_route_table" "hashicat" {
        id               = "rtb-05d15616067d71b7b"
      + tags             = {}
        # (6 unchanged attributes hidden)
    }
  # aws_eip.hashicat has been changed
  ~ resource "aws_eip" "hashicat" {
        id                   = "eipalloc-09b8550bcd02d005d"
      + tags                 = {}
        # (12 unchanged attributes hidden)
    }

Unless you have made equivalent changes to your configuration, or ignored the relevant
attributes using ignore_changes, the following plan may include actions to undo or respond to
these changes.

─────────────────────────────────────────────────────────────────────────────────────────────

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # null_resource.configure-cat-app must be replaced
-/+ resource "null_resource" "configure-cat-app" {
      ~ id       = "6642248633071939512" -> (known after apply)
      ~ triggers = {
          - "build_number" = "2022-01-19T03:20:59Z"
        } -> (known after apply) # forces replacement
    }

Plan: 1 to add, 0 to change, 1 to destroy.

Changes to Outputs:
  + catapp_url = "http://ec2-54-219-4-25.us-west-1.compute.amazonaws.com"

─────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take
exactly these actions if you run "terraform apply" now.

# Now run an apply to build the HashiCat application
# terraform apply -auto-approve
# It can take up to five minutes for the application to finish deploying. 
# You will know it is complete when you see the Terraform output with your application URL at the end of the run.
#If the application won't load just run terraform apply again. 
# This will attempt to reinstall the webserver and start your application if it's not running.
#Open your web application in a new browser tab by clicking on the URL in the catapp_url output.
root@workstation:~/hashicat-aws# terraform apply -auto-approve
tls_private_key.hashicat: Refreshing state... [id=976fc16e7f3ba91c3f6e699d88ae61418dd3af20]
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]
aws_key_pair.hashicat: Refreshing state... [id=minman-ssh-key.pem]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-0c5db75b6e4598e1a]
aws_security_group.hashicat: Refreshing state... [id=sg-0512901f8d650cc19]
aws_subnet.hashicat: Refreshing state... [id=subnet-03be28a3358e39958]
aws_route_table.hashicat: Refreshing state... [id=rtb-05d15616067d71b7b]
aws_instance.hashicat: Refreshing state... [id=i-02291afd8a0a4cfc1]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-07db145cb4684c3f5]
aws_eip.hashicat: Refreshing state... [id=eipalloc-09b8550bcd02d005d]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-0e4db23a59f475b5c]
null_resource.configure-cat-app: Refreshing state... [id=6642248633071939512]

Note: Objects have changed outside of Terraform

Terraform detected the following changes made outside of Terraform since the last "terraform
apply":

  # aws_instance.hashicat has been changed
  ~ resource "aws_instance" "hashicat" {
        id                                   = "i-02291afd8a0a4cfc1"
      ~ public_dns                           = "ec2-54-183-42-20.us-west-1.compute.amazonaws.com" -> "ec2-54-219-4-25.us-west-1.compute.amazonaws.com"
      ~ public_ip                            = "54.183.42.20" -> "54.219.4.25"
        tags                                 = {
            "Name" = "minman-hashicat-instance"
        }
        # (27 unchanged attributes hidden)





        # (5 unchanged blocks hidden)
    }
  # aws_key_pair.hashicat has been changed
  ~ resource "aws_key_pair" "hashicat" {
        id          = "minman-ssh-key.pem"
      + tags        = {}
        # (6 unchanged attributes hidden)
    }
  # aws_route_table.hashicat has been changed
  ~ resource "aws_route_table" "hashicat" {
        id               = "rtb-05d15616067d71b7b"
      + tags             = {}
        # (6 unchanged attributes hidden)
    }
  # aws_eip.hashicat has been changed
  ~ resource "aws_eip" "hashicat" {
        id                   = "eipalloc-09b8550bcd02d005d"
      + tags                 = {}
        # (12 unchanged attributes hidden)
    }

Unless you have made equivalent changes to your configuration, or ignored the relevant
attributes using ignore_changes, the following plan may include actions to undo or respond to
these changes.

─────────────────────────────────────────────────────────────────────────────────────────────

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # null_resource.configure-cat-app must be replaced
-/+ resource "null_resource" "configure-cat-app" {
      ~ id       = "6642248633071939512" -> (known after apply)
      ~ triggers = {
          - "build_number" = "2022-01-19T03:20:59Z"
        } -> (known after apply) # forces replacement
    }

Plan: 1 to add, 0 to change, 1 to destroy.

Changes to Outputs:
  + catapp_url = "http://ec2-54-219-4-25.us-west-1.compute.amazonaws.com"
null_resource.configure-cat-app: Destroying... [id=6642248633071939512]
null_resource.configure-cat-app: Destruction complete after 0s
null_resource.configure-cat-app: Creating...
null_resource.configure-cat-app: Provisioning with 'file'...
null_resource.configure-cat-app: Provisioning with 'remote-exec'...
null_resource.configure-cat-app (remote-exec): Connecting to remote host via SSH...
null_resource.configure-cat-app (remote-exec):   Host: 54.219.4.25
null_resource.configure-cat-app (remote-exec):   User: ubuntu
null_resource.configure-cat-app (remote-exec):   Password: false
null_resource.configure-cat-app (remote-exec):   Private key: true
null_resource.configure-cat-app (remote-exec):   Certificate: false

...

null_resource.configure-cat-app (remote-exec): Reading state information... 0%
null_resource.configure-cat-app (remote-exec): Reading state information... Done
null_resource.configure-cat-app (remote-exec): apache2 is already the newest version (2.4.29-1ubuntu4.21).
null_resource.configure-cat-app (remote-exec): 0 upgraded, 0 newly installed, 0 to remove and0 not upgraded.
null_resource.configure-cat-app (remote-exec): Script complete.
null_resource.configure-cat-app: Creation complete after 26s [id=4208660317826343136]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.

Outputs:

catapp_url = "http://ec2-54-219-4-25.us-west-1.compute.amazonaws.com"
```

위에 http://ec2-54-219-4-25.us-west-1.compute.amazonaws.com 을 클릭해서 접속하면 아래와 같은 화면을 확인할 수 있다.

<img width="967" alt="2" src="https://user-images.githubusercontent.com/41605276/150126249-3c5977fc-2047-4b1c-b8c6-2fe69066c3c5.png">


```console
root@workstation:~/hashicat-aws# vim main.tf

...

provisioner "remote-exec" {
    inline = [
      "sudo apt -y update",
      "sleep 15",
      "sudo apt -y update",
      "sudo apt -y install apache2",
      "sudo systemctl start apache2",
      "sudo chown -R ubuntu:ubuntu /var/www/html",
      "chmod +x *.sh",
      "PLACEHOLDER=${var.placeholder} WIDTH=${var.width} HEIGHT=${var.height} PREFIX=${var.pre
fix} ./deploy_app.sh",
      "sudo apt -y install cowsay",     <-- 요기 두줄 추가
      "cowsay Mooooooooooo!",
    ]

...

root@workstation:~/hashicat-aws# terraform apply -auto-approve
tls_private_key.hashicat: Refreshing state... [id=976fc16e7f3ba91c3f6e699d88ae61418dd3af20]
aws_key_pair.hashicat: Refreshing state... [id=minman-ssh-key.pem]
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]
aws_subnet.hashicat: Refreshing state... [id=subnet-03be28a3358e39958]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-0c5db75b6e4598e1a]
aws_security_group.hashicat: Refreshing state... [id=sg-0512901f8d650cc19]
aws_route_table.hashicat: Refreshing state... [id=rtb-05d15616067d71b7b]
aws_instance.hashicat: Refreshing state... [id=i-02291afd8a0a4cfc1]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-07db145cb4684c3f5]
aws_eip.hashicat: Refreshing state... [id=eipalloc-09b8550bcd02d005d]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-0e4db23a59f475b5c]
null_resource.configure-cat-app: Refreshing state... [id=4208660317826343136]

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # null_resource.configure-cat-app must be replaced
-/+ resource "null_resource" "configure-cat-app" {
      ~ id       = "4208660317826343136" -> (known after apply)
      ~ triggers = {
          - "build_number" = "2022-01-19T03:24:07Z"
        } -> (known after apply) # forces replacement
    }

Plan: 1 to add, 0 to change, 1 to destroy.
null_resource.configure-cat-app: Destroying... [id=4208660317826343136]
null_resource.configure-cat-app: Destruction complete after 0s
null_resource.configure-cat-app: Creating...
null_resource.configure-cat-app: Provisioning with 'file'...

...

null_resource.configure-cat-app (remote-exec): (Reading database ... 95%
null_resource.configure-cat-app (remote-exec): (Reading database ... 100%
null_resource.configure-cat-app (remote-exec): (Reading database ... 58316 files and directories currently installed.)
null_resource.configure-cat-app (remote-exec): Preparing to unpack .../cowsay_3.03+dfsg2-4_all.deb ...
Progress: [ 17%] [##................] e-exec): Unpacking cowsay (3.03+dfsg2-4) ...
Progress: [ 50%] [#########.........] e-exec): Setting up cowsay (3.03+dfsg2-4) ...
Progress: [ 83%] [###############...] e-exec): Processing triggers for man-db (2.8.3-2ubuntu0.1) ...

null_resource.configure-cat-app (remote-exec):  _______________
null_resource.configure-cat-app (remote-exec): < Mooooooooooo! >
null_resource.configure-cat-app (remote-exec):  ---------------
null_resource.configure-cat-app (remote-exec):         \   ^__^
null_resource.configure-cat-app (remote-exec):          \  (oo)\_______
null_resource.configure-cat-app (remote-exec):             (__)\       )\/\
null_resource.configure-cat-app (remote-exec):                 ||----w |
null_resource.configure-cat-app (remote-exec):                 ||     ||
null_resource.configure-cat-app: Creation complete after 28s [id=8896159478913568901]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.

Outputs:

catapp_url = "http://ec2-54-219-4-25.us-west-1.compute.amazonaws.com"





#add an Output
#Open the outputs.tf file on the "Code Editor" tab. Note the public_dns output in the file.
#Add a second output for the public_ip of your web server. 
# It's going to look almost exactly like the first output but will instead output an IP address instead of a DNS name.
#Name your output catapp_ip.
#You may refer to the docs page to see what types of outputs are valid:
#Terraform AWS Docs - Click Here Terraform Outputs Docs - Click Here
#Be sure to save the changes to the outputs.tf file.
#Run the Terraform refresh command to view your new output:
root@workstation:~/hashicat-aws# terraform refresh
tls_private_key.hashicat: Refreshing state... [id=976fc16e7f3ba91c3f6e699d88ae61418dd3af20]
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]
aws_key_pair.hashicat: Refreshing state... [id=minman-ssh-key.pem]
aws_subnet.hashicat: Refreshing state... [id=subnet-03be28a3358e39958]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-0c5db75b6e4598e1a]
aws_security_group.hashicat: Refreshing state... [id=sg-0512901f8d650cc19]
aws_route_table.hashicat: Refreshing state... [id=rtb-05d15616067d71b7b]
aws_instance.hashicat: Refreshing state... [id=i-02291afd8a0a4cfc1]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-07db145cb4684c3f5]
aws_eip.hashicat: Refreshing state... [id=eipalloc-09b8550bcd02d005d]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-0e4db23a59f475b5c]
null_resource.configure-cat-app: Refreshing state... [id=8896159478913568901]

Outputs:

catapp_url = "http://ec2-54-219-4-25.us-west-1.compute.amazonaws.com"




root@workstation:~/hashicat-aws# terraform output
catapp_url = "http://ec2-54-219-4-25.us-west-1.compute.amazonaws.com"





# Fun With Variables
#There are several ways to configure Terraform variables. So far we've been using the terraform.tfvars file to set our variables. 
# Try re-deploying your application with different height and width variables on the command line. 
# Reload the webapp after each apply to observe any changes.
root@workstation:~/hashicat-aws# terraform apply -auto-approve -var height=600 -var width=800
tls_private_key.hashicat: Refreshing state... [id=976fc16e7f3ba91c3f6e699d88ae61418dd3af20]
aws_key_pair.hashicat: Refreshing state... [id=minman-ssh-key.pem]
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]
aws_subnet.hashicat: Refreshing state... [id=subnet-03be28a3358e39958]
aws_security_group.hashicat: Refreshing state... [id=sg-0512901f8d650cc19]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-0c5db75b6e4598e1a]
aws_route_table.hashicat: Refreshing state... [id=rtb-05d15616067d71b7b]
aws_instance.hashicat: Refreshing state... [id=i-02291afd8a0a4cfc1]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-07db145cb4684c3f5]
aws_eip.hashicat: Refreshing state... [id=eipalloc-09b8550bcd02d005d]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-0e4db23a59f475b5c]
null_resource.configure-cat-app: Refreshing state... [id=8896159478913568901]

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # null_resource.configure-cat-app must be replaced
-/+ resource "null_resource" "configure-cat-app" {
      ~ id       = "8896159478913568901" -> (known after apply)
      ~ triggers = {
          - "build_number" = "2022-01-19T04:10:40Z"
        } -> (known after apply) # forces replacement
    }

Plan: 1 to add, 0 to change, 1 to destroy.
null_resource.configure-cat-app: Destroying... [id=8896159478913568901]
null_resource.configure-cat-app: Destruction complete after 0s
null_resource.configure-cat-app: Creating...
null_resource.configure-cat-app: Provisioning with 'file'...
null_resource.configure-cat-app: Provisioning with 'remote-exec'...
null_resource.configure-cat-app (remote-exec): Connecting to remote host via SSH...

...

null_resource.configure-cat-app (remote-exec): Reading state information... Done
null_resource.configure-cat-app (remote-exec): cowsay is already the newest version (3.03+dfsg2-4).
null_resource.configure-cat-app (remote-exec): 0 upgraded, 0 newly installed, 0 to remove and0 not upgraded.
null_resource.configure-cat-app (remote-exec):  _______________
null_resource.configure-cat-app (remote-exec): < Mooooooooooo! >
null_resource.configure-cat-app (remote-exec):  ---------------
null_resource.configure-cat-app (remote-exec):         \   ^__^
null_resource.configure-cat-app (remote-exec):          \  (oo)\_______
null_resource.configure-cat-app (remote-exec):             (__)\       )\/\
null_resource.configure-cat-app (remote-exec):                 ||----w |
null_resource.configure-cat-app (remote-exec):                 ||     ||
null_resource.configure-cat-app: Creation complete after 27s [id=6620377421343917730]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.

Outputs:

catapp_url = "http://ec2-54-219-4-25.us-west-1.compute.amazonaws.com"

```

위에 URL을 클릭하면 아래와 같은 화면을 확인할 수 있다.

<img width="828" alt="3" src="https://user-images.githubusercontent.com/41605276/150137504-ceacdf8d-fa69-4eb0-a98f-27786b1e7ff9.png">


```console
# Next try setting an environment variable that Terraform can read. Run this command to set the placeholder variable:
root@workstation:~/hashicat-aws# export TF_VAR_placeholder=placedog.net


root@workstation:~/hashicat-aws# terraform apply -auto-approve
tls_private_key.hashicat: Refreshing state... [id=976fc16e7f3ba91c3f6e699d88ae61418dd3af20]
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]
aws_key_pair.hashicat: Refreshing state... [id=minman-ssh-key.pem]
aws_security_group.hashicat: Refreshing state... [id=sg-0512901f8d650cc19]
aws_subnet.hashicat: Refreshing state... [id=subnet-03be28a3358e39958]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-0c5db75b6e4598e1a]
aws_route_table.hashicat: Refreshing state... [id=rtb-05d15616067d71b7b]
aws_instance.hashicat: Refreshing state... [id=i-02291afd8a0a4cfc1]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-07db145cb4684c3f5]
aws_eip.hashicat: Refreshing state... [id=eipalloc-09b8550bcd02d005d]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-0e4db23a59f475b5c]
null_resource.configure-cat-app: Refreshing state... [id=6620377421343917730]

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # null_resource.configure-cat-app must be replaced
-/+ resource "null_resource" "configure-cat-app" {
      ~ id       = "6620377421343917730" -> (known after apply)
      ~ triggers = {
          - "build_number" = "2022-01-19T04:16:30Z"
        } -> (known after apply) # forces replacement
    }

Plan: 1 to add, 0 to change, 1 to destroy.
null_resource.configure-cat-app: Destroying... [id=6620377421343917730]
null_resource.configure-cat-app: Destruction complete after 0s
null_resource.configure-cat-app: Creating...
null_resource.configure-cat-app: Provisioning with 'file'...
null_resource.configure-cat-app: Provisioning with 'remote-exec'...
null_resource.configure-cat-app (remote-exec): Connecting to remote host via SSH...
null_resource.configure-cat-app (remote-exec):   Host: 54.219.4.25

...

null_resource.configure-cat-app (remote-exec): < Mooooooooooo! >
null_resource.configure-cat-app (remote-exec):  ---------------
null_resource.configure-cat-app (remote-exec):         \   ^__^
null_resource.configure-cat-app (remote-exec):          \  (oo)\_______
null_resource.configure-cat-app (remote-exec):             (__)\       )\/\
null_resource.configure-cat-app (remote-exec):                 ||----w |
null_resource.configure-cat-app (remote-exec):                 ||     ||
null_resource.configure-cat-app: Creation complete after 27s [id=1683815062203551535]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.

Outputs:

catapp_url = "http://ec2-54-219-4-25.us-west-1.compute.amazonaws.com"
```

위에 URL을 클릭하면 아래와 같은 화면으로 접속할 것이다.

<img width="713" alt="4" src="https://user-images.githubusercontent.com/41605276/150137982-d772e5e6-f1d9-4dfe-af06-5c55cf131c2a.png">

```console
# Now try it again with the same variable set differently on the command line:
# Which variable took precedence? Why?
root@workstation:~/hashicat-aws# terraform apply -auto-approve -var placeholder=placebear.com
tls_private_key.hashicat: Refreshing state... [id=976fc16e7f3ba91c3f6e699d88ae61418dd3af20]
aws_key_pair.hashicat: Refreshing state... [id=minman-ssh-key.pem]
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]
aws_subnet.hashicat: Refreshing state... [id=subnet-03be28a3358e39958]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-0c5db75b6e4598e1a]
aws_security_group.hashicat: Refreshing state... [id=sg-0512901f8d650cc19]
aws_route_table.hashicat: Refreshing state... [id=rtb-05d15616067d71b7b]
aws_instance.hashicat: Refreshing state... [id=i-02291afd8a0a4cfc1]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-07db145cb4684c3f5]
aws_eip.hashicat: Refreshing state... [id=eipalloc-09b8550bcd02d005d]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-0e4db23a59f475b5c]
null_resource.configure-cat-app: Refreshing state... [id=1683815062203551535]

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # null_resource.configure-cat-app must be replaced
-/+ resource "null_resource" "configure-cat-app" {
      ~ id       = "1683815062203551535" -> (known after apply)
      ~ triggers = {
          - "build_number" = "2022-01-19T04:18:29Z"
        } -> (known after apply) # forces replacement
    }

Plan: 1 to add, 0 to change, 1 to destroy.
null_resource.configure-cat-app: Destroying... [id=1683815062203551535]
null_resource.configure-cat-app: Destruction complete after 0s

...

sg2-4).
null_resource.configure-cat-app (remote-exec): 0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
null_resource.configure-cat-app (remote-exec):  _______________
null_resource.configure-cat-app (remote-exec): < Mooooooooooo! >
null_resource.configure-cat-app (remote-exec):  ---------------
null_resource.configure-cat-app (remote-exec):         \   ^__^
null_resource.configure-cat-app (remote-exec):          \  (oo)\_______
null_resource.configure-cat-app (remote-exec):             (__)\       )\/\
null_resource.configure-cat-app (remote-exec):                 ||----w |
null_resource.configure-cat-app (remote-exec):                 ||     ||
null_resource.configure-cat-app: Creation complete after 26s [id=1691619177554243973]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.

Outputs:

catapp_url = "http://ec2-54-219-148-40.us-west-1.compute.amazonaws.com"
```

위에 URL로 접속하면 아래와 같은 화면을 확인할 수 있다.

<img width="750" alt="5" src="https://user-images.githubusercontent.com/41605276/150138374-f88d0d5a-03a3-4557-9fc3-85f3e0cf3fab.png">

그런 다음에 아래 가이드에 따라 진행해준다.

```text
Terraform Cloud Setup
During this challenge and the next one, you'll use the remote state feature of Terraform Cloud to store your state file in the cloud. In order to do this you'll need a Terraform Cloud account. Click on the URL below and sign up for a free account if you don't have one already:

https://app.terraform.io/signup/account

If you already have an account you can simply sign in with your existing credentials.

Once you're signed into Terraform Cloud create a new organization called YOURNAME-training. Replace YOURNAME with your own name or other text.

Next you'll be prompted to create a workspace. You can skip the VCS integration step by clicking the "CLI-driven workflow" panel. Name your workspace hashicat-aws.

Note: If you already have a hashicat-aws workspace, please delete the workspace by selecting the Settings >> Destruction and Deletion menu, clicking the "Delete from Terraform Cloud" button, typing "hashicat-aws" to confirm, and then clicking the "Delete workspace" button. Then re-create it as above. Doing this avoids possible problems with mis-matched state versions when executing local runs after having executed remote runs. This could happen if you played the Terraform Cloud with AWS track and then played this track.

Run terraform version on the "Shell" tab and then set the Terraform Version to match on the workspace's Settings >> General settings page.

Also, change the Execution Mode to Local.

Be sure to save your settings by clicking the "Save settings" button at the bottom of the page! This will allow us to run Terraform commands from our workstation with local variables, which we'll do in the next challenge.
```

위에 가이드에 따라 아래 그림과 같이 진행한다.

<img width="1275" alt="6" src="https://user-images.githubusercontent.com/41605276/150139799-0a653bc0-1c99-4ff1-b609-d8a7636ea3fe.png">

<img width="1102" alt="7" src="https://user-images.githubusercontent.com/41605276/150139827-ff0c7dcb-b53a-4a51-bd07-4e8c5443c587.png">

<img width="893" alt="8" src="https://user-images.githubusercontent.com/41605276/150139871-31dcd8f6-c656-4029-9533-132379738a50.png">

<img width="1325" alt="9" src="https://user-images.githubusercontent.com/41605276/150139911-2145d36b-7e20-4bbd-983f-ee666f20cbbd.png">

<img width="1094" alt="10" src="https://user-images.githubusercontent.com/41605276/150139955-a828e7ed-1481-4590-a906-7421d519a60d.png">

위에 이미지에서 로컬 버튼 누르기 전에 반드시 !!!! 아래 부분에서 테라폼 버전을 1.0.7로 변경해주고 로컬로 변경해줘야 함

그런 다음에 아래 가이드에 따라 진행해준다.

```text
Configure Remote State
During this challenge we'll configure Terraform Cloud as a remote state backend, and then migrate our existing state file to Terraform Cloud.

Edit the "remote_backend.tf" file and replace the YOURORGANIZATION placeholder with your org name. Be sure to save the file.

Let's generate a new user token for use on your workstation. Visit the User Settings > Tokens page in Terraform Cloud:

https://app.terraform.io/app/settings/tokens

Click on the Create an API token button. You can name the token whatever you like. Copy the entire token using your mouse or the small copy-paste icon.

Back in the Instruqt track, you need to add your API token to a file called "credentials.tfrc.json". Select the "Credentials File" tab and open the /root/.terraform.d/credentials.tfrc.json file directly.

Replace the part that says YOURTOKEN with what you copied from Terraform Cloud. Be sure to save the file.

Your token is now safely stored in the /root/.terraform.d/credentials.tfrc.json file.

Now run 'terraform init' to migrate your state to Terraform Cloud.

Type "yes" when it prompts you to migrate your state into Terraform Cloud.

Your state is now safely stored in Terraform Cloud. You can verify this on the "States" tab of your workspace in the TFC UI.

Run a terraform apply -auto-approve and watch your state file evolve with each change. You can browse through previous state files with the Terraform Cloud UI.
```

```console
root@workstation:~/hashicat-aws# cat remote_backend.tf
terraform {
  backend "remote" {
    hostname = "app.terraform.io"
    organization = "YOURORGANIZATION"
    workspaces {
      name = "hashicat-aws"
    }
  }
}

root@workstation:~/hashicat-aws# vim remote_backend.tf

YOURORGANIZATION 을 위에서 설정한 minsupark-training로 변경

root@workstation:~/hashicat-aws# cat remote_backend.tf
terraform {
  backend "remote" {
    hostname = "app.terraform.io"
    organization = "minsupark-training"
    workspaces {
      name = "hashicat-aws"
    }
  }
}
```

https://app.terraform.io/app/settings/tokens 접속 후 토큰발급후 토큰 복사

```console
root@workstation:~/hashicat-aws# cat /root/.terraform.d/credentials.tfrc.json
{
  "credentials": {
    "app.terraform.io": {
      "token": "YOURTOKEN"
    }
  }
}

# 아래와 같이 복사한 토큰 입력
root@workstation:~/hashicat-aws# cat /root/.terraform.d/credentials.tfrc.json
{
  "credentials": {
    "app.terraform.io": {
      "token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    }
  }
}

# 그런다음에 테라폼 이닛
root@workstation:~/hashicat-aws# terraform init

Initializing the backend...
Acquiring state lock. This may take a few moments...
Do you want to copy existing state to the new backend?
  Pre-existing state was found while migrating the previous "local" backend to the
  newly configured "remote" backend. No existing state was found in the newly
  configured "remote" backend. Do you want to copy this state to the new "remote"
  backend? Enter "yes" to copy and "no" to start with an empty state.

  Enter a value: yes


Successfully configured the backend "remote"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file
- Reusing previous version of hashicorp/null from the dependency lock file
- Reusing previous version of hashicorp/tls from the dependency lock file
- Using previously-installed hashicorp/aws v3.42.0
- Using previously-installed hashicorp/null v3.1.0
- Using previously-installed hashicorp/tls v3.1.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.




root@workstation:~/hashicat-aws# terraform apply -auto-approve
tls_private_key.hashicat: Refreshing state... [id=20039f01bc14f9c91987c0a67948f7ba50315166]
aws_key_pair.hashicat: Refreshing state... [id=minsupark2-ssh-key.pem]
aws_vpc.hashicat: Refreshing state... [id=vpc-0b6ede93432252507]
aws_subnet.hashicat: Refreshing state... [id=subnet-0e6421d0372f32726]
aws_security_group.hashicat: Refreshing state... [id=sg-05eccb74d6332b526]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-010e6c940845afa11]
aws_route_table.hashicat: Refreshing state... [id=rtb-00fd38c38eedec7e3]
aws_instance.hashicat: Refreshing state... [id=i-0b1f006506ac0e39e]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-0c44d34d98e48a303]
aws_eip.hashicat: Refreshing state... [id=eipalloc-01a376ba384b20f61]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-016058b8cbcf31d0e]
null_resource.configure-cat-app: Refreshing state... [id=1691619177554243973]

Terraform used the selected providers to generate the following execution plan. Resource actions
are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # null_resource.configure-cat-app must be replaced
-/+ resource "null_resource" "configure-cat-app" {
      ~ id       = "1691619177554243973" -> (known after apply)
      ~ triggers = {
          - "build_number" = "2022-01-19T13:15:44Z"
        } -> (known after apply) # forces replacement
    }

Plan: 1 to add, 0 to change, 1 to destroy.
null_resource.configure-cat-app: Destroying... [id=1691619177554243973]
null_resource.configure-cat-app: Destruction complete after 0s
null_resource.configure-cat-app: Creating...
null_resource.configure-cat-app: Provisioning with 'file'...

...

null_resource.configure-cat-app (remote-exec): Reading state information... 0%
null_resource.configure-cat-app (remote-exec): Reading state information... 0%
null_resource.configure-cat-app (remote-exec): Reading state information... Done
null_resource.configure-cat-app (remote-exec): cowsay is already the newest version (3.03+dfsg2-4).
null_resource.configure-cat-app (remote-exec): 0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
null_resource.configure-cat-app (remote-exec):  _______________
null_resource.configure-cat-app (remote-exec): < Mooooooooooo! >
null_resource.configure-cat-app (remote-exec):  ---------------
null_resource.configure-cat-app (remote-exec):         \   ^__^
null_resource.configure-cat-app (remote-exec):          \  (oo)\_______
null_resource.configure-cat-app (remote-exec):             (__)\       )\/\
null_resource.configure-cat-app (remote-exec):                 ||----w |
null_resource.configure-cat-app (remote-exec):                 ||     ||
null_resource.configure-cat-app: Creation complete after 27s [id=5430972914951113765]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.

Outputs:

catapp_url = "http://ec2-54-219-148-40.us-west-1.compute.amazonaws.com"

# Terraform Destroy
# Run the following command to destroy your infrastructure: terraform destroy
# You'll need to type "yes" when prompted to destroy your infrastructure. 
# This is a safety feature to help prevent accidental deletion of important resources.
# Wait until the destroy action has completely finished before clicking on the Check button.
# Congratulations on completing the Intro to Terraform track!
root@workstation:~/hashicat-aws# terraform destroy
Acquiring state lock. This may take a few moments...
tls_private_key.hashicat: Refreshing state... [id=20039f01bc14f9c91987c0a67948f7ba50315166]
aws_key_pair.hashicat: Refreshing state... [id=minsupark2-ssh-key.pem]
aws_vpc.hashicat: Refreshing state... [id=vpc-0b6ede93432252507]
aws_security_group.hashicat: Refreshing state... [id=sg-05eccb74d6332b526]

...

aws_security_group.hashicat: Destruction complete after 2s
aws_vpc.hashicat: Destroying... [id=vpc-0b6ede93432252507]
aws_vpc.hashicat: Destruction complete after 1s

Destroy complete! Resources: 12 destroyed.
```