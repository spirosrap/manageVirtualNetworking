# Rubric 7 screenshots — Load Balancer (HA)

Put Rubric 7 screenshots in this folder.

## What to capture (portal)

Open `lb-sales-ilb` and capture:

1. Overview
2. Frontend IP configuration (show `fe-sales` and `10.20.1.4`)
3. Backend pools (show `be-sales` with `sales-web-1` and `sales-web-2`)
4. Health probes (show `probe-http`)
5. Load balancing rules (show `rule-http`)

## Optional (CLI evidence screenshots)

```bash
RG=udacity-project-starter
az network lb show -g $RG -n lb-sales-ilb -o table
az network lb frontend-ip list -g $RG --lb-name lb-sales-ilb -o table
az network lb address-pool list -g $RG --lb-name lb-sales-ilb -o table
az network lb probe list -g $RG --lb-name lb-sales-ilb -o table
az network lb rule list -g $RG --lb-name lb-sales-ilb -o table
```

## Optional (functional test screenshot)

From `udatech-starter-vm` (via Bastion SSH):

```bash
for i in {1..10}; do curl -s http://10.20.1.4/; done
```

