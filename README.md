# Infrastructure as Code with Terraform

**Author:** Dylan Bryson    
**Video Walkthrough:** [▶ Watch on Loom](https://www.loom.com/share/6cce304ef6934a0a8f58d73d218c311e)

---

## Table of Contents

- [Overview](#overview)
- [How Terraform Works](#how-terraform-works)
- [Architecture](#architecture)
- [Technologies Used](#technologies-used)
- [Phase 1 — Open Cloud Shell](#phase-1--open-cloud-shell)
- [Phase 2 — Write the Configuration](#phase-2--write-the-configuration)
- [Phase 3 — Deploy](#phase-3--deploy)
- [Phase 4 — Verify in the Azure Portal](#phase-4--verify-in-the-azure-portal)
- [Phase 5 — Add a Resource to a Live Environment](#phase-5--add-a-resource-to-a-live-environment)
- [Phase 6 — Destroy](#phase-6--destroy)
- [Troubleshooting](#troubleshooting)
- [Summary](#summary)
- [Next Steps](#next-steps)

---

## Overview

In this lab, Azure infrastructure is deployed using **Terraform** — an Infrastructure as Code (IaC) tool. Instead of clicking through the Azure portal, a plain text configuration file describes the resources to build, and Terraform creates, updates, and destroys them automatically.

The lab also demonstrates one of Terraform's most important behaviors: when a new resource is added to a live environment, Terraform updates **only what changed** — leaving existing infrastructure untouched.

By the end, the following are deployed and torn down entirely from code:

- Resource Group
- Virtual Network
- Subnet
- Network Security Group

---

## How Terraform Works

Before running any commands, these three concepts are worth understanding — they make every step below make sense.

**Terraform describes WHAT, not HOW.** Rather than clicking through steps in the portal, you write a file that says "I want a resource group called X in East US" and Terraform determines the steps to make that happen. This is called declarative infrastructure.

**The state file is Terraform's memory.** After Terraform creates resources, it writes a `terraform.tfstate` file to your project folder. This is its record of everything it has created — resource IDs, configuration values, and relationships. Every `plan` and `apply` reads this file to know what already exists. Never delete it.

**The four commands — always in this order:**

| Command | What It Does | When to Run |
|---------|-------------|-------------|
| `terraform init` | Downloads the Azure provider plugin | Once per project — the very first step |
| `terraform plan` | Previews what will change without doing anything | Before every apply — never skip this |
| `terraform apply` | Executes the plan and creates/updates resources | After reviewing and approving the plan |
| `terraform destroy` | Removes every resource tracked in the state file | When tearing down the environment |

---

## Architecture

```
Azure (Terraform-managed)
│
├── Resource Group — rg-lab04-tf-dylan
│   ├── Virtual Network — vnet-terraform (10.0.0.0/16)
│   │   └── Subnet — snet-backend (10.0.1.0/24)
│   └── Network Security Group — nsg-web
```

> No portal clicks. This entire environment is defined in a single `main.tf` file and reproducible with four commands.

---

## Technologies Used

| Technology | Purpose |
|------------|---------|
| Terraform | Infrastructure as Code provisioning |
| Azure Cloud Shell | Browser-based terminal — no local install required |
| Azure Resource Group | Logical container for all resources |
| Azure Virtual Network | Network isolation layer |
| Azure Subnet | Network segmentation within the VNet |
| Azure NSG | Firewall rules for inbound/outbound traffic control |

---

## Phase 1 — Open Cloud Shell

1. Log in to the [Azure Portal](https://portal.azure.com)
2. Click the **>_** icon in the top-right toolbar to open Cloud Shell
3. Select **Bash** — the commands in this lab will not work in PowerShell mode
4. Create the project folder:

```bash
mkdir terraform-lab
cd terraform-lab
```

Your prompt should now end with `~/terraform-lab$`

> **No installation needed.** Terraform is pre-installed in every Azure Cloud Shell session. Nothing needs to be installed on your local machine.

---

## Phase 2 — Write the Configuration

Open the built-in Cloud Shell editor:

```bash
code main.tf
```

Paste the following configuration. **Before saving, change `dylan` on line 6 to your own first name** — lowercase, no spaces.

```hcl
# ── 1. Tell Terraform which provider to use ──────────────────────────────
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

# features {} is required by azurerm even when empty.
provider "azurerm" {
  features {}
}

# ── 2. Resource Group ─────────────────────────────────────────────────────
# CHANGE "dylan" to YOUR name below.
resource "azurerm_resource_group" "rg" {
  name     = "rg-lab04-tf-dylan"   # <-- CHANGE THIS TO YOUR NAME
  location = "East US"
}

# ── 3. Virtual Network ────────────────────────────────────────────────────
# References the resource group above — Terraform creates the RG first automatically.
resource "azurerm_virtual_network" "vnet" {
  name                = "vnet-terraform"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  address_space       = ["10.0.0.0/16"]
}

# ── 4. Subnet ─────────────────────────────────────────────────────────────
resource "azurerm_subnet" "subnet" {
  name                 = "snet-backend"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}
```

Save with `Ctrl + S`, then close the editor with `Ctrl + Q`.

Confirm the file saved correctly:

```bash
cat main.tf
```

---

## Phase 3 — Deploy

Run the following three commands in order. Never skip `plan`.

**Initialize — download the Azure provider:**

```bash
terraform init
```

Expected output: `Terraform has been successfully initialized!`

**Plan — preview what will be created:**

```bash
terraform plan
```

Expected output: `Plan: 3 to add, 0 to change, 0 to destroy.`

The three resources are the Resource Group, Virtual Network, and Subnet. If the number is different, check `main.tf` for a missing or extra resource block.

**Apply — build the infrastructure:**

```bash
terraform apply
```

Type `yes` when prompted — the full word is required. Terraform will not accept `y` alone.

Expected output: `Apply complete! Resources: 3 added, 0 changed, 0 destroyed.`

> Terraform resolves dependency order automatically — the Resource Group is created first, then the VNet (which depends on the RG), then the Subnet (which depends on the VNet).

---

## Phase 4 — Verify in the Azure Portal

1. In the Azure Portal search bar, navigate to **Resource Groups**
2. Open `rg-lab04-tf-dylan`
3. Confirm the following resources are present:

| Resource | Name |
|----------|------|
| Resource Group | `rg-lab04-tf-dylan` |
| Virtual Network | `vnet-terraform` |
| Subnet | `snet-backend` (10.0.1.0/24) |

---

## Phase 5 — Add a Resource to a Live Environment

This phase demonstrates one of Terraform's most important behaviors: it only changes what is different. Adding a new resource to an already-running environment will not touch anything that already exists.

Reopen the editor:

```bash
code main.tf
```

Scroll to the bottom of the file and add the following block **after** the closing `}` of the subnet resource. Do not modify any existing code.

```hcl
# ── 5. Network Security Group ────────────────────────────────────────────
resource "azurerm_network_security_group" "nsg" {
  name                = "nsg-web"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
}
```

Save (`Ctrl + S`) and close (`Ctrl + Q`), then plan and apply:

```bash
terraform plan
```

Expected output: `Plan: 1 to add, 0 to change, 0 to destroy.`

This is the key insight — Terraform compares `main.tf` against the state file, sees the Resource Group, VNet, and Subnet already exist unchanged, and correctly identifies that only the NSG needs to be created.

```bash
terraform apply
```

Type `yes` to confirm.

Expected output: `Apply complete! Resources: 1 added, 0 changed, 0 destroyed.`

Refresh the resource group in the Azure Portal — all four resources should now be visible.

---

## Phase 6 — Destroy

Remove all resources cleanly with a single command:

```bash
terraform destroy
```

Expected plan output: `Plan: 0 to add, 0 to change, 4 to destroy.`

Type `yes` to confirm.

Expected output: `Destroy complete! Resources: 4 destroyed.`

> **Always use `terraform destroy` instead of deleting through the portal.** If a resource group is manually deleted in the portal, the `terraform.tfstate` file still records those resources as existing. The next `terraform plan` will throw errors because Terraform cannot find resources it believes are there. `terraform destroy` keeps the state file and Azure environment perfectly in sync.

Verify in the Azure Portal that `rg-lab04-tf-dylan` no longer appears in the Resource Groups list.

---

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| "Resource Group already exists" | A resource group with this name already exists in your subscription | Change the name in `main.tf` to something unique, or delete the existing RG in the portal first |
| `terraform: command not found` | Cloud Shell session dropped | Close and reopen Cloud Shell from the portal toolbar — Terraform is pre-installed in every new session |
| Syntax error on line X | Missing `}`, `"`, or extra character introduced during editing | Open `code main.tf`, go to the line shown in the error, and check for mismatched brackets or quotes |
| `Plan: 0 to add` | Resources already exist and are recorded in the state file | Run `terraform destroy` first, then apply again |
| `apply` fails partway through | Transient Azure API error or quota limit | Run `terraform apply` again — Terraform skips what already exists and retries only what failed |

---

## Summary

- ✅ Deployed a Resource Group, Virtual Network, Subnet, and NSG entirely from a Terraform configuration file
- ✅ Ran the full `init → plan → apply` workflow inside Azure Cloud Shell — no local installation required
- ✅ Added a Network Security Group to a live environment and confirmed Terraform only changed the new resource
- ✅ Destroyed the entire environment with a single `terraform destroy` command
- ✅ Understood why IaC is the professional standard for managing cloud infrastructure at scale

---

## Next Steps

| Enhancement | Description |
|-------------|-------------|
| **Variables** | Refactor `main.tf` to use `variables.tf` and `terraform.tfvars` for reusable, parameterized configs |
| **Terraform Remote State** | Store `terraform.tfstate` in Azure Blob Storage to enable team collaboration and state locking |
| **NSG Rules** | Add inbound security rules to `nsg-web` to control specific ports and source IP ranges |
| **Deploy a VM** | Extend this configuration to provision a virtual machine inside the subnet |
| **Terraform Modules** | Organize resources into reusable modules for larger, multi-environment deployments |

---

*© Dylan Bryson — Cloud Security Specialist*
