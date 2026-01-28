# Final Architecture Document — UdaTech Azure Virtual Networking

This document consolidates the final architecture and key configurations for the UdaTech Azure virtual networking project.

## Scope

Provide a secure, scalable Azure network for the **HR**, **Sales**, and **DevOps** departments with:

- Department segmentation
- Secure remote access (internal-only VM administration)
- Private PaaS access (private endpoints + private DNS)
- High availability (load balancing)
- Validation via Azure diagnostic tools (Network Watcher)

## Resource group / region

- **Resource group**: `udacity-project-starter`
- **Region**: `westeurope`

## Topology (hub-and-spoke)

- **Hub**: `udatech-starter-vnet (10.0.0.0/16)`
- **Spokes**:
  - HR: `vnet-hr (10.10.0.0/16)`
  - Sales: `vnet-sales (10.20.0.0/16)`
  - DevOps: `vnet-devops (10.30.0.0/16)`

Diagram:

- `docs/network-diagram.png`

## Address plan (VNets/subnets)

### Hub — `udatech-starter-vnet`

- `default`: `10.0.0.0/24`
- `AzureBastionSubnet`: `10.0.1.0/26`

### HR — `vnet-hr`

- `snet-hr-app`: `10.10.1.0/24`
- `snet-hr-data`: `10.10.2.0/24`
- `snet-hr-pe`: `10.10.3.0/24`

### Sales — `vnet-sales`

- `snet-sales-app`: `10.20.1.0/24`
- `snet-sales-data`: `10.20.2.0/24`
- `snet-sales-pe`: `10.20.3.0/24`

### DevOps — `vnet-devops`

- `snet-devops-tools`: `10.30.1.0/24`
- `snet-devops-agents`: `10.30.2.0/24`
- `snet-devops-pe`: `10.30.3.0/24`

Evidence:

- `docs/RUBRIC_03_VNETS_SUBNETS_DEPLOYED.md`
- Screenshots: `docs/screenshots/rubric-03-vnets-subnets/`

## Routing & connectivity

- **VNet peering**:
  - Hub ↔ HR
  - Hub ↔ Sales
  - Hub ↔ DevOps
- **Inter-department traffic** is restricted by subnet-attached NSGs (see Security).

## Security controls

### NSGs (Network Security Groups)

- HR: `nsg-hr` (attached to all HR subnets)
- Sales: `nsg-sales` (attached to all Sales subnets)
- DevOps: `nsg-devops` (attached to all DevOps subnets)

Baseline posture:

- Explicit deny rules between:
  - \(10.10.0.0/16\) (HR)
  - \(10.20.0.0/16\) (Sales)
  - \(10.30.0.0/16\) (DevOps)

Evidence:

- `docs/RUBRIC_04_NSG_ASG_PRIVATE_ENDPOINTS.md`
- Screenshots: `docs/screenshots/rubric-04-security-controls/`

### ASGs (Application Security Groups)

Created to group workloads by role for scalable policy:

- `asg-hr-app`, `asg-hr-data`
- `asg-sales-app`, `asg-sales-data`
- `asg-devops-tools`, `asg-devops-agents`

Evidence:

- `docs/RUBRIC_04_NSG_ASG_PRIVATE_ENDPOINTS.md`

## Secure remote access (Azure Bastion)

- **Bastion**: `bastion-udatech` deployed in hub VNet on `AzureBastionSubnet`
- **Internal-only enforcement**:
  - `udatech-starter-vm` uses private IP `10.0.0.4` and has no public IP attached
  - Bastion SSH session to VM confirmed

Evidence:

- `docs/RUBRIC_05_BASTION_INTERNAL_ONLY_VM_ACCESS.md`
- Screenshots: `docs/screenshots/rubric-05-bastion/`

## DNS & name resolution (Private DNS)

Approach:

- Azure Private DNS for private endpoint name resolution (no custom DNS servers).

Implemented:

- Private DNS zone: `privatelink.blob.core.windows.net`
- Storage account: `udatechsa4c79f445`
- Storage private endpoint IP (observed): `10.30.3.4`
- Cross-VNet resolution verified from hub VM (`udatech-starter-vm`):
  - `udatechsa4c79f445.blob.core.windows.net` resolves to `10.30.3.4`

Evidence:

- `docs/RUBRIC_06_DNS_NAME_RESOLUTION.md`
- Screenshots: `docs/screenshots/rubric-06-dns/`

## Load balancing (High availability)

Implemented:

- **Internal Load Balancer**: `lb-sales-ilb`
- **Frontend IP**: `10.20.1.4`
- **Backend pool**: `be-sales` with two backend VMs:
  - `sales-web-1 (10.20.1.5)`
  - `sales-web-2 (10.20.1.6)`
- **Health probe / rule**: TCP/80

Functional test confirmed requests are distributed across both backends.

Evidence:

- `docs/RUBRIC_07_LOAD_BALANCER_HA.md`
- Screenshots: `docs/screenshots/rubric-07-load-balancer/`

## Validation (Azure diagnostic tools)

Network Watcher validation performed (from Cloud Shell):

- VM → Sales ILB (`10.20.1.4:80`) → **Reachable**
- VM → Storage PE (`10.30.3.4:443`) → **Reachable**
- Next hop:
  - to `10.20.1.4`: **VirtualNetworkPeering**
  - to `10.30.3.4`: **PrivateEndpoint**

Evidence:

- `docs/RUBRIC_08_NETWORK_WATCHER_VALIDATION.md`
- `docs/VALIDATION_TESTS.md`
- Screenshots: `docs/screenshots/rubric-08-network-watcher/`

## Appendix: Rubric evidence index

- Rubric 1: `docs/RUBRIC_01_EXISTING_ENV_ASSESSMENT.md`
- Rubric 2: `docs/RUBRIC_02_SEGMENTED_NETWORK_DESIGN.md` + diagram `docs/network-diagram.png`
- Rubric 3: `docs/RUBRIC_03_VNETS_SUBNETS_DEPLOYED.md`
- Rubric 4: `docs/RUBRIC_04_NSG_ASG_PRIVATE_ENDPOINTS.md`
- Rubric 5: `docs/RUBRIC_05_BASTION_INTERNAL_ONLY_VM_ACCESS.md`
- Rubric 6: `docs/RUBRIC_06_DNS_NAME_RESOLUTION.md`
- Rubric 7: `docs/RUBRIC_07_LOAD_BALANCER_HA.md`
- Rubric 8: `docs/RUBRIC_08_NETWORK_WATCHER_VALIDATION.md`

