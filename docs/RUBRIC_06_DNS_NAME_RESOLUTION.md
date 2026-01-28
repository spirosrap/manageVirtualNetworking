# Rubric 6 — DNS & Name Resolution (Evidence)

## Rubric text (from project overview)

- **Criteria**: Configure DNS in Azure and resolve names across VNets, troubleshooting issues as needed.
- **Submission requirements**: Documentation and/or screenshots of DNS setup, testing results for resolution across VNets, and steps taken to resolve connectivity issues.

## DNS design used

- **Approach**: Azure Private DNS for private endpoints (no custom DNS servers).
- **Goal**: When workloads inside VNets resolve a PaaS FQDN (e.g., Storage Blob), they receive the **private endpoint IP** instead of a public endpoint.

## What is implemented

### Private DNS zone (Blob)

- Private DNS zone: `privatelink.blob.core.windows.net`
- A record present for the storage account: `udatechsa4c79f445` → private IP (expected in DevOps PE subnet `10.30.3.0/24`)

### VNet links

- Private DNS zone is linked to VNets that must resolve the private endpoint name:
  - `udatech-starter-vnet` (hub)
  - `vnet-devops` (private endpoint lives here)

## Proof / testing (what to capture)

### 1) DNS record evidence (Cloud Shell)

Run and screenshot:

```bash
RG=udacity-project-starter
az network private-dns record-set a list -g $RG -z privatelink.blob.core.windows.net -o table
```

### 2) Cross-VNet resolution evidence (VM via Bastion)

From `udatech-starter-vm` (hub VNet), run and screenshot:

```bash
nslookup udatechsa4c79f445.blob.core.windows.net
curl -I https://udatechsa4c79f445.blob.core.windows.net/
```

Expected:

- `nslookup` returns an address in `10.30.3.0/24` (observed private endpoint IP was `10.30.3.4`).
- `curl -I` returns an HTTP response (even `400/403` is fine for connectivity proof).

## Troubleshooting notes (what we did when things didn’t work)

- **Avoided a common pitfall**: running `az ...` commands inside the VM shell. The VM does not have Azure CLI installed; `az` must be run from **Azure Cloud Shell**.
- **Validated from the correct network context**:
  - Cloud Shell is not inside the VNets, so its DNS results may differ (often public resolution).
  - The definitive test is from a VM inside the VNet(s) (Bastion SSH).

If a VM resolves to a public IP instead of the private endpoint:

- Confirm the Private DNS zone exists and contains an **A record**
- Confirm the zone is **linked to the VNet** where the client VM resides
- Confirm the private endpoint has a DNS zone group attached
- Retry name resolution (optionally restart resolver: `sudo systemctl restart systemd-resolved`)

## Where to store screenshots in this repo

- `docs/screenshots/rubric-06-dns/`

