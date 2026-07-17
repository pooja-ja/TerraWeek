# 🚀 TerraWeek Day 6 — Advanced Terraform + Capstone Project

**Date:** Friday, 17th July 2026
---

## 🎯 Learning Goals

- Manage multiple environments with **workspaces**.
- Automatically **format, validate, and test** Terraform (`fmt`, `validate`, `test`).
- **Scan for security issues** before you apply.
- Automate Terraform in **CI/CD** (GitHub Actions).
- Know the **production best practices** — and prove them in a capstone.
---

## 📝 Tasks

### Task 1: Workspaces & Environments
- Learn what **workspaces** are and how they isolate state per environment.
  Both approaches are shown side by side on purpose: `var.environment` drives the values that
the `tftest.hcl` file needs to override for testing, while `terraform.workspace` is used
directly to prove I understand the built-in mechanism the task actually asked for.

Ran through the full workspace lifecycle:
```bash
terraform workspace list
terraform workspace new staging
terraform workspace select staging
terraform workspace show
```
<img width="1600" height="720" alt="image" src="https://github.com/user-attachments/assets/6c25d015-7057-40f0-9afa-767237e67df8" />

- Use `terraform.workspace` in your config (e.g. size things differently per env):
```hcl
locals {
  instance_type = terraform.workspace == "prod" ? "t3.medium" : "t3.micro"
}
```
Applying in `prod` picks up the bigger instance size automatically:

```
current_workspace       = "prod"
instance_type            = "t3.micro"
workspace_instance_type  = "t3.medium"
```
<img width="1428" height="720" alt="image" src="https://github.com/user-attachments/assets/a9d0b0c2-013a-4488-aac7-0d019f9d5d35" />

...while `staging` stays on the smaller size:

```
current_workspace       = "staging"
instance_type            = "t3.micro"
workspace_instance_type  = "t3.micro"
```
<img width="1516" height="720" alt="image" src="https://github.com/user-attachments/assets/3b757c95-eb86-4471-b937-bd88caff178d" />

⚠️ Cleanup note: workspaces here were deleted with `terraform workspace delete -force`, which
is safe **only** because these workspaces held nothing but a `random_pet` resource — no real
cloud infra. If real infrastructure existed in a workspace, the correct order is always
`terraform destroy` first, then delete the workspace. Never `-force` delete a workspace that
still has real resources tracked in its state.
- Discuss the **trade-offs**: workspaces vs separate directories/backends per environment.
<img width="1350" height="720" alt="image" src="https://github.com/user-attachments/assets/33440262-6941-4478-98a9-1ccc07eee17e" />

### Task 2: Quality Gates — `fmt`, `validate`, `test`
- Format and validate everything:
```bash
terraform fmt -recursive
terraform validate
```
- Write a **native test** with the **`terraform test`** framework (Terraform 1.6+). See [`./example/tests/basic.tftest.hcl`](./example/tests/basic.tftest.hcl):
```bash
cd example
terraform init
terraform test
```
Explain the difference between a `plan`-based `command` and an `apply`-based one in a test.
<img width="1181" height="720" alt="image" src="https://github.com/user-attachments/assets/90d83684-625b-409d-bf5b-3e80edd383c4" />
This is a genuinely useful pattern — the test suite runs entirely locally with no AWS
credentials and no real resources created, since it's exercising `random_pet` and local logic,
not cloud infra. That's exactly what makes it CI-friendly.


### Task 3: Security & Cost Scanning
Run a static analysis tool against your Day 3 or Day 5 code and fix what it flags:
- **[Trivy](https://github.com/aquasecurity/trivy)**: `trivy config .`
- or **[Checkov](https://www.checkov.io/)**: `checkov -d .`
- or **[tfsec](https://github.com/aquasecurity/tfsec)** (now part of Trivy).
- **Bonus:** estimate cloud cost of a plan with **[Infracost](https://www.infracost.io/)**.

### Task 4: CI/CD with GitHub Actions
- Use the starter workflow at [`./example/.github-workflow-example.yml`](./example/.github-workflow-example.yml).
- Copy it to `.github/workflows/terraform.yml` in your repo.
- It runs `fmt -check`, `init`, `validate`, and `plan` on every PR. Explain each step.

### Task 5: Best Practices Checklist
Document how your capstone honors these:
- ✅ Remote state with locking (Day 4) — **never commit `.tfstate`**.
- ✅ **Pin** Terraform + provider + module versions.
- ✅ Reusable **modules** (Day 5), consistent naming & tagging.
- ✅ **No hard-coded secrets** — use variables / env / a secrets manager.
- ✅ `fmt` + `validate` + `test` + a security scan in CI.
- ✅ A clear **`README.md`** and a working `terraform destroy`.

---

**Requirements:**
1. Use at least **one custom module** and **one registry module**.
2. Use **remote state with native S3 locking**.
3. Drive everything with **variables** + sensible **outputs**.
4. Pass **`fmt`**, **`validate`**, a **security scan**, and at least one **`terraform test`**.
5. Wire up the **GitHub Actions** workflow.
6. Write a **`README.md`** with an architecture diagram and run instructions.
7. **`terraform destroy`** cleanly when done.
---

### 🎉 Congratulations on completing the TWS TerraWeek Challenge 2026! Happy Terraforming! 🌍💻
