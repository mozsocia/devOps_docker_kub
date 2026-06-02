**Terraform Basics – Complete Step-by-Step Guide**

This guide combines everything we’ve discussed so far into one structured, beginner-friendly tutorial.

---

### 1. What is Terraform?

Terraform is an **Infrastructure as Code (IaC)** tool that lets you define, provision, and manage infrastructure using code.

**Key Features:**
- Declarative (you say *what* you want, not *how*)
- Works with AWS, Azure, GCP, Docker, Local, etc.
- Version controllable (store in Git)
- Idempotent (safe to run multiple times)

---

### 2. Installation

1. Download Terraform from: https://developer.hashicorp.com/terraform/downloads
2. Add it to your system PATH
3. Verify installation:
   ```bash
   terraform version
   ```

---

### 3. Terraform Configuration Language (HCL)

All Terraform code is written in files with `.tf` extension using **HCL (HashiCorp Configuration Language)**.

Everything in Terraform is built using **Blocks**.

---

### 4. Full Structure of a Terraform Block (Most Important Section)

Here is the complete anatomy of a Terraform block:

```hcl
# Block Type     Label 1           Label 2
resource "aws_instance" "web_server" {

  # Arguments
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"

  # Meta-arguments
  count      = 2
  depends_on = [aws_iam_role.example]

  # Nested Blocks
  tags = {
    Name        = "Web-Server"
    Environment = "Production"
  }

  root_block_device {
    volume_size = 20
    volume_type = "gp3"
  }

  # Dynamic Nested Block (Advanced)
  dynamic "ingress" {
    for_each = var.allowed_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  # Lifecycle Block
  lifecycle {
    create_before_destroy = true
    ignore_changes        = [tags]
  }
}
```

#### Detailed Breakdown of Each Part:

| Part                  | Description                                                                 | Example |
|-----------------------|-----------------------------------------------------------------------------|---------|
| **Block Type**        | The first keyword that defines what this block represents                   | `resource`, `provider`, `variable`, `output`, `data`, `module` |
| **Labels**            | One or two strings after the block type                                     | `"aws_instance"` (Label 1), `"web_server"` (Label 2) |
| **Arguments**         | Key-value pairs that configure the block                                    | `ami = "..."`, `instance_type = "t2.micro"` |
| **Meta-arguments**    | Special arguments that control Terraform behavior                           | `count`, `for_each`, `depends_on`, `provider`, `lifecycle` |
| **Nested Blocks**     | Blocks inside the main block for grouped configuration                      | `tags { }`, `root_block_device { }` |
| **Dynamic Blocks**    | Create nested blocks dynamically using loops                                | `dynamic "ingress" { ... }` |
| **Lifecycle Block**   | Special nested block to control resource creation/destruction behavior      | `lifecycle { create_before_destroy = true }` |

---

### 5. Most Common Block Types

| Block Type     | Purpose                                      | Labels | Example |
|----------------|----------------------------------------------|--------|--------|
| `resource`     | Create/manage infrastructure                 | 2      | `resource "aws_instance" "web" {}` |
| `provider`     | Configure cloud/API connection               | 1      | `provider "aws" {}` |
| `variable`     | Declare input variables                      | 1      | `variable "region" {}` |
| `output`       | Return values after deployment               | 1      | `output "public_ip" {}` |
| `data`         | Read existing data (read-only)               | 2      | `data "aws_ami" "latest" {}` |
| `locals`       | Define local computed values                 | 0      | `locals { name = "..." }` |
| `terraform`    | Configure Terraform settings                 | 0      | `terraform { required_version = "..." }` |
| `module`       | Use reusable code                            | 1      | `module "vpc" {}` |

---

### 6. Step-by-Step First Project

**Step 1:** Create a new folder and file `main.tf`

```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.0"
    }
  }
}

resource "local_file" "example" {
  filename = "hello-terraform.txt"
  content  = <<-EOT
    Hello from Terraform!
    This is my first resource.
    Created on: ${timestamp()}
  EOT
}

output "file_name" {
  value = local_file.example.filename
}
```

**Step 2:** Initialize
```bash
terraform init
```

**Step 3:** Plan
```bash
terraform plan
```

**Step 4:** Apply
```bash
terraform apply
```

**Step 5:** Destroy (cleanup)
```bash
terraform destroy
```

---

### 7. Variables & Outputs

**variables.tf**
```hcl
variable "filename" {
  type    = string
  default = "default.txt"
}

variable "content" {
  type    = string
}

variable "tags" {
  type = map(string)
  default = {}
}
```

**Use in main.tf**
```hcl
resource "local_file" "example" {
  filename = var.filename
  content  = var.content
  tags     = var.tags
}

output "created_file" {
  value       = local_file.example.filename
  description = "Name of the created file"
}
```

---

### 8. Terraform Workflow Commands

| Command                    | Purpose |
|---------------------------|--------|
| `terraform init`          | Initialize, download providers |
| `terraform fmt`           | Format code |
| `terraform validate`      | Check syntax |
| `terraform plan`          | Preview changes |
| `terraform apply`         | Apply changes |
| `terraform destroy`       | Delete all resources |
| `terraform state list`    | Show managed resources |
| `terraform show`          | Show current state |

---

### 9. Key Concepts Summary

- **Provider**: Plugin that connects to cloud
- **Resource**: The actual thing you create
- **State**: Terraform’s record of what it manages (`terraform.tfstate`)
- **Idempotency**: Safe to rerun
- **Declarative**: Focus on desired end state

---

### 10. Best Practices for Beginners

1. Always run `terraform fmt`
2. Use `terraform validate`
3. Never commit sensitive information
4. Use variables for everything configurable
5. Keep configuration files clean and organized
6. Use meaningful resource names

---

Would you like me to continue this guide with the **next level** topics?  
I can create Part 2 covering:

- `count` vs `for_each`
- Modules
- Remote State (S3 backend)
- AWS real example (VPC + EC2)
- Best folder structure

Just say **“Continue to Part 2”** or tell me what you want next.  

You’re doing great! Keep asking questions.
