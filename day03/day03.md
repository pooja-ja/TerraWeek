# ☁️ TerraWeek Day 3 — Providers, Resources & Your First Cloud Infra

**Date:** Wednesday, 15th July 2026

Time to touch **real cloud infrastructure**! Today you'll configure a **provider**, use **data sources** and **meta-arguments** (`for_each`, `count`, `depends_on`, `lifecycle`), and provision a small network + compute stack on the cloud of your choice. 🏗️

---

## 🎯 Learning Goals

- Configure a **provider** properly with **version pinning** and **region**.
- Understand **resources** vs **data sources**.
- Use meta-arguments: **`count`**, **`for_each`**, **`depends_on`**, **`lifecycle`**.
- Provision, update, and destroy real cloud resources safely.

---

## 🗺️ 60-Second Networking Primer (read this first!)

Today jumps from a single container to a real cloud network. Don't panic — here are the **6 building blocks** you'll create, in plain English:

| Block | What it is | Real-world analogy |
|-------|------------|--------------------|
| **VPC** | Your own private, isolated network in the cloud (a range of IPs like `10.0.0.0/16`) | Your own gated neighborhood |
| **Subnet** | A slice of the VPC's IPs (`10.0.1.0/24`), lives in one Availability Zone | A street in that neighborhood |
| **Internet Gateway (IGW)** | The door between your VPC and the public internet | The neighborhood's main gate |
| **Route Table** | Rules that say "traffic for the internet → go via the IGW" | Road signs / GPS routes |
| **Security Group (SG)** | A stateful virtual firewall on the instance (which ports are open) | A bouncer checking who gets in |
| **EC2 Instance** | The actual virtual machine running your app | A house on the street |

**How they connect:** an **EC2 instance** lives in a **subnet**, inside a **VPC**. To reach the internet, the subnet's **route table** sends traffic through the **IGW**, and the **security group** decides which ports (e.g. 80/HTTP) are allowed in.

```
Internet ──▶ [IGW] ──▶ [Route Table] ──▶ [ Public Subnet ] ──▶ [SG] ──▶ [EC2]
                                          (inside the VPC)
```

> 💡 You'll build exactly this stack in Task 3. Re-read this table if a resource name ever feels confusing.

---

## 📝 Tasks

### Task 1: Providers & Version Pinning
- Add a `terraform` block with `required_version` and `required_providers` (pin with `~>`).
  ```hcl
terraform {
  required_version = ">= 1.10"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.6"
    }
  }
}
---
- Explain **why version pinning matters** and what the `~>` (pessimistic) operator does.
  without it, `terraform init` can pull down whatever the latest provider version happens to be on that day — which can silently introduce breaking changes months after the config was written. Pinning guarantees the same config behaves the same way regardless of when or where it's run.
  the pessimistic constraint operator. `~> 6.0` means "allow any 6.x version (6.1, 6.2, 6.99...) but never jump to 7.0 automatically." You get bug fixes and minor updates without risking a major breaking change sneaking in on a routine `init`.
- **Bonus:** configure a second provider **alias** (e.g. a second AWS region) and explain when you'd use it.
```hcl
provider "aws" {
  alias  = "backup"
  region = var.backup_region
  ...
}
```
A Terraform provider alias is a named, secondary provider configuration that lets you use the same provider with different settings in one run. It is used to target multiple regions, accounts, or endpoints without duplicating code. Aliases improve clarity when managing cross-region or cross-account deployments.

### Task 2: Resources vs Data Sources
- **🛠️ Resource** (creates/manages something): `aws_vpc.main`, `aws_instance.web`, `aws_s3_bucket.protected_demo` — Terraform owns the full lifecycle of these, create through destroy.
- **🔍 Data source** (only reads): `data.aws_ami.al2023` and `data.aws_availability_zones.available` — these read information that already exists outside Terraform's control (AWS's own published AMIs, the region's AZ list) so the config can reference it without hardcoding IDs that change over time.

### Task 3: Provision a Cloud Stack
Built the full network + compute stack: VPC, public subnet, internet gateway, route table, security group (port 80 open), and an EC2 instance that installs Nginx via `user_data` on boot — no SSH key pair needed.

```bash
cd example
terraform init
terraform validate
terraform plan
terraform apply      # type: yes
terraform state list # see everything Terraform now manages
```
**terraform plan:**
<img width="1540" height="720" alt="image" src="https://github.com/user-attachments/assets/88c03131-b07d-4c9b-8b34-2bff84452572" />
**terraform apply:**
<img width="1333" height="720" alt="image" src="https://github.com/user-attachments/assets/77972bd6-26e5-4086-acb0-0f004935b144" />
**Verified in the browser (public IP from the output):**
<img width="1296" height="720" alt="image" src="https://github.com/user-attachments/assets/35836f0b-4ba0-46bc-b4ac-f42ea503e533" />

### Task 4: Meta-Arguments in Action
Extend the config to practice each of these:
- **`count`** — create N identical resources (e.g. N EC2 instances).
- **🔢 `count`** — two identical, interchangeable "worker" EC2 instances. They don't need individual names or identities, so `count` is the simpler fit here.
```hcl
resource "aws_instance" "worker" {
  count = var.extra_worker_count
  ...
}
```
- **`for_each`** — create resources from a `map`/`set` (preferred over `count` for named things).
- **🔁 `for_each`** — two named private subnets (`app`, `db`) built from a map, so each one has a stable identity. Deleting one later won't reshuffle the other, unlike an indexed list.
```hcl
resource "aws_subnet" "private" {
  for_each          = var.private_subnets
  cidr_block        = each.value
  ...
}
```
- **`depends_on`** — force an explicit ordering.
- **⛓️ `depends_on`** — Terraform infers most ordering from attribute references automatically, but a `null_resource` that doesn't reference any attribute of the web server still needed to run only after it existed, so the dependency had to be stated explicitly.
```hcl
resource "null_resource" "post_boot_check" {
  depends_on = [aws_instance.web]
  ...
}
```
- **`lifecycle`** — try `create_before_destroy`, `prevent_destroy`, and `ignore_changes`.

```hcl
lifecycle {
  create_before_destroy = true
  ignore_changes        = [tags["LastModified"]]
}
```
**Confirmed in the AWS Console:**
<img width="1188" height="720" alt="image" src="https://github.com/user-attachments/assets/1d40139f-80b6-4b18-b769-4cba64e7114a" />
<img width="1479" height="720" alt="image" src="https://github.com/user-attachments/assets/c14fb0dc-c230-4cf3-a2eb-b6ef00f71d15" />
**Created S3 bucket in us-west-2 region using provider alias:**
<img width="1428" height="720" alt="image" src="https://github.com/user-attachments/assets/27843a3f-aac4-483b-9ef1-38156cf95547" />

### Task 5: Update & Destroy
- Change a `tag` or the `instance_type`, run `terraform plan`, and read the diff — notice what forces **replace** vs **in-place update**.
- **Always** finish with:
```bash
terraform destroy   # type: yes  — avoid surprise bills!
```
<img width="1531" height="720" alt="image" src="https://github.com/user-attachments/assets/d88d531e-d8fb-4cfb-86fd-f846af71b16d" />

---
### Happy Terraforming! 🌍💻
