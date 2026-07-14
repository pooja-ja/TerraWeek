# 🧩 TerraWeek Day 2 — HCL Deep Dive: Variables, Types & Expressions

**Date:** Tuesday, 14th July 2026
---

## 🎯 Learning Goals

- Understand HCL **blocks, arguments, and expressions**.
- Use **input variables** with types, defaults, validation, and `sensitive`.
- Use **`locals`**, **`outputs`**, and built-in **functions**.
- Understand **variable precedence** (`tfvars`, `-var`, env vars).

---

## 📝 Tasks

### Task 1: Master HCL Syntax
Explain (with examples) in your notes:
- The anatomy of a **block**: `block_type "label_one" "label_two" { argument = value }`.
```
block_type "label_one" "label_two" {
  argument = value
}
```
`resource "docker_container" "web" { ... }` — `resource` is the block type, `"docker_container"` is resource type `"web"` resource name, and everything inside `{}` is arguments.
- The difference between an **argument** and a **block**.
An argument is a single `key = value` line (like `name = "nginx"`). A block is a nested structure that can itself contain more arguments or blocks — like the `validation { }` block inside a variable, or the `ports { }` and `dynamic "labels" { }` blocks inside the docker container resource. Arguments assign a value; blocks configure a nested structure.
- **Expressions**: string interpolation `"${...}"`, references (`resource.name.attr`), and operators.
- String interpolation: `"${local.name_prefix}-${var.container_name}"` — mixes literal text with references inside a string.
- References: `docker_image.nginx.image_id` — pointing at another resource's exported attribute.
- Operators: standard `==`, `!=`, `&&`, `||`, plus the ternary `condition ? true_val : false_val`.
  
### Task 2: Variables, Types & Validation
Create a `variables.tf` and define variables covering **each major type**:
- Primitives: `string`, `number`, `bool`
```hcl
variable "environment" {
  description = "Deployment environment name"
  type        = string
  default     = "dev"
}

variable "instance_count" {
  description = "Number of instances to create"
  type        = number
  default     = 2
}
variable "enable_monitoring" {
  description = "Enable monitoring for the deployment"
  type        = bool
  default     = true
}
```
- Collections: `list(string)`, `map(string)`, `set(string)`
```hcl
variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

variable "common_tags" {
  description = "Map of common resource tags"
  type        = map(string)
  default = {
    Environment = "dev"
    Owner       = "network-team"
  }
}
variable "allowed_cidrs" {
  description = "Set of allowed CIDR blocks"
  type        = set(string)
  default     = ["10.0.0.0/24", "192.168.1.0/24"]
}
```
- Structural: `object({...})`, `tuple([...])`
```hcl
variable "app_config" {
  description = "Structured application configuration"
  type = object({
    name           = string
    port           = number
    enable_tls     = bool
    instance_type  = string
  })
  default = {
    name          = "demo-app"
    port          = 8080
    enable_tls    = true
    instance_type = "t3.micro"
  }
}
variable "connection_info" {
  description = "Fixed-position tuple for connection details"
  type        = tuple([string, number, bool])
  default     = ["db.internal.local", 5432, true]
}
```
Add at least one variable with:
- a **`default`**,
- a **`validation`** block (e.g. only allow certain values),
- the **`sensitive = true`** flag.

```hcl
variable "external_port" {
  description = "Host port to expose the container on."
  type        = number
  default     = 8080

  validation {
    condition     = var.external_port > 1024 && var.external_port < 65535
    error_message = "external_port must be between 1025 and 65534."
  }
}
```
### Task 3: Locals, Outputs & Functions
- Use a **`locals`** block to compute a value (e.g. a common `name_prefix` or merged tags).
  `locals` computes `name_prefix` (string interpolation) and `common_labels` (merging default tags with `var.extra_labels` via `merge()`). Outputs expose the container's name, URL, and computed values so I don't have to dig through `docker ps` to find them.
- Add **`outputs`** that expose useful values.
- Use at least **3 built-in functions** — e.g. `upper()`, `merge()`, `join()`, `lookup()`, `length()`, `format()`.
  Explore them live with `terraform console`:
```bash
terraform console
> upper("terraweek")
> merge({a=1}, {b=2})
> join("-", ["tws", "terraweek", "2026"])
```
Tried the built-in functions live in `terraform console`:
<img width="1434" height="720" alt="image" src="https://github.com/user-attachments/assets/225f7311-f7ac-4b66-9e6f-1542fd511a38" />


### Task 4: Build Something Real (Docker provider — no cloud cost)
Used the `kreuzwerker/docker` provider to pull Nginx and run a container, with the image tag, port, and labels all coming from variables — no cloud account, no cost.
```bash
cd example
terraform init
terraform plan  -var 'container_name=tws-web' -var 'external_port=8080'
terraform apply -var 'container_name=tws-web' -var 'external_port=8080'
# visit http://localhost:8080
terraform output
terraform destroy -var 'container_name=tws-web' -var 'external_port=8080'
```
**terraform init:**
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/3271a965-e3d6-46ae-8fa3-ed020e88634c" />
**terraform plan:**
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/05aba127-9d7a-45de-92c7-b4d32e7548f3" />
**terraform apply:**
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/1a43d058-44a9-4af3-8277-62cc568caf30" />
**nginx running on localhost 8090**
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/3fff2d81-fa0f-4600-9f97-8889422ae63d" />
**terraform Destroy:**
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/96dd7f05-ebec-4aee-b853-6922fee248de" />

Tried the same run using a **`terraform.tfvars`** file instead of `-var` flags and note the difference.
**terraform plan:**
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/f0f4a5ad-6eba-4200-a900-957f9d0cd2b3" />
**terraform apply:**
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/85461cad-b747-4ad6-9ba7-3a83fd379aff" />
**terraform Destroy:**
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/8ba42894-1fab-492e-b3c8-8650aade8a05" />
verified using docker ps also.

---

## 📊 Variable Precedence (highest wins)
```
-var / -var-file  ▶  *.auto.tfvars  ▶  terraform.tfvars  ▶  TF_VAR_ env vars  ▶  default
```

---

## 🍫 Bonus (Brownie Points)
- **For expression:** `local.upper_names = [for n in var.names : upper(n)]` — transforms every item in the `names` list to uppercase.
- **Conditional expression:** `local.effective_memory_mb = var.environment == "prod" ? 512 : var.resource_profile.memory_mb` — picks a memory value based on environment.
- **`optional()` attribute:** the `resource_profile` object type marks `notes` as optional, so it defaults to `null` if not explicitly set.
---


### Happy Terraforming! 🌍💻
