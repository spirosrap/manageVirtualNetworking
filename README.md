# UdaTech Azure Virtual Networking (Design-First)

This repository starts with **assessment + network architecture documentation** for a secure, scalable Microsoft Azure virtual network for UdaTech.

## Scope (phased)

- Assess existing environment and identify gaps in **connectivity**, **security**, and **scalability**
- Propose and implement a topology and segmentation plan for UdaTech departments:
  - **HR**
  - **Sales**
  - **DevOps**
- Define network security controls to be implemented later:
  - **NSGs**, **ASGs**, **private endpoints**, **Azure Bastion**, **DNS**, **Load Balancers**, and **diagnostics**

## Documents

- `docs/ASSESSMENT_GAPS.md` — assessment checklist and gap analysis framework
- `docs/NETWORK_TOPOLOGY.md` — proposed network architecture, segmentation, address plan, and security controls
- `docs/ARCHITECTURE_FINAL.md` — consolidated “final architecture” document for submission
- `docs/RUBRIC_10_FINAL_REVIEW.md` — executive summary / checklist mapping solution to requirements

## Non-goals (for this phase)

- This repo is not meant to be a reusable IaC module (it documents one project run) 

