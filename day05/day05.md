# 📦 TerraWeek Day 5 — Modules: Reusable, Composable Infrastructure

**Date:** Thursday, 16th July 2026
## 🎯 Learning Goals

- Understand **what modules are** and why they're the backbone of scalable Terraform.
- Understand the **root module** vs **child modules**.
- Write a **local module** with clear **inputs (variables)** and **outputs**.
- Consume modules from the **Terraform Registry** and **Git**, and **pin versions**.
- Use **`for_each`** to instantiate a module multiple times.

---

## 📝 Tasks

### Task 1: Modules — the Why
Answer in your notes:
- What is a **module**? What counts as the **root module**?
  A module is just a folder of `.tf` files — inputs in, resources managed, outputs out. The **root module** is wherever you run `terraform apply` from (here, `day05/example/`); anything it calls (`./modules/ec2_instance`, or a registry module) is a **child module**.
- What are the benefits — **reusability, consistency, encapsulation, versioning, testing**?
- 🔁 **Reusability** — write the EC2-launching logic once, call it for `web`, `app`, `worker`, `cache` without repeating a single resource block.
- 🎯 **Consistency** — every instance gets the same tagging scheme automatically.
- 📦 **Encapsulation** — callers only see inputs/outputs, not the resource internals.
- 🔒 **Versioning** — pin a module to a specific version/tag, upgrade deliberately.
- 🧪 **Testing** — a small, self-contained module is far easier to validate in isolation.
- What files make up a well-structured module (`main.tf`, `variables.tf`, `outputs.tf`, `README.md`)?
  `main.tf` (resources), `variables.tf` (inputs + validation), `outputs.tf` (exports), `README.md` (usage docs) — all present in `modules/ec2_instance/`.

### Task 2: Write Your Own Module
Use the **starter code in [`./example`](./example)**. It contains:
- A reusable module at [`./example/modules/ec2_instance`](./example/modules/ec2_instance)
- A **root module** ([`./example`](./example)) that calls it.

Study how the root resolves shared lookups **once** (AMI, subnet, security group) and passes them as **inputs**, then reads the module's **outputs**:
```hcl
module "web_server" {
  source                 = "./modules/ec2_instance"
  name                   = "web"
  instance_type          = "t2.micro"
  environment            = "dev"
  ami                    = data.aws_ami.al2023.id   # resolved in the root
  subnet_id              = local.subnet_id
  vpc_security_group_ids = local.security_group_ids
}

output "web_public_ip" {
  value = module.web_server.public_ip
}
```

> 💡 Notice the module takes **IDs as inputs** instead of doing its own lookups. That keeps it reusable and avoids running the same data source once per instance.
```bash
cd example
terraform init      # note how Terraform initializes the module
terraform plan
terraform apply
terraform destroy
```
<img width="1373" height="720" alt="image" src="https://github.com/user-attachments/assets/5cdbb5aa-f39a-4256-9e8f-08e5c083f71c" />
<img width="1449" height="720" alt="image" src="https://github.com/user-attachments/assets/a4438369-c524-4148-90d9-658f5443778c" />
<img width="1449" height="720" alt="image" src="https://github.com/user-attachments/assets/a4283971-ba74-49eb-8c6e-6cbaef9eb706" />

### Task 3: Modular Composition (`for_each`)
Instantiate the **same module multiple times** to build multiple servers cleanly:
```hcl
module "servers" {
  source   = "./modules/ec2_instance"
  for_each = toset(["app", "worker", "cache"])

  name                   = each.key
  instance_type          = "t2.micro"
  environment            = "dev"
  ami                    = data.aws_ami.al2023.id
  subnet_id              = local.subnet_id
  vpc_security_group_ids = local.security_group_ids
}
```
Add this to the root module and observe the plan.
<img width="1404" height="720" alt="image" src="https://github.com/user-attachments/assets/118a5fca-85bf-42b3-ba1c-c0460d1a82d3" />
Confirmed live in the AWS Console — 4 instances total
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/5c976c39-bf71-4a9c-bece-29dac37dcda0" />

### Task 4: Consume a Registry Module + Version Locking
Use a real, popular module from the **[Terraform Registry](https://registry.terraform.io/)** — e.g. the official AWS VPC module — and **pin its version**:
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"   # ✅ always pin registry/git module versions

  name = "terraweek-vpc"
  cidr = "10.0.0.0/16"
  # ...
}
```
Init output confirms Terraform pulling in the local module
<img width="1473" height="720" alt="image" src="https://github.com/user-attachments/assets/a4ce563a-dd44-411f-8c62-a469a2547b01" />
<img width="1437" height="720" alt="image" src="https://github.com/user-attachments/assets/a1818497-8310-445b-8592-636be16adfcd" />
Confirmed live in the AWS Console
<img width="1177" height="720" alt="image" src="https://github.com/user-attachments/assets/10288d5d-01f1-4790-b89b-6f3f7c1dd42d" />


### Task 5: Ways to Lock Module Versions
Document, with code snippets, **each** way to pin a module source:
- **Registry:** `version = "~> 5.0"` (also `= 5.1.2`, `>= 5.0, < 6.0`).
- **Git tag/ref:** `source = "git::https://github.com/org/repo.git//path?ref=v1.2.0"`.
- **Git commit SHA** for immutability: `?ref=<full-sha>`.
Explain why **pinning matters** (reproducible builds, no surprise breaking changes).

---

> 📚 **Reference the companion repo:** [`aws_module_project/`](https://github.com/LondheShubham153/terraform-for-devops/tree/main/aws_module_project) is a real multi-environment example — one reusable [`my_app_infra_module`](https://github.com/LondheShubham153/terraform-for-devops/tree/main/aws_module_project/my_app_infra_module) (EC2 + S3 + DynamoDB) instantiated three times for **dev / stg / prd** with different instance sizes. Study how [`main.tf`](https://github.com/LondheShubham153/terraform-for-devops/blob/main/aws_module_project/main.tf) passes inputs and reads outputs.

## 🧠 `~>` (Pessimistic Constraint) Cheatsheet
- `~> 5.0` → allows `5.x`, **not** `6.0`.
- `~> 5.1.0` → allows `5.1.x`, **not** `5.2.0`.

---


### Happy Terraforming! 🌍💻
