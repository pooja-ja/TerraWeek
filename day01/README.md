# 🌱 TerraWeek Day 1 — Introduction to IaC & Terraform Basics

**Date:** Monday, 13th July 2026

Welcome to **Day 1** of the TerraWeek Challenge!
---

## 🎯 Learning Goals

By the end of today you should be able to:
- Explain what **Infrastructure as Code (IaC)** is and why it matters.
- Describe what **Terraform** is and how it fits into the DevOps workflow.
- Install Terraform and verify it works.
- Understand the **core Terraform workflow** and key terminology.
- Provision your **first resource** — with zero cloud cost.

---

## 📝 Tasks

### Task 1: Understand IaC & Terraform
- What is **Infrastructure as Code**, and what problems does it solve compared to clicking around a cloud console?
  Creating a cloud infrastructure via cloud console might work for personal project but its difficult when the project is    big and need to provosion various resourses via console. So IaC tool like Terraform is a must thing while working with     projects. Using Terraform its easier to provision resources and the code can be made resuable by creating modules. 
- What is **Terraform**, and why is it so popular? (Hint: declarative, provider-agnostic, huge ecosystem.)
  Terraform is a **Infrastructure as Code** tool. Its popular because with the help of this tool we can provision            resources in any cloud or on premises. Its declarative approach makes it easy to use.
- **Terraform vs alternatives** — write one line each on how Terraform compares to **OpenTofu**, **Pulumi**, **CloudFormation**, and **Ansible**.
  | Tool | Comparison |
|-------|------------|
| Terraform | IaC tool using HCL and used for multicloud |
| OpenTofu | Open-source fork of a Terraform |
| Pulimi | Works on programming languages like Python, Go |
| Clodformation | Works only for AWS cloud |
| Ansible | Main purpose is for configuration management |


### Task 2: Install Terraform (latest version)
- Install **Terraform ≥ 1.15** using the [official install guide](https://developer.hashicorp.com/terraform/install).
- Verify your install and **paste the output** in your notes:

```bash
terraform version
terraform -help
<img width="1347" height="720" alt="image" src="https://github.com/user-attachments/assets/82a5ba3c-ece4-44b8-b2ea-08577c621c9f" />

```
- Install the **HashiCorp Terraform** extension in VS Code for syntax highlighting and autocomplete.
  <img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/47cf67e5-df92-4abf-9c16-b04435e17c84" />


### Task 3: Learn 6 Crucial Terraform Terminologies
Explain each of these **in your own words** with a one-line example:
1. **Provider** — a plugin that lets Terraform talk to a platform (AWS, Azure, Docker…).
   -Its a plugin which helps terraform to interact with platforms.
   <img width="137" height="48" alt="image" src="https://github.com/user-attachments/assets/0b7a82e0-73c0-457b-abb0-12080772acf5" />
2. **Resource** — a piece of infrastructure you want to create (an EC2 instance, an S3 bucket…).
   Resource defines the kind of infrastructure object.
   <img width="206" height="67" alt="image" src="https://github.com/user-attachments/assets/196d78a4-26db-43b5-8c18-9ad8810daa64" />
3. **State** — Terraform's record of what it manages (the `terraform.tfstate` file).
   This file stores information about your managed infrastructure and configuration. 
4. **Plan** — a preview of the changes Terraform will make.
   It allows to see the changes Terraform will make to your infrastructure before applying them.
   ```bash
   terraform plan
   ```
5. **HCL** — HashiCorp Configuration Language, the syntax you write Terraform in.
   Its a language used to write terraform configuration.
9. **Module** — a reusable, packaged group of Terraform configuration.
   Its a collection of standard configuration files.
   <img width="236" height="65" alt="image" src="https://github.com/user-attachments/assets/575c8a05-f6d5-421b-9c02-8422ced5c97a" />

### Task 4: Your First Terraform Config (no cloud account needed!)
Use the **starter code in [`./example`](./example)** — it uses the `local` and `random` providers, so it costs **nothing** and needs **no credentials**.

Run the **core Terraform workflow** and capture the output of each step:
```bash
cd example
terraform init      # download providers, initialize the working directory
terraform fmt       # format your code
terraform validate  # check for syntax errors
terraform plan      # preview what will be created
terraform apply     # create the resources (type: yes)
cat greeting.txt    # see the file Terraform generated
terraform destroy   # clean up (type: yes)
```

---
<img width="1313" height="720" alt="image" src="https://github.com/user-attachments/assets/fd934c31-4a9f-4dd8-900a-8914f7163ff9" />
<img width="1330" height="720" alt="image" src="https://github.com/user-attachments/assets/307676d5-6d08-4fd3-aade-a4d4107c5e8c" />
<img width="1540" height="720" alt="image" src="https://github.com/user-attachments/assets/342578da-6087-4dad-b921-b5c45e2e146f" />


## 🔁 The Core Terraform Workflow

```
  Write  ──▶  Init  ──▶  Plan  ──▶  Apply  ──▶  Destroy
  (.tf)     (init)     (preview)   (create)    (clean up)
```

---
## Understanding
During initialization terraform generated .terraform.lock.hcl which contains provider version, provider checksum, Dependency information. This ensures that everyone in team should use same provider version.

### Happy Terraforming! 🌍💻
