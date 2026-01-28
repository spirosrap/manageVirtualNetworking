# Rubric 1 — Existing Azure Environment Assessment

## Rubric text (from project overview)

- **Criteria**: Analyze an existing Azure network, identify gaps in the current connectivity, security, and scalability of VNets, subnets, and VMs.
- **Submission requirements**: Submit a written assessment (PDF or DOC) listing all discovered VNets, subnets, VMs, and their features, as well as identified gaps/issues.

## Deliverable: Written assessment outline (copy/paste into a DOC/PDF)

### 1) Scope & context

- **Subscription**: `1180b2af-f0d7-4bb6-bba0-36c8c9796fc9`
- **Resource group assessed**: `udacity-project-starter`
- **Region(s) in use (observed)**: `westeurope`
- **Date/time of inventory**: 2026-01-28 (UTC, approximate)

> Note: This rubric section captures the **initial discovered environment**. Later project steps
> intentionally remediate some gaps (e.g., deploy Bastion, remove public SSH exposure, add VNets).

### 2) Inventory — VNets

| VNet | Region | Address space(s) | Subnets (count) | Peerings | DNS setting | Notes |
|---|---|---|---:|---|---|---|
| `udatech-starter-vnet` | `westeurope` | `10.0.0.0/16` | 1 | None observed | Default (Azure-provided) | `DDoSProtection=False` (observed); starter flat network |

### 3) Inventory — Subnets

| VNet/Subnet | CIDR | NSG attached | Route table (UDR) | Delegations / service endpoints | Private endpoints present | Notes |
|---|---|---|---|---|---|---|
| `udatech-starter-vnet/default` | `10.0.0.0/24` | `udatech-starter-nsg` | None observed | None observed | None observed | Private endpoint network policies: **Disabled** (observed); Private link service network policies: **Enabled** (observed) |

### 4) Inventory — VMs

| VM name | RG | VNet/Subnet | Private IP | Public IP (Y/N) | OS | NIC NSG? | Backup/Availability | Notes |
|---|---|---|---|---:|---|---|---|---|
| `udatech-starter-vm` | `udacity-project-starter` | `udatech-starter-vnet/default` | Not captured in CLI output | Y | Linux (Ubuntu, from template) | Subnet NSG applies | Not captured | VM running; public IP `20.71.24.10`; FQDN `udatech-starter-vm-arpujukqkbbca.westeurope.cloudapp.azure.com` |

### 5) Findings — gaps/issues

Record issues against the three rubric lenses.

#### Connectivity

- **Gap: Flat, single-VNet design (no departmental segmentation yet)**
  - **Evidence**: Only one VNet (`udatech-starter-vnet`) and one subnet (`default`) discovered.
  - **Impact**: No network boundary between HR/Sales/DevOps; increases blast radius and complicates policy enforcement.
  - **Recommendation**: Implement hub/spoke or multi-VNet segmentation with separate VNets/subnets per department and explicit routing/NSG policy.
  - **Priority**: High

#### Security

- **Gap: Publicly reachable VM management path (public IP + inbound SSH from Internet)**
  - **Evidence**:
    - Public IP `20.71.24.10` attached (via `udatech-starter-pip`)
    - NSG rule `Allow-SSH-Internet` allows inbound TCP/22 from `Internet` (priority 1000)
  - **Impact**: Exposes VM to internet attack surface (brute force, credential stuffing, scanning).
  - **Recommendation**: Remove public management exposure by deploying **Azure Bastion**, removing/limiting SSH-from-Internet, and using least-privilege NSG rules (or JIT access).
  - **Priority**: High

### 5b) Remediation notes (implemented later in project)

- Azure Bastion deployed: `bastion-udatech` (hub VNet)
- Public SSH exposure removed:
  - NSG rule `Allow-SSH-Internet` deleted from `udatech-starter-nsg`
  - Public IP detached from `udatech-starter-vm` NIC; VM now uses private IP only (observed: `10.0.0.4`)

#### Scalability / operations

- **Gap: Limited network operational visibility captured**
  - **Evidence**: No Network Watcher / flow logs / diagnostic settings captured in current inventory outputs.
  - **Impact**: Harder to troubleshoot connectivity and validate security posture.
  - **Recommendation**: Enable Network Watcher in-region and configure NSG flow logs + Log Analytics for visibility (later project phase).
  - **Priority**: Medium

### 6) “Overprovisioned / missing components” quick checks

- Public IPs on workloads (should be avoided; prefer Bastion/LB): **Present** (`udatech-starter-vm` → `20.71.24.10`)
- Flat networks (missing segmentation): **Present** (single VNet/subnet)
- Missing NSGs or overly permissive NSG rules: **Risk present** (SSH from `Internet` allowed)
- Missing private endpoints / relies on public PaaS endpoints:
- No diagnostics (Network Watcher, NSG flow logs, etc.):
- Address space too small / no growth plan:

### 7) Screenshot checklist (for your own tracking)

Capture only what the course allows, but typically helpful:

- VNet list + details (address spaces)
- Subnet list for each VNet (CIDRs, NSGs, route tables)
- NSG rules (inbound/outbound) and effective rules (where applicable)
- VM list + NIC/networking blade (private/public IPs)

