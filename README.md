# terraform_proxmox

A beginner-friendly Terraform starter for provisioning **Proxmox VMs** in a repeatable way.

---

## 1) What Terraform is (and is not)

Terraform is an **Infrastructure as Code (IaC)** tool:
- You describe infrastructure in `.tf` files (desired state).
- Terraform compares desired state vs real state.
- Terraform creates/updates/destroys resources to match it.

Terraform is best for:
- VM lifecycle (create/update/delete)
- Networks, disks, cloud-init parameters
- Reproducible environments (dev/staging/prod)

Terraform is **not** a full configuration manager for in-guest software setup. For installing packages and hardening inside the VM, pair it with:
- **Cloud-init** (first-boot bootstrap)
- **Ansible** (post-provision config)
- Or image baking (Packer) for golden images

---

## 2) Big concepts to understand first

Before you start writing many files, get comfortable with these:

1. **Providers**
   - Plugins Terraform uses to talk to APIs (for you: Proxmox provider).

2. **State (`terraform.tfstate`)**
   - Terraform’s source of truth for managed resources.
   - Treat state like sensitive data; it can include IDs and metadata.

3. **Plan → Apply workflow**
   - `terraform plan` shows intended changes.
   - `terraform apply` executes them.

4. **Variables and outputs**
   - Variables make templates reusable.
   - Outputs expose useful values like VM IP or IDs.

5. **Modules**
   - Reusable units (e.g., one module for Linux VM + networking conventions).

6. **Idempotency**
   - Running Terraform repeatedly should converge to the same result if config didn’t change.

7. **Drift**
   - Manual edits in Proxmox can drift from Terraform config.
   - Detect with `terraform plan`.

---

## 3) Recommended learning path for your objective

Your objective: automated VM creation with preinstalled services, hardening, network, and storage.

Use this phased approach:

### Phase A — Provision VM reliably
- Create VM from a Proxmox template.
- Attach CPU, RAM, disk, network.
- Pass cloud-init user, SSH key, static IP.

### Phase B — Bootstrap service basics
- Use cloud-init to install baseline packages and users.
- Keep scripts minimal and deterministic.

### Phase C — Hardening + app config
- Use Ansible after VM is reachable.
- Enforce SSH policy, firewall rules, packages, CIS-lite controls.

### Phase D — Reuse at scale
- Wrap VM pattern in a Terraform module.
- Parameterize hostname, VLAN, storage class, disk size, role profile.

---

## 4) Prerequisites checklist

- Proxmox VE reachable from your machine/runner.
- API token/user with minimal required permissions.
- One prepared VM template (cloud-init ready).
- Terraform installed (`>= 1.6` recommended).
- SSH keypair for VM access.
- Git repository for versioning IaC.

---

## 5) Suggested repository layout

```text
terraform_proxmox/
├─ main.tf
├─ providers.tf
├─ variables.tf
├─ outputs.tf
├─ terraform.tfvars.example
├─ modules/
│  └─ vm/
│     ├─ main.tf
│     ├─ variables.tf
│     └─ outputs.tf
└─ ansible/
   ├─ inventory.tpl
   └─ playbook.yml
```

---

## 6) Minimal first-run workflow

1. Initialize providers:
   ```bash
   terraform init
   ```

2. Format and validate config:
   ```bash
   terraform fmt -recursive
   terraform validate
   ```

3. Preview changes:
   ```bash
   terraform plan -out=tfplan
   ```

4. Apply safely:
   ```bash
   terraform apply tfplan
   ```

5. Destroy (for test environments):
   ```bash
   terraform destroy
   ```

---

## 7) Security and team best practices (important)

- Never commit secrets or real token values.
- Use environment variables or secret manager for API credentials.
- Add `.gitignore` for:
  - `.terraform/`
  - `*.tfstate*`
  - `*.tfvars` (except example)
- Use remote backend for team work (S3-compatible, Terraform Cloud, etc.).
- Lock state to avoid concurrent apply conflicts.

---

## 8) What to automate where

- **Terraform:** VM, NIC, disk, metadata, network attachments.
- **Cloud-init:** first-boot user setup, SSH keys, light bootstrap.
- **Ansible:** package installation, service config, hardening policies.
- **Packer (optional):** prebuilt image with baseline software.

This separation keeps your automation clean and maintainable.

---

## 9) Common beginner mistakes to avoid

- Treating Terraform like a shell script runner.
- Hardcoding secrets in `.tf` files.
- Editing infrastructure manually in Proxmox after apply.
- Skipping `plan` review before `apply`.
- Building one giant root config instead of modules.

---

## 10) A practical “first milestone”

Aim for this first:
- One reusable VM template definition.
- Inputs: hostname, CPU, RAM, disk, IP, VLAN.
- Output: VM ID + IP.
- Cloud-init: user + SSH key.

After that, connect Ansible and add hardening/service roles.

---

## 11) If you want, next step

If you want, next I can generate a complete **starter Terraform skeleton for Proxmox** (`providers.tf`, `variables.tf`, `main.tf`, module structure, and safe `.gitignore`) so you can run `terraform init/plan` immediately.
