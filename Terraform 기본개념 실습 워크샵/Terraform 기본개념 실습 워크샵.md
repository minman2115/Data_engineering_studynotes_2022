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
  # 아래와 같이 테라폼 프로바이더로 aws를 쓸거라고 설정된 부분을 확인할 수 있다.
  # 이부분은 사용자가 일일히 타이핑을해서 치는게 아니라 테라폼 홈페이지가면
  # 레지스트리 라는 메뉴가 있는데 거기에 프로바이더로 aws, gcp, 에저 등 코드 샘플들이 있는데
  # 거기에서 템플릿을 가져와서 사용하면 된다.
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





# 아래와 같이 테라폼 이닛을 하게 되면 위에서 확인했던 main.tf 내용 상단에 명시된 aws 프로바이더를 다운로드 받게된다.
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





# 그러면 아래와 같이 .terraform이라는 히든 디렉토리 하위에 aws 라는 폴더에 프로바이더를 다운로드 받게된다.
# 테라폼은 이 프로바이더를 통해서 aws와 통신을 하게 된다.
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





# terraform validate 명령어를 사용하게 되면 해당 디렉토리 안에 모든 .tf 파일을 검수해서
# hcl에 맞게 잘 구성이 되어 있는지 체크를 해준다.
# 예를 들어서 main.tf를 수정해서 일부러 문법에러를 만들게해서 저장을 한 후에 다시 terraform validate
# 명령어를 실행하게 되면 에러가 발생한 스크립트 부분을 지적하면서 에러를 일으키게된다.
root@workstation:~/hashicat-aws# terraform validate
Success! The configuration is valid.





# 테라폼 플랜을 실행하면 아래와 같이 prefix value에 대해서 변수를 임의로 입력하라고 나온다. 그냥 이름으로 입력한다.
# 테라폼에서는 커맨드라인으로 변수를 선언하거나 .tfvar 파일을 만들어서 변수를 선언할수도 있는데 아무곳에도 변수가 선언이
# 되어 있지 않은데 main.tf에서 변수를 사용하고 있으면 아래와 같이 변수를 입력하고 엔터를 치라고 나오게 된다.
# 위에 main.tf 스크립트를 보면 tags에 ${var.prefix}와 ${var.region} 문구를 확인할 수 있는데 이 두개의 변수는
# variables.tf 파일에서 프리픽스와 리전을 가져와서 넣으라는 의미이다. 리전은 디폴트로 us-east-1으로 명시되어 있는것을
# 확인할 수 있는데 프리픽스는 디폴트로 선언이 안되어 있다. 그래서 아래와 같이 테라폼 플랜 실행할때 프리픽스 변수를 입력하라고
# 나오는 것이다.
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





# 일반적인 경우에는 테라폼 변수들을 variables.tf에 선언하고 이거를 활용을 하지만
# terraform.tfvars 라는 파일을 만들어서 여기에 변수를 선언해서 활용할 수도 있다.
# 그래서 위에서 명시를 하지 않고 실행했던 프리픽스 변수를 terraform.tfvars에 선언해서 활용해보자.
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






# 위에서 terraform.tfvars에 변수를 선언하고 다시 테라폼 플랜을 실행하면
# 아까처럼 변수를 입력하라고 물어보지 않고 terraform.tfvars에서 변수를 가져와서 main.tf에 치환을 해서 실행하게 된다.
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






# terraform.tfvars에 이번에는 리전변수를 선언해서
# 기존에 variables.tf에 선언된 us-east-1 값을 디폴트로 가져와서 사용하는게 아니라 
# terraform.tfvars에 선언한 값을 가져와서 오버라이딩 할 수도 있다.
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





# 결론적으로 main.tf에 변수를 사용하고 싶으면 variables.tf에 변수들을 선언해서 사용을 하는데
# 사용자에 따라 또는 상황에 따라 유저 커스텀하게 사용하고 싶으면 terraform.tfvars 파일에 선언해서
# 오버라이딩해서 사용할 수도 있다.






# 테라폼 그래프 라는 명령어를 실행하면 리소스간의 연결관계와 종속성 확인할 수 있다.
# 아래와 같이 실행해서 보면 사실 무슨 얘기인지 이해가 안간다.
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






# 그래서 위에서 테라폼 그래프 명령어를 갖고 blast-radius 라는 어플리케이션을 활용하면 실행하면 그래프뷰로 그림으로 볼 수 있다.
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
# 그런다음에 main.tf에 있는 파일에 명시되어 있는 내용에 대해서 아래와 같이 테라폼 플랜을 떠보자
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






# 그런 다음에 테라폼 어플라이 명령어를 실행하게 되면 main.tf에 명시된 내용을 실제로 배포하게 된다.
# 현재 main.tf에 명시된 내용은 위에서 확인할 수 있지만 간단한 vpc만 생성하는 내용이다.
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







# 위와 같이 테라폼 어플라이를 배포하고 main.tf에 별다른 수정이 없다면 
# 그 이후에 실행하는 테라폼 플랜 명령어는 아래와 같이 변경내역이 없다고 나오게 된다.
root@workstation:~/hashicat-aws# terraform plan
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no
differences, so no changes are needed.







# main.tf를 따로 변경하지 않았기 때문에 테라폼 어플라이를 해도 아래와 같이 별다른 액션을 하지 않게 된다.
root@workstation:~/hashicat-aws# terraform apply
aws_vpc.hashicat: Refreshing state... [id=vpc-0b2ad272619c62ae9]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no
differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.






# terraform.tfvars 프리픽스 변수를 다시 바꿔서 다시 테라폼 어플라이를 해보자.
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






# 그런 다음에 다시 테라폼 어플라이를 하게되면 위에서 변경한 프리픽스 내용으로 다시 배포하게 된다.
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







# 이번에는 main.tf에서 아래와 같이 vpc의 테그에 environment = "Production" 를 추가해서 다시 배포해보자.
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





# vpc 테그에 우리가 추가했던 내용이 추가되어 배포가 다시 된다.
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






# 위에서는 vpc를 만들었으니 이번에는 그 vpc 안에 서브넷을 만들어서 배포해보자.
# 아래 내용과 같이 서브넷 영역 부분만 주석을 해제한 다음에 저장해준다.
root@workstation:~/hashicat-aws# vim main.tf
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

 ...

# resource "aws_key_pair" "hashicat" {
#   key_name   = local.private_key_filename
#   public_key = tls_private_key.hashicat.public_key_openssh
# }






# main.tf 변경후에 다시 terraform plan을 떠보자
# 그러면 기존에 있는 것은 건들이지 않고 서브넷 정보만 추가된 것을 확인할 수 있다.
root@workstation:~/hashicat-aws# terraform plan
aws_vpc.hashicat: Refreshing state... [id=vpc-070d52d1481db9f4e]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

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
          + "name" = "minman2-subnet"
        }
      + tags_all                        = {
          + "name" = "minman2-subnet"
        }
      + vpc_id                          = "vpc-070d52d1481db9f4e"
    }

Plan: 1 to add, 0 to change, 0 to destroy.

────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these
actions if you run "terraform apply" now.







# 그런 다음에 테라폼 어플라이 명령어를 실행하면 추가한 서브넷이 배포되게 된다.
root@workstation:~/hashicat-aws# terraform apply
aws_vpc.hashicat: Refreshing state... [id=vpc-070d52d1481db9f4e]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

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
          + "name" = "minman2-subnet"
        }
      + tags_all                        = {
          + "name" = "minman2-subnet"
        }
      + vpc_id                          = "vpc-070d52d1481db9f4e"
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_subnet.hashicat: Creating...
aws_subnet.hashicat: Creation complete after 2s [id=subnet-0a5f3d0c45fdb6e27]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.



# 참고사항
# 만약에 테라폼을 CICD로 태워서 배포해야 한다면 terraform apply 할때마다 yes를 추가로 입력해줘야 하는데 이러면 안되니까
# 어플라이해서 적용할때 terraform apply -auto-approve 명령어를 사용한다면 CICD로 태울때 자동으로 적용이 가능하다.






# 이번에는 아래와 같이 main.tf에서 모든 내용을 주석처리를 해제한 다음에 terraform apply 명령어를 실행하여 배포해보자.
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

# We're using a little trick here so we can run the provisioner without
# destroying the VM. Do not do this in production.

# If you need ongoing management (Day N) of your virtual machines a tool such
# as Chef or Puppet is a better choice. These tools track the state of
# individual files and can keep them in the correct configuration.

# Here we do the following steps:
# Sync everything in files/ to the remote VM.
# Set up some environment variables for our script.
# Add execute permissions to our scripts.
# Run the deploy_app.sh script.
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








root@workstation:~/hashicat-aws# terraform plan
aws_vpc.hashicat: Refreshing state... [id=vpc-070d52d1481db9f4e]
aws_subnet.hashicat: Refreshing state... [id=subnet-0a5f3d0c45fdb6e27]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
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
      + key_name                             = "minman2-ssh-key.pem"
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
      + subnet_id                            = "subnet-0a5f3d0c45fdb6e27"
      + tags                                 = {
          + "Name" = "minman2-hashicat-instance"
        }
      + tags_all                             = {
          + "Name" = "minman2-hashicat-instance"
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
          + "Name" = "minman2-internet-gateway"
        }
      + tags_all = {
          + "Name" = "minman2-internet-gateway"
        }
      + vpc_id   = "vpc-070d52d1481db9f4e"
    }

  # aws_key_pair.hashicat will be created
  + resource "aws_key_pair" "hashicat" {
      + arn         = (known after apply)
      + fingerprint = (known after apply)
      + id          = (known after apply)
      + key_name    = "minman2-ssh-key.pem"
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
      + vpc_id           = "vpc-070d52d1481db9f4e"
    }

  # aws_route_table_association.hashicat will be created
  + resource "aws_route_table_association" "hashicat" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = "subnet-0a5f3d0c45fdb6e27"
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
      + name                   = "minman2-security-group"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags                   = {
          + "Name" = "minman2-security-group"
        }
      + tags_all               = {
          + "Name" = "minman2-security-group"
        }
      + vpc_id                 = "vpc-070d52d1481db9f4e"
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

Plan: 10 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + catapp_url = (known after apply)

────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these
actions if you run "terraform apply" now.






# auto approve 옵션을 주고 어플라이를 해보자.
root@workstation:~/hashicat-aws# terraform apply -auto-approve
aws_vpc.hashicat: Refreshing state... [id=vpc-070d52d1481db9f4e]
aws_subnet.hashicat: Refreshing state... [id=subnet-0a5f3d0c45fdb6e27]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_eip.hashicat will be created
  + resource "aws_eip" "hashicat" {
      + allocation_id        = (known after apply)
      + association_id       = (known after apply)
      + carrier_ip           = (known after apply)


...



null_resource.configure-cat-app (remote-exec): Processing triggers for systemd (237-3ubuntu10.53) ...
null_resource.configure-cat-app (remote-exec): Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
null_resource.configure-cat-app (remote-exec): Processing triggers for ufw (0.36-0ubuntu0.18.04.2) ...
null_resource.configure-cat-app (remote-exec): Processing triggers for ureadahead (0.100.0-21) ...

null_resource.configure-cat-app (remote-exec): Script complete.
null_resource.configure-cat-app: Still creating... [50s elapsed]
null_resource.configure-cat-app: Creation complete after 50s [id=6575887821605929847]

Apply complete! Resources: 10 added, 0 changed, 0 destroyed.

Outputs:

catapp_url = "http://ec2-52-53-56-250.us-west-1.compute.amazonaws.com"
```

위에 http://ec2-52-53-56-250.us-west-1.compute.amazonaws.com 을 웹브라우저로 접속하면 아래와 같은 화면을 확인할 수 있다.

<img width="967" alt="2" src="https://user-images.githubusercontent.com/41605276/150126249-3c5977fc-2047-4b1c-b8c6-2fe69066c3c5.png">



결론적으로 위와 같이 테라폼 프로비저너를 이용해서 가상머신 인프라를 띄우고 어플리케이션을 구성해서 배포할 수 있다.


```console
# main.tf를 열고 아래와 같이 두줄을 추가해준다.
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
      "cowsay Mooooooooooo!",           <-- 
    ]

...




# 위와 같이 main.tf를 수정하고 개행이나 이쁘게 스크립트가 정렬되도록 도와주는 명령어도 있다.
# terraform fmt 명령어를 실행하면 main.tf 내용의 스크립트를 이쁘게 정렬을 해준다.
root@workstation:~/hashicat-aws# terraform fmt
main.tf






# 그런 다음에 다시 배포를 해보자 그러면 위에서 추가한 cowsay가 설치가되고 아래와 같이 "움머~" 메세지를 확인할 수 있다.
# 아래에 내용을 보면 -/+ destroy and then create replacement 문구와 같이 기존에 있던 인스턴스는 날리고 새로
# 인스턴스를 띄워서 cowsay를 설치하고 아래와 같이 "움머~" 메세지를 띄워주게 된다.
root@workstation:~/hashicat-aws# terraform apply -auto-approve
tls_private_key.hashicat: Refreshing state... [id=f2de8f8993340627e01f44f6b92518467e14995e]
aws_key_pair.hashicat: Refreshing state... [id=minman2-ssh-key.pem]
aws_vpc.hashicat: Refreshing state... [id=vpc-070d52d1481db9f4e]
aws_security_group.hashicat: Refreshing state... [id=sg-0dc6708d0a2454574]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-0de5cc872dbeededd]
aws_subnet.hashicat: Refreshing state... [id=subnet-0a5f3d0c45fdb6e27]
aws_route_table.hashicat: Refreshing state... [id=rtb-0e4f5c408f68c48ca]
aws_instance.hashicat: Refreshing state... [id=i-0515d8a419ad6d273]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-0e9ab2e4d16cc8bbc]
aws_eip.hashicat: Refreshing state... [id=eipalloc-01bf1cc0a3e802fa8]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-00c56c1b3da672219]
null_resource.configure-cat-app: Refreshing state... [id=6575887821605929847]

Note: Objects have changed outside of Terraform

Terraform detected the following changes made outside of Terraform since the last "terraform apply":

  # aws_eip.hashicat has been changed
  ~ resource "aws_eip" "hashicat" {
        id                   = "eipalloc-01bf1cc0a3e802fa8"
      + tags                 = {}
        # (12 unchanged attributes hidden)
    }
  # aws_instance.hashicat has been changed
  ~ resource "aws_instance" "hashicat" {
        id                                   = "i-0515d8a419ad6d273"
      ~ public_dns                           = "ec2-54-193-51-71.us-west-1.compute.amazonaws.com" -> "ec2-52-53-56-250.us-west-1.compute.amazonaws.com"
      ~ public_ip                            = "54.193.51.71" -> "52.53.56.250"
        tags                                 = {
            "Name" = "minman2-hashicat-instance"
        }
        # (27 unchanged attributes hidden)





        # (5 unchanged blocks hidden)
    }
  # aws_key_pair.hashicat has been changed
  ~ resource "aws_key_pair" "hashicat" {
        id          = "minman2-ssh-key.pem"
      + tags        = {}
        # (6 unchanged attributes hidden)
    }
  # aws_route_table.hashicat has been changed
  ~ resource "aws_route_table" "hashicat" {
        id               = "rtb-0e4f5c408f68c48ca"
      + tags             = {}
        # (6 unchanged attributes hidden)
    }

Unless you have made equivalent changes to your configuration, or ignored the relevant attributes using
ignore_changes, the following plan may include actions to undo or respond to these changes.

────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # null_resource.configure-cat-app must be replaced
-/+ resource "null_resource" "configure-cat-app" {
      ~ id       = "6575887821605929847" -> (known after apply)
      ~ triggers = {
          - "build_number" = "2022-01-22T03:53:08Z"
        } -> (known after apply) # forces replacement
    }

Plan: 1 to add, 0 to change, 1 to destroy.
null_resource.configure-cat-app: Destroying... [id=6575887821605929847]
null_resource.configure-cat-app: Destruction complete after 0s
null_resource.configure-cat-app: Creating...
null_resource.configure-cat-app: Provisioning with 'file'...
null_resource.configure-cat-app: Provisioning with 'remote-exec'...
null_resource.configure-cat-app (remote-exec): Connecting to remote host via SSH...
null_resource.configure-cat-app (remote-exec):   Host: 52.53.56.250

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

catapp_url = "http://ec2-52-53-56-250.us-west-1.compute.amazonaws.com"








# 이번에는 output을 바꿔보자 위와 같이 catapp_url = "http://ec2-52-53-56-250.us-west-1.compute.amazonaws.com" 형태로
# main.tf를 배포하게 되면 아웃풋으로 출력하게 되는데 이는 outputs.tf에 명시되어 있다.
root@workstation:~/hashicat-aws# cat outputs.tf
# Outputs file
output "catapp_url" {
  value = "http://${aws_eip.hashicat.public_dns}"
}






# 아래와 같이 output 포맷을 하나 더 추가해보자
root@workstation:~/hashicat-aws# cat outputs.tf
# Outputs file
output "catapp_url" {
  value = "http://${aws_eip.hashicat.public_dns}"
}
output "catapp_ip" {
  value = "http://${aws_eip.hashicat.public_ip}"
}





# 아래와 같이 terraform refresh 명령어를 실행하게 되면
# 현재 실제 배포되어진 환경과 현재 갖고 있는 tf 파일과 피교해서 변경사항이 있다면 서로 sync를 해주고
# 마지막에 아웃풋 파일을 출력해주게 된다.
root@workstation:~/hashicat-aws# terraform refresh
tls_private_key.hashicat: Refreshing state... [id=f2de8f8993340627e01f44f6b92518467e14995e]
aws_vpc.hashicat: Refreshing state... [id=vpc-070d52d1481db9f4e]
aws_key_pair.hashicat: Refreshing state... [id=minman2-ssh-key.pem]
aws_security_group.hashicat: Refreshing state... [id=sg-0dc6708d0a2454574]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-0de5cc872dbeededd]
aws_subnet.hashicat: Refreshing state... [id=subnet-0a5f3d0c45fdb6e27]
aws_route_table.hashicat: Refreshing state... [id=rtb-0e4f5c408f68c48ca]
aws_instance.hashicat: Refreshing state... [id=i-0515d8a419ad6d273]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-0e9ab2e4d16cc8bbc]
aws_eip.hashicat: Refreshing state... [id=eipalloc-01bf1cc0a3e802fa8]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-00c56c1b3da672219]
null_resource.configure-cat-app: Refreshing state... [id=707923439097649920]

Outputs:

catapp_ip = "http://52.53.56.250"
catapp_url = "http://ec2-52-53-56-250.us-west-1.compute.amazonaws.com"





# 테라폼 아웃풋 명령어로 아웃풋 포맷도 확인할 수 있다.
root@workstation:~/hashicat-aws# terraform output
catapp_ip = "http://52.53.56.250"
catapp_url = "http://ec2-52-53-56-250.us-west-1.compute.amazonaws.com"





# 참고사항 : 변수 우선순위
# Terraform variables have five levels of precedence. 1=highest 5=lowest:
# 1. Command line flag - run as a command line switch
# 2. Configuration file - set in your terraform.tfvars file
# 3. Environment variable - part of your shell environment
# 4. Default Config - default value in variables.tf
# 5. User manual entry - if not specified, prompt the user for entry






# Fun With Variables
# 이번에는 아래와 같이 커맨드라인으로 고양이 사진의 가로 세로 변수를 조정을 해서 다시 배포를 해보자.
# 당연한 얘기겠지만 아래에 가로 세로 변수는 variables.tf에 명시가 되어 있다.
# 위에서 언급한대로 커맨드라인으로 주는 변수의 우선순위가 높기 때문에 600 * 800 으로 고양이 사진이 크기가 변경될 것이다.
# 그러면 아래와 같이 ec2를 죽였다가 다시 띄우고 우리가 변경한 가로세로 이미지 크기로 다시 배포하게 된다.
root@workstation:~/hashicat-aws# terraform apply -auto-approve -var height=600 -var width=800
tls_private_key.hashicat: Refreshing state... [id=f2de8f8993340627e01f44f6b92518467e14995e]
aws_key_pair.hashicat: Refreshing state... [id=minman2-ssh-key.pem]
aws_vpc.hashicat: Refreshing state... [id=vpc-070d52d1481db9f4e]
aws_subnet.hashicat: Refreshing state... [id=subnet-0a5f3d0c45fdb6e27]
aws_security_group.hashicat: Refreshing state... [id=sg-0dc6708d0a2454574]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-0de5cc872dbeededd]
aws_route_table.hashicat: Refreshing state... [id=rtb-0e4f5c408f68c48ca]
aws_instance.hashicat: Refreshing state... [id=i-0515d8a419ad6d273]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-0e9ab2e4d16cc8bbc]
aws_eip.hashicat: Refreshing state... [id=eipalloc-01bf1cc0a3e802fa8]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-00c56c1b3da672219]
null_resource.configure-cat-app: Refreshing state... [id=707923439097649920]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # null_resource.configure-cat-app must be replaced
-/+ resource "null_resource" "configure-cat-app" {
      ~ id       = "707923439097649920" -> (known after apply)
      ~ triggers = {
          - "build_number" = "2022-01-22T04:11:01Z"
        } -> (known after apply) # forces replacement
    }

Plan: 1 to add, 0 to change, 1 to destroy.
null_resource.configure-cat-app: Destroying... [id=707923439097649920]
null_resource.configure-cat-app: Destruction complete after 0s

...

null_resource.configure-cat-app (remote-exec): Building dependency tree... 50%
null_resource.configure-cat-app (remote-exec): Building dependency tree
null_resource.configure-cat-app (remote-exec): Reading state information... 0%
null_resource.configure-cat-app (remote-exec): Reading state information... 0%
null_resource.configure-cat-app (remote-exec): Reading state information... Done
null_resource.configure-cat-app (remote-exec): cowsay is already the newest version (3.03+dfsg2-4).
null_resource.configure-cat-app (remote-exec): 0 upgraded, 0 newly installed, 0 to remove and 5 not upgraded.
null_resource.configure-cat-app (remote-exec):  _______________
null_resource.configure-cat-app (remote-exec): < Mooooooooooo! >
null_resource.configure-cat-app (remote-exec):  ---------------
null_resource.configure-cat-app (remote-exec):         \   ^__^
null_resource.configure-cat-app (remote-exec):          \  (oo)\_______
null_resource.configure-cat-app (remote-exec):             (__)\       )\/\
null_resource.configure-cat-app (remote-exec):                 ||----w |
null_resource.configure-cat-app (remote-exec):                 ||     ||
null_resource.configure-cat-app: Creation complete after 26s [id=570116522482004336]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.

Outputs:

catapp_ip = "http://52.53.56.250"
catapp_url = "http://ec2-52-53-56-250.us-west-1.compute.amazonaws.com"
```

위에 URL을 클릭하면 아래와 같은 화면을 확인할 수 있다.

<img width="828" alt="3" src="https://user-images.githubusercontent.com/41605276/150137504-ceacdf8d-fa69-4eb0-a98f-27786b1e7ff9.png">


```console
# 이번에는 variables.tf의 플레이스 홀더 변수를 os의 환경변수를 지정해서 오버라이딩 해보자.
root@workstation:~/hashicat-aws# export TF_VAR_placeholder=placedog.net

root@workstation:~/hashicat-aws# env | grep TF_VAR
TF_VAR_placeholder=placedog.net



# 그런 다음에 다시 배포하면 아래와 같이 강아지 사진으로 변경된 것을 확인할 수 있다.
root@workstation:~/hashicat-aws# terraform apply -auto-approve
tls_private_key.hashicat: Refreshing state... [id=f2de8f8993340627e01f44f6b92518467e14995e]
aws_key_pair.hashicat: Refreshing state... [id=minman2-ssh-key.pem]
aws_vpc.hashicat: Refreshing state... [id=vpc-070d52d1481db9f4e]
aws_security_group.hashicat: Refreshing state... [id=sg-0dc6708d0a2454574]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-0de5cc872dbeededd]
aws_subnet.hashicat: Refreshing state... [id=subnet-0a5f3d0c45fdb6e27]
aws_route_table.hashicat: Refreshing state... [id=rtb-0e4f5c408f68c48ca]
aws_instance.hashicat: Refreshing state... [id=i-0515d8a419ad6d273]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-0e9ab2e4d16cc8bbc]
aws_eip.hashicat: Refreshing state... [id=eipalloc-01bf1cc0a3e802fa8]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-00c56c1b3da672219]
null_resource.configure-cat-app: Refreshing state... [id=570116522482004336]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # null_resource.configure-cat-app must be replaced
-/+ resource "null_resource" "configure-cat-app" {
      ~ id       = "570116522482004336" -> (known after apply)
      ~ triggers = {
          - "build_number" = "2022-01-22T04:28:12Z"
        } -> (known after apply) # forces replacement
    }

Plan: 1 to add, 0 to change, 1 to destroy.
null_resource.configure-cat-app: Destroying... [id=570116522482004336]
null_resource.configure-cat-app: Destruction complete after 0s
null_resource.configure-cat-app: Creating...
null_resource.configure-cat-app: Provisioning with 'file'...

...

null_resource.configure-cat-app (remote-exec): cowsay is already the newest version (3.03+dfsg2-4).
null_resource.configure-cat-app (remote-exec): 0 upgraded, 0 newly installed, 0 to remove and 5 not upgraded.
null_resource.configure-cat-app (remote-exec):  _______________
null_resource.configure-cat-app (remote-exec): < Mooooooooooo! >
null_resource.configure-cat-app (remote-exec):  ---------------
null_resource.configure-cat-app (remote-exec):         \   ^__^
null_resource.configure-cat-app (remote-exec):          \  (oo)\_______
null_resource.configure-cat-app (remote-exec):             (__)\       )\/\
null_resource.configure-cat-app (remote-exec):                 ||----w |
null_resource.configure-cat-app (remote-exec):                 ||     ||
null_resource.configure-cat-app: Creation complete after 26s [id=404062229813896000]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.

Outputs:

catapp_ip = "http://52.53.56.250"
catapp_url = "http://ec2-52-53-56-250.us-west-1.compute.amazonaws.com"
```

위에 URL을 클릭하면 아래와 같은 화면으로 접속할 것이다.

<img width="713" alt="4" src="https://user-images.githubusercontent.com/41605276/150137982-d772e5e6-f1d9-4dfe-af06-5c55cf131c2a.png">

```console
# 또는 아래와 같이 곰사진으로 바꾸도록 명령어를 줘서 바꿀수도 있다.
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

catapp_ip = "http://52.53.56.250"
catapp_url = "http://ec2-52-53-56-250.us-west-1.compute.amazonaws.com"
```

위에 URL로 접속하면 아래와 같이 곰사진으로 바뀐것을 확인할 수 있다.

<img width="750" alt="5" src="https://user-images.githubusercontent.com/41605276/150138374-f88d0d5a-03a3-4557-9fc3-85f3e0cf3fab.png">

```console
# 참고사항 : Terraform State
# 테라폼은 stateful 한 어플리케이션이다. 다시말해서 state file 내부에서 빌드한 모든 내용을 추적한다.
# 아래와 같이 terraform.tfstate, terraform.tfstate.backup 파일을 확인할 수 있는데
# 이거는 테라폼 어플라이 명령어를 해서 현재 적용된 상태를 저장한 파일이다.
# 제이슨 형태로 되어 있고 현재 main.tf를 이용해서 aws에 배포된 상태를 저장하고 있다.
root@workstation:~/hashicat-aws# ll
total 124
drwxr-xr-x 7 root root  4096 Jan 22 04:38 ./
drwx------ 8 root root  4096 Jan 22 04:38 ../
drwxr-xr-x 2 root root  4096 Jan 22 02:44 .circleci/
drwxr-xr-x 8 root root  4096 Jan 22 02:44 .git/
-rw-r--r-- 1 root root   177 Jan 22 02:44 .gitignore
drwxr-xr-x 3 root root  4096 Jan 22 03:31 .terraform/
-rw-r--r-- 1 root root  3045 Jan 22 03:45 .terraform.lock.hcl
-rw-r--r-- 1 root root 11357 Jan 22 02:44 LICENSE
-rw-r--r-- 1 root root   244 Jan 22 02:44 README.md
drwxr-xr-x 2 root root  4096 Jan 22 02:44 exercises/
drwxr-xr-x 2 root root  4096 Jan 22 02:44 files/
-rw-r--r-- 1 root root  4526 Jan 22 04:09 main.tf
-rw-r--r-- 1 root root   160 Jan 22 04:18 outputs.tf
-rw-r--r-- 1 root root   162 Jan 22 04:38 remote_backend.tf
-rw-r--r-- 1 root root 24011 Jan 22 04:36 terraform.tfstate
-rw-r--r-- 1 root root 24011 Jan 22 04:35 terraform.tfstate.backup
-rw-r--r-- 1 root root   199 Jan 22 03:37 terraform.tfvars
-rw-r--r-- 1 root root  1578 Jan 22 02:44 variables.tf
```

그런 다음에 아래 가이드에 따라 진행해준다.

- 테라폼 클라우드

테라폼 클라우드는 테라폼을 사용하여 코드로 인프라를 작성하고 구축하기 위한 워크플로우를 제공하는 사스형태의 어플리케이션임. 무료버전과 상용버전 둘다 있음

- 테라폼 리모트 state

기본적으로 테라폼은 테라폼 명령어를 실행하는 머신에 상태파일을 저장한다. 하지만 운영환경에서 이를 운영하려고 한다면 당연히 이 상태파일을 안전하게 저장해야 한다. 테라폼은 이 상태 파일을 안전하게 저장하도록 제공해주는 옵션이 있다. 이를 위해서 테라폼 클라우드에 상태파일을 저장할 수 있도록 기능을 제공해주는데 이게 테라폼 리모트 스테이트이다.

- 테라폼 클라우드 execution mode

1) 로컬실행

말 그대로 테라폼 명령을 로컬에서 실행하며 상태파일도 로컬에 저장됨

2) 원격실행

테라폼 명령어가 테라폼 클라우드의 컨테이너에 저장됨


이번에는 무료 테라폼 클라우드 계정을 생성하고 상태파일을 테라폼 클라우드의 원격저장소에 저장하도록 해보자.


- Terraform Cloud Setup

아래에 URL 로 접속해서 

https://app.terraform.io/signup/account 

minsupark-training 이라는 이름으로 organization을 생생해준다.

그런 다음에 반드시 hashicat-aws 라는 이름으로 워크스페이스를 아래 그림과 같이 생성해준다.

<img width="1275" alt="6" src="https://user-images.githubusercontent.com/41605276/150139799-0a653bc0-1c99-4ff1-b609-d8a7636ea3fe.png">

<img width="1102" alt="7" src="https://user-images.githubusercontent.com/41605276/150139827-ff0c7dcb-b53a-4a51-bd07-4e8c5443c587.png">

<img width="893" alt="8" src="https://user-images.githubusercontent.com/41605276/150139871-31dcd8f6-c656-4029-9533-132379738a50.png">

<img width="1325" alt="9" src="https://user-images.githubusercontent.com/41605276/150139911-2145d36b-7e20-4bbd-983f-ee666f20cbbd.png">

<img width="1094" alt="10" src="https://user-images.githubusercontent.com/41605276/150139955-a828e7ed-1481-4590-a906-7421d519a60d.png">

위에 이미지에서 로컬 버튼 누르기 전에 반드시 !!!! 아래 부분에서 테라폼 버전을 1.0.7로 변경해주고 로컬로 변경해줘야 함

왜냐하면 맨위에서 테라폼 버전을 확인했듯이 1.0.7로 되어 있기 때문이다.

그런 다음에 리모트 스테이트를 구성해보자.

```console
# 아래와 같이 remote_backend.tf 파일이라는게 있는데 이거를 수정해줘야 한다.
root@workstation:~/hashicat-aws# ll
total 124
drwxr-xr-x 7 root root  4096 Jan 22 04:38 ./
drwx------ 8 root root  4096 Jan 22 05:07 ../
drwxr-xr-x 2 root root  4096 Jan 22 02:44 .circleci/
drwxr-xr-x 8 root root  4096 Jan 22 02:44 .git/
-rw-r--r-- 1 root root   177 Jan 22 02:44 .gitignore
drwxr-xr-x 3 root root  4096 Jan 22 03:31 .terraform/
-rw-r--r-- 1 root root  3045 Jan 22 03:45 .terraform.lock.hcl
-rw-r--r-- 1 root root 11357 Jan 22 02:44 LICENSE
-rw-r--r-- 1 root root   244 Jan 22 02:44 README.md
drwxr-xr-x 2 root root  4096 Jan 22 02:44 exercises/
drwxr-xr-x 2 root root  4096 Jan 22 02:44 files/
-rw-r--r-- 1 root root  4526 Jan 22 04:09 main.tf
-rw-r--r-- 1 root root   160 Jan 22 04:18 outputs.tf
-rw-r--r-- 1 root root   162 Jan 22 04:38 remote_backend.tf
-rw-r--r-- 1 root root 24011 Jan 22 04:36 terraform.tfstate
-rw-r--r-- 1 root root 24011 Jan 22 04:35 terraform.tfstate.backup
-rw-r--r-- 1 root root   199 Jan 22 03:37 terraform.tfvars
-rw-r--r-- 1 root root  1578 Jan 22 02:44 variables.tf

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


# 그런 다음에 https://app.terraform.io/app/settings/tokens 접속해서 토큰발급후 토큰 복사한다.
# 복사한 토큰은 아래와 같이 크레덴셜 파일에 넣어주면 된다.
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






# 아래와 같이 어플라이를 해서 다시 적용을 해주면
# 
rroot@workstation:~/hashicat-aws# terraform apply -auto-approve
tls_private_key.hashicat: Refreshing state... [id=f2de8f8993340627e01f44f6b92518467e14995e]
aws_key_pair.hashicat: Refreshing state... [id=minman2-ssh-key.pem]
aws_vpc.hashicat: Refreshing state... [id=vpc-070d52d1481db9f4e]
aws_subnet.hashicat: Refreshing state... [id=subnet-0a5f3d0c45fdb6e27]
aws_security_group.hashicat: Refreshing state... [id=sg-0dc6708d0a2454574]
aws_internet_gateway.hashicat: Refreshing state... [id=igw-0de5cc872dbeededd]
aws_route_table.hashicat: Refreshing state... [id=rtb-0e4f5c408f68c48ca]
aws_instance.hashicat: Refreshing state... [id=i-0515d8a419ad6d273]
aws_route_table_association.hashicat: Refreshing state... [id=rtbassoc-0e9ab2e4d16cc8bbc]
aws_eip.hashicat: Refreshing state... [id=eipalloc-01bf1cc0a3e802fa8]
aws_eip_association.hashicat: Refreshing state... [id=eipassoc-00c56c1b3da672219]
null_resource.configure-cat-app: Refreshing state... [id=404062229813896000]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # null_resource.configure-cat-app must be replaced
-/+ resource "null_resource" "configure-cat-app" {
      ~ id       = "404062229813896000" -> (known after apply)
      ~ triggers = {
          - "build_number" = "2022-01-22T04:35:44Z"
        } -> (known after apply) # forces replacement
    }

Plan: 1 to add, 0 to change, 1 to destroy.
null_resource.configure-cat-app: Destroying... [id=404062229813896000]
null_resource.configure-cat-app: Destruction complete after 0s

...

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

catapp_ip = "http://52.53.56.250"
catapp_url = "http://ec2-52-53-56-250.us-west-1.compute.amazonaws.com"
```

그런 다음에 웹브라우저에 테라폼 클라우드의 나의 워크스페이스로 이동하면 아래 그림과 같이 상태정보가 저장된 것을 확인할 수 있다.

<img width="1440" alt="스크린샷 2022-01-22 오후 2 19 18" src="https://user-images.githubusercontent.com/41605276/150625909-2d1612d0-711b-4c37-b8cf-976d63e00036.png">



```console
# 실습종료 후 테라폼 삭제
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