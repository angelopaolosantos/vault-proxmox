# Vault in Proxmox

This project provisions a **HashiCorp Vault server** inside a **Proxmox LXC container** using **Terraform** and configures it with **Ansible**.  

It also supports **AWS KMS auto-unseal**, so Vault can start automatically without manual unseal keys.

---

## ⚡ Quick Start

```bash
# 1. Install transcrypt and decrypt files
transcrypt

# 2. Provision LXC container with Terraform
terraform init --backend-config=config.pg.tfbackend
terraform apply

# 3. Install Terraform collection for Ansible
ansible-galaxy collection install cloud.terraform

# 4. Run Ansible playbook to install Vault
export ANSIBLE_HOST_KEY_CHECKING=False
ansible-playbook -i ./ansible/inventory.yaml ./ansible/playbook.yaml -K

```
terraform init --backend-config=config.pg.tfbackend 
terraform plan
terraform apply
```

## Run Ansible to install Vault
### Install Terraform Collection for Ansible
`ansible-galaxy collection install cloud.terraform`

### Print Terraform Inventory
`ansible-inventory -i ./ansible/inventory.yaml --graph --vars`

### Run Ansible Playbook
```
export ANSIBLE_HOST_KEY_CHECKING=False
ansible-playbook -i ./ansible/inventory.yaml ./ansible/playbook.yaml
```

### SSH into container
`ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -i .ssh/my-private-key.pem root@192.168.254.217`

### Download terraform state from backend
`terraform state pull > terraform.tfstate`

### View terraform state
`terraform show -json`

## Initialize
Go to https://vault.deviantlab.duckdns.org/ to initialize vault

enter Initial Root Token to login

## Manual Start and Stop Vault server
### Start Vault Server
`vault server -config config.hcl`

### Stop Vault Server
`pkill -9 vault`

### Run vault as service
https://developer.hashicorp.com/vault/docs/run-as-service

### 2️⃣ Create a KMS Key in AWS

- Use AWS Console or CLI to create a **Customer Managed Key (CMK)**.  
- Note the **KMS Key ARN**, which you’ll use in Vault config.  

---

### 3️⃣ Create IAM Policy for Vault

Example policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt",
                "kms:DescribeKey"
            ],
            "Resource": "arn:aws:kms:us-east-1:123456789012:key/your-kms-key-id"
        }
    ]
}
```

- Replace `your-kms-key-id` with your actual KMS key ID.  
- Attach this policy to the IAM role assigned to the Vault LXC (or instance profile).  

---

### 4️⃣ Configure Vault for Auto-Unseal

Edit `config.hcl`:

```hcl
seal "awskms" {
  region    = "us-east-1"
  kms_key_id = "arn:aws:kms:us-east-1:123456789012:key/your-kms-key-id"
}
```

- Ensure Vault has network access to AWS KMS.  
- Vault will now auto-unseal on startup.  

---
## 🔑 Accessing the Container

```bash
ssh -o UserKnownHostsFile=/dev/null \
    -o StrictHostKeyChecking=no \
    -i .ssh/my-private-key.pem \
    root@192.168.254.217
```

---

## 📂 Working with Terraform State

Download and view the Terraform state stored in the backend:

```bash
terraform state pull > terraform.tfstate
terraform show -json terraform.tfstate
```

---

## 🚀 Managing the Vault Server

### Start Vault Manually
```bash
vault server -config config.hcl
```

### Stop Vault
```bash
pkill -9 vault
# or
systemctl stop vault
```

### Run Vault as a Service
Follow the official documentation:  
👉 [Running Vault as a Service](https://developer.hashicorp.com/vault/docs/run-as-service)

---

## 🧹 Clean Up Resources

Destroy the LXC container and remove all associated resources:

```bash
terraform destroy
```

---

## 🔗 References

- [HashiCorp Vault](https://developer.hashicorp.com/vault)  
- [Proxmox VE](https://www.proxmox.com/en/)  
- [Terraform](https://www.terraform.io/)  
- [Ansible](https://www.ansible.com/)  
- [elasticdog/transcrypt](https://github.com/elasticdog/transcrypt)  
- [AWS KMS Auto-Unseal Docs](https://developer.hashicorp.com/vault/docs/configuration/seal/awskms)  

https://github.com/elasticdog/transcrypt