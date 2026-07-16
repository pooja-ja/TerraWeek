# 🗄️ TerraWeek Day 4 — State & Remote Backends (Native Locking)

**Date:** Thursday, 16th July 2026

Terraform's **state** is the single most important concept for working on a **team**. Today you'll understand what state is, why it's sensitive, and how to store it **remotely and safely** — using the **modern S3 native state locking** (no DynamoDB needed anymore!). 🔐

---

## 🎯 Learning Goals

- Understand **what Terraform state is** and why it exists.
- Use the **`terraform state`** commands to inspect and manipulate state.
- Move from **local** to **remote** state with an **S3 backend**.
- Enable **S3 native state locking** with `use_lockfile` (the 2026 way).
- Safely **import** existing resources into state.

---

## 📝 Tasks

### 📝Task 1: Why State Matters
Explain in your notes:
- What is the **`terraform.tfstate`** file and what does it store?
  A JSON file that records everything Terraform manages — real-world resource IDs, all their attributes, and the dependency graph between them. It's Terraform's only record of what actually exists, mapped back to the resource blocks in code.
- Why should you **never** edit it by hand or commit it to Git?
  the state can contain **secrets in plaintext** — database passwords, API keys, anything that ends up as a resource attribute. Committing it to Git exposes those permanently in history. Hand-editing it risks a mismatch between what the file says exists and what's actually in the cloud, which then causes Terraform to make wrong decisions on the next apply.
- What is **state drift**, and how does `terraform plan` / `terraform refresh` relate to it?
  when real infrastructure changes outside of Terraform (someone edits a setting in the console, or a resource gets deleted manually) without Terraform's state being updated. `terraform plan` compares state against the real infrastructure and shows the difference; `terraform refresh` (now folded into `plan`/`apply` by default) re-syncs state to match reality without changing anything.
- Why is state **sensitive** (it can contain secrets in plaintext)?
  because it's a plaintext record of resource attributes, which can include credentials, connection strings, and other secrets depending on what resources are managed — hence encryption at rest and restricted access are mandatory for any remote backend.
---
### 📝Task 2: Explore Local State & `terraform state`
Start from **any** working config (reuse Day 3's, or the [`./backend_demo`](./backend_demo) here). After an `apply`, practice:
```bash
terraform state list                       # list all managed resources
terraform state show <resource_address>    # inspect one resource
terraform state mv <src> <dest>            # rename/move within state
terraform state rm <resource_address>      # stop managing (does NOT delete infra)
terraform show                             # human-readable state
```
Document what each command does and when you'd use it.
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/b969a5e6-c98c-46b0-b07d-f6a19d567241" />
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/2d2b4e8a-a8d3-4962-b213-da8974323188" />

- **`state list`** — lists every resource address Terraform currently manages.
- **`state show <address>`** — prints every attribute of one specific resource, straight from state.
- **`state mv`** — renames/moves a resource's address inside state without touching real infrastructure — used here to rename `random_pet.demo` to `random_pet.app_name` matching a code rename.
- **`state rm`** — tells Terraform to stop tracking a resource. The infrastructure itself is untouched; only Terraform's awareness of it is removed. Used here on `local_file.scratch` as prep for the `removed` block bonus demo.
- **`terraform show`** — prints the full state in human-readable form, equivalent to `state show` for every resource at once.
---
### 📝Task 3: Bootstrap the Backend Infrastructure
The S3 bucket that *holds* your state must exist **before** you configure the backend. Use [`./backend_infra`](./backend_infra) to create it (**local** state for this bootstrap step only):
```bash
cd backend_infra
terraform init
terraform apply    # creates the versioned, encrypted S3 state bucket
```
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/5dd5a025-fdd9-4e37-983d-4b727ae73cea" />
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/271b5191-362c-415f-95a8-303ccce39b63" />

### Task 4: Configure the Remote Backend with Native Locking
Now point a real config at that bucket. See [`./backend_demo`](./backend_demo):
```hcl
terraform {
  backend "s3" {
    bucket       = "your-unique-terraweek-state-bucket"
    key          = "day04/terraform.tfstate"
    region       = "us-east-1"
    encrypt      = true
    use_lockfile = true   # ✅ native S3 state locking — no DynamoDB!
  }
}
```
```bash
cd backend_demo
terraform init     # Terraform will offer to migrate local state → S3
terraform apply
```
Verify in the S3 console that your `terraform.tfstate` is uploaded, and watch a `.tflock` file appear/disappear during an apply.
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/5803261c-6a6a-4675-b1d8-18926f5d6c0a" />

### Task 5: Import an Existing Resource
Manually created an S3 bucket in the console (`terra-week-bucket-import`), then wrote the import block:

```hcl
import {
  to = aws_s3_bucket.imported
  id = "terra-week-bucket-import"
}
```
Generated the matching resource config instead of writing it by hand:

```
terraform plan -generate-config-out=generated.tf
```
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/12cd7b69-fc4a-4bf7-911b-a567ff7872f6" />
Reviewed `generated.tf`, then applied to bring it fully under management:
<img width="1421" height="720" alt="image" src="https://github.com/user-attachments/assets/1d36dcb2-407e-4f75-9219-2606a71598a4" />

---

## 🧹 Cleanup
```bash
cd backend_demo && terraform destroy
cd ../backend_infra && terraform destroy   # empty the bucket first if versioning blocks it
```
Destroyed `backend_demo` first, then `backend_infra`
---

## 🍫 Bonus (Brownie Points)
- Compare remote backends: **S3**, **HCP Terraform (Terraform Cloud)**, **Azure Storage**, **GCS**.
  | Backend | Notes |
|---|---|
| **S3** ☁️ | Cheapest, most common for AWS-only setups. Native locking since Terraform 1.11 means no DynamoDB table is needed anymore. |
| **HCP Terraform (Terraform Cloud)** 🌐 | Fully managed — built-in locking, a run history UI, and team access controls out of the box. Free tier has usage limits. |
| **Azure Storage** 🔷 | Azure's equivalent of the S3 backend — blob storage with native state locking support. |
| **GCS** 🟡 | Google Cloud's equivalent, similar feature set to S3. |
- Enable **S3 bucket versioning** and recover a previous state version.
- Try the **`moved`** block to refactor resource addresses without destroy/recreate ([`examples/moved.tf`](https://github.com/LondheShubham153/terraform-for-devops/blob/main/examples/moved.tf)).
- Use a **`removed`** block to stop managing a resource *without deleting it* ([`examples/removed.tf`](https://github.com/LondheShubham153/terraform-for-devops/blob/main/examples/removed.tf)).
- Add a **`check`** block for a continuous health assertion ([`examples/check.tf`](https://github.com/LondheShubham153/terraform-for-devops/blob/main/examples/check.tf)).

---

### Happy Terraforming! 🌍💻
