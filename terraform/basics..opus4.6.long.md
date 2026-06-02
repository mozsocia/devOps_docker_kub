# Terraform Basics: A Complete Beginner's Guide (Step-by-Step)

---

## Table of Contents

1. [What is Terraform?](#1-what-is-terraform)
2. [HCL Syntax Fundamentals](#2-hcl-syntax-fundamentals)
3. [Block Types](#3-block-types)
4. [Labels](#4-labels)
5. [Arguments](#5-arguments)
6. [Nested Blocks](#6-nested-blocks)
7. [Variables](#7-variables)
8. [Outputs](#8-outputs)
9. [Data Sources](#9-data-sources)
10. [State](#10-state)
11. [Providers](#11-providers)
12. [Modules](#12-modules)
13. [Expressions & Functions](#13-expressions--functions)
14. [Meta-Arguments](#14-meta-arguments)
15. [Lifecycle](#15-lifecycle)
16. [Provisioners](#16-provisioners)
17. [Backends](#17-backends)
18. [Workspaces](#18-workspaces)
19. [The Core Workflow](#19-the-core-workflow)
20. [Complete Real-World Example](#20-complete-real-world-example)

---

## 1. What is Terraform?

Terraform is an **Infrastructure as Code (IaC)** tool built by HashiCorp. It lets you define, provision, and manage cloud infrastructure using a **declarative** configuration language called **HCL (HashiCorp Configuration Language)**.

### Key Concepts

```
You DESCRIBE what you want     →   Terraform FIGURES OUT how to create it
(Declarative)                      (Execution Plan)
```

### Install Terraform

```bash
# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Ubuntu/Debian
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Windows (chocolatey)
choco install terraform

# Verify
terraform -version
```

---

## 2. HCL Syntax Fundamentals

Everything in Terraform is written in **HCL**. The anatomy of HCL is:

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   <BLOCK_TYPE> "<LABEL_1>" "<LABEL_2>" {                    │
│       <ARGUMENT_NAME> = <ARGUMENT_VALUE>                    │
│                                                             │
│       <NESTED_BLOCK> {                                      │
│           <ARGUMENT_NAME> = <ARGUMENT_VALUE>                │
│       }                                                     │
│   }                                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Concrete example (annotated):

```hcl
#  BLOCK TYPE   LABEL 1        LABEL 2
#     │            │              │
#     ▼            ▼              ▼
  resource    "aws_instance"   "web_server" {

      # ARGUMENT_NAME   =   ARGUMENT_VALUE
      #      │                    │
      #      ▼                    ▼
          ami           =   "ami-0c55b159cbfafe1f0"
          instance_type =   "t2.micro"

      #  NESTED BLOCK
      #      │
      #      ▼
          tags {
              Name = "MyServer"     # ← Argument inside nested block
          }
  }
```

---

## 3. Block Types

A **block** is a container for configuration. Terraform has **several top-level block types**:

```
┌──────────────────────────────────────────────────────────────────┐
│                    TERRAFORM BLOCK TYPES                         │
├──────────────────┬───────────────────────────────────────────────┤
│ Block Type       │ Purpose                                      │
├──────────────────┼───────────────────────────────────────────────┤
│ terraform        │ Terraform settings (version, backend, etc.)  │
│ provider         │ Configure a cloud/service provider           │
│ resource         │ Define infrastructure objects to CREATE       │
│ data             │ Fetch/read EXISTING infrastructure info      │
│ variable         │ Declare input parameters                     │
│ output           │ Declare output values                        │
│ locals           │ Define local/computed values                  │
│ module           │ Call/reuse a module                           │
│ provisioner      │ Run scripts on resources (inside resource)   │
│ moved            │ Track resource refactoring                   │
│ import           │ Import existing infrastructure (v1.5+)       │
│ check            │ Post-apply health checks (v1.5+)             │
└──────────────────┴───────────────────────────────────────────────┘
```

### 3.1 — `terraform` Block

```hcl
terraform {
  # Which version of Terraform CLI is required
  required_version = ">= 1.5.0"

  # Which providers are needed and their versions
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  # Where to store the state file
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"
  }
}
```

### 3.2 — `provider` Block

```hcl
provider "aws" {
  region  = "us-east-1"
  profile = "my-aws-profile"
}

# You can have MULTIPLE providers (aliased)
provider "aws" {
  alias  = "europe"
  region = "eu-west-1"
}
```

### 3.3 — `resource` Block

```hcl
# Creates infrastructure
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
```

### 3.4 — `data` Block

```hcl
# Reads existing infrastructure (does NOT create anything)
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}
```

### 3.5 — `variable` Block

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}
```

### 3.6 — `output` Block

```hcl
output "instance_ip" {
  description = "Public IP of the EC2 instance"
  value       = aws_instance.web.public_ip
}
```

### 3.7 — `locals` Block

```hcl
locals {
  environment = "production"
  name_prefix = "myapp-${local.environment}"
  common_tags = {
    Environment = local.environment
    ManagedBy   = "Terraform"
  }
}
```

### 3.8 — `module` Block

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"
}
```

---

## 4. Labels

Labels are the **names/identifiers** that come after the block type keyword. Different block types take different numbers of labels.

```
┌──────────────────┬────────────┬───────────────────────────────────────┐
│ Block Type       │ # Labels   │ Format                                │
├──────────────────┼────────────┼───────────────────────────────────────┤
│ terraform        │ 0          │ terraform { }                         │
│ locals           │ 0          │ locals { }                            │
│ provider         │ 1          │ provider "<NAME>" { }                 │
│ variable         │ 1          │ variable "<NAME>" { }                 │
│ output           │ 1          │ output "<NAME>" { }                   │
│ resource         │ 2          │ resource "<TYPE>" "<NAME>" { }        │
│ data             │ 2          │ data "<TYPE>" "<NAME>" { }            │
│ module           │ 1          │ module "<NAME>" { }                   │
│ provisioner      │ 1          │ provisioner "<TYPE>" { }  (nested)    │
└──────────────────┴────────────┴───────────────────────────────────────┘
```

### Label Breakdown for `resource`

```hcl
resource "aws_instance" "web_server" { }
#           │                │
#           │                └── LABEL 2: Local name (your identifier)
#           │                    Used to reference THIS resource in config
#           │                    e.g., aws_instance.web_server.id
#           │
#           └── LABEL 1: Resource type
#               Format: <PROVIDER>_<RESOURCE>
#               "aws"      = provider name
#               "instance" = resource kind
```

### Label Breakdown for `data`

```hcl
data "aws_ami" "latest_ubuntu" { }
#       │             │
#       │             └── LABEL 2: Local name
#       │                 Referenced as: data.aws_ami.latest_ubuntu.id
#       │
#       └── LABEL 1: Data source type
```

### Label Naming Rules

```hcl
# ✅ VALID names
resource "aws_instance" "web_server" { }
resource "aws_instance" "web-server" { }    # hyphens ok but not recommended
resource "aws_instance" "server_1" { }

# ❌ INVALID names
resource "aws_instance" "1_server" { }      # cannot start with a number
resource "aws_instance" "web server" { }    # no spaces
resource "aws_instance" "web.server" { }    # no dots
```

---

## 5. Arguments

Arguments are **key = value** pairs inside a block. They configure the behavior of that block.

### 5.1 — Argument Syntax

```hcl
resource "aws_instance" "example" {
  #  ARGUMENT_NAME  =  ARGUMENT_VALUE
  #       │                 │
  ami           = "ami-0c55b159cbfafe1f0"    # String
  instance_type = "t2.micro"                  # String
  count         = 3                           # Number
  monitoring    = true                        # Boolean
  tags          = { Name = "Example" }        # Map
  security_groups = ["sg-123", "sg-456"]      # List
}
```

### 5.2 — Value Types

```hcl
# ┌─────────────────────────────────────────────────────────┐
# │                  TERRAFORM VALUE TYPES                   │
# └─────────────────────────────────────────────────────────┘

# STRING — Quoted text
ami = "ami-0c55b159cbfafe1f0"

# NUMBER — No quotes, integer or float
count = 3
cpu   = 1.5

# BOOLEAN — true or false (no quotes)
monitoring = true
public     = false

# LIST (Tuple) — Ordered collection in [ ]
availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
ports              = [80, 443, 8080]

# MAP (Object) — Key-value pairs in { }
tags = {
  Name        = "WebServer"
  Environment = "production"
}

# SET — Like list, but unordered with no duplicates
# (Declared via type constraint, syntax same as list)
security_groups = toset(["sg-123", "sg-456"])

# NULL — Represents absence of a value
special_config = null
```

### 5.3 — Required vs Optional Arguments

```hcl
resource "aws_instance" "example" {
  # REQUIRED — Must be provided, no default exists
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  # OPTIONAL — Has a default or can be omitted
  monitoring                  = true          # default: false
  disable_api_termination     = false         # default: false
  associate_public_ip_address = true          # default: depends on subnet
}
```

### 5.4 — Argument References (Accessing Other Resources)

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  # REFERENCE another resource's attribute
  #     RESOURCE_TYPE.LOCAL_NAME.ATTRIBUTE
  #          │             │         │
  vpc_id = aws_vpc.main.id    # ← This is an argument reference

  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
}
```

### Reference Syntax Cheat Sheet

```hcl
# Resource attribute
aws_instance.web.id
aws_instance.web.public_ip

# Variable
var.instance_type

# Local value
local.common_tags

# Data source
data.aws_ami.ubuntu.id

# Module output
module.vpc.vpc_id

# Terraform built-ins
terraform.workspace
path.module
path.root
```

---

## 6. Nested Blocks

A **nested block** is a block defined **inside** another block. They represent **sub-configurations** or **repeated sub-structures**.

### 6.1 — Basic Nested Block

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  # ┌───────────────────────────────────┐
  # │       NESTED BLOCK (tags)         │
  # └───────────────────────────────────┘
  tags = {
    Name = "WebServer"
  }

  # ┌───────────────────────────────────┐
  # │   NESTED BLOCK (root_block_device)│
  # └───────────────────────────────────┘
  root_block_device {
    volume_size = 20
    volume_type = "gp3"
    encrypted   = true
  }

  # ┌───────────────────────────────────┐
  # │ NESTED BLOCK (network_interface)  │
  # └───────────────────────────────────┘
  network_interface {
    network_interface_id = aws_network_interface.web.id
    device_index         = 0
  }
}
```

### 6.2 — Multiple Nested Blocks of the Same Type

```hcl
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id

  # FIRST ingress rule
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow HTTP"
  }

  # SECOND ingress rule (same block type, repeated)
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow HTTPS"
  }

  # Egress rule
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow all outbound"
  }
}
```

### 6.3 — Deeply Nested Blocks

```hcl
resource "aws_lb_listener" "front_end" {
  load_balancer_arn = aws_lb.front_end.arn
  port              = 443
  protocol          = "HTTPS"

  # LEVEL 1 NESTED BLOCK
  default_action {
    type = "forward"

    # LEVEL 2 NESTED BLOCK (nested inside nested!)
    forward {

      # LEVEL 3 NESTED BLOCK
      target_group {
        arn    = aws_lb_target_group.primary.arn
        weight = 80
      }

      # Another LEVEL 3 NESTED BLOCK
      target_group {
        arn    = aws_lb_target_group.canary.arn
        weight = 20
      }

      stickiness {
        enabled  = true
        duration = 600
      }
    }
  }
}
```

### 6.4 — Nested Blocks vs Arguments: Know the Difference

```hcl
resource "aws_instance" "example" {

  # This is an ARGUMENT (uses = sign with { })
  # It's a MAP type value
  tags = {
    Name = "Example"
  }

  # This is a NESTED BLOCK (NO = sign, just { })
  # It has its own schema/structure
  root_block_device {
    volume_size = 20
  }

  # RULE OF THUMB:
  # ┌──────────────────────────────────────────────────┐
  # │ ARGUMENT:     name = { ... }   (has = sign)     │
  # │ NESTED BLOCK: name { ... }     (NO = sign)      │
  # └──────────────────────────────────────────────────┘
}
```

### 6.5 — `dynamic` Blocks (Dynamic Nested Blocks)

When you need to generate nested blocks programmatically:

```hcl
variable "ingress_rules" {
  type = list(object({
    port        = number
    description = string
    cidr_blocks = list(string)
  }))
  default = [
    { port = 80,  description = "HTTP",  cidr_blocks = ["0.0.0.0/0"] },
    { port = 443, description = "HTTPS", cidr_blocks = ["0.0.0.0/0"] },
    { port = 22,  description = "SSH",   cidr_blocks = ["10.0.0.0/8"] },
  ]
}

resource "aws_security_group" "web" {
  name   = "web-sg"
  vpc_id = aws_vpc.main.id

  # DYNAMIC BLOCK — generates multiple 'ingress' nested blocks
  dynamic "ingress" {                          # ← "ingress" = block type to generate
    for_each = var.ingress_rules               # ← iterate over this collection
    content {                                  # ← defines the BODY of each block
      from_port   = ingress.value.port         # ← ingress.value = current item
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ingress.value.cidr_blocks
      description = ingress.value.description
    }
  }
}

# The above is EQUIVALENT to writing 3 separate ingress { } blocks manually
```

---

## 7. Variables (Input Variables)

### 7.1 — Declaring Variables

```hcl
# ─── variables.tf ─────────────────────────────────────────

# SIMPLE variable with default
variable "region" {
  description = "AWS region to deploy resources"
  type        = string
  default     = "us-east-1"
}

# REQUIRED variable (no default = must be provided)
variable "project_name" {
  description = "Name of the project"
  type        = string
  # No default! This MUST be provided.
}

# NUMBER variable with validation
variable "instance_count" {
  description = "Number of EC2 instances"
  type        = number
  default     = 1

  validation {
    condition     = var.instance_count > 0 && var.instance_count <= 10
    error_message = "Instance count must be between 1 and 10."
  }
}

# BOOLEAN variable
variable "enable_monitoring" {
  description = "Enable detailed monitoring"
  type        = bool
  default     = false
}

# LIST variable
variable "availability_zones" {
  description = "List of AZs"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

# MAP variable
variable "extra_tags" {
  description = "Additional tags"
  type        = map(string)
  default     = {}
}

# COMPLEX OBJECT variable
variable "database_config" {
  description = "Database configuration"
  type = object({
    engine         = string
    engine_version = string
    instance_class = string
    storage_gb     = number
    multi_az       = bool
  })
  default = {
    engine         = "postgres"
    engine_version = "15.3"
    instance_class = "db.t3.micro"
    storage_gb     = 20
    multi_az       = false
  }
}

# SENSITIVE variable (won't show in logs/output)
variable "db_password" {
  description = "Database master password"
  type        = string
  sensitive   = true
}
```

### 7.2 — Using Variables

```hcl
# ─── main.tf ─────────────────────────────────────────

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type          # ← var.<name>
  count         = var.instance_count
  monitoring    = var.enable_monitoring

  tags = merge(var.extra_tags, {
    Name    = "${var.project_name}-web-${count.index}"
    Project = var.project_name
  })
}
```

### 7.3 — Providing Variable Values (5 Methods, in Order of Precedence)

```
┌────┬────────────────────────────────────────────────────────────┐
│ #  │ Method (Lowest → Highest precedence)                      │
├────┼────────────────────────────────────────────────────────────┤
│ 1  │ default value in variable block                           │
│ 2  │ terraform.tfvars or terraform.tfvars.json (auto-loaded)   │
│ 3  │ *.auto.tfvars or *.auto.tfvars.json (auto-loaded)         │
│ 4  │ -var-file flag                                            │
│ 5  │ -var flag or TF_VAR_ environment variable                 │
└────┴────────────────────────────────────────────────────────────┘
```

```hcl
# ─── terraform.tfvars ─────────────────────────────────
region         = "us-west-2"
project_name   = "myapp"
instance_count = 2
extra_tags = {
  Owner = "devteam"
}
```

```hcl
# ─── prod.tfvars ──────────────────────────────────────
region             = "us-east-1"
instance_count     = 5
enable_monitoring  = true
```

```bash
# Command line methods
terraform plan -var="project_name=myapp"
terraform plan -var-file="prod.tfvars"

# Environment variable
export TF_VAR_project_name="myapp"
export TF_VAR_db_password="supersecret"
terraform plan
```

### 7.4 — Variable Type Constraints (Full Reference)

```hcl
# PRIMITIVE TYPES
type = string
type = number
type = bool

# COLLECTION TYPES
type = list(string)                    # ["a", "b", "c"]
type = set(string)                     # Unique, unordered
type = map(string)                     # { key = "value" }

# STRUCTURAL TYPES
type = object({
  name = string
  age  = number
  tags = map(string)
})

type = tuple([string, number, bool])   # ["hello", 42, true]

# ANY type (accepts anything)
type = any

# NESTED collections
type = list(object({
  name = string
  port = number
}))

type = map(list(string))
```

---

## 8. Outputs

Outputs expose values after `terraform apply`. They're used for:
- Displaying info to the user
- Passing data between modules
- Feeding into other tools/scripts

```hcl
# ─── outputs.tf ───────────────────────────────────────

# BASIC output
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

# OUTPUT with multiple values
output "instance_public_ips" {
  description = "Public IPs of all instances"
  value       = aws_instance.web[*].public_ip       # splat expression
}

# SENSITIVE output (hidden from CLI display)
output "db_connection_string" {
  description = "Database connection string"
  value       = "postgresql://${var.db_user}:${var.db_password}@${aws_db_instance.main.endpoint}/mydb"
  sensitive   = true
}

# CONDITIONAL output
output "lb_dns_name" {
  description = "Load balancer DNS name"
  value       = var.create_lb ? aws_lb.main[0].dns_name : null
}

# COMPLEX output
output "instance_details" {
  value = {
    id         = aws_instance.web.id
    public_ip  = aws_instance.web.public_ip
    private_ip = aws_instance.web.private_ip
    az         = aws_instance.web.availability_zone
  }
}
```

```bash
# View outputs after apply
terraform output
terraform output instance_id
terraform output -json
terraform output -raw instance_id    # No quotes, useful for scripting
```

---

## 9. Data Sources

Data sources let you **read** information from existing infrastructure or external sources. They do **NOT create** anything.

```hcl
# ─── data.tf ──────────────────────────────────────────

# Fetch the latest Ubuntu AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]    # Canonical's AWS account ID

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Get current AWS account info
data "aws_caller_identity" "current" {}

# Get available AZs
data "aws_availability_zones" "available" {
  state = "available"
}

# Read an existing VPC
data "aws_vpc" "existing" {
  id = "vpc-0123456789abcdef0"
}

# Read a file
data "local_file" "config" {
  filename = "${path.module}/config.json"
}

# Render a template
data "template_file" "user_data" {
  template = file("${path.module}/userdata.tpl")
  vars = {
    server_name = var.project_name
    environment = var.environment
  }
}

# ─── Usage ────────────────────────────────────────────

resource "aws_instance" "web" {
  ami               = data.aws_ami.ubuntu.id              # ← data.<TYPE>.<NAME>.<ATTR>
  instance_type     = "t2.micro"
  availability_zone = data.aws_availability_zones.available.names[0]

  tags = {
    Account = data.aws_caller_identity.current.account_id
  }
}
```

---

## 10. State

Terraform keeps track of your infrastructure in a **state file** (`terraform.tfstate`).

```
┌─────────────────────────────────────────────────────────────┐
│                    TERRAFORM STATE                           │
│                                                             │
│  Your Config (.tf)  ←→  State File  ←→  Real Infrastructure│
│                                                             │
│  "What you want"        "What TF        "What actually     │
│                          knows about"    exists"            │
└─────────────────────────────────────────────────────────────┘
```

### State Commands

```bash
# List all resources in state
terraform state list

# Show details of a specific resource
terraform state show aws_instance.web

# Move a resource (rename in state)
terraform state mv aws_instance.web aws_instance.app

# Remove from state (doesn't destroy the real resource)
terraform state rm aws_instance.web

# Import existing resource into state
terraform import aws_instance.web i-0123456789abcdef0

# Pull remote state to local
terraform state pull

# Force push local state to remote
terraform state push
```

---

## 11. Providers

Providers are **plugins** that let Terraform interact with APIs (AWS, Azure, GCP, Kubernetes, etc.).

```hcl
# ─── providers.tf ─────────────────────────────────────

terraform {
  required_providers {
    # AWS Provider
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"          # >= 5.0, < 6.0
    }

    # Random Provider (for generating random values)
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }

    # Docker Provider
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0"
    }
  }
}

# Configure the AWS provider
provider "aws" {
  region  = var.region
  profile = var.aws_profile

  default_tags {
    tags = {
      ManagedBy   = "Terraform"
      Project     = var.project_name
    }
  }
}

# Multiple providers with ALIASES
provider "aws" {
  alias   = "us_west"
  region  = "us-west-2"
}

provider "aws" {
  alias   = "eu"
  region  = "eu-west-1"
}

# Using aliased provider
resource "aws_instance" "west_server" {
  provider      = aws.us_west          # ← specify which provider
  ami           = "ami-xxxxx"
  instance_type = "t2.micro"
}

resource "aws_instance" "eu_server" {
  provider      = aws.eu
  ami           = "ami-yyyyy"
  instance_type = "t2.micro"
}
```

### Version Constraints

```hcl
version = "5.0.0"       # Exact version
version = ">= 5.0"      # 5.0 or newer
version = "~> 5.0"      # >= 5.0, < 6.0 (minor version flexibility)
version = "~> 5.0.0"    # >= 5.0.0, < 5.1.0 (patch version flexibility)
version = ">= 5.0, < 6.0"  # Explicit range
```

---

## 12. Modules

Modules are **reusable packages** of Terraform configuration. Every Terraform config is technically a module (the **root module**).

### 12.1 — Module Structure

```
project/
├── main.tf                    # Root module
├── variables.tf
├── outputs.tf
├── terraform.tfvars
│
└── modules/                   # Child modules
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    ├── ec2/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    └── rds/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### 12.2 — Creating a Module

```hcl
# ─── modules/ec2/variables.tf ────────────────────────
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "ami_id" {
  type = string
}

variable "subnet_id" {
  type = string
}

variable "name" {
  type = string
}

variable "tags" {
  type    = map(string)
  default = {}
}
```

```hcl
# ─── modules/ec2/main.tf ─────────────────────────────
resource "aws_instance" "this" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id

  tags = merge(var.tags, {
    Name = var.name
  })
}
```

```hcl
# ─── modules/ec2/outputs.tf ──────────────────────────
output "instance_id" {
  value = aws_instance.this.id
}

output "public_ip" {
  value = aws_instance.this.public_ip
}

output "private_ip" {
  value = aws_instance.this.private_ip
}
```

### 12.3 — Calling a Module

```hcl
# ─── main.tf (root module) ───────────────────────────

# Local module
module "web_server" {
  source = "./modules/ec2"              # ← local path

  name          = "web-server"
  ami_id        = data.aws_ami.ubuntu.id
  instance_type = "t3.small"
  subnet_id     = module.vpc.public_subnet_ids[0]
  tags          = local.common_tags
}

# Terraform Registry module
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = true
}

# Git module
module "security" {
  source = "git::https://github.com/myorg/terraform-modules.git//security?ref=v1.2.0"
}

# S3 module
module "config" {
  source = "s3::https://s3-eu-west-1.amazonaws.com/my-bucket/modules/config.zip"
}

# Access module outputs
output "web_ip" {
  value = module.web_server.public_ip     # ← module.<NAME>.<OUTPUT>
}
```

---

## 13. Expressions & Functions

### 13.1 — String Interpolation & Templates

```hcl
# String interpolation
name = "server-${var.environment}-${count.index + 1}"

# Heredoc (multi-line strings)
description = <<-EOT
  This is a multi-line
  description for the
  ${var.project_name} project.
EOT

# Directive (conditional in templates)
user_data = <<-EOF
  #!/bin/bash
  %{ if var.environment == "production" }
  echo "Production mode"
  %{ else }
  echo "Development mode"
  %{ endif }
EOF
```

### 13.2 — Operators

```hcl
# ARITHMETIC
count = var.base_count + 2
size  = var.base_size * 3

# COMPARISON
condition = var.environment == "production"
condition = var.instance_count > 5
condition = var.instance_count >= 1
condition = var.instance_count != 0

# LOGICAL
condition = var.enable_monitoring && var.environment == "prod"
condition = var.use_spot || var.environment == "dev"
condition = !var.disable_feature
```

### 13.3 — Conditional Expression (Ternary)

```hcl
# condition ? true_value : false_value

instance_type = var.environment == "production" ? "t3.large" : "t3.micro"

count = var.create_resource ? 1 : 0

subnet_id = var.is_public ? aws_subnet.public.id : aws_subnet.private.id
```

### 13.4 — Commonly Used Functions

```hcl
# ── STRING FUNCTIONS ──────────────────────────────────
upper("hello")                          # "HELLO"
lower("HELLO")                          # "hello"
title("hello world")                    # "Hello World"
trimspace("  hello  ")                  # "hello"
replace("hello-world", "-", "_")        # "hello_world"
substr("hello world", 0, 5)             # "hello"
join(", ", ["a", "b", "c"])             # "a, b, c"
split(",", "a,b,c")                     # ["a", "b", "c"]
format("Hello, %s! You are %d.", "Bob", 25)   # "Hello, Bob! You are 25."
regex("^(?:web|app)-(.+)$", "web-server")     # ["server"]

# ── NUMERIC FUNCTIONS ─────────────────────────────────
min(1, 2, 3)                            # 1
max(1, 2, 3)                            # 3
ceil(4.3)                               # 5
floor(4.9)                              # 4
abs(-10)                                # 10

# ── COLLECTION FUNCTIONS ──────────────────────────────
length(["a", "b", "c"])                 # 3
concat(["a"], ["b"], ["c"])             # ["a", "b", "c"]
merge({a=1}, {b=2})                     # {a=1, b=2}
flatten([["a"], ["b", "c"]])            # ["a", "b", "c"]
distinct(["a", "b", "a"])               # ["a", "b"]
sort(["c", "a", "b"])                   # ["a", "b", "c"]
contains(["a", "b", "c"], "b")          # true
index(["a", "b", "c"], "b")            # 1
element(["a", "b", "c"], 1)             # "b"
slice(["a", "b", "c", "d"], 1, 3)       # ["b", "c"]
zipmap(["a", "b"], [1, 2])              # {a=1, b=2}
keys({a=1, b=2})                        # ["a", "b"]
values({a=1, b=2})                      # [1, 2]
lookup({a=1, b=2}, "a", 0)              # 1  (default: 0)
toset(["a", "b", "a"])                  # toset(["a", "b"])
tolist(toset(["c", "b", "a"]))          # ["a", "b", "c"]
tomap({a = "1"})                        # {a = "1"}

# ── TYPE CONVERSION ───────────────────────────────────
tostring(42)                            # "42"
tonumber("42")                          # 42
tobool("true")                          # true

# ── FILE FUNCTIONS ────────────────────────────────────
file("${path.module}/script.sh")                # Read file contents
fileexists("${path.module}/script.sh")          # true/false
templatefile("${path.module}/tmpl.tftpl", {     # Render template
  name = "web"
  port = 8080
})
filebase64("${path.module}/file.bin")           # Base64 encode
base64decode("SGVsbG8=")                        # "Hello"
jsonencode({a = 1, b = "two"})                  # '{"a":1,"b":"two"}'
jsondecode("{\"a\":1}")                         # {a = 1}
yamlencode({a = 1})                             # YAML string
yamldecode("a: 1\n")                            # {a = 1}

# ── DATE/TIME ─────────────────────────────────────────
timestamp()                              # "2024-01-15T10:30:00Z"
formatdate("YYYY-MM-DD", timestamp())    # "2024-01-15"

# ── HASH/CRYPTO ───────────────────────────────────────
md5("hello")                             # MD5 hash
sha256("hello")                          # SHA256 hash
uuid()                                   # Random UUID
base64encode("hello")                    # "aGVsbG8="

# ── IP NETWORK ────────────────────────────────────────
cidrsubnet("10.0.0.0/16", 8, 1)         # "10.0.1.0/24"
cidrhost("10.0.1.0/24", 5)              # "10.0.1.5"
cidrnetmask("10.0.0.0/16")              # "255.255.0.0"

# ── FOR EXPRESSION ────────────────────────────────────
# List comprehension
[for s in var.list : upper(s)]                       # ["A", "B", "C"]
[for s in var.list : upper(s) if s != "b"]           # filtered

# Map comprehension
{for s in var.list : s => upper(s)}                  # {a="A", b="B"}
{for k, v in var.map : k => "${v}-modified"}         # transform values
```

### 13.5 — Splat Expressions

```hcl
# Splat expression (shorthand for extracting attributes from lists)

# Get ALL instance IDs
output "all_ids" {
  value = aws_instance.web[*].id
  # Equivalent to: [for i in aws_instance.web : i.id]
}

# With nested attributes
output "all_private_ips" {
  value = aws_instance.web[*].private_ip
}
```

---

## 14. Meta-Arguments

Meta-arguments change the **behavior** of a resource block itself. They work on any resource type.

### 14.1 — `count`

```hcl
# Create multiple identical resources
resource "aws_instance" "web" {
  count = 3                              # Creates 3 instances

  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"

  tags = {
    Name = "web-server-${count.index}"   # count.index = 0, 1, 2
  }
}

# Reference: aws_instance.web[0], aws_instance.web[1], ...

# Conditional creation (create or skip)
resource "aws_instance" "bastion" {
  count = var.create_bastion ? 1 : 0     # 1 = create, 0 = skip

  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}

# Reference when count is conditional:
# aws_instance.bastion[0].id  (must use index)
```

### 14.2 — `for_each`

```hcl
# ── for_each with a SET ──────────────────────────────
resource "aws_iam_user" "developers" {
  for_each = toset(["alice", "bob", "charlie"])

  name = each.value                      # each.value = current item
}
# Reference: aws_iam_user.developers["alice"]

# ── for_each with a MAP ──────────────────────────────
variable "instances" {
  default = {
    web = { type = "t3.small",  az = "us-east-1a" }
    api = { type = "t3.medium", az = "us-east-1b" }
    db  = { type = "t3.large",  az = "us-east-1c" }
  }
}

resource "aws_instance" "servers" {
  for_each = var.instances

  ami               = data.aws_ami.ubuntu.id
  instance_type     = each.value.type    # each.key = "web", "api", "db"
  availability_zone = each.value.az      # each.value = { type=..., az=... }

  tags = {
    Name = "${each.key}-server"
  }
}
# Reference: aws_instance.servers["web"].id
```

### `count` vs `for_each`

```
┌─────────────────────────────────────────────────────────────────┐
│           count                    │        for_each            │
├────────────────────────────────────┼────────────────────────────┤
│ Indexed by NUMBER (0, 1, 2...)     │ Indexed by STRING key      │
│ Removing item 0 shifts all others  │ Each item is independent   │
│ Good for: identical resources      │ Good for: unique resources │
│ Good for: conditional creation     │ Good for: maps/sets        │
│                                    │                            │
│ ⚠️  Removing from middle causes   │ ✅ Stable references       │
│    recreation of subsequent items  │    No shifting             │
└────────────────────────────────────┴────────────────────────────┘
```

### 14.3 — `depends_on`

```hcl
# Explicit dependency (when Terraform can't auto-detect it)
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"

  depends_on = [
    aws_iam_role_policy.s3_access,       # Wait for IAM policy first
    aws_security_group.web,              # Wait for SG first
  ]
}
```

### 14.4 — `provider`

```hcl
# Use a specific (aliased) provider
resource "aws_instance" "eu_server" {
  provider = aws.eu                       # Use the "eu" aliased provider

  ami           = "ami-xxxxxxxx"
  instance_type = "t2.micro"
}
```

---

## 15. Lifecycle

The `lifecycle` nested block controls how Terraform handles resource creation, update, and destruction.

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  lifecycle {
    # ── create_before_destroy ──────────────────────────
    # Create the replacement BEFORE destroying the old one
    # (Prevents downtime)
    create_before_destroy = true

    # ── prevent_destroy ────────────────────────────────
    # Terraform will ERROR if you try to destroy this resource
    # (Protects critical resources like databases)
    prevent_destroy = true

    # ── ignore_changes ─────────────────────────────────
    # Ignore changes to specific attributes
    # (Useful when external processes modify these)
    ignore_changes = [
      tags,                    # Ignore tag changes
      ami,                     # Ignore AMI changes
      user_data,               # Ignore user data changes
    ]

    # Ignore ALL changes (manage creation/deletion only)
    # ignore_changes = all

    # ── replace_triggered_by ───────────────────────────
    # Force replacement when referenced resource changes
    replace_triggered_by = [
      aws_ami.custom.id,
    ]

    # ── precondition ───────────────────────────────────
    precondition {
      condition     = var.instance_type != "t2.nano"
      error_message = "t2.nano is too small for production."
    }

    # ── postcondition ──────────────────────────────────
    postcondition {
      condition     = self.public_ip != ""
      error_message = "Instance must have a public IP."
    }
  }
}
```

---

## 16. Provisioners

Provisioners run **scripts** or **commands** on a resource after creation. They are a **last resort** — prefer native resource configuration.

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  key_name      = aws_key_pair.deployer.key_name

  # ── file provisioner ────────────────────────────────
  # Upload files to the remote machine
  provisioner "file" {
    source      = "scripts/setup.sh"
    destination = "/tmp/setup.sh"

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }

  # ── remote-exec provisioner ─────────────────────────
  # Run commands on the remote machine
  provisioner "remote-exec" {
    inline = [
      "chmod +x /tmp/setup.sh",
      "sudo /tmp/setup.sh",
      "sudo systemctl start nginx",
    ]

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }

  # ── local-exec provisioner ──────────────────────────
  # Run commands on YOUR LOCAL machine
  provisioner "local-exec" {
    command = "echo ${self.public_ip} >> inventory.txt"
  }

  # ── Destroy-time provisioner ────────────────────────
  # Runs when the resource is DESTROYED
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Destroying instance ${self.id}' >> destroy.log"
  }

  # ── on_failure ──────────────────────────────────────
  provisioner "local-exec" {
    command    = "some-risky-command"
    on_failure = continue    # Don't fail the apply (default: fail)
  }
}
```

---

## 17. Backends

Backends define **where** Terraform stores its state file.

```hcl
# ── Local backend (default) ──────────────────────────
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}

# ── S3 backend (most common for AWS) ─────────────────
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/network/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"     # State locking
  }
}

# ── Azure Storage backend ────────────────────────────
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

# ── GCS backend ──────────────────────────────────────
terraform {
  backend "gcs" {
    bucket = "my-terraform-state"
    prefix = "prod"
  }
}

# ── Terraform Cloud backend ──────────────────────────
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "my-workspace"
    }
  }
}
```

---

## 18. Workspaces

Workspaces let you manage **multiple environments** (dev, staging, prod) with the same configuration.

```bash
# List workspaces
terraform workspace list

# Create a new workspace
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Switch workspace
terraform workspace select prod

# Show current workspace
terraform workspace show

# Delete a workspace
terraform workspace delete dev
```

```hcl
# Use workspace name in configuration
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = terraform.workspace == "prod" ? "t3.large" : "t3.micro"

  tags = {
    Environment = terraform.workspace
    Name        = "web-${terraform.workspace}"
  }
}

locals {
  env_config = {
    dev     = { instance_type = "t3.micro",  count = 1 }
    staging = { instance_type = "t3.small",  count = 2 }
    prod    = { instance_type = "t3.large",  count = 3 }
  }

  current = local.env_config[terraform.workspace]
}

resource "aws_instance" "app" {
  count         = local.current.count
  instance_type = local.current.instance_type
  ami           = data.aws_ami.ubuntu.id
}
```

---

## 19. The Core Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TERRAFORM CORE WORKFLOW                          │
│                                                                     │
│    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    │
│    │  Write   │───▶│   Init   │───▶│   Plan   │───▶│  Apply   │    │
│    │  (.tf)   │    │          │    │          │    │          │    │
│    └──────────┘    └──────────┘    └──────────┘    └──────────┘    │
│                                                         │          │
│                    ┌──────────┐                          │          │
│                    │ Destroy  │◀─── (when done) ────────┘          │
│                    └──────────┘                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Step-by-Step Commands

```bash
# ──────────────────────────────────────────────────────
# STEP 1: Initialize — Downloads providers & modules
# ──────────────────────────────────────────────────────
terraform init

# Upgrade providers to latest allowed versions
terraform init -upgrade

# Reconfigure backend
terraform init -reconfigure

# ──────────────────────────────────────────────────────
# STEP 2: Format — Auto-format your .tf files
# ──────────────────────────────────────────────────────
terraform fmt            # Format current directory
terraform fmt -recursive # Format all subdirectories too
terraform fmt -check     # Check if files are formatted (CI/CD)

# ──────────────────────────────────────────────────────
# STEP 3: Validate — Check syntax and internal consistency
# ──────────────────────────────────────────────────────
terraform validate

# ──────────────────────────────────────────────────────
# STEP 4: Plan — Preview changes (dry run)
# ──────────────────────────────────────────────────────
terraform plan

# Save plan to a file (for review/apply later)
terraform plan -out=tfplan

# Plan with specific variable file
terraform plan -var-file="prod.tfvars"

# Plan for destruction
terraform plan -destroy

# Target specific resource only
terraform plan -target=aws_instance.web

# ──────────────────────────────────────────────────────
# STEP 5: Apply — Create/update infrastructure
# ──────────────────────────────────────────────────────
terraform apply

# Apply without confirmation prompt
terraform apply -auto-approve

# Apply a saved plan
terraform apply tfplan

# Apply with variables
terraform apply -var="instance_type=t3.large"

# ──────────────────────────────────────────────────────
# STEP 6: Destroy — Tear down all infrastructure
# ──────────────────────────────────────────────────────
terraform destroy

# Destroy without confirmation
terraform destroy -auto-approve

# Destroy specific resource only
terraform destroy -target=aws_instance.web

# ──────────────────────────────────────────────────────
# OTHER USEFUL COMMANDS
# ──────────────────────────────────────────────────────
terraform show                # Show current state
terraform graph               # Generate dependency graph (DOT format)
terraform graph | dot -Tpng > graph.png   # Visual graph
terraform providers            # Show required providers
terraform refresh              # Sync state with real infrastructure
terraform taint aws_instance.web    # Mark for recreation (deprecated)
terraform untaint aws_instance.web  # Unmark
terraform console              # Interactive console (test expressions)
```

---

## 20. Complete Real-World Example

Let's build a **complete AWS web server setup** from scratch:

### File Structure

```
aws-web-project/
├── main.tf              # Main configuration
├── variables.tf         # Variable declarations
├── outputs.tf           # Output declarations
├── providers.tf         # Provider configuration
├── data.tf              # Data sources
├── locals.tf            # Local values
├── terraform.tfvars     # Variable values
├── userdata.tftpl       # EC2 user data template
└── modules/
    └── security-group/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### `providers.tf`

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }
  }

  # For production, use remote backend:
  # backend "s3" {
  #   bucket         = "my-tf-state"
  #   key            = "web-project/terraform.tfstate"
  #   region         = "us-east-1"
  #   encrypt        = true
  #   dynamodb_table = "tf-locks"
  # }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}
```

### `variables.tf`

```hcl
# ──────────────────────────────────────────────────────
# General
# ──────────────────────────────────────────────────────
variable "project_name" {
  description = "Name of the project"
  type        = string

  validation {
    condition     = length(var.project_name) > 0
    error_message = "Project name cannot be empty."
  }
}

variable "environment" {
  description = "Environment (dev, staging, prod)"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

# ──────────────────────────────────────────────────────
# AWS
# ──────────────────────────────────────────────────────
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

# ──────────────────────────────────────────────────────
# Networking
# ──────────────────────────────────────────────────────
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "public_subnet_cidrs" {
  description = "CIDR blocks for public subnets"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24"]
}

# ──────────────────────────────────────────────────────
# Compute
# ──────────────────────────────────────────────────────
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "instance_count" {
  description = "Number of web server instances"
  type        = number
  default     = 2

  validation {
    condition     = var.instance_count >= 1 && var.instance_count <= 10
    error_message = "Must be between 1 and 10."
  }
}

variable "enable_monitoring" {
  description = "Enable detailed monitoring"
  type        = bool
  default     = false
}

variable "ssh_allowed_cidrs" {
  description = "CIDR blocks allowed for SSH"
  type        = list(string)
  default     = []
}

variable "extra_tags" {
  description = "Additional tags to apply"
  type        = map(string)
  default     = {}
}
```

### `locals.tf`

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"

  common_tags = merge(var.extra_tags, {
    Name        = local.name_prefix
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "Terraform"
    CreatedAt   = formatdate("YYYY-MM-DD", timestamp())
  })

  # Environment-specific settings
  env_config = {
    dev = {
      instance_type = "t3.micro"
      volume_size   = 20
      monitoring    = false
    }
    staging = {
      instance_type = "t3.small"
      volume_size   = 30
      monitoring    = true
    }
    prod = {
      instance_type = "t3.medium"
      volume_size   = 50
      monitoring    = true
    }
  }

  current_config = local.env_config[var.environment]
}
```

### `data.tf`

```hcl
# Get latest Ubuntu 22.04 AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Get available AZs
data "aws_availability_zones" "available" {
  state = "available"
}

# Get current caller identity
data "aws_caller_identity" "current" {}
```

### `modules/security-group/variables.tf`

```hcl
variable "name" {
  type = string
}

variable "vpc_id" {
  type = string
}

variable "ingress_rules" {
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
    description = string
  }))
}

variable "tags" {
  type    = map(string)
  default = {}
}
```

### `modules/security-group/main.tf`

```hcl
resource "aws_security_group" "this" {
  name        = var.name
  description = "Security group for ${var.name}"
  vpc_id      = var.vpc_id

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
      description = ingress.value.description
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow all outbound traffic"
  }

  tags = merge(var.tags, {
    Name = var.name
  })

  lifecycle {
    create_before_destroy = true
  }
}
```

### `modules/security-group/outputs.tf`

```hcl
output "security_group_id" {
  description = "ID of the security group"
  value       = aws_security_group.this.id
}

output "security_group_arn" {
  description = "ARN of the security group"
  value       = aws_security_group.this.arn
}
```

### `userdata.tftpl`

```bash
#!/bin/bash
set -euxo pipefail

# Update system
apt-get update -y
apt-get upgrade -y

# Install Nginx
apt-get install -y nginx

# Create custom page
cat > /var/www/html/index.html <<HTML
<!DOCTYPE html>
<html>
<head><title>${project_name}</title></head>
<body>
  <h1>Welcome to ${project_name}!</h1>
  <p>Environment: ${environment}</p>
  <p>Server: $(hostname)</p>
  <p>Instance Index: ${instance_index}</p>
</body>
</html>
HTML

# Start Nginx
systemctl enable nginx
systemctl start nginx
```

### `main.tf`

```hcl
# ──────────────────────────────────────────────────────
# RANDOM SUFFIX (for unique naming)
# ──────────────────────────────────────────────────────
resource "random_id" "suffix" {
  byte_length = 4
}

# ──────────────────────────────────────────────────────
# VPC
# ──────────────────────────────────────────────────────
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-vpc"
  })
}

# ──────────────────────────────────────────────────────
# INTERNET GATEWAY
# ──────────────────────────────────────────────────────
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-igw"
  })
}

# ──────────────────────────────────────────────────────
# PUBLIC SUBNETS
# ──────────────────────────────────────────────────────
resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)

  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-subnet-${count.index + 1}"
    Tier = "public"
  })
}

# ──────────────────────────────────────────────────────
# ROUTE TABLE
# ──────────────────────────────────────────────────────
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-rt"
  })
}

resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# ──────────────────────────────────────────────────────
# SECURITY GROUP (using our module)
# ──────────────────────────────────────────────────────
module "web_sg" {
  source = "./modules/security-group"

  name   = "${local.name_prefix}-web-sg"
  vpc_id = aws_vpc.main.id

  ingress_rules = concat(
    [
      {
        from_port   = 80
        to_port     = 80
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
        description = "Allow HTTP"
      },
      {
        from_port   = 443
        to_port     = 443
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
        description = "Allow HTTPS"
      },
    ],
    # Conditionally add SSH rule
    length(var.ssh_allowed_cidrs) > 0 ? [
      {
        from_port   = 22
        to_port     = 22
        protocol    = "tcp"
        cidr_blocks = var.ssh_allowed_cidrs
        description = "Allow SSH from allowed CIDRs"
      }
    ] : []
  )

  tags = local.common_tags
}

# ──────────────────────────────────────────────────────
# EC2 INSTANCES
# ──────────────────────────────────────────────────────
resource "aws_instance" "web" {
  count = var.instance_count

  ami                    = data.aws_ami.ubuntu.id
  instance_type          = coalesce(var.instance_type, local.current_config.instance_type)
  subnet_id              = aws_subnet.public[count.index % length(aws_subnet.public)].id
  vpc_security_group_ids = [module.web_sg.security_group_id]
  monitoring             = var.enable_monitoring || local.current_config.monitoring

  user_data = templatefile("${path.module}/userdata.tftpl", {
    project_name   = var.project_name
    environment    = var.environment
    instance_index = count.index
  })
  
  root_block_device {
    volume_size = local.current_config.volume_size
    volume_type = "gp3"
    encrypted   = true

    tags = merge(local.common_tags, {
      Name = "${local.name_prefix}-web-${count.index}-root-vol"
    })
  }

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web-${count.index + 1}"
    Role = "webserver"
  })

  lifecycle {
    create_before_destroy = true

    # Ignore changes to user_data to prevent recreation on template changes
    ignore_changes = [user_data]

    precondition {
      condition     = data.aws_ami.ubuntu.architecture == "x86_64"
      error_message = "AMI must be x86_64 architecture."
    }

    postcondition {
      condition     = self.public_ip != ""
      error_message = "Instance must receive a public IP."
    }
  }
}

# ──────────────────────────────────────────────────────
# ELASTIC IPs (one per instance, only in production)
# ──────────────────────────────────────────────────────
resource "aws_eip" "web" {
  count = var.environment == "prod" ? var.instance_count : 0

  instance = aws_instance.web[count.index].id
  domain   = "vpc"

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web-eip-${count.index + 1}"
  })

  depends_on = [aws_internet_gateway.main]
}

# ──────────────────────────────────────────────────────
# APPLICATION LOAD BALANCER
# ──────────────────────────────────────────────────────
resource "aws_lb" "web" {
  name               = "${local.name_prefix}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [module.web_sg.security_group_id]
  subnets            = aws_subnet.public[*].id

  enable_deletion_protection = var.environment == "prod" ? true : false

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-alb"
  })
}

# ──────────────────────────────────────────────────────
# TARGET GROUP
# ──────────────────────────────────────────────────────
resource "aws_lb_target_group" "web" {
  name     = "${local.name_prefix}-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    enabled             = true
    path                = "/"
    port                = "traffic-port"
    protocol            = "HTTP"
    healthy_threshold   = 3
    unhealthy_threshold = 3
    timeout             = 5
    interval            = 30
    matcher             = "200"
  }

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-tg"
  })
}

# ──────────────────────────────────────────────────────
# TARGET GROUP ATTACHMENT (register instances)
# ──────────────────────────────────────────────────────
resource "aws_lb_target_group_attachment" "web" {
  count = var.instance_count

  target_group_arn = aws_lb_target_group.web.arn
  target_id        = aws_instance.web[count.index].id
  port             = 80
}

# ──────────────────────────────────────────────────────
# ALB LISTENER
# ──────────────────────────────────────────────────────
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.web.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-http-listener"
  })
}
```

---

### `outputs.tf`

```hcl
# ──────────────────────────────────────────────────────
# INSTANCE OUTPUTS
# ──────────────────────────────────────────────────────
output "instance_ids" {
  description = "IDs of all web server instances"
  value       = aws_instance.web[*].id
}

output "instance_public_ips" {
  description = "Public IPs of all web server instances"
  value       = aws_instance.web[*].public_ip
}

output "instance_private_ips" {
  description = "Private IPs of all web server instances"
  value       = aws_instance.web[*].private_ip
}

# ──────────────────────────────────────────────────────
# NETWORK OUTPUTS
# ──────────────────────────────────────────────────────
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "vpc_cidr" {
  description = "CIDR block of the VPC"
  value       = aws_vpc.main.cidr_block
}

output "public_subnet_ids" {
  description = "IDs of public subnets"
  value       = aws_subnet.public[*].id
}

# ──────────────────────────────────────────────────────
# LOAD BALANCER OUTPUTS
# ──────────────────────────────────────────────────────
output "alb_dns_name" {
  description = "DNS name of the Application Load Balancer"
  value       = aws_lb.web.dns_name
}

output "alb_url" {
  description = "Full URL of the load balancer"
  value       = "http://${aws_lb.web.dns_name}"
}

output "alb_arn" {
  description = "ARN of the ALB"
  value       = aws_lb.web.arn
}

# ──────────────────────────────────────────────────────
# SECURITY GROUP OUTPUTS
# ──────────────────────────────────────────────────────
output "web_security_group_id" {
  description = "ID of the web security group"
  value       = module.web_sg.security_group_id
}

# ──────────────────────────────────────────────────────
# META / DEBUG OUTPUTS
# ──────────────────────────────────────────────────────
output "aws_account_id" {
  description = "AWS Account ID"
  value       = data.aws_caller_identity.current.account_id
}

output "ami_id_used" {
  description = "AMI ID used for instances"
  value       = data.aws_ami.ubuntu.id
}

output "availability_zones_used" {
  description = "AZs where subnets were created"
  value       = aws_subnet.public[*].availability_zone
}

# COMPLEX structured output
output "instance_details" {
  description = "Detailed info about each web server"
  value = [
    for i, instance in aws_instance.web : {
      index      = i
      id         = instance.id
      public_ip  = instance.public_ip
      private_ip = instance.private_ip
      az         = instance.availability_zone
      subnet_id  = instance.subnet_id
    }
  ]
}
```

---

### `terraform.tfvars`

```hcl
# ──────────────────────────────────────────────────────
# Project Configuration
# ──────────────────────────────────────────────────────
project_name = "mywebapp"
environment  = "dev"
aws_region   = "us-east-1"

# ──────────────────────────────────────────────────────
# Network
# ──────────────────────────────────────────────────────
vpc_cidr            = "10.0.0.0/16"
public_subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]

# ──────────────────────────────────────────────────────
# Compute
# ──────────────────────────────────────────────────────
instance_type     = "t3.micro"
instance_count    = 2
enable_monitoring = false
ssh_allowed_cidrs = ["203.0.113.0/32"]   # Your IP here

# ──────────────────────────────────────────────────────
# Tags
# ──────────────────────────────────────────────────────
extra_tags = {
  Owner      = "DevTeam"
  CostCenter = "Engineering"
}
```

---

### `prod.tfvars` (production overrides)

```hcl
project_name      = "mywebapp"
environment       = "prod"
aws_region        = "us-east-1"
instance_type     = "t3.medium"
instance_count    = 4
enable_monitoring = true
ssh_allowed_cidrs = []   # No SSH in production

extra_tags = {
  Owner      = "PlatformTeam"
  CostCenter = "Production"
  Compliance = "SOC2"
}
```

---

## Running the Project: Full Walkthrough

```bash
# ──────────────────────────────────────────────────────
# STEP 1: Navigate to project directory
# ──────────────────────────────────────────────────────
cd aws-web-project/

# ──────────────────────────────────────────────────────
# STEP 2: Initialize Terraform
#         Downloads providers and modules
# ──────────────────────────────────────────────────────
terraform init

# Expected output:
# Initializing the backend...
# Initializing provider plugins...
# - Finding hashicorp/aws versions matching "~> 5.0"...
# - Finding hashicorp/random versions matching "~> 3.5"...
# - Installing hashicorp/aws v5.xx.x...
# - Installing hashicorp/random v3.x.x...
# Terraform has been successfully initialized!

# ──────────────────────────────────────────────────────
# STEP 3: Format files
# ──────────────────────────────────────────────────────
terraform fmt -recursive

# ──────────────────────────────────────────────────────
# STEP 4: Validate configuration
# ──────────────────────────────────────────────────────
terraform validate

# Expected output:
# Success! The configuration is valid.

# ──────────────────────────────────────────────────────
# STEP 5: Plan (preview changes)
# ──────────────────────────────────────────────────────

# Dev environment (uses terraform.tfvars automatically)
terraform plan

# Production environment (uses prod.tfvars)
terraform plan -var-file="prod.tfvars"

# Save plan to a file for review
terraform plan -out=dev.tfplan

# Expected output:
# Plan: 13 to add, 0 to change, 0 to destroy.
#
# Changes to Outputs:
#   + alb_dns_name        = (known after apply)
#   + instance_ids        = (known after apply)
#   + instance_public_ips = (known after apply)
#   ...

# ──────────────────────────────────────────────────────
# STEP 6: Apply (create infrastructure)
# ──────────────────────────────────────────────────────

# Apply the saved plan (no confirmation needed)
terraform apply dev.tfplan

# OR apply directly (will prompt for confirmation)
terraform apply

# OR auto-approve (use in CI/CD, not recommended manually)
terraform apply -auto-approve

# Expected output:
# aws_vpc.main: Creating...
# aws_vpc.main: Creation complete after 3s [id=vpc-0abcd1234]
# aws_internet_gateway.main: Creating...
# ...
# Apply complete! Resources: 13 added, 0 changed, 0 destroyed.
#
# Outputs:
# alb_dns_name = "mywebapp-dev-alb-1234567890.us-east-1.elb.amazonaws.com"
# alb_url      = "http://mywebapp-dev-alb-1234567890.us-east-1.elb.amazonaws.com"
# instance_ids = ["i-0abc123", "i-0def456"]
# ...

# ──────────────────────────────────────────────────────
# STEP 7: Inspect outputs
# ──────────────────────────────────────────────────────
terraform output
terraform output alb_url
terraform output -json instance_details

# ──────────────────────────────────────────────────────
# STEP 8: Inspect state
# ──────────────────────────────────────────────────────
terraform state list

# Expected output:
# data.aws_ami.ubuntu
# data.aws_availability_zones.available
# data.aws_caller_identity.current
# random_id.suffix
# aws_vpc.main
# aws_internet_gateway.main
# aws_subnet.public[0]
# aws_subnet.public[1]
# aws_route_table.public
# aws_route_table_association.public[0]
# aws_route_table_association.public[1]
# module.web_sg.aws_security_group.this
# aws_instance.web[0]
# aws_instance.web[1]
# aws_lb.web
# aws_lb_target_group.web
# aws_lb_target_group_attachment.web[0]
# aws_lb_target_group_attachment.web[1]
# aws_lb_listener.http

# Show details of a specific resource
terraform state show aws_instance.web[0]

# ──────────────────────────────────────────────────────
# STEP 9: Make a change (e.g., add instance)
# ──────────────────────────────────────────────────────
# Edit terraform.tfvars: instance_count = 3
# Then:
terraform plan       # Shows: 2 to add (new instance + target group attachment)
terraform apply      # Adds the new instance

# ──────────────────────────────────────────────────────
# STEP 10: Destroy specific resource (optional)
# ──────────────────────────────────────────────────────
terraform destroy -target=aws_eip.web

# ──────────────────────────────────────────────────────
# STEP 11: Destroy everything
# ──────────────────────────────────────────────────────
terraform destroy

# Expected output:
# Plan: 0 to add, 0 to change, 13 to destroy.
#
# Do you really want to destroy all resources? (yes)
# aws_lb_listener.http: Destroying...
# ...
# aws_vpc.main: Destruction complete after 1s
#
# Destroy complete! Resources: 13 destroyed.
```

---

## Quick Reference Cheat Sheet

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    TERRAFORM CHEAT SHEET                                 │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  FILE NAMING CONVENTIONS (all are optional, just best practice):         │
│  ─────────────────────────────────────────────────────────────            │
│  main.tf           → Primary resources                                   │
│  variables.tf      → Input variable declarations                         │
│  outputs.tf        → Output declarations                                 │
│  providers.tf      → Provider config + terraform block                   │
│  data.tf           → Data sources                                        │
│  locals.tf         → Local values                                        │
│  versions.tf       → Version constraints (alternative to providers.tf)   │
│  terraform.tfvars  → Variable values (auto-loaded)                       │
│  *.auto.tfvars     → Variable values (auto-loaded)                       │
│  backend.tf        → Backend configuration                               │
│                                                                          │
│  REFERENCE SYNTAX:                                                       │
│  ─────────────────                                                       │
│  var.name                          → Input variable                      │
│  local.name                        → Local value                         │
│  resource_type.name.attribute      → Resource attribute                  │
│  data.type.name.attribute          → Data source attribute               │
│  module.name.output_name           → Module output                       │
│  terraform.workspace               → Current workspace name              │
│  path.module                       → Current module directory            │
│  path.root                         → Root module directory               │
│  path.cwd                          → Current working directory           │
│  self.attribute                    → Self-reference (in provisioners)    │
│  each.key / each.value             → for_each iterator                   │
│  count.index                       → count iterator                      │
│                                                                          │
│  COMMANDS QUICK REFERENCE:                                               │
│  ─────────────────────────                                               │
│  terraform init              → Initialize / download providers           │
│  terraform init -upgrade     → Upgrade providers                         │
│  terraform fmt               → Auto-format .tf files                     │
│  terraform validate          → Validate syntax                           │
│  terraform plan              → Preview changes                           │
│  terraform plan -out=FILE    → Save plan                                 │
│  terraform apply             → Apply changes                             │
│  terraform apply FILE        → Apply saved plan                          │
│  terraform destroy           → Destroy all resources                     │
│  terraform output            → Show outputs                              │
│  terraform console           → Interactive REPL                          │
│  terraform state list        → List resources in state                   │
│  terraform state show X      → Show resource details                     │
│  terraform import TYPE.NAME ID → Import existing resource               │
│  terraform graph             → Dependency graph (DOT)                    │
│  terraform workspace list    → List workspaces                           │
│  terraform taint X           → Force recreation                          │
│  terraform providers         → Show providers                            │
│                                                                          │
│  ARGUMENT vs NESTED BLOCK:                                               │
│  ─────────────────────────                                               │
│  tags = { Name = "X" }       → ARGUMENT (has = sign)                    │
│  root_block_device { ... }   → NESTED BLOCK (no = sign)                 │
│                                                                          │
│  IMPORTANT SYMBOLS:                                                      │
│  ──────────────────                                                      │
│  =          Assignment                                                   │
│  ${ }       String interpolation                                         │
│  [*]        Splat expression (get all)                                   │
│  [ ]        Index / List                                                 │
│  { }        Map / Block body                                             │
│  =>         Map key-value separator (in for expressions)                │
│  ...        Expansion (spread list into function args)                   │
│  #          Single-line comment                                          │
│  //         Single-line comment (alternative)                            │
│  /* */      Multi-line comment                                           │
│                                                                          │
│  PLAN OUTPUT SYMBOLS:                                                    │
│  ─────────────────────                                                   │
│  +   Create (new resource)                                               │
│  -   Destroy (remove resource)                                           │
│  ~   Update in-place                                                     │
│  -/+ Destroy and recreate (replacement)                                  │
│  <=  Read (data source)                                                  │
│                                                                          │
│  VERSION CONSTRAINTS:                                                    │
│  ─────────────────────                                                   │
│  = 5.0.0          Exact                                                  │
│  >= 5.0           Greater than or equal                                  │
│  ~> 5.0           >= 5.0, < 6.0 (pessimistic / compatible)              │
│  ~> 5.0.0         >= 5.0.0, < 5.1.0                                     │
│  >= 5.0, < 6.0    Explicit range                                         │
│                                                                          │
│  DEPENDENCY FLOW:                                                        │
│  ─────────────────                                                       │
│  Implicit: Terraform detects references automatically                    │
│            vpc_id = aws_vpc.main.id  ← TF knows to create VPC first    │
│  Explicit: depends_on = [aws_iam_role_policy.x]                         │
│            Use when TF can't auto-detect the dependency                  │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Concept Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                    HOW IT ALL FITS TOGETHER                          │
│                                                                     │
│  ┌─────────────┐                                                    │
│  │  terraform   │─── Settings, versions, backend                    │
│  │  block       │                                                    │
│  └──────┬──────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌─────────────┐                                                    │
│  │  provider    │─── Connect to cloud (AWS, Azure, GCP...)          │
│  │  block       │                                                    │
│  └──────┬──────┘                                                    │
│         │                                                            │
│         ├──────────────┬──────────────┬─────────────┐               │
│         ▼              ▼              ▼             ▼               │
│  ┌───────────┐  ┌───────────┐  ┌──────────┐  ┌──────────┐         │
│  │ variable  │  │  locals   │  │   data   │  │  module  │         │
│  │ blocks    │  │  block    │  │  blocks  │  │  blocks  │         │
│  │           │  │           │  │          │  │          │         │
│  │ Inputs    │  │ Computed  │  │ Read     │  │ Reusable │         │
│  │ from user │  │ values    │  │ existing │  │ packages │         │
│  └─────┬─────┘  └─────┬─────┘  └────┬─────┘  └────┬─────┘         │
│        │              │              │              │               │
│        └──────────────┴──────┬───────┴──────────────┘               │
│                              │                                       │
│                              ▼                                       │
│                     ┌─────────────────┐                              │
│                     │    resource     │                              │
│                     │    blocks       │                              │
│                     │                 │                              │
│                     │ ┌─────────────┐ │                              │
│                     │ │ arguments   │ │  ← key = value              │
│                     │ ├─────────────┤ │                              │
│                     │ │ nested      │ │  ← sub-configuration        │
│                     │ │ blocks      │ │                              │
│                     │ ├─────────────┤ │                              │
│                     │ │ meta-args   │ │  ← count, for_each,        │
│                     │ │             │ │    depends_on, provider,     │
│                     │ │             │ │    lifecycle                  │
│                     │ ├─────────────┤ │                              │
│                     │ │ provisioners│ │  ← scripts (last resort)    │
│                     │ └─────────────┘ │                              │
│                     └────────┬────────┘                              │
│                              │                                       │
│                              ▼                                       │
│                     ┌─────────────────┐                              │
│                     │    output       │                              │
│                     │    blocks       │─── Expose values             │
│                     └─────────────────┘                              │
│                              │                                       │
│                              ▼                                       │
│                     ┌─────────────────┐                              │
│                     │   State File    │─── terraform.tfstate         │
│                     │   (.tfstate)    │    Tracks real infra         │
│                     └─────────────────┘                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Best Practices Summary

```
┌──────────────────────────────────────────────────────────────────────┐
│                     TERRAFORM BEST PRACTICES                         │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. STATE MANAGEMENT                                                 │
│     ✅ Always use remote backend (S3, GCS, Azure Storage)            │
│     ✅ Enable state locking (DynamoDB for S3)                        │
│     ✅ Enable encryption for state files                             │
│     ❌ Never commit .tfstate to git                                  │
│     ❌ Never manually edit state files                               │
│                                                                      │
│  2. CODE ORGANIZATION                                                │
│     ✅ Split config into logical files (main/vars/outputs/etc.)      │
│     ✅ Use modules for reusable components                           │
│     ✅ Use consistent naming: snake_case for resources               │
│     ✅ Always run terraform fmt                                      │
│     ✅ Add description to every variable and output                  │
│                                                                      │
│  3. VARIABLES & SECRETS                                              │
│     ✅ Use variable validation blocks                                │
│     ✅ Mark sensitive variables with sensitive = true                 │
│     ✅ Use .tfvars files per environment (dev.tfvars, prod.tfvars)   │
│     ❌ Never hardcode secrets in .tf files                           │
│     ❌ Never commit .tfvars with secrets to git                      │
│                                                                      │
│  4. RESOURCES                                                        │
│     ✅ Prefer for_each over count (stable keys)                      │
│     ✅ Use lifecycle rules to protect critical resources              │
│     ✅ Tag everything consistently                                   │
│     ✅ Use data sources to reference existing infrastructure         │
│     ❌ Avoid provisioners; use cloud-init/user_data instead          │
│                                                                      │
│  5. WORKFLOW                                                         │
│     ✅ Always run plan before apply                                  │
│     ✅ Save plans with -out for production applies                   │
│     ✅ Use workspaces or directory structure for environments        │
│     ✅ Pin provider versions with ~> constraint                      │
│     ✅ Review plan output carefully before approving                 │
│                                                                      │
│  6. GIT                                                              │
│     ✅ .gitignore should include:                                    │
│        .terraform/                                                    │
│        *.tfstate                                                      │
│        *.tfstate.backup                                               │
│        *.tfplan                                                       │
│        .terraform.lock.hcl  (or commit it — both schools of thought) │
│        *.auto.tfvars (if contains secrets)                           │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### `.gitignore` for Terraform Projects

```gitignore
# Terraform directories
.terraform/
.terraform.lock.hcl

# State files (NEVER commit these)
*.tfstate
*.tfstate.*
*.tfstate.backup

# Plan files
*.tfplan
*.out

# Sensitive variable files
*.auto.tfvars
secret.tfvars

# OS files
.DS_Store
Thumbs.db

# IDE
.idea/
.vscode/
*.swp
*.swo
```

---

This completes the full Terraform basics guide. The progression is:

```
1.  Understand HCL syntax (blocks → labels → arguments → nested blocks)
2.  Learn the block types (terraform, provider, resource, data, variable, output, locals, module)
3.  Master expressions & functions (interpolation, conditionals, for, splat)
4.  Use meta-arguments (count, for_each, depends_on, lifecycle)
5.  Manage state (remote backends, locking, imports)
6.  Organize with modules (reusable components)
7.  Follow the workflow (init → fmt → validate → plan → apply → destroy)
```
