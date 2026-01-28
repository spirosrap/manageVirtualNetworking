# Proposed Azure Network Topology (UdaTech)

This document proposes a secure, scalable Azure network design with clear segmentation for:

- **HR**
- **Sales**
- **DevOps**

## Design principles

- **Least privilege by default** (deny by default; explicitly allow required flows)
- **Separation of duties** via network segmentation + role-based access
- **Private-by-default** for PaaS access (private endpoints + private DNS)
- **No public admin access** to VMs (Azure Bastion)
- **Scalable topology** (hub/spoke pattern to add spokes or subnets without redesign)

## Assumptions (documented until validated)

- Single primary Azure region for initial deployment (optionally paired region later).
- Each department has its own application tier(s) that should not be broadly reachable by other departments.
- Shared services exist/will exist (DNS, Bastion, logging, shared tooling).

## Topology overview (hub/spoke)

### Hub VNet (shared services)

**Purpose**: Centralize shared connectivity and platform services.

Recommended hub components (phased):

- **Azure Bastion** (for VM administration)
- **Private DNS zone links** (hub-linked zones, linked to spokes)
- Optional future: centralized egress (Azure Firewall/NVA), VPN/ExpressRoute gateway

#### Deployed (current)

- Hub VNet: `udatech-starter-vnet` (`10.0.0.0/16`)
- Bastion: `bastion-udatech` (resource group `udacity-project-starter`, region `westeurope`)
- Hub↔spoke peerings (Connected): `hub-to-hr`, `hub-to-sales`, `hub-to-devops` and corresponding spoke-to-hub peerings
- Internal load balancer (Sales): `lb-sales-ilb` with frontend IP `10.20.1.4` (backend VMs: `sales-web-1`, `sales-web-2`)

### Spoke VNets (workload isolation)

- **HR Spoke VNet**: `vnet-hr` (`10.10.0.0/16`)
- **Sales Spoke VNet**: `vnet-sales` (`10.20.0.0/16`)
- **DevOps Spoke VNet**: `vnet-devops` (`10.30.0.0/16`)

Spokes peer with the hub (VNet peering). Inter-spoke traffic is allowed only if explicitly required and permitted by policy (typically via hub routing in later phases; for early phases, keep inter-spoke minimal).

## Addressing plan (example RFC1918)

Choose a non-overlapping RFC1918 plan sized for growth.

Example:

- **Hub VNet**: `10.0.0.0/16`
  - `default` (starter subnet): `10.0.0.0/24`
  - `AzureBastionSubnet`: `10.0.1.0/26` (required name)
- **HR Spoke VNet**: `10.10.0.0/16`
  - `snet-hr-app`: `10.10.1.0/24`
  - `snet-hr-data`: `10.10.2.0/24`
  - `snet-hr-pe`: `10.10.3.0/24`
- **Sales Spoke VNet**: `10.20.0.0/16`
  - `snet-sales-app`: `10.20.1.0/24`
  - `snet-sales-data`: `10.20.2.0/24`
  - `snet-sales-pe`: `10.20.3.0/24`
- **DevOps Spoke VNet**: `10.30.0.0/16`
  - `snet-devops-tools`: `10.30.1.0/24`
  - `snet-devops-agents`: `10.30.2.0/24`
  - `snet-devops-pe`: `10.30.3.0/24`

Notes:

- Reserve space for additional subnets (shared AKS, integration, etc.).
- Confirm no overlap with on-prem or other clouds before finalizing.

## Segmentation strategy

Segmentation is enforced by:

- **Separate VNets per department (spokes)** for strong isolation boundaries
- **Subnets per tier** (app/data/private endpoints)
- **NSGs on subnets** for traffic enforcement
- **ASGs** to simplify rules (group workloads by role, not IP)
- **Private endpoints** for PaaS; minimize public exposure

## Security controls

### 1) NSG strategy (high level)

Apply NSGs at **subnet** level (primary) and optionally at NIC level (exception cases only).

Baseline approach:

- Keep Azure defaults that allow VNet-internal unless overridden by explicit denies.
- Add explicit allows for required traffic.
- Add explicit denies for cross-department access when not needed.

Recommended rule categories per subnet NSG:

- **Inbound**
  - Allow required intra-subnet / intra-tier traffic
  - Allow from Bastion / admin management plane only where needed
  - Allow from load balancer (if ILB)
  - Deny other spoke traffic by default
- **Outbound**
  - Allow to private endpoints / required Azure services
  - Allow to hub shared services (DNS, logging) as needed
  - Restrict internet egress where feasible (future phase with central egress)

#### Deployed (current)

- Department NSGs (attached at subnet level):
  - HR: `nsg-hr`
  - Sales: `nsg-sales`
  - DevOps: `nsg-devops`
- Baseline policy:
  - Inter-department traffic is explicitly **denied** using address-prefix denies (e.g., HR denies \(10.20.0.0/16\) and \(10.30.0.0/16\) inbound/outbound).

### 2) ASG strategy (workload identity)

Create ASGs per role, for example:

- `asg-hr-app`, `asg-hr-data`
- `asg-sales-app`, `asg-sales-data`
- `asg-devops-tools`, `asg-devops-agents`

Use NSG rules referencing ASGs rather than IP CIDRs to reduce operational overhead.

### 3) Private endpoints (PaaS hardening)

Use private endpoints for services such as:

- Key Vault, Storage, SQL, Container Registry, etc.

Pattern:

- Place private endpoints in each spoke’s `*-PrivateEndpoints` subnet (or in hub if shared).
- Ensure corresponding **Private DNS zones** exist and are linked to all VNets that must resolve names.

#### Deployed (current)

- Storage account (Blob) secured with **private endpoint**:
  - Storage account: `udatechsa4c79f445`
  - Private endpoint IP (observed from VM DNS resolution): `10.30.3.4`
  - Private DNS zone: `privatelink.blob.core.windows.net`
  - DNS resolution validated from `udatech-starter-vm`:
    - `udatechsa4c79f445.blob.core.windows.net` → CNAME to `privatelink...` → A record `10.30.3.4`

### 4) Azure Bastion (secure admin access)

Deploy Bastion in the **hub VNet** in `AzureBastionSubnet` and use it for:

- SSH/RDP to spoke VMs over private IPs
- No public IPs on VMs

### 5) DNS & name resolution

Preferred (Azure-native) approach:

- Use **Azure Private DNS zones** for private endpoints
- Link zones to:
  - Hub VNet
  - Each spoke VNet that needs resolution

If custom DNS is required (e.g., AD DS), document:

- DNS server IPs
- VNet DNS settings for each VNet
- Conditional forwarders for private endpoint zones (if applicable)

## Load balancing & high availability

General guidance:

- Use **Internal Load Balancer (ILB)** for private applications (east-west).
- Use **Public Load Balancer** only when a workload must be internet-facing (and preferably fronted by additional controls in later phases).

Document per application:

- Frontend entry point type: ILB/Public LB
- Backend pool subnets
- Health probe paths/ports

## Diagnostics & validation plan

Use Azure diagnostics to validate connectivity and security:

- **Network Watcher**
  - IP flow verify (NSG rule evaluation)
  - Connection troubleshoot
  - Next hop (route validation)
- **NSG flow logs** (to Log Analytics / storage)
- **Azure Monitor** and alerting for critical components (Bastion, LB health)

## “To be implemented later” checklist (explicitly not done yet)

- Create VNets/subnets and peerings
- Deploy NSGs/ASGs and apply rules
- Deploy private endpoints + private DNS zones/links
- Deploy Azure Bastion in hub
- Deploy Load Balancers for required apps
- Enable diagnostics (Network Watcher, flow logs, Log Analytics integration)

