# Simplifique-o-Gerenciamento-de-Infraestrutura-com-Terraform-na-AWS

Para simplificar o gerenciamento de infraestrutura com Terraform na AWS, vamos estruturar o projeto em módulos e alinhado à esquerda, como solicitado. Abaixo estão os passos e exemplos de código para configurar um ambiente básico na AWS usando Terraform.

### Passo 1: Configuração do Ambiente

1. **Instalação do Terraform:**
   Certifique-se de ter o Terraform instalado localmente. Você pode baixá-lo em [terraform.io](https://www.terraform.io/downloads.html) e seguir as instruções de instalação para seu sistema operacional.

2. **Configuração das Credenciais da AWS:**
   Configure suas credenciais da AWS localmente para que o Terraform possa acessar sua conta AWS. Você pode configurar isso através do AWS CLI ou diretamente no arquivo de configuração `~/.aws/credentials`.

### Passo 2: Estrutura do Projeto

Estruture seu projeto Terraform da seguinte maneira:

```plaintext
terraform-aws-project/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
└── modules/
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    ├── ec2/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── iam/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### Passo 3: Implementação dos Módulos

#### 3.1. Módulo VPC

Crie um módulo para configurar a Virtual Private Cloud (VPC) na AWS.

**`modules/vpc/main.tf`**
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr_block
  enable_dns_support = true
  enable_dns_hostnames = true

  tags = {
    Name = var.vpc_name
  }
}

resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${var.vpc_name}-gw"
  }
}

output "vpc_id" {
  value = aws_vpc.main.id
}
```

**`modules/vpc/variables.tf`**
```hcl
variable "vpc_name" {
  description = "Name of the VPC"
  type        = string
}

variable "vpc_cidr_block" {
  description = "CIDR block for the VPC"
  type        = string
}
```

**`modules/vpc/outputs.tf`**
```hcl
output "vpc_id" {
  description = "ID of the created VPC"
  value       = aws_vpc.main.id
}
```

#### 3.2. Módulo EC2

Crie um módulo para configurar instâncias EC2 na AWS.

**`modules/ec2/main.tf`**
```hcl
resource "aws_instance" "web" {
  ami           = var.ec2_ami
  instance_type = var.ec2_instance_type
  subnet_id     = var.subnet_id

  tags = {
    Name = "Web Server"
  }
}

output "ec2_instance_id" {
  value = aws_instance.web.id
}
```

**`modules/ec2/variables.tf`**
```hcl
variable "ec2_ami" {
  description = "AMI ID for EC2 instance"
  type        = string
}

variable "ec2_instance_type" {
  description = "Instance type for EC2 instance"
  type        = string
}

variable "subnet_id" {
  description = "ID of the subnet to launch EC2 instance"
  type        = string
}
```

**`modules/ec2/outputs.tf`**
```hcl
output "ec2_instance_id" {
  description = "ID of the created EC2 instance"
  value       = aws_instance.web.id
}
```

#### 3.3. Módulo IAM

Crie um módulo para configurar políticas e papéis do IAM na AWS.

**`modules/iam/main.tf`**
```hcl
resource "aws_iam_role" "example" {
  name = "example-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect    = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
        Action    = "sts:AssumeRole"
      }
    ]
  })
}

output "iam_role_arn" {
  value = aws_iam_role.example.arn
}
```

**`modules/iam/variables.tf`**
```hcl
variable "role_name" {
  description = "Name of the IAM role"
  type        = string
}
```

**`modules/iam/outputs.tf`**
```hcl
output "iam_role_arn" {
  description = "ARN of the created IAM role"
  value       = aws_iam_role.example.arn
}
```

### Passo 4: Configuração Principal (main.tf, variables.tf, outputs.tf)

**`main.tf`**
```hcl
provider "aws" {
  region = var.aws_region
}

module "vpc" {
  source      = "./modules/vpc"
  vpc_name    = var.vpc_name
  vpc_cidr_block = var.vpc_cidr_block
}

module "ec2" {
  source            = "./modules/ec2"
  ec2_ami           = var.ec2_ami
  ec2_instance_type = var.ec2_instance_type
  subnet_id         = module.vpc.vpc_id
}

module "iam" {
  source    = "./modules/iam"
  role_name = var.role_name
}
```

**`variables.tf`**
```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
}

variable "vpc_name" {
  description = "Name of the VPC"
  type        = string
}

variable "vpc_cidr_block" {
  description = "CIDR block for the VPC"
  type        = string
}

variable "ec2_ami" {
  description = "AMI ID for EC2 instance"
  type        = string
}

variable "ec2_instance_type" {
  description = "Instance type for EC2 instance"
  type        = string
}

variable "role_name" {
  description = "Name of the IAM role"
  type        = string
}
```

**`outputs.tf`**
```hcl
output "vpc_id" {
  description = "ID of the created VPC"
  value       = module.vpc.vpc_id
}

output "ec2_instance_id" {
  description = "ID of the created EC2 instance"
  value       = module.ec2.ec2_instance_id
}

output "iam_role_arn" {
  description = "ARN of the created IAM role"
  value       = module.iam.iam_role_arn
}
```

### Passo 5: Execução do Terraform

1. **Inicialize o Terraform:**
   ```bash
   terraform init
   ```

2. **Visualize as Mudanças Planejadas:**
   ```bash
   terraform plan
   ```

3. **Aplicar as Mudanças:**
   ```bash
   terraform apply
   ```

### Observações Finais

Certifique-se de substituir os valores das variáveis (`vpc_cidr_block`, `ec2_ami`, `ec2_instance_type`, etc.) pelos valores apropriados para sua configuração específica na AWS. Este exemplo oferece uma base para começarmos a gerenciar a nossa infraestrutura na AWS de forma automatizada usando Terraform, permitindo escalabilidade, controle e fácil atualização dos recursos.

Este projeto pode ser expandido adicionando mais módulos e recursos conforme nossas necessidades de infraestrutura crescem.
