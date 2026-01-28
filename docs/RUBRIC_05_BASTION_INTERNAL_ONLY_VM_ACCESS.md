# Rubric 5 — Azure Bastion (Internal-only VM access)

## Rubric text (from project overview)

- **Criteria**: Configure Azure Bastion for secure, internal-only VM access.
- **Submission requirements**: Screenshots showing Azure Bastion deployment; confirmation that direct public IP access for VMs is disabled/enforced.

## What is implemented

- **Azure Bastion**: `bastion-udatech` deployed in the hub VNet `udatech-starter-vnet` (West Europe).
- **Hub subnet requirement**: `AzureBastionSubnet` exists in hub with CIDR `10.0.1.0/26`.
- **Internal-only admin access**:
  - The VM `udatech-starter-vm` is reachable via Bastion SSH.
  - The VM has **no public IP** attached to its NIC (private IP observed: `10.0.0.4`).

## Evidence to include (screenshots/tables)

### Bastion deployment evidence

- Bastion resource overview (Portal) **or** CLI table showing Bastion provisioning state = `Succeeded`.

### Internal-only VM access evidence

- VM networking blade showing **no public IP** (Portal), and/or CLI output where VM **PublicIps** is blank:

```bash
RG=udacity-project-starter
az vm list -g $RG -d -o table
```

### Bastion connectivity evidence

- Screenshot of a successful Bastion SSH session to `udatech-starter-vm` (prompt like `azureadmin@udatech-starter-vm:~$`).

## Where screenshots live in this repo

- `docs/screenshots/rubric-05-bastion/`

