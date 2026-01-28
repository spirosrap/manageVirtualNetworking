# Rubric 7 — Load Balancer (High Availability) Evidence

## Rubric text (from project overview)

- **Criteria**: Deploy and configure load balancers for Azure VMs to ensure high availability.
- **Submission requirements**: Screenshots of load balancer deployment and configuration, including backend pools and rules.

## What is implemented

- **Load Balancer**: `lb-sales-ilb` (Internal Load Balancer, Standard SKU)
- **Frontend IP**: `10.20.1.4` (in `vnet-sales/snet-sales-app`)
- **Backend pool**: `be-sales`
- **Backends**: `sales-web-1 (10.20.1.5)`, `sales-web-2 (10.20.1.6)`
- **Health probe**: `probe-http` on port `80`
- **LB rule**: `rule-http` TCP `80` → `80` (frontend `fe-sales` → backend pool `be-sales`)

## Proof / what to screenshot

### Portal screenshots (recommended)

Azure Portal → Resource group `udacity-project-starter` → Load balancer `lb-sales-ilb`:

1. **Overview** (shows LB type + frontend IP configuration)
2. **Frontend IP configuration** (shows `fe-sales` and private IP `10.20.1.4`)
3. **Backend pools** (shows `be-sales` containing `sales-web-1` + `sales-web-2` NICs)
4. **Health probes** (shows `probe-http` port `80`)
5. **Load balancing rules** (shows `rule-http`)

### CLI tables (optional, also acceptable as evidence)

```bash
RG=udacity-project-starter

az network lb show -g $RG -n lb-sales-ilb -o table
az network lb frontend-ip list -g $RG --lb-name lb-sales-ilb -o table
az network lb address-pool list -g $RG --lb-name lb-sales-ilb -o table
az network lb probe list -g $RG --lb-name lb-sales-ilb -o table
az network lb rule list -g $RG --lb-name lb-sales-ilb -o table
```

### Functional test (nice bonus)

From `udatech-starter-vm` (via Bastion SSH):

```bash
for i in {1..10}; do curl -s http://10.20.1.4/; done
```

Expected: output includes both `hello from sales-web-1` and `hello from sales-web-2`.

## Where to store screenshots in this repo

- `docs/screenshots/rubric-07-load-balancer/`

