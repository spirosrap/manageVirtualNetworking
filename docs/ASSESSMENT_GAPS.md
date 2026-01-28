# Environment Assessment & Gap Analysis (UdaTech Azure Networking)

This document is used to assess the **current state** (if any) and identify gaps in:

- **Connectivity**
- **Security**
- **Scalability / Operations**

## Assumptions (until validated)

- Greenfield deployment in Azure (no existing VNets), or existing networking is minimal.
- Departments requiring segmentation: **HR**, **Sales**, **DevOps**.
- Workloads will require:
  - Private east-west traffic within Azure
  - Limited north-south (internet) exposure via controlled entry points
  - Secure administrative access (no public IP RDP/SSH)

## 1) Inventory checklist (what exists today)

> For the rubric “existing environment assessment” submission template, see
> `docs/RUBRIC_01_EXISTING_ENV_ASSESSMENT.md`.

### Subscriptions / management groups

- Subscription(s) in scope:
- Tenant / Entra ID:
- Management group structure (if any):
- Policy baseline (if any):

### Network resources

- Existing **VNets** and address spaces:
- Existing **subnets** and route tables (UDRs):
- Existing **VNet peering** relationships:
- Existing **VPN Gateway / ExpressRoute**:
- Existing **Azure Firewall / NVA**:
- Existing **Private DNS zones** and links:
- Existing **private endpoints**:

### Security controls

- Existing **NSGs** and effective rules:
- Existing **ASGs** usage:
- Existing **DDoS Protection** plan (Standard or none):
- Existing **Azure Bastion** (if any):
- Any public IPs on workloads:

### Workloads & dependencies

- HR applications/services (inbound/outbound dependencies):
- Sales applications/services (inbound/outbound dependencies):
- DevOps tooling (CI/CD runners, agents, registries, Key Vault, etc.):
- Shared services (AD DS / DNS, logging, monitoring, jump access):
- External dependencies (SaaS, APIs, on-prem):

### Observability / operations

- Log Analytics workspace(s):
- Network Watcher enabled? Region(s)?
- NSG flow logs enabled? Traffic analytics?
- Any existing runbooks / automation?

## 2) Connectivity requirements (target outcomes)

- **Department isolation**: HR, Sales, DevOps separated by subnets/VNets and policy.
- **Controlled inter-department traffic**:
  - Explicit allow rules only (default deny stance).
- **No direct inbound to VMs from internet**:
  - Admin access via **Azure Bastion**.
- **Private access to PaaS**:
  - Prefer **private endpoints** + private DNS.
- **Name resolution**:
  - Consistent DNS across VNets (Azure Private DNS + links; optional custom DNS if required).
- **High availability**:
  - Load balancing for critical apps (Internal LB for private apps; Public LB only if needed).

## 3) Gap analysis template

Use this table to record findings and required remediations.

| Area | Current state | Gap / risk | Recommendation | Priority |
|---|---|---|---|---|
| Address space planning | TBD | Overlaps / unmanaged growth | Define RFC1918 plan w/ growth | High |
| Segmentation | TBD | Flat network / lateral movement | Hub-spoke + per-dept subnets | High |
| Admin access | TBD | Public RDP/SSH | Azure Bastion + JIT/PIM | High |
| PaaS access | TBD | Public endpoints | Private endpoints + private DNS | High |
| Egress control | TBD | Unrestricted outbound | Central egress policy (future) | Medium |
| DNS | TBD | Split-brain / inconsistent | Private DNS zones + links | Medium |
| Monitoring | TBD | No flow logs / visibility | Network Watcher + NSG flow logs | Medium |

## 4) Acceptance criteria for “design complete”

- Proposed topology documented with:
  - VNets/subnets, address spaces, peering, DNS, and security boundaries
- NSG/ASG strategy defined (default deny + explicit flows)
- Bastion plan defined (where, how accessed, what subnets)
- Private endpoint + private DNS approach documented
- Load balancing approach documented (what uses ILB/Public LB)
- Diagnostics plan documented (what logs, where, and why)

