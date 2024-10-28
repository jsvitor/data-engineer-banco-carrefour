# Terraform Cheat Sheet

> This cheat sheet is a quick reference for Terraform commands, syntax, and best practices. It is not a comprehensive guide, but rather a collection of useful snippets and examples. For more detailed information, please refer to the [Terraform Documentation](https://www.terraform.io/docs/index.html). Also you can find a awsome and more complete cheat sheet for Terraform [here](https://cheat-sheets.nicwortel.nl/terraform-cheat-sheet.pdf).

Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently. An Awsome resource to see the Terraform landscape is [Terraform Roadmap](https://roadmap.sh/terraform).

## Terraform architecture

![Terraform Architecture](./images/terraform-architecture.png)

Terraform architecture consists in a set of components that work together to manage infrastructure. This components are:

- **Terraform CLI**: The command line interface for Terraform. It is used to interact with the Terraform runtime.
- **Terraform Runtime**: The runtime is responsible for executing the Terraform code and managing the state of the infrastructure.
- **Terraform Providers**: Providers are plugins that allow Terraform to manage infrastructure. For example, the AWS provider allows Terraform to manage AWS infrastructure.
- **Terraform Configuration**: The configuration is the code that describes the infrastructure that you want to manage. It is written in the Terraform language (HCL).
- **Terraform State**: The state is an important component of Terraform. It is a file that stores the details of the managed infrastructure. The state is used to track the details of the infrastructure and to ensure that the desired state is achieved.
- **Terraform Modules**: Modules are a way to package and reuse infrastructure code. They are a fundamental concept in Terraform that allow you to create reusable, modular, and maintainable infrastructure.

## Basic Commands

```bash
# Initialize Terraform working directory
terraform init

# Preview changes
terraform plan -out=tfplan

# Apply changes
terraform apply "tfplan"

# Taint a resource, this is useful when you need to force a replacement of the resource
terraform taint aws_instance.example && terraform apply "tfplan"

# Destroy infrastructure
terraform destroy

# Format code
terraform fmt

# Validate configuration
terraform validate

# Show current state
terraform show

# Import existing infrastructure
terraform import aws_instance.example i-1234567890abcdef0
```

## Basic Syntax

```hcl
# Provider configuration
provider "aws" {
  region = "us-west-2"
  profile = "default"
}

# Resource block
resource "aws_instance" "example" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
  
  tags = {
    Name = "example-instance"
  }
}

# Data source
data "aws_ami" "ubuntu" {
  most_recent = true
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

# Variables
variable "environment" {
  type        = string
  default     = "dev"
  description = "Environment name"
}

# Outputs
output "instance_ip" {
  value = aws_instance.example.public_ip
}

# Local values
locals {
  common_tags = {
    Environment = var.environment
    Project     = "MyProject"
  }
}
```

## Variables and References

> - You can specify descriptions, default values, types and constraints, validation rules, and sensitive values.
> - Also you can use variables to reference other variables, locals, and data sources.
> - Variables can be passed in at runtime, from a file, from environment variables, or from a remote secrets management service.
> - Variables types can be `string`, `number`, `bool`, `list` (or `tuple`), `set`, `map` (or `object`), one special type that has no type: `null`.
> - see more at [Types and Values](https://developer.hashicorp.com/terraform/language/expressions/types)

```hcl
# Variable types
variable "ports" {
  type    = list(number)
  default = [80, 443]
}

variable "settings" {
  type = map(string)
  default = {
    "key1" = "value1"
    "key2" = "value2"
  }
}

# Variable references
${var.environment}
# Local references, these are useful when you need to reference a value that is defined in a local block.
${local.common_tags}
```

### Ways to populate variables

- Hardcoded values
- From a file, `terraform.tfvars` or `*.auto.tfvars` or `*.tfvars.json`
  - From command line, `terraform plan -var="environment=prod"`
  - From command line, `terraform plan -var-file="terraform.tfvars"`
  - `tfvars` example:
    ```hcl
    environment = "prod"
    region      = "us-west-2"
    zone        = "us-west-2a"
    instance_type = "t2.micro"
    ```
- From environment variables, `TF_VAR_environment`
- From a remote secrets management service like AWS Secrets Manager, Google Secrets Manager, or HashiCorp Vault

## Common Functions

```hcl
# Variable types
variable "ports" {
  type    = list(number)
  default = [80, 443]
}

variable "settings" {
  type = map(string)
  default = {
    "key1" = "value1"
    "key2" = "value2"
  }
}

# Variable references
${var.environment}
${local.common_tags}
````

## Conditional Expressions

```hcl
# Ternary operator
condition ? true_val : false_val

# Dynamic blocks
dynamic "setting" {
  for_each = var.settings
  content {
    name  = setting.key
    value = setting.value
  }
}
```

## Count and For Each

```hcl
# Count example
resource "aws_instance" "server" {
  count = 3
  ami   = "ami-123456"
  tags  = {
    Name = "server-${count.index}"
  }
}

# For each example
resource "aws_iam_user" "users" {
  for_each = toset(["user1", "user2", "user3"])
  name     = each.key
}
```

## State Management

```bash
# State commands
terraform state list
terraform state show aws_instance.example
terraform state mv aws_instance.example aws_instance.new
terraform state rm aws_instance.example
terraform state pull
terraform state push
```

## Workspaces

```bash
# Workspace commands
terraform workspace new dev
terraform workspace select prod
terraform workspace list
terraform workspace delete dev
```

## Importing Existing Infrastructure

```bash
# Terraform Import And Outputs
# import EC2 instance with id i-abcd1234 into the Terraform resource named "new_ec2_instance" of type "aws_instance"
terraform import aws_instance.new_ec2_instance i-abcd1234 
# same as above, imports a real-world resource into an instance of Terraform resource
terraform import 'aws_instance.new_ec2_instance[0]' i-abcd1234
# List all outputs as stated in code
terraform output
# List out a specific declared output
terraform output instance_public_ip
# List all outputs in JSON format
terraform output -json
```

## Modules

Terraform modules are a way to package and reuse infrastructure code. They are a fundamental concept in Terraform that allow you to create reusable, modular, and maintainable infrastructure.

In Terraform you can use modules from the [Terraform Registry](https://registry.terraform.io/) or you can create your own modules. But I highly recommend using modules from the registry as they are tested and maintained by the Terraform team. So you can focus on building your infrastructure without worrying about the underlying details of the module.

- [Terraform Modules](https://www.terraform.io/docs/language/modules/index.html)
- [Terraform Module Registry](https://registry.terraform.io/modules)
- [Terraform Module Marketplace](https://registry.terraform.io/marketplace)
- [Terraform Module Snippets](https://registry.terraform.io/browse/modules)
- [Terraform Module Development](https://www.terraform.io/docs/modules/development.html)
- [Terraform Module Structure](https://www.terraform.io/docs/modules/create.html#module-structure)

## Best Practices

- Use consistent naming conventions
- Organize code into modules
- Version control your Terraform code
- Use remote state storage
- Lock state files when working in teams
- Use data sources instead of hardcoding values
- Implement proper variable validation
- Use terragrunt for keeping your code DRY
- Always use version constraints for providers and modules. This ensures that the module is compatible with the provider version.

Remember to always run `terraform plan` before applying changes and maintain proper documentation for your infrastructure code!

## References and additional learning resources

- [Terraform Documentation](https://www.terraform.io/docs/index.html)
- [Terraform Registry](https://registry.terraform.io/)
- [Terraform Course](https://learn.hashicorp.com/terraform)
- [The Ultimate Terraform Cheatsheet](https://www.pluralsight.com/resources/blog/cloud/the-ultimate-terraform-cheatsheet)
- [Terraform on Google Cloud](https://cloud.google.com/docs/terraform)
- [Terraform on Google Cloud](https://www.youtube.com/watch?v=nvV6yobU710&list=PLpZQVidZ65jO_wtOpLv-HmC9uJgTRB8GT&index=1)
- [Terraform explained in 15 mins | Terraform Tutorial for Beginners](https://www.youtube.com/watch?v=l5k1ai_GBDE)
