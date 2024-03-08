---
layout: post
title: Terraform Tutorials
categories: devops
tags: devops
---

테라폼은 HashiCorp의 Infrastructure as Code (IaC)이다. Terraform은 인프라를 구성 파일을 이용하여 관리할 수 있게 해주며, 안전하고 일관되며 반복 가능한 방식으로 인프라를 구축, 변경 및 관리할 수 있다.

테라폼을 사용함으로써 여러 이점이 있다.

- 여러 클라우드 플랫폼의 인프라를 관리할 수 있다.
- 읽기 쉬운 설정 언어로 빠르게 인프라 코드를 작성할 수 있다.
- 테라폼의 상태는 배포 전체의 리소스 변경을 추적할 수 있다.
- 설정 코드에 대한 커밋은 버전 관리와 협업에 용이하다.

> [https://developer.hashicorp.com/terraform/tutorials/aws-get-started/infrastructure-as-code](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/infrastructure-as-code)

### Install Terraform

```sh
# terraform
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

brew update
brew upgrade hashicorp/tap/terraform

# awscli
brew install awscli
brew tap dkanejs/aws-session-manager-plugin
brew install aws-session-manager-plugin
```

> [https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)

### Build infrastructure

IAM 자격 증명을 이용하여 AWS provider를 인증

```sh
export AWS_ACCESS_KEY_ID={ACCESS_KEY}
export AWS_SECRET_ACCESS_KEY={SECRET_ACCESS_KEY}
```

```tf
# main.tf
terraform { # 테라폼 세팅을 포함. hostname, namespace, provider type을 정의
  # provider는 기본적으로 https://registry.terraform.io/에서 설치
  # aws prodiver source는 `hashicorp/aws`에 정의되어 있고 `registry.terraform.io/hashicorp/aws`의 약자
  required_providers { # required_providers에 provider의 버전을 정의할 수 있고 버전을 명시하는 것을 권장한다. 이유는 버전을 명시하지 않으면 항상 latest로 다운받게 되고 세팅한 구성이 버전이 달라 정상적으로 작동하지 않을 수 있기 때문이다.
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}

# 다른 `provider`에서 리소스를 함께 관리하기 위해 여러 개의 `provider` block을 사용할 수 있다.
provider "aws" { # 체적인 `provider`를 설정한다. 이것은 플러그인이고 테라폼은 리소스를 생성하고 관리하는데 사용
  region  = "us-west-2"
}

# 인프라의 컴포넌트를 나타낸다. `resource`는 EC2 인스턴스와 같은 물리적 또는 가상의 컴포넌트일 수도 있고 Heroku Application와 같은 논리적인 리소스일 수도 있다.
# "aws_instance" "app_server"와 같이 resource type과 name이 존재한다. resource type의 prefix는 `provider`의 name과 매핑이 된다. 이 둘은 resource의 `aws_instance.app_server`와 같이 unique key를 형성한다.
resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```

#### Command

```sh
# 새로운 설정 파일 또는 기존 설정 파일을 check out 할 때 사용
# 정의된 구성 파일에 provider를 설치한다.
# - 하위 서브 디렉터리(.terraform)에 provider를 설치
# - .terraform.lock.hcl에 provider version을 명시
#   - 사용하고 있는 provider를 업데이트할 때 컨트롤하기 위해 해당 파일 이용
$ terraform init
# 모든 설정 파일의 일관된 포맷팅을 사용하기 위해 추천
# 현재 디렉터리의 설정 파일들을 가독성과 일관성을 위해 자동으로 업데이트한다.
# 수정된 파일이 있으면 파일 이름을 출력하고 그렇지 않으면 아무것도 출력하지 않는다.
$ terraform fmt
# 설정 파일의 문법적 오류나 내부적으로 일관성이 있는지 확인
# 검증에 성공하면 Success! The configuration is valid. 문구를 출력한다.
$ terraform validate
# 설정 정보를 적용할 때 사용
# 변경 사항을 적용하기 전에 실행 계획을 출력한다.
# - 실행 계획은 `git diff`와 유사하다.
# - `+` 다음으로는 리소스 생성을 의미한다.
# - (known after apply) 라는 문구는 리소스가 생성되기 전까지 알 수 없다는 것이다.
# 실행 계획을 모두 확인하고 `yes`를 입력하면 적용이 시작된다.
$ terraform apply
```

#### Inspect State

`terraform apply`로 설정 정보를 적용하면 `terraform.tfstate` 파일에 테라폼에서 관리하는 ID와 프로퍼티를 저장하고 관리한다. 이는 리소스를 업데이트하고 삭제하기 위함이다.

해당 파일은 테라폼이 관리하고 있는 리소스들을 추적하기 위한 유일한 방법이며 가끔 민감한 정보를 포함한다. 그렇기 때문에 안전하게 관리하기 위해 해당 파일은 신뢰하는 동료들만 액세스할 수 있도록 제한해야 한다. Terraform Cloud 또는 Terraform Enterprise에 [원격으로 상태를 저장](https://developer.hashicorp.com/terraform/tutorials/cloud/cloud-migrate)하는 것을 추천한다.

```sh
# 현재 상태를 검사할 때 사용
# 리소스를 생성할 때 얻은 메타 데이터들을 상태 파일에 저장한다.
$ terraform show
# 상태 관리를 전문적으로 다루기 위해 사용
# list 서브 커맨드는 프로젝트 리소스들을 나열해서 출력한다.
$ terraform state list
```

> [https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-build](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-build)

### Change infrastructure

```tf
# main.tf
terraform {
...blah blah
resource "aws_instance" "app_server" {
  ami           = "ami-08d70e59c07c61a3a"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```

```sh
$ terraform apply
Terraform will perform the following actions:

  # aws_instance.app_server must be replaced
-/+ resource "aws_instance" "app_server" {
      ~ ami                                  = "ami-830c94e3" -> "ami-08d70e59c07c61a3a" # forces replacement
```

> [https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-change](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-change)

### Destroy infrastructure

`terraform destroy` 명령어는 테라폼에서 관리하고 있는 리소스를 종료한다.

```sh
$ terraform destroy
Terraform will perform the following actions:

  # aws_instance.app_server will be destroyed
  - resource "aws_instance" "app_server" {
      - ami                                  = "ami-08d70e59c07c61a3a" -> null
```

> [https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-destroy](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-destroy)

### Define input variables

하드 코딩된 값이 아닌 변수를 이용해 동적이고 유연한 설정을 만들 수 있다.

```tf
# variables.tf
variable "instance_name" {
  description = "Value of the Name tag for the EC2 instance"
  type        = string
  default     = "AppServerInstance"
}
```

`-var` 옵션을 이용하여 변수를 할당할 수 있고 그렇지 않으면 `default`의 값이 설정된다.

```sh
$ terraform apply -var "instance_name=ChangedAppServerInstance"
Terraform will perform the following actions:

  # aws_instance.app_server will be updated in-place
  ~ resource "aws_instance" "app_server" {
        id                                   = "i-0faacb7af0a626dda"
      ~ tags                                 = {
          ~ "Name" = "AppServerInstance" -> "ChangedAppServerInstance"
        }
      ~ tags_all                             = {
          ~ "Name" = "AppServerInstance" -> "ChangedAppServerInstance"
        }
```

> [https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-variables](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-variables)

### Query data with outputs

`output` 은 테라폼을 실행한 후 생성된 인프라에 대한 정보를 출력하거나 다른 테라폼 모듈에서 이 값을 참조할 수 있다.

```sh
# outputs.tf
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.app_server.id
}

output "instance_public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.app_server.public_ip
}
```

```sh
$ terraform apply

Changes to Outputs:
  + instance_id        = (known after apply)
  + instance_public_ip = (known after apply)
...blah blah 
Outputs:

instance_id = "i-02c34550825b6e339"
instance_public_ip = "18.236.168.140"

$ terraform output

instance_id = "i-02c34550825b6e339"
instance_public_ip = "18.236.168.140"
```

> [https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-outputs](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-outputs)

### Store remote state

테스트 환경에서는 로컬에서 인프라를 빌드, 변경, 제거하는 것은 좋은 개발일 수 있다. 하지만 실제 제품에서 팀원들만 접근하여 협업할 수 있도록 상태를 안전하게 유지해야 한다.

이를 위해 가장 좋은 방법은 테라폼 클라우드에서 상태 접근 권한을 받아 원격 환경에서 테라폼을 실행시키는 것이다.

```sh
# main.tf
terraform {
  cloud {
    organization = "organization-name"
    workspaces {
      name = "learn-tfc-aws"
    }
  }
  ...blah blah 
}

Token for app.terraform.io:
  Enter a value: {TOKEN}

                                          -
                                          -----                           -
                                          ---------                      --
                                          ---------  -                -----
                                           ---------  ------        -------
                                             -------  ---------  ----------
                                                ----  ---------- ----------
                                                  --  ---------- ----------
   Welcome to Terraform Cloud!                     -  ---------- -------
                                                      ---  ----- ---
   Documentation: terraform.io/docs/cloud             --------   -
                                                      ----------
                                                      ----------
                                                       ---------
                                                           -----
                                                               -
```

```sh
$ terraform init

Initializing Terraform Cloud...
Do you wish to proceed?
  As part of migrating to Terraform Cloud, Terraform can optionally copy your
  current workspace state to the configured Terraform Cloud workspace.

  Answer "yes" to copy the latest state snapshot to the configured
  Terraform Cloud workspace.

$ yes

Terraform Cloud has been successfully initialized!
```

테라폼 클라우드에 상태를 마이그레이션 한 후에는 로컬에 있는 상태 파일을 삭제한다.

```sh
$ rm terraform.tfstate
```

이제 로컬이 아닌 원격에서 provider에 접근하기 때문에 관련해서 인증 토큰이 필요하다.

**Workspace variables**에 들어가서 **Environment variable**에 'Sensitive'를 체크하여 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`를 등록한다.

`apply` 명령어를 통해 트리거를 작동시키면 원격에서 수행하는 것을 볼 수 있다.

```sh
$ terraform apply
```

> [https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-remote](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-remote)