# Rubric 2 — Segmented Network Design (Multi-Department)

## Rubric text (from project overview)

- **Criteria**: Design a segmented network plan for multiple departments in Azure.
- **Submission requirements**: Submit a network design diagram and written documentation of proposed VNets, subnets, and routing.

## Written design (proposed and mapped to deployed resources)

### Topology

- **Pattern**: Hub-and-spoke
- **Hub**: shared services + secure admin access (Azure Bastion), and (later) shared DNS/private endpoints if required
- **Spokes**: separate VNets per department (HR, Sales, DevOps) for isolation and scalability

### VNets and subnets

#### Hub

- `udatech-starter-vnet` — `10.0.0.0/16`
  - `default` — `10.0.0.0/24` (starter subnet; contains `udatech-starter-vm`)
  - `AzureBastionSubnet` — `10.0.1.0/26` (required name for Bastion)

#### HR spoke

- `vnet-hr` — `10.10.0.0/16`
  - `snet-hr-app` — `10.10.1.0/24`
  - `snet-hr-data` — `10.10.2.0/24`
  - `snet-hr-pe` — `10.10.3.0/24` (private endpoints)

#### Sales spoke

- `vnet-sales` — `10.20.0.0/16`
  - `snet-sales-app` — `10.20.1.0/24` (contains ILB + web backends)
  - `snet-sales-data` — `10.20.2.0/24`
  - `snet-sales-pe` — `10.20.3.0/24` (private endpoints)

#### DevOps spoke

- `vnet-devops` — `10.30.0.0/16`
  - `snet-devops-tools` — `10.30.1.0/24`
  - `snet-devops-agents` — `10.30.2.0/24`
  - `snet-devops-pe` — `10.30.3.0/24` (private endpoints)

### Routing / connectivity strategy

- **VNet peering**:
  - Hub ↔ HR (bidirectional peering)
  - Hub ↔ Sales (bidirectional peering)
  - Hub ↔ DevOps (bidirectional peering)
- **Inter-department traffic**:
  - Default posture is **deny** using subnet-attached NSGs (`nsg-hr`, `nsg-sales`, `nsg-devops`) with explicit deny rules between \(10.10.0.0/16\), \(10.20.0.0/16\), \(10.30.0.0/16\).
  - Only required flows should be explicitly allowed in later steps (using ASGs where possible).
- **Admin access**:
  - Use **Azure Bastion** in the hub (`bastion-udatech`) for SSH/RDP over private IPs; no public admin access to VMs.
- **PaaS access**:
  - Use **private endpoints** per department in `*-pe` subnets, with **Private DNS zones** linked to VNets that need name resolution.

### Evidence / diagram

- **Written topology and current deployed mapping**: `docs/NETWORK_TOPOLOGY.md`
- **Suggested diagram approach** (for submission):
  - Create a hub-and-spoke diagram showing hub `udatech-starter-vnet` peered to `vnet-hr`, `vnet-sales`, `vnet-devops`, including the subnet CIDRs listed above.
  - Include `bastion-udatech` in the hub and show `lb-sales-ilb (10.20.1.4)` in Sales.

