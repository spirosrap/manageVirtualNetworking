# Rubric 4 — NSGs, ASGs, and Private Endpoints (Evidence)

## Rubric text (from project overview)

- **Criteria**: Implement NSGs, ASGs, and private endpoints to protect and segment resources.
- **Submission requirements**: Screenshots/tables demonstrating NSG, ASG, and private endpoint configuration; a summary of the protection goals.

## Summary of protection goals

- **Department isolation**: HR, Sales, DevOps are separated into distinct VNets/subnets with a default “deny inter-department” posture.
- **Reduce attack surface**: No public SSH/RDP exposure for administrative access; use **Azure Bastion**.
- **Private PaaS access**: Access PaaS services via **private endpoints** and **Private DNS** rather than public endpoints.
- **Operational simplicity**: Use **ASGs** to group workloads by role for scalable policy (even if not fully applied to NSGs yet).

## Implemented controls (what exists in Azure)

### Network Security Groups (NSGs)

- **Department NSGs** (attached to department subnets):
  - `nsg-hr`
  - `nsg-sales`
  - `nsg-devops`
- **Policy**:
  - Explicit deny rules between department address spaces \(10.10.0.0/16\), \(10.20.0.0/16\), \(10.30.0.0/16\) (inbound + outbound).

### Application Security Groups (ASGs)

- `asg-hr-app`, `asg-hr-data`
- `asg-sales-app`, `asg-sales-data`
- `asg-devops-tools`, `asg-devops-agents`

### Private endpoints

- Storage account **`udatechsa4c79f445`** is accessed privately via:
  - Private endpoint in DevOps PE subnet: `vnet-devops/snet-devops-pe (10.30.3.0/24)`
  - Observed private endpoint IP: **`10.30.3.4`**
  - Private DNS zone: `privatelink.blob.core.windows.net`
  - Resolution validated from `udatech-starter-vm`: `*.blob.core.windows.net` → CNAME to `privatelink...` → A record `10.30.3.4`

## Evidence to include (screenshots and/or CLI tables)

Run these in **Cloud Shell** and screenshot the outputs (or paste into your submission doc):

```bash
RG=udacity-project-starter

# NSGs + key rules
az network nsg list -g $RG -o table
az network nsg rule list -g $RG --nsg-name nsg-hr -o table
az network nsg rule list -g $RG --nsg-name nsg-sales -o table
az network nsg rule list -g $RG --nsg-name nsg-devops -o table

# ASGs
az network asg list -g $RG -o table

# Private endpoint + DNS records
az network private-endpoint list -g $RG -o table
az network private-dns zone list -g $RG -o table
az network private-dns record-set a list -g $RG -z privatelink.blob.core.windows.net -o table
```

VM-side validation screenshot (via Bastion SSH):

```bash
nslookup udatechsa4c79f445.blob.core.windows.net
curl -I https://udatechsa4c79f445.blob.core.windows.net/
```

## Where to store screenshots in this repo

Use: `docs/screenshots/rubric-04-security-controls/`

