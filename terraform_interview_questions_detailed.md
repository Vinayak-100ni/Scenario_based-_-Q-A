# Terraform Interview Questions -- Detailed Answers with Real-World Examples

This guide covers 10 common Terraform interview questions in an
interview-ready format. Each topic includes the concept, a practical
AWS-oriented example, and a sample answer you can explain to an
interviewer.

> **Note:** The AWS IDs, AMI IDs, passwords, and configuration values in
> these examples are placeholders. Replace them with valid values for
> your environment. Review all plans before applying changes, especially
> in production.

------------------------------------------------------------------------

## 1. What is the difference between `terraform import` and `terraform taint`?

### Understand the concept

These are different operations:

-   **Import** associates an existing infrastructure object with a
    Terraform resource address in state. It is useful when a resource
    was created outside Terraform and you want Terraform to manage it.
-   **Taint** was the older mechanism for marking a resource for
    replacement. The `terraform taint` command is deprecated; use
    `terraform apply -replace` instead.

### Example: Import an existing EC2 instance

First, define the resource in Terraform configuration:

``` hcl
resource "aws_instance" "my_ec2" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}
```

Then import the existing instance:

``` bash
terraform import aws_instance.my_ec2 i-0123456789abcdef0
```

The resource address is `aws_instance.my_ec2`; the instance ID
identifies the existing EC2 instance.

Import updates Terraform state. It does **not** automatically write a
complete resource configuration for you. Review and reconcile the
configuration with the real resource before applying changes.

### Example: Replace a resource

If an EC2 instance needs to be recreated:

``` bash
terraform plan -replace="aws_instance.my_ec2"
terraform apply -replace="aws_instance.my_ec2"
```

Review the plan carefully because replacement generally means destroying
and recreating the resource, which may cause downtime or data loss.

### Key differences

  -----------------------------------------------------------------------
  Import                              Replace
  ----------------------------------- -----------------------------------
  Brings an existing resource under   Recreates a resource already
  Terraform state management          managed by Terraform

  Does not create a new cloud         Plans destruction and recreation
  resource                            

  Useful for adopting manually        Useful when a resource must be
  created infrastructure              recreated
  -----------------------------------------------------------------------

### Interview-ready answer

"Terraform import is used to bring an already existing infrastructure
resource into Terraform state management. For example, if an EC2
instance was created manually through the AWS Console, I can import that
instance into Terraform without creating another one. I still need to
write and verify the matching Terraform configuration.

Terraform taint was previously used to mark a resource for recreation.
In modern Terraform, that command is deprecated, and we use the
`-replace` option with `terraform apply` instead.

So, import is for adopting existing infrastructure, whereas replace is
for recreating infrastructure."

------------------------------------------------------------------------

## 2. How do you manage secrets in Terraform without hardcoding them?

### Understand the concept

Secrets include database passwords, API tokens, private keys, and
credentials. They should not be hardcoded in Terraform files or
committed to Git.

Avoid this:

``` hcl
variable "db_password" {
  default = "MyPassword123"
}
```

Even when a variable is marked sensitive, its value may still be stored
in Terraform state or exposed through other systems that handle the
value.

### Method 1: AWS Secrets Manager

Store the secret in AWS Secrets Manager and retrieve it when needed:

``` hcl
data "aws_secretsmanager_secret_version" "db" {
  secret_id = "production/database"
}

locals {
  db_password = jsondecode(
    data.aws_secretsmanager_secret_version.db.secret_string
  )["password"]
}
```

A resource can then use the value:

``` hcl
resource "aws_db_instance" "database" {
  identifier        = "production-db"
  engine            = "mysql"
  instance_class    = "db.t3.micro"
  username          = "admin"
  password          = local.db_password
  skip_final_snapshot = true
}
```

This is an illustrative snippet, not a complete production database
configuration. Retrieving a secret into Terraform does not keep it out
of Terraform state if it is passed to a managed resource.

### Method 2: Environment variables

Declare the input variable:

``` hcl
variable "db_password" {
  type      = string
  sensitive = true
}
```

In Windows CMD:

``` cmd
set TF_VAR_db_password=MySecretPassword
```

In Linux:

``` bash
export TF_VAR_db_password="MySecretPassword"
```

Then run:

``` bash
terraform plan
terraform apply
```

### Method 3: HashiCorp Vault

Terraform can integrate with HashiCorp Vault to retrieve secrets from a
centralized secrets-management system.

### Security best practices

-   Use a secure remote backend with encryption at rest.
-   Restrict state access with IAM or equivalent permissions.
-   Do not commit secret-bearing `.tfvars` files.
-   Use `sensitive = true` to reduce accidental display in CLI output.
-   Remember that `sensitive = true` masks output; it does not encrypt
    or remove values from state.
-   Rotate credentials and use least-privilege access.

### Interview-ready answer

"In real-time projects, I never hardcode passwords, API keys, or cloud
credentials inside Terraform files. I prefer storing secrets in AWS
Secrets Manager or HashiCorp Vault and retrieving them when required.
For input variables, I can also use environment variables such as
`TF_VAR_db_password`.

I mark sensitive variables using `sensitive = true`, but I understand
that this only masks values in normal Terraform output; it does not
prevent them from being stored in state. Therefore, I secure the remote
backend, restrict state access, and avoid committing secret files to
Git."

------------------------------------------------------------------------

## 3. What is the difference between `count` and `for_each`? Give a real-world use case.

### Understand the concept

Both `count` and `for_each` create multiple instances of a resource or
module.

-   `count` identifies instances using numeric indexes.
-   `for_each` identifies instances using map keys or set elements.

### Example: `count`

``` hcl
resource "aws_instance" "servers" {
  count         = 3
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}
```

Terraform addresses the instances as:

``` text
aws_instance.servers[0]
aws_instance.servers[1]
aws_instance.servers[2]
```

You can refer to the first instance with:

``` hcl
aws_instance.servers[0].id
```

Use `count` when you need a number of similar instances and their
identities do not need meaningful keys.

### Example: `for_each`

``` hcl
variable "instances" {
  type = map(string)

  default = {
    dev   = "t3.micro"
    stage = "t3.small"
    prod  = "t3.medium"
  }
}

resource "aws_instance" "servers" {
  for_each      = var.instances
  ami           = "ami-12345678"
  instance_type = each.value

  tags = {
    Name = each.key
  }
}
```

Terraform addresses the instances as:

``` text
aws_instance.servers["dev"]
aws_instance.servers["stage"]
aws_instance.servers["prod"]
```

-   `each.key` is the map key.
-   `each.value` is the corresponding map value.

### Key differences

  -----------------------------------------------------------------------
  `count`                             `for_each`
  ----------------------------------- -----------------------------------
  Uses numerical indexes              Uses meaningful keys or set
                                      elements

  Useful for a fixed number of        Useful when instances have distinct
  similar instances                   identities or values

  Uses `count.index`                  Uses `each.key` and `each.value`

  Inserting or removing list elements Stable keys help preserve resource
  can shift indexes                   identity when other entries are
                                      added or removed
  -----------------------------------------------------------------------

### Real-world use case

If a team needs three identical web servers, `count` may be suitable. If
it needs different configurations for dev, staging, and production,
`for_each` is often clearer because each instance has a meaningful key.

### Interview-ready answer

"Both `count` and `for_each` are used to create multiple resources in
Terraform. Count uses numerical indexes such as 0, 1, and 2, so I use it
when I need multiple similar resources. For example, I can create three
identical EC2 instances using `count = 3`.

For_each uses unique keys from a map or set. I prefer it when resources
have different configurations or meaningful identifiers. For example, if
I need EC2 instances for dev, staging, and production with different
instance types, I can use `for_each`.

The main advantage of `for_each` is that resources are identified by
stable keys rather than numerical indexes, which makes changes to the
collection easier to manage."

------------------------------------------------------------------------

## 4. How do you handle drift detection in Terraform?

### Understand the concept

Drift occurs when the real infrastructure differs from the configuration
Terraform is intended to manage.

For example, Terraform configuration specifies:

``` hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
}
```

An engineer manually changes the EC2 instance type in the AWS Console to
`t3.medium`. The actual infrastructure now differs from the desired
configuration.

### Detect drift

Run:

``` bash
terraform plan
```

Terraform refreshes its view of managed resources and compares the
observed values with the desired configuration. The plan can show the
difference and the action Terraform proposes.

For a refresh-only review:

``` bash
terraform plan -refresh-only
```

This shows how observed remote changes would update state, without
proposing normal configuration changes.

### Handle drift

**If the manual change was not approved:** keep the desired
configuration and review the normal plan. If the plan is correct, apply
it to bring infrastructure back into line.

``` bash
terraform plan
terraform apply
```

**If the manual change was approved:** update the Terraform
configuration to reflect the intended value, then run and review a new
plan.

``` hcl
instance_type = "t3.medium"
```

Then:

``` bash
terraform plan
terraform apply
```

### Real-world examples of drift

-   Someone changes an EC2 instance type.
-   A security group rule is edited manually.
-   An S3 bucket setting is changed.
-   An RDS parameter is modified outside Terraform.

A CI/CD pipeline can run scheduled plans to help detect drift. Review
and approval should happen before applying changes.

### Interview-ready answer

"Drift detection means identifying differences between actual cloud
infrastructure and the desired configuration defined in Terraform.

For example, suppose Terraform created an EC2 instance with a `t3.micro`
instance type, but someone manually changes it to `t3.medium` in the AWS
Console. When I execute `terraform plan`, Terraform refreshes its view
of the infrastructure and identifies the difference.

If the manual change is unauthorized, I review a plan to restore the
desired configuration. If it is approved, I update the Terraform code
and review the new plan. In production, I also recommend scheduled plans
through CI/CD pipelines to detect drift early."

------------------------------------------------------------------------

## 5. What is a Terraform remote backend, and why is it important?

### Understand the concept

Terraform state records information about the resources Terraform
manages, such as EC2 instance IDs, VPC IDs, subnet IDs, and resource
attributes.

By default, Terraform stores state locally in `terraform.tfstate`. This
can create collaboration and consistency problems when multiple
engineers use separate local state files.

A **remote backend** stores state in a shared remote location.

Examples include:

-   Amazon S3
-   Azure Storage
-   Google Cloud Storage
-   HCP Terraform / Terraform Cloud

### AWS S3 backend example

``` hcl
terraform {
  backend "s3" {
    bucket       = "company-terraform-state"
    key          = "production/terraform.tfstate"
    region       = "ap-south-1"
    use_lockfile  = true
    encrypt       = true
  }
}
```

Initialize the backend:

``` bash
terraform init
```

The S3 backend stores state in the configured bucket and key. Modern S3
backends support native state locking with `use_lockfile = true`.

### Why is a remote backend important?

1.  Centralized state management.
2.  Shared state for team collaboration.
3.  State locking to prevent conflicting concurrent operations.
4.  Encryption and access controls.
5.  Versioning and recovery when configured on the storage bucket.
6.  Better integration with CI/CD workflows.

### Production best practices

-   Enable S3 bucket versioning.
-   Restrict access using IAM policies.
-   Enable encryption.
-   Use state locking.
-   Separate state keys for environments.
-   Treat state as sensitive because it can contain secret values.

### Interview-ready answer

"A Terraform backend defines where Terraform stores its state file. By
default, Terraform stores the state locally, but in a team environment
that can create synchronization problems because engineers may have
different copies of state.

To solve this, we can use a remote backend such as AWS S3. In an AWS
project, I configure an S3 bucket and state key, enable encryption,
configure bucket versioning, and use S3 native lockfiles to prevent
conflicting operations.

This gives the team centralized state management and safer
collaboration. I also restrict access to the state because it may
contain sensitive information."

------------------------------------------------------------------------

## 6. How do you manage multiple environments (dev, staging, prod) in Terraform?

### Understand the concept

Organizations commonly maintain separate environments:

  Environment   Purpose                   Example EC2 type
  ------------- ------------------------- ------------------
  Dev           Development and testing   `t3.micro`
  Staging       Pre-production testing    `t3.small`
  Production    Live application          `t3.medium`

The infrastructure may be similar, but sizing, networking, access, and
availability requirements can differ.

### Method 1: Separate directories with reusable modules

A common structure is:

``` text
terraform/
├── modules/
│   ├── ec2/
│   ├── vpc/
│   └── security-group/
├── dev/
│   ├── main.tf
│   ├── variables.tf
│   └── terraform.tfvars
├── staging/
│   ├── main.tf
│   ├── variables.tf
│   └── terraform.tfvars
└── prod/
    ├── main.tf
    ├── variables.tf
    └── terraform.tfvars
```

Dev configuration:

``` hcl
module "ec2" {
  source        = "../modules/ec2"
  instance_type = "t3.micro"
  environment   = "dev"
}
```

Production configuration:

``` hcl
module "ec2" {
  source        = "../modules/ec2"
  instance_type = "t3.medium"
  environment   = "prod"
}
```

Use a separate remote state key for each environment, for example:

``` text
dev/terraform.tfstate
staging/terraform.tfstate
prod/terraform.tfstate
```

Each directory should have an appropriately configured backend and its
own initialization.

### Method 2: Terraform workspaces

Workspaces provide multiple state instances for the same configuration.

Create workspaces:

``` bash
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod
```

List them:

``` bash
terraform workspace list
```

Select one:

``` bash
terraform workspace select dev
```

Show the current workspace:

``` bash
terraform workspace show
```

Example configuration:

``` hcl
locals {
  instance_type = {
    dev     = "t3.micro"
    staging = "t3.small"
    prod    = "t3.medium"
  }
}

resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = local.instance_type[terraform.workspace]

  tags = {
    Environment = terraform.workspace
  }
}
```

### Method 3: Separate repositories

Some organizations use separate repositories for environments when they
need stronger workflow, ownership, or access separation.

### Which approach is preferable?

For many production systems, separate environment directories or root
configurations with reusable modules and separate remote states provide
clear isolation and independent deployment workflows.

Workspaces can be useful for multiple state instances of the same
configuration, but they are not a security boundary by themselves.

### Interview-ready answer

"In real-time projects, we generally maintain separate environments such
as dev, staging, and production. I prefer reusable Terraform modules
with separate environment configurations.

For example, I create an EC2 module once and reuse it in dev, staging,
and production with different variable values. Dev might use a
`t3.micro`, staging a `t3.small`, and production a `t3.medium`.

I also maintain separate remote state files for each environment using
different backend keys. This helps isolate environments, manage
deployments independently, and reduce the risk of accidental production
changes.

Terraform workspaces are another option when we need multiple state
instances for the same configuration, but I don't treat workspaces as a
security boundary."

------------------------------------------------------------------------

## 7. What is the difference between `local-exec` and `remote-exec` provisioners?

### Understand the concept

Terraform provisioners can run commands during resource creation or
destruction. Two commonly discussed provisioners are `local-exec` and
`remote-exec`.

### A. `local-exec`

`local-exec` runs a command on the machine executing Terraform, such as
a developer's workstation or a CI/CD runner.

``` hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"

  provisioner "local-exec" {
    command = "echo EC2 instance created successfully"
  }
}
```

The command runs locally, not on the EC2 instance.

Potential use cases include running a local script or triggering a local
notification.

### B. `remote-exec`

`remote-exec` runs commands on a remote machine, commonly through SSH or
WinRM.

``` hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
  key_name      = "my-key"

  provisioner "remote-exec" {
    inline = [
      "sudo yum install nginx -y",
      "sudo systemctl start nginx",
      "sudo systemctl enable nginx"
    ]

    connection {
      type        = "ssh"
      user        = "ec2-user"
      private_key = file("my-key.pem")
      host        = self.public_ip
    }
  }
}
```

This example assumes an appropriate Linux AMI, package manager, network
access, SSH access, and key configuration. The commands may need
adjustment for the chosen operating system.

### Key differences

  -----------------------------------------------------------------------
  `local-exec`                        `remote-exec`
  ----------------------------------- -----------------------------------
  Executes on the Terraform execution Executes on the target remote
  host                                machine

  Does not require SSH to the target  Usually requires SSH or WinRM

  Useful for local scripts            Useful for remote commands
  -----------------------------------------------------------------------

### Important production point

Provisioners are generally a last resort because they are less
predictable than declarative resource configuration. Prefer:

-   EC2 user data
-   cloud-init
-   Ansible or other configuration-management tools
-   Prebuilt machine images

### Interview-ready answer

"The main difference between local-exec and remote-exec is where the
command executes.

Local-exec runs commands on the machine where Terraform is running. For
example, after creating an EC2 instance, I can execute a local script or
trigger a notification from my CI/CD runner.

Remote-exec runs commands on the remote infrastructure, such as an EC2
instance, using SSH or WinRM. For example, I can connect to an EC2
instance and install Nginx.

However, in production, I prefer EC2 user data, cloud-init, Ansible, or
prebuilt images instead of relying heavily on Terraform provisioners."

------------------------------------------------------------------------

## 8. How do you safely roll back infrastructure changes after a failed deployment?

### Understand the concept

Rollback means recovering from a failed infrastructure change and
returning the system to a known-good state.

For example, an EC2 instance type is changed from `t3.medium` to
`t3.large`, but the application becomes unhealthy after deployment.

### Step 1: Identify the failure

Check:

-   Terraform apply output.
-   Cloud provider events.
-   Application and system logs.
-   Infrastructure health.
-   CI/CD pipeline logs and monitoring alerts.

### Step 2: Revert the Terraform configuration

If the previous configuration was known to be stable, revert the
relevant code change using Git.

For example, change:

``` hcl
instance_type = "t3.large"
```

Back to:

``` hcl
instance_type = "t3.medium"
```

### Step 3: Review the plan

``` bash
terraform plan
```

Confirm that the plan proposes the expected recovery changes. Check
whether any changes will force resource replacement or cause downtime.

### Step 4: Apply the rollback

``` bash
terraform apply
```

Only apply after reviewing the plan and confirming that the proposed
changes are safe.

### Step 5: Validate recovery

Check:

-   EC2 instance health.
-   Application availability.
-   Load balancer health checks.
-   Application logs.
-   Monitoring and alerts.

### What if Terraform state is damaged?

First stop further Terraform operations and preserve a backup of the
current state.

To pull the current state:

``` bash
terraform state pull
```

To save a backup:

``` bash
terraform state pull > backup.tfstate
```

If using S3, review object versions and identify the correct version.
Before restoring any old state, verify that it corresponds to the actual
infrastructure. An old state file may not reflect changes made since
that version.

### Production best practices

-   Track configuration changes in Git.
-   Enable remote state versioning and backups.
-   Require deployment reviews and approvals.
-   Review plans before applying.
-   Use blue-green or other safe deployment strategies where
    appropriate.
-   Maintain database backups and snapshots when needed.
-   Understand that not every infrastructure change can be rolled back
    without downtime or data recovery.

### Interview-ready answer

"In real-time projects, I handle rollback by first identifying the
failed infrastructure change and understanding its impact.

If the issue is caused by a Terraform configuration change, I revert the
code to the previous stable version using Git. Then I run
`terraform plan` to review the proposed rollback and verify that it is
safe.

After reviewing the plan, I execute `terraform apply` and validate the
infrastructure using health checks, monitoring, and application logs.

For state recovery, I use remote backend versioning and backups, but I
never blindly restore an old state file because it may not represent the
actual infrastructure. For critical production workloads, I also
consider blue-green deployment strategies to minimize downtime."

------------------------------------------------------------------------

## 9. Explain `terraform refresh` vs `terraform plan`.

### Understand the concept

The commands have different purposes:

-   Historically, `terraform refresh` updated Terraform state to reflect
    observed remote infrastructure.
-   `terraform plan` compares the desired configuration with the
    observed infrastructure and state, then proposes changes.

The standalone `terraform refresh` command is deprecated. Modern
Terraform workflows use refresh-only planning and applying.

### Refresh-only plan

``` bash
terraform plan -refresh-only
```

This reviews how changes observed in the remote infrastructure would
update Terraform state. It does not propose normal configuration
changes.

To apply the refresh-only state updates:

``` bash
terraform apply -refresh-only
```

Review the proposed changes before applying them.

### Normal plan

``` bash
terraform plan
```

This refreshes Terraform's view of remote objects by default, compares
the observed values with the desired configuration, and generates a plan
for bringing infrastructure toward the configuration.

### Example

Suppose the configuration specifies:

``` hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
}
```

Someone manually changes the instance type to `t3.small`.

A normal `terraform plan` can identify the difference and propose
changing the instance back to the configured value.

A refresh-only plan instead shows how the observed remote change would
be reflected in state, without proposing the normal configuration
correction.

### Key differences

  -----------------------------------------------------------------------
  Refresh-only workflow               Normal `terraform plan`
  ----------------------------------- -----------------------------------
  Reviews observed remote changes to  Reviews proposed changes to
  state                               converge on configuration

  Does not propose normal             Can propose infrastructure changes
  infrastructure changes              

  Applied with                        Applied with normal
  `terraform apply -refresh-only`     `terraform apply` after review
  -----------------------------------------------------------------------

### Interview-ready answer

"Terraform refresh and Terraform plan have different purposes.
Historically, `terraform refresh` was used to synchronize Terraform
state with actual infrastructure. The standalone command is deprecated,
and we now use `terraform plan -refresh-only` to review changes observed
in the remote infrastructure.

Terraform plan is used to compare the desired configuration in Terraform
files with the observed infrastructure and generate an execution plan.

For example, if someone manually changes an EC2 instance from `t3.micro`
to `t3.small`, a normal plan can show the change needed to bring it back
to the configuration. A refresh-only plan instead shows how the observed
remote change would update state.

So, refresh-only is for reviewing state synchronization, while a normal
plan is for reviewing proposed infrastructure changes."

------------------------------------------------------------------------

## 10. How do you write reusable Terraform modules?

### Understand the concept

A Terraform module is a collection of Terraform configuration files
grouped together to create reusable infrastructure components.

Instead of duplicating the same EC2 configuration for dev, staging, and
production, you can create one EC2 module and call it with different
input values.

### Step 1: Create the project structure

``` text
terraform-project/
├── main.tf
└── modules/
    └── ec2/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### Step 2: Define the resource

File: `modules/ec2/main.tf`

``` hcl
resource "aws_instance" "this" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name        = var.instance_name
    Environment = var.environment
  }
}
```

### Step 3: Define input variables

File: `modules/ec2/variables.tf`

``` hcl
variable "ami_id" {
  type = string
}

variable "instance_type" {
  type = string
}

variable "instance_name" {
  type = string
}

variable "environment" {
  type = string
}
```

### Step 4: Define outputs

File: `modules/ec2/outputs.tf`

``` hcl
output "instance_id" {
  value = aws_instance.this.id
}

output "public_ip" {
  value = aws_instance.this.public_ip
}
```

Outputs expose useful values from the module to the calling
configuration.

### Step 5: Call the module

Root `main.tf`:

``` hcl
module "dev_ec2" {
  source        = "./modules/ec2"
  ami_id        = "ami-12345678"
  instance_type = "t3.micro"
  instance_name = "dev-server"
  environment   = "dev"
}

module "prod_ec2" {
  source        = "./modules/ec2"
  ami_id        = "ami-12345678"
  instance_type = "t3.medium"
  instance_name = "prod-server"
  environment   = "prod"
}
```

The same module is now reused with different values.

### Step 6: Run Terraform commands

``` bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

### Example enterprise module structure

``` text
modules/
├── vpc/
├── ec2/
├── eks/
├── rds/
├── s3/
├── iam/
└── security-group/
```

Different environment configurations can reuse these modules with their
own values.

### Advantages of modules

1.  Code reusability.
2.  Reduced duplication.
3.  Standardized infrastructure.
4.  Easier maintenance.
5.  Better project organization.
6.  Easier collaboration.
7.  Consistent infrastructure across environments.

### Interview-ready answer

"Terraform modules are used to organize infrastructure code into
reusable components.

For example, if my organization needs EC2 instances in dev, staging, and
production, instead of writing the same EC2 resource configuration
multiple times, I create one reusable EC2 module.

Inside that module, I define the resource in `main.tf`, input variables
in `variables.tf`, and outputs in `outputs.tf`.

Then I call the module from the root configuration and pass different
values such as AMI ID, instance type, environment name, and server name.

For example, development can use `t3.micro`, while production can use
`t3.medium`.

This approach reduces code duplication, improves maintainability,
standardizes infrastructure, and makes it easier to manage large-scale
DevOps projects."

------------------------------------------------------------------------

# Bonus: How to perform well in a Terraform interview

Knowing definitions is not enough. Interviewers often want to know
whether you understand how Terraform is used in real projects.

Use this answer structure:

1.  **Definition:** Explain what the concept means.
2.  **Purpose:** Explain why it is used.
3.  **Practical example:** Describe an AWS or production scenario.
4.  **Commands or code:** Mention the relevant Terraform commands or
    configuration.
5.  **Best practices:** Explain how you would use it safely in
    production.

## Important Terraform commands cheat sheet

  -----------------------------------------------------------------------------------
  Command                                         Purpose
  ----------------------------------------------- -----------------------------------
  `terraform init`                                Initialize Terraform and configure
                                                  providers and backend

  `terraform fmt`                                 Format Terraform files

  `terraform validate`                            Validate configuration syntax and
                                                  structure

  `terraform plan`                                Preview proposed infrastructure
                                                  changes

  `terraform apply`                               Apply approved infrastructure
                                                  changes

  `terraform destroy`                             Destroy resources managed by the
                                                  configuration

  `terraform state list`                          List resources tracked in state

  `terraform state show`                          Show a resource's state details

  `terraform state pull`                          Download the current state

  `terraform import`                              Import an existing resource into
                                                  state

  `terraform plan -refresh-only`                  Review how observed remote changes
                                                  would update state

  `terraform workspace list`                      List workspaces

  `terraform output`                              Display root-module outputs

  `terraform apply -replace="RESOURCE_ADDRESS"`   Replace a selected resource after
                                                  reviewing the plan
  -----------------------------------------------------------------------------------

## Five important areas to master

1.  **State management:** State files, remote backends, locking, and
    drift.
2.  **Resource management:** Import, replace, `count`, and `for_each`.
3.  **Security:** Secrets Manager, Vault, sensitive variables, and state
    protection.
4.  **Code organization:** Modules, variables, outputs, and
    environments.
5.  **Production operations:** Rollback, plan, apply, and CI/CD.

## Final interview tip

Do not memorize every answer word for word. Understand the concept,
practice explaining it aloud, and use a real-world example. Be ready for
follow-up questions such as:

-   What happens if two engineers run `terraform apply` at the same
    time?
-   What happens if the state file is deleted?
-   Can Terraform manage resources created manually?
-   What is the difference between a data source and a resource?
-   What happens when a resource is removed from Terraform
    configuration?
-   How do you manage Terraform in a CI/CD pipeline?

The ability to explain the reasoning behind your approach is more
valuable than simply remembering commands.
