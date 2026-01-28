# Rubric 5 screenshots — Azure Bastion + internal-only VM access

Put Rubric 5 screenshots in this folder.

## Suggested set

1. **Bastion exists / succeeded**
   - Bastion resource overview in Portal, or CLI table screenshot.
2. **AzureBastionSubnet exists**
   - Hub VNet subnets list showing `AzureBastionSubnet (10.0.1.0/26)`.
3. **VM has no public IP**
   - VM networking blade showing Public IP = none, or CLI `az vm list -d -o table` showing blank PublicIps.
4. **Bastion session works**
   - Screenshot of Bastion SSH terminal (`azureadmin@udatech-starter-vm:~$`).
