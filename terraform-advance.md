## Terraform Advanced Scenario-Based Questions & Answers (2025)



### 1. Two developers run `terraform apply` on the same module at the same time. What happens and how do you prevent state corruption?

**Behavior**

- **First apply**: Acquires the state lock and proceeds.
- **Second apply**: Fails with a lock error, for example:

```text
Error: Error acquiring the state lock
```

**How to prevent corruption**

- **Use a remote backend with state locking**:
  - **AWS S3 + DynamoDB** (DynamoDB table for state locking)
  - **Terraform Cloud / Enterprise**
  - **Consul** (as a backend with locking)
- **Best practice**: Run Terraform via a **CI/CD pipeline** and **block local applies** to ensure serialized, auditable changes.

---

### 2. Changing the AMI ID in a Launch Template forces ASG recreation. How do you achieve zero (or near-zero) downtime?

**Goal**: Update the Auto Scaling Group (ASG) to use a new AMI while avoiding downtime.

**Solution**

- Use `create_before_destroy` to create new resources before destroying the old ones:

```hcl
lifecycle {
  create_before_destroy = true
}
```

- Optionally ignore Launch Template version drift:

```hcl
lifecycle {
  ignore_changes = [latest_version]
}
```

**Why**

- ASG updates to its Launch Template often trigger **replacement**.
- `create_before_destroy` ensures new instances are launched and become healthy **before** old instances are terminated.

---

### 3. Terraform wants to recreate an entire RDS instance due to a minor parameter change, but you must avoid downtime. What do you do?

**Problem**: Some RDS attributes force replacement; you only want a non-disruptive parameter change.

**Solution**

- Ignore specific parameter changes at the DB instance level:

```hcl
lifecycle {
  ignore_changes = [parameter_group_name]
}
```

- Or move configuration into a separate **DB parameter group** and update that instead of the instance resource.

---

### 4. After `terraform import` of an EC2 instance, every `plan` shows changes even though AWS hasn’t changed. Why?

**Reason**

- `terraform import` synchronizes **state only**, not your **code**.
- Your resource block does not exactly match the real resource attributes.

**Fix**

1. Run:

   ```bash
   terraform show
   ```

2. Inspect the imported resource’s attributes.
3. Update the Terraform resource block so that all relevant arguments match the **actual** values.
4. Run `terraform plan` until the plan is clean (no changes).

---

### 5. Terraform marks a resource as tainted and keeps recreating it on every apply. How do you find and fix the cause?

**How to inspect**

```bash
terraform state show <resource_address>
```

**Common reasons**

- **Drift**: The resource was changed manually in the cloud.
- **Missing attributes** in the code that Terraform infers as changed.
- **Provider bug** or flapping computed values.
- A field is **computed** and changes on every refresh.

**Fix**

- Explicitly replace the resource once:

```bash
terraform apply -replace="aws_instance.example"
```

- Then adjust your code (or `ignore_changes`) to prevent repeated “differences”.

---

### 6. One module deploys VPCs for dev, stage, and prod. Dev/stage use 2 subnets, prod uses 6. How do you design the module?

**Goal**: Reuse a single module with a different number of subnets per environment.

**Solution**

- Use `for_each` with a map and avoid `count` (which is order-sensitive and can cause recreation):

```hcl
variable "subnets" {
  type = map(list(string))
}

resource "aws_subnet" "this" {
  for_each = toset(var.subnets[var.env])

  vpc_id     = var.vpc_id
  cidr_block = each.value
  # ... other arguments ...
}
```

- Optionally use **dynamic blocks** for nested arguments that vary per environment.

---

### 7. You want to deploy the same infra across 10 AWS accounts from one code base. How do you design it?

**Solution**

- Define multiple AWS providers with aliases:

```hcl
provider "aws" {
  alias  = "acc1"
  region = "ap-south-1"
}

provider "aws" {
  alias  = "acc2"
  region = "eu-west-1"
}
```

- Override the provider in module calls:

```hcl
module "network_acc1" {
  source    = "./network"
  providers = { aws = aws.acc1 }
  # ...variables...
}
```

- Use **STS AssumeRole** in provider configuration for secure, cross-account access.

---

### 8. Terraform plan is slow because it queries thousands of resources. How do you optimize performance?

**Strategies**

- Use `lifecycle ignore_changes` for attributes that don’t matter for your drift detection.
- Use **data sources only when required**; avoid overusing them to fetch large inventories.
- **Split** the configuration into multiple root modules / state files:
  - Separate state for “network”, “database”, “app”, etc.
  - Use **remote state data sources** to share outputs.
  - Optionally use **Terragrunt** to orchestrate multiple stacks.

---

### 9. A sensitive variable (like a password) accidentally got committed to Git. How do you prevent this in the future?

**Terraform configuration**

```hcl
variable "password" {
  type      = string
  sensitive = true
}
```

**Practices to prevent future leaks**

- Add `terraform.tfvars` (and any secrets files) to **`.gitignore`**.
- Use **pre-commit hooks** to block committing secrets.
- Store secrets in:
  - **HashiCorp Vault**
  - **AWS SSM Parameter Store**
  - **AWS Secrets Manager** or similar secret managers

---

### 10. Terraform wants to recreate an ALB every time because AWS injects default tags. How do you stop this?

**Problem**

- AWS adds or modifies **default tags**, which causes perpetual diffs.

**Solution**

- Ignore provider- or AWS-managed tags:

```hcl
lifecycle {
  ignore_changes = [tags]
}
```

**Note**: This pattern is common for **EKS**, **ALB**, **EC2**, **S3**, and other managed resources that auto-add tags.

---

### 11. You want to share outputs from one Terraform project into another. How do you do it?

**Solution: Remote state data source**

```hcl
data "terraform_remote_state" "vpc" {
  backend = "s3"

  config = {
    bucket = "tf-state"
    key    = "vpc/terraform.tfstate"
    region = "ap-south-1"
  }
}
```

Then you can consume:

```hcl
vpc_id = data.terraform_remote_state.vpc.outputs.vpc_id
```

---

### 12. `terraform destroy` is stuck because RDS requires a final snapshot. How do you handle this?

**Options**

- Skip final snapshot:

```hcl
skip_final_snapshot = true
```

- Or explicitly create a final snapshot:

```hcl
final_snapshot_identifier = "backup-before-destroy"
```

---

### 13. Terraform keeps recreating an S3 bucket because AWS changes ACL/ownership defaults. How do you stabilize it?

**Problem**

- AWS’s S3 **Object Ownership** and ACL features can auto-adjust settings, causing drift.

**Solution**

- Ignore those fields:

```hcl
lifecycle {
  ignore_changes = [
    acl,
    grant,
    object_ownership
  ]
}
```

This is very common after AWS introduced S3 Object Ownership defaults.

---

### 14. Terraform cannot delete a resource because it’s still attached (e.g., ENI, IAM role). How do you resolve cyclic dependencies?

**Approaches**

- Use explicit `depends_on` to control order:

```hcl
depends_on = [
  aws_iam_role_policy.this,
  aws_security_group.this
]
```

- If necessary, **split** the teardown into multiple phases:
  - First detach or delete dependent resources.
  - Then destroy the primary resource.

---

### 15. You have 10 environment folders (dev → prod) with high code duplication. How do you make it DRY?

**Solution: Use Terragrunt**

- Example layout:

```text
/live/dev/network/terragrunt.hcl
/live/prod/network/terragrunt.hcl
/modules/network/main.tf
```

- Terragrunt helps with:
  - **Remote state** configuration
  - **State locking**
  - **Backend config** reuse
  - **Environment variables**
  - A DRY, hierarchical structure for multiple environments

#### how you handle the state management in your env
```
In my environment, I handle Terraform state management using a remote backend to ensure collaboration, security, and consistency across the team.

I usually store the Terraform state file in an AWS backend like Amazon Web Services S3 with state locking enabled through Amazon Web Services DynamoDB.

This helps prevent multiple engineers from modifying the infrastructure at the same time and avoids state corruption.

I also separate state files based on environments like dev, staging, and production to keep resources isolated.

Sensitive data inside the state file is protected using encryption and restricted IAM access.

For large projects, I use modular Terraform design and sometimes split state files by application or infrastructure layer to reduce state size and improve performance.

Additionally, before applying changes, I review terraform plan output and maintain state backups/versioning for recovery if anything fails
```
#### how to do state locking?
```
Terraform state locking is used to prevent multiple users from changing the same infrastructure at the same time.

You can explain it in an interview like this:

“State locking is implemented to avoid concurrent Terraform operations.
When one user runs terraform apply, Terraform locks the state file so no other user can modify it until the operation completes.
In AWS, I usually configure remote state using an S3 backend and enable locking through a DynamoDB table.
Terraform creates a lock entry in DynamoDB before applying changes and removes it after completion.
This prevents state corruption in team environments.”
```
#### where do we use the terraform workspace ? what is the exact use of it?  
```
Terraform workspace is used when we want to deploy the same infrastructure code into different environments such as dev, staging, and production without duplicating code.
Each workspace maintains its own separate state file, which means resources created in one workspace do not affect another workspace.
This helps isolate environments while reusing the same Terraform configuration.
```

#### you have to env dev and qa and you unfortunately deleted the prod workspace so what will happen now?
```
If a Terraform workspace like prod is deleted accidentally, the infrastructure usually still exists in the cloud provider, but Terraform loses direct workspace tracking for that environment.
The main risk is losing access to the associated state file mapping.
If the backend is remote, I can often recover by recreating the workspace and reconnecting to the existing state.
Before recovery, I would verify whether the infrastructure still exists and check the remote backend state.”

What happens step by step:

Workspace deleted
Terraform removes workspace metadata locally.
Infrastructure may still exist
Resources in Amazon Web Services AWS are usually not deleted automatically.
State file risk
If using remote backend (like S3), the prod state file may still exist.
If local backend, recovery becomes harder.
Possible recovery
Recreate workspace:
terraform workspace new prod
Reconnect state
Pull/import existing resources if needed.

How to check safely:

terraform workspace list
terraform state list

Best practice to avoid this:

Use remote backend + versioning in S3.
Enable backend backups.
Restrict delete permissions for production workspaces.
```

### you have stored everythink in s3 backend so only workspace is deleted . so can we bring that state once again in a new workspace? will it work?
```
Yes — if you are using a remote backend like Amazon Web Services S3 and only the Terraform workspace was deleted, you can usually recover it because the state file still exists in S3.

You can tell the interviewer like this:

“If only the Terraform workspace is deleted but the backend state still exists in S3, I can recreate the workspace and reconnect Terraform to the existing state file. Since the actual state is stored remotely, deleting the workspace does not remove infrastructure or backend state automatically.”

How recovery works:

Verify the state file still exists in the S3 backend.

Example path:
env:/prod/terraform.tfstate
Recreate the workspace:
terraform workspace new prod
Select the workspace:
terraform workspace select prod
Run:
terraform init
Terraform reconnects to the backend state.

Important point:

If the S3 state file still exists → recovery works.
If both workspace and state file are deleted → recovery becomes difficult.

For workspace-based state in remote backend, Terraform often stores state like:

s3://bucket-name/env:/prod/terraform.tfstate
```

### what happens if a provider is getting upgraded ? for ex you are in an aws 7.5 provider, and that needs to be update to 7.6 it's a major version . so how do you plan for this?
```
When a Terraform provider upgrade is required, I never upgrade directly in production. I follow a controlled upgrade process to avoid breaking infrastructure changes.

First, I review the provider release notes and changelog to identify breaking changes, deprecated resources, or argument modifications.

Then I update the provider version in Terraform configuration and test it in a lower environment like dev or QA before touching production.

I run terraform init -upgrade to download the new provider version and then execute terraform plan to compare infrastructure changes.

If the plan shows unexpected resource recreation or schema changes, I fix compatibility issues first.

After validation in non-production environments, I promote the same tested version to production through CI/CD.

I also keep a backup of the Terraform state and use version pinning so rollback is possible if required.”
```

### what are the cost optimization initiatives you have taken in your current env?
```
In my environment, I worked on multiple AWS cost optimization initiatives focused on reducing unused resources and improving utilization.

First, I identified idle or underutilized EC2 instances using monitoring metrics and either rightsized them or scheduled automatic shutdown during non-business hours.

I also moved stable workloads to Reserved Instances or Savings Plans to reduce long-term compute costs.

For storage optimization, I cleaned up unused EBS volumes, old snapshots, and unattached Elastic IPs.

In Kubernetes environments, I optimized node sizing and enabled cluster autoscaling to avoid overprovisioning.

I implemented S3 lifecycle policies to move old data into cheaper storage classes like Glacier.

I also enabled monitoring and tagging strategies to track cost per application, team, or environment.

Additionally, I reviewed load balancer usage, removed unused resources, and optimized logging retention periods to reduce CloudWatch storage costs.
```
