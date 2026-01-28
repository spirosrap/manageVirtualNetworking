# Rubric 10 — Final Review (Business + Technical Requirements)

## Rubric text (from project overview)

- **Criteria**: Evaluate the deployed Azure network against business and technical requirements.
- **Submission requirements**: A written final review (executive summary or checklist) explaining how the solution meets **connectivity**, **security**, **scalability**, and **business goals**.

## Executive summary

UdaTech’s Azure network was redesigned from a flat, internet-exposed starter environment into a **segmented hub-and-spoke architecture** that supports HR, Sales, and DevOps as separate network domains, improves the security posture with **private-by-default controls**, and validates expected connectivity via **Network Watcher** diagnostics.

Key outcomes:

- **Segmentation**: Each department is isolated in its own VNet and tiered subnets.
- **Security**: VM administration moved to **Azure Bastion** with **no public IP** on the administrative VM; private endpoints are used for PaaS access.
- **Availability**: Sales app tier uses an **Internal Load Balancer** with two backend VMs.
- **Validation**: Connectivity and routing were verified using **Network Watcher** and VM-side tests.

## Checklist — requirements met

### Connectivity

- **Hub-and-spoke connectivity via peering**:
  - Hub `udatech-starter-vnet (10.0.0.0/16)` is peered with `vnet-hr (10.10.0.0/16)`, `vnet-sales (10.20.0.0/16)`, and `vnet-devops (10.30.0.0/16)`.
- **Functional connectivity to key services**:
  - `udatech-starter-vm (10.0.0.4)` can reach the Sales ILB `10.20.1.4:80` (**Reachable** in Network Watcher).
  - `udatech-starter-vm (10.0.0.4)` can reach the Storage private endpoint `10.30.3.4:443` (**Reachable** in Network Watcher).

Evidence:

- `docs/ARCHITECTURE_FINAL.md`
- `docs/RUBRIC_08_NETWORK_WATCHER_VALIDATION.md`
- `docs/VALIDATION_TESTS.md`

### Security

- **Reduced public exposure**:
  - Public SSH exposure was removed (starter VM now uses private IP only and is accessed via Bastion).
- **Secure remote admin**:
  - Azure Bastion (`bastion-udatech`) provides SSH access without exposing VMs to the public internet.
- **Department isolation controls**:
  - Subnet NSGs (`nsg-hr`, `nsg-sales`, `nsg-devops`) enforce a default “deny inter-department” posture (explicit deny rules between \(10.10.0.0/16\), \(10.20.0.0/16\), \(10.30.0.0/16\)).
- **Private PaaS access**:
  - Storage account `udatechsa4c79f445` is accessed via a private endpoint (observed `10.30.3.4`) with Private DNS (`privatelink.blob.core.windows.net`).

Evidence:

- `docs/RUBRIC_04_NSG_ASG_PRIVATE_ENDPOINTS.md`
- `docs/RUBRIC_05_BASTION_INTERNAL_ONLY_VM_ACCESS.md`
- `docs/RUBRIC_06_DNS_NAME_RESOLUTION.md`

### Scalability / operations

- **Address plan sized for growth**:
  - Separate /16 VNets per department allow additional subnets/tier expansion without redesign.
- **Operational diagnostics**:
  - Network Watcher used to validate routing and connectivity (reachability + next-hop types).
- **Load balancing pattern established**:
  - Sales uses an ILB with a backend pool of two VMs and health probe/rule configuration; this pattern scales horizontally by adding additional backends.

Evidence:

- `docs/RUBRIC_07_LOAD_BALANCER_HA.md`
- `docs/RUBRIC_08_NETWORK_WATCHER_VALIDATION.md`

### Business goals (mapped)

- **HR / Sales / DevOps separation**: Achieved through distinct VNets and subnetting strategy.
- **Minimize blast radius**: Inter-department denies reduce lateral movement risk.
- **Secure administration**: Bastion removes direct internet admin paths.
- **High availability for critical apps**: Sales ILB demonstrates HA approach.
- **Secure access to cloud services**: Private endpoint + Private DNS for Storage ensures private-only access.

## Gaps / recommendations (next steps)

These are not required for the rubric, but are standard production hardening steps:

- **Centralized egress control**: Add Azure Firewall/NVA in hub + UDRs to control/inspect outbound traffic.
- **NSG flow logs + Log Analytics**: Enable flow logs to support auditing and incident response.
- **Use ASGs in NSG rules**: Replace CIDR-based rules with ASG-based rules for easier operations as VM counts grow.
- **Policy governance**: Apply Azure Policy to prevent public IPs on NICs/VMs and enforce required tags.

## Where this fits in the repo

- Consolidated architecture: `docs/ARCHITECTURE_FINAL.md`
- This final review: `docs/RUBRIC_10_FINAL_REVIEW.md`

