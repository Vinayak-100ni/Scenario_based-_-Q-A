### **1. Scenario: Zero-Downtime Deployment of an EC2 Instance**

**Question:** You need to update an EC2 instance’s AMI ID **without downtime**. Terraform wants to destroy and recreate the instance. How do you avoid downtime?

**Answer:**

- Use **Create Before Destroy** in lifecycle rules:
    
    ```hcl
    
    resource "aws_instance" "example" {
      ami           = var.ami_id
      instance_type = "t3.micro"
    
      lifecycle {
        create_before_destroy = true
      }
    }
    
    ```
    
- This ensures the new instance is created before the old one is deleted.
- Alternatively, use an **Auto Scaling Group (ASG)** with rolling updates.

---

### **2. Scenario: Managing Cross-Region Infrastructure**

**Question:** You need to deploy an S3 bucket in `us-east-1` and an EC2 instance in `ap-south-1` using the same Terraform configuration. How do you achieve this?

**Answer:**

Use **multiple provider configurations**:

```hcl

provider "aws" {
  alias  = "us-east"
  region = "us-east-1"
}

provider "aws" {
  alias  = "ap-south"
  region = "ap-south-1"
}

resource "aws_s3_bucket" "example" {
  provider = aws.us-east
  bucket   = "my-bucket-us-east"
}

resource "aws_instance" "example" {
  provider      = aws.ap-south
  ami           = "ami-123456"
  instance_type = "t2.micro"
}

```

- Each resource is explicitly assigned a provider alias.

---

### **3. Scenario: Handling Terraform Drift in Production**

**Question:** Your Terraform-managed AWS infrastructure was modified manually by another team. Terraform does not show changes, but the AWS console does. How do you detect and correct this?

**Answer:**

- Run `terraform plan -refresh-only` to detect drift without making changes.
- Use `terraform state list` to inspect tracked resources.
- If a resource is missing, re-import it:
    
    ```bash
    
    terraform import aws_instance.example i-1234567890abcdef0
    
    ```
    
- If necessary, `terraform apply` to restore the expected configuration.

---

### **4. Scenario: Enforcing Terraform Security Policies**

**Question:** Your company wants to ensure that only t2.micro instances are used to control AWS costs. How do you enforce this in Terraform?

**Answer:**

Use a Terraform validation rule in the variables.tf file to restrict instance types.
- Example 
    
    ```
    
   variable "instance_type" {
  description = "AWS EC2 instance type"
  type        = string

  validation {
    condition     = contains(["t2.micro"], var.instance_type)
    error_message = "Only t2.micro instance type is allowed."
  }
    }
    
    ```
    
- This prevents users from applying Terraform with non-approved instance types.

---

### **5. Scenario: Blue-Green Deployment with Terraform**

**Question:** You need to implement a **blue-green deployment strategy** using Terraform for an Auto Scaling Group (ASG). How do you achieve this?

**Answer:**

- Use two ASGs (`blue` and `green`) behind an **Elastic Load Balancer (ELB)**.
- Example approach:
    
    ```hcl
    
    resource "aws_launch_template" "blue" {
      name = "blue-template"
      image_id = var.ami_blue
    }
    
    resource "aws_launch_template" "green" {
      name = "green-template"
      image_id = var.ami_green
    }
    
    resource "aws_autoscaling_group" "blue" {
      launch_template {
        id = aws_launch_template.blue.id
      }
    }
    
    resource "aws_autoscaling_group" "green" {
      launch_template {
        id = aws_launch_template.green.id
      }
    }
    
    resource "aws_lb_listener_rule" "switch" {
      listener_arn = aws_lb_listener.http.arn
      priority     = 100
      conditions {
        field  = "path-pattern"
        values = ["*"]
      }
      actions {
        type             = "forward"
        target_group_arn = aws_lb_target_group.green.arn
      }
    }
    
    ```
    
- Change the ALB target group when deploying a new version.

---

### **6. Scenario: Terraform Apply Failure Due to API Rate Limits**

**Question:** You are creating 100+ AWS resources in a single `terraform apply`, but the process fails due to AWS **API rate limits**. How do you fix this?

**Answer:**

- Use **retry settings** in the AWS provider:
    
    ```hcl
    provider "aws" {
      region = "us-east-1"
      max_retries = 5
    }
    
    ```
    
- Use `depends_on` to **stagger** resource creation:
    
    ```hcl
    
    resource "aws_instance" "one" {
      ami = "ami-123456"
    }
    
    resource "aws_instance" "two" {
      ami       = "ami-123456"
      depends_on = [aws_instance.one]
    }
    
    ```
    
- Use **Terraform Workspaces** to split workloads.

---

### **7. Scenario: Preventing Accidental Deletion of Critical Resources**

**Question:** You want to prevent the accidental deletion of a production RDS database managed by Terraform. How do you enforce this?

**Answer:**

Use `prevent_destroy` lifecycle rules:

```hcl

resource "aws_db_instance" "production_db" {
  identifier = "prod-db"
  engine     = "mysql"
  instance_class = "db.t3.large"

  lifecycle {
    prevent_destroy = true
  }
}

```

- If `terraform destroy` is run, it will **fail** for this resource.

---

### **8. Scenario: Managing Multi-Tenant AWS Accounts with Terraform**

**Question:** Your organization has multiple AWS accounts (Dev, QA, Prod). How do you manage deployments efficiently?

**Answer:**

Use **Terraform Workspaces** for multi-environment management:

```bash

terraform workspace new dev
terraform workspace new prod
terraform workspace select prod

```

In Terraform configuration:

```hcl

provider "aws" {
  region = "us-east-1"
  alias  = terraform.workspace
}

resource "aws_s3_bucket" "example" {
  bucket = "my-bucket-${terraform.workspace}"
}

```
### **9. Scenario: Handling Terraform State Locking Issues**

**Question:** Your team is using Terraform with remote state in S3. A team member’s Terraform run failed, leaving the state locked. How do you resolve this issue?

**Answer:**

Terraform automatically locks the state in DynamoDB when using `terraform { backend "s3" { } }`. If a lock persists, run:

```

terraform force-unlock <LOCK_ID>

```

Use `terraform state list` and `terraform state show` to verify the state before unlocking.

---

### **10. Scenario: Rolling Back a Failed Terraform Deployment**

**Question:** Your Terraform apply modified resources incorrectly, causing an outage. How can you quickly roll back to the previous state?

**Answer:**

1. If you have a previous state file stored remotely (S3, Terraform Cloud), restore it:
    
    ```
    
    terraform state pull > backup.tfstate
    terraform state push backup.tfstate
    
    ```
    
2. Revert the incorrect code and apply Terraform again.
3. If necessary, manually restore critical resources before running Terraform apply.

---

### **11. Scenario: Dynamic Resource Scaling with Terraform Modules**

**Question:** Your infrastructure requires different EC2 instance types based on environment (`dev`, `prod`). How can you dynamically assign instance types in a Terraform module?

**Answer:**

Use `terraform.tfvars` or a map inside `variables.tf`:

```hcl

variable "instance_type_map" {
  type = map(string)
  default = {
    dev  = "t2.micro"
    prod = "t3.large"
  }
}

resource "aws_instance" "example" {
  ami           = "ami-123456"
  instance_type = var.instance_type_map[var.environment]
}

```

Pass `environment = "prod"` in Terraform variables to get the correct instance type.

### **12. Scenario: Managing Secrets Securely in Terraform**

**Question:** Your Terraform code requires AWS credentials. How do you securely manage these secrets without hardcoding them?

**Answer:**

- **Use AWS Secrets Manager** and retrieve secrets dynamically:

```hcl

data "aws_secretsmanager_secret_version" "db_creds" {
  secret_id = "my-db-creds"
}

resource "aws_db_instance" "db" {
  username = jsondecode(data.aws_secretsmanager_secret_version.db_creds.secret_string)["username"]
  password = jsondecode(data.aws_secretsmanager_secret_version.db_creds.secret_string)["password"]
}

```

- Alternatively, use Terraform **variables** with environment variables (`TF_VAR_secret`).

---

### **13. Scenario: Managing Large Terraform State Efficiently**

**Question:** Your Terraform state file is growing too large, causing slow performance. How can you manage it efficiently?

**Answer:**

- **Use Terraform State Splitting:** Separate resources into multiple state files using different workspaces or backends.
- **Enable Terraform State Locking:** Store state in **S3 with DynamoDB** to prevent concurrency issues:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
  }
}

```

- **Use Terraform Workspaces** to isolate environments (`terraform workspace new dev`).

---

### **14. Scenario: Auto-Scaling Worker Nodes in EKS**

**Question:** Your Kubernetes cluster runs on AWS EKS. How do you configure Terraform to auto-scale worker nodes dynamically?

**Answer:**

- Use **AWS Auto Scaling Groups** with Terraform:

```hcl

resource "aws_autoscaling_group" "eks_nodes" {
  min_size         = 2
  max_size         = 10
  desired_capacity = 3

  tag {
    key                 = "kubernetes.io/cluster/my-cluster"
    value               = "owned"
    propagate_at_launch = true
  }
}

```

- Combine it with **Cluster Autoscaler** to adjust nodes dynamically.

---

### **15. Scenario: Avoiding Resource Recreation in Terraform**

**Question:** You updated an EC2 instance type, but Terraform wants to **destroy and recreate** it instead of modifying it in-place. How do you prevent this?

**Answer:**

- Use the **`ignore_changes`** lifecycle rule to keep the existing resource:

```hcl

resource "aws_instance" "example" {
  ami           = "ami-123456"
  instance_type = "t2.micro"

  lifecycle {
    ignore_changes = [instance_type]
  }
}

```

- If modification is required, **manually update it** outside Terraform, then import the changes using:
    
    ```
    sh
    CopyEdit
    terraform import aws_instance.example i-1234567890abcdef
    
    ```


### **1. How does Terraform handle state, and what are the best practices for managing Terraform state in a team?**

### **Answer:**

Terraform maintains state to track resources it manages. It stores this state in a file (by default, `terraform.tfstate`). Best practices for managing state in a team include:

- Use **remote backends** like AWS S3 with DynamoDB locking, Terraform Cloud, or Consul for state management.
- Enable **state locking** to prevent concurrent modifications.
- Use **workspaces** for managing different environments.
- Encrypt state files, especially if they contain sensitive data.
- Use **state commands** (`terraform state mv`, `terraform state rm`) cautiously.

---

### **2. Explain Terraform’s dependency resolution mechanism.**

### **Answer:**

Terraform determines dependencies using:

- **Implicit dependencies:** Based on how resources reference each other using `depends_on` and interpolations.
- **Explicit dependencies:** Defined using the `depends_on` argument to force Terraform to execute resources in a specific order.

Terraform builds a **dependency graph** internally to determine the correct execution order of resources.

---

### **3. What happens when you manually delete a resource that Terraform manages?**

### **Answer:**

If a resource is deleted outside of Terraform (e.g., via AWS console):

- Running `terraform plan` will detect that the resource is missing.
- Running `terraform apply` will recreate the resource.
- If the resource was removed from the `.tf` file and `terraform apply` is run, Terraform won’t manage it anymore.

To remove it from state without recreating:

```

terraform state rm <resource_address>

```

---

### **4. How do you manage secrets in Terraform?**

### **Answer:**

- **Environment Variables:** Use `TF_VAR_<variable_name>` with sensitive values.
- **Terraform Vault Provider:** Store secrets in HashiCorp Vault.
- **AWS Secrets Manager / Azure Key Vault:** Retrieve secrets dynamically.
- **Sensitive Variables:** Mark variables as sensitive in Terraform:
    
    ```hcl
    
    variable "db_password" {
      type      = string
      sensitive = true
    }
    
    ```
    
- **Do not commit `terraform.tfstate`** to version control as it may contain secrets.

---

### **5. What are Terraform Workspaces, and how do they differ from Modules?**

### **Answer:**

- **Workspaces** allow you to manage multiple instances of Terraform configurations within the same backend. Example:
    
    ```
    
    terraform workspace new dev
    terraform workspace select dev
    
    ```
    
- **Modules** are reusable Terraform configurations that help with abstraction and code organization.

Use workspaces for **environment separation within the same backend**, while modules help **reusability and modularity**.

---

### **6. How does Terraform handle drift detection?**

### **Answer:**

Terraform detects configuration drift by running:

```

terraform plan

```

If the actual state differs from the expected state, Terraform highlights the drift and prompts an update. To prevent drift:

- Implement CI/CD checks.
- Use `terraform state list` to inspect current resources.
- Use `terraform import` to sync manually created resources.

---

### **7. How does Terraform’s `for_each` differ from `count`?**

### **Answer:**

- `count` is index-based (`count.index`), useful for simple lists.
- `for_each` works with sets and maps, allowing dynamic key-value associations.

Example using `count`:

```hcl

resource "aws_instance" "example" {
  count = 3
  ami   = "ami-123456"
}

```

Example using `for_each`:

```hcl

resource "aws_instance" "example" {
  for_each = toset(["dev", "qa", "prod"])
  ami      = "ami-123456"
  tags     = { Name = each.key }
}

```

---

### **8. What is the purpose of the `terraform refresh` command?**

### **Answer:**

`terraform refresh` updates the state file with the real-world state but **does not apply changes**.

Example usage:

```

terraform refresh

```

Since Terraform v1.1, `terraform refresh` is deprecated and `terraform apply -refresh-only` should be used.

---

### **9. What is Terraform Import, and how do you use it?**

### **Answer:**

Terraform `import` allows importing existing infrastructure into Terraform state without modifying resources.

Example:

```

terraform import aws_instance.example i-1234567890abcdef

```

After importing, update the `.tf` file to match the real-world configuration.

---

### **10. How do you use the `terraform taint` command?**

### **Answer:**

`terraform taint` marks a resource for recreation in the next `terraform apply`.

Example:

```

terraform taint aws_instance.example
terraform apply

```

In Terraform v0.15+, `terraform taint` is removed. Instead, use:

```

terraform apply -replace="aws_instance.example"

```

---

### **11. Explain the difference between `terraform destroy` and `terraform apply -destroy`.**

### **Answer:**

- `terraform destroy`: Destroys all resources in the state file.
- `terraform apply -destroy`: Also destroys resources but allows for additional plan checks before applying.

---

### **12. How can you handle cross-account deployments in Terraform?**

### **Answer:**

- Use multiple AWS profiles:
    
    ```
    
    export AWS_PROFILE=dev
    terraform apply
    
    ```
    
- Use Terraform providers with different aliases:
    
    ```hcl
    provider "aws" {
      alias  = "dev"
      region = "us-east-1"
    }
    
    provider "aws" {
      alias  = "prod"
      region = "us-west-2"
    }
    
    ```
    

---

### **13. What is the purpose of the `terraform data` source?**

### **Answer:**

`data` sources allow Terraform to **fetch existing data** without creating new resources.

Example:

```hcl

data "aws_vpc" "existing_vpc" {
  id = "vpc-123456"
}

```

The `data` block does not create resources, it only reads existing ones.

---

### **14. How does Terraform handle provider versioning?**

### **Answer:**

Terraform allows version constraints for providers:

```hcl

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

```

Use `terraform providers` to check installed versions.

---

### **15. How do you optimize Terraform performance for large infrastructures?**

### **Answer:**

- **Enable parallelism** (`terraform apply -parallelism=10`).
- **Use modules** to break down configurations.
- **Use caching** for remote states (`terraform refresh`).
- **Use state locking** to prevent concurrency issues.
- **Use the `target` flag** to apply changes to specific resources.
