# Rubric 3 — Deploy VNets & Subnets (Evidence)

## Rubric text (from project overview)

- **Criteria**: Deploy VNets and subnets according to the segmentation plan.
- **Submission requirements**: Screenshots showing actual VNets and subnets, matching the designed plan.

## Evidence checklist

### Screenshots (portal)

Saved/organized under `docs/screenshots/rubric-03-vnets-subnets/` (see that folder’s `README.md` for mapping).

### CLI proof (Cloud Shell)

```bash
RG=udacity-project-starter
az network vnet list -g $RG -o table

for v in udatech-starter-vnet vnet-hr vnet-sales vnet-devops; do
  echo "== $v subnets ==";
  az network vnet subnet list -g $RG --vnet-name $v -o table;
done
```

Observed output (abridged):

- VNets:
  - `udatech-starter-vnet` — `10.0.0.0/16` — 2 subnets
  - `vnet-hr` — `10.10.0.0/16` — 3 subnets
  - `vnet-sales` — `10.20.0.0/16` — 3 subnets
  - `vnet-devops` — `10.30.0.0/16` — 3 subnets

- Subnets:
  - Hub `udatech-starter-vnet`: `default (10.0.0.0/24)`, `AzureBastionSubnet (10.0.1.0/26)`
  - HR `vnet-hr`: `snet-hr-app (10.10.1.0/24)`, `snet-hr-data (10.10.2.0/24)`, `snet-hr-pe (10.10.3.0/24)`
  - Sales `vnet-sales`: `snet-sales-app (10.20.1.0/24)`, `snet-sales-data (10.20.2.0/24)`, `snet-sales-pe (10.20.3.0/24)`
  - DevOps `vnet-devops`: `snet-devops-tools (10.30.1.0/24)`, `snet-devops-agents (10.30.2.0/24)`, `snet-devops-pe (10.30.3.0/24)`

## Mapping to design

Design is documented in:

- `docs/RUBRIC_02_SEGMENTED_NETWORK_DESIGN.md`
- `docs/NETWORK_TOPOLOGY.md`
- `docs/network-diagram.png`

