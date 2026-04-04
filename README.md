Watch Me Do The Lab Here 
https://www.loom.com/share/6cce304ef6934a0a8f58d73d218c311e

#  Infrastructure as Code with Terraform

**Author:** Dylan Bryson  

## 📌 Overview

In this lab, I deployed Azure infrastructure using **Terraform**, an Infrastructure as Code (IaC) tool. Instead of clicking through the Azure portal, I wrote a configuration file describing the resources I wanted. Terraform created, updated, and destroyed them automatically.

By the end, I had deployed a **Resource Group, Virtual Network, Subnet, and Network Security Group** entirely from a text file.

---

## 🚀 Technologies Used

- Microsoft Azure  
- Terraform  
- Azure Resource Group  
- Azure Virtual Network  
- Azure Subnet  
- Azure Network Security Group (NSG)  

---

## 🏗️ Architecture
User → Azure Resources (Terraform-managed)
├─ Resource Group (rg-lab04-tf-dylan)
├─ Virtual Network (vnet-terraform)
├─ Subnet (snet-backend)
└─ Network Security Group (nsg-web)

## ⚙️ Steps Performed

### 1. Open Cloud Shell
- Logged into the Azure portal  
- Opened **Cloud Shell** (Bash)  
- Created project folder:
```bash
mkdir terraform-lab
cd terraform-lab

2. Write Terraform Configuration
Created main.tf in Cloud Shell editor
Defined:
Resource Group
Virtual Network
Subnet
Network Security Group
Updated resource group name with my first name: dylan

3. Deploy with Terraform
Initialize: terraform init
Preview plan: terraform plan
Apply changes: terraform apply

Typed yes to confirm
Resources were created successfully

4. Verify in Azure Portal
Resource Group: rg-lab04-tf-dylan
Virtual Network: vnet-terraform
Subnet: snet-backend
Network Security Group: nsg-web

5. Destroy Resources
To clean up: terraform destroy
Typed yes to confirm
All resources were removed successfully
