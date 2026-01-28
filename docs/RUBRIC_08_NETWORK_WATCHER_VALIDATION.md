# Rubric 8 — Network Watcher / Diagnostic Validation

## Rubric text (from project overview)

- **Criteria**: Verify network connectivity and security using Azure diagnostic tools.
- **Submission requirements**: Screenshots from Network Watcher/testing tools showing successful validation of expected connectivity and security.

## What to validate (expected results)

### Expected connectivity paths

- **Hub VM → Sales ILB**: `udatech-starter-vm (10.0.0.4)` → `lb-sales-ilb (10.20.1.4:80)` should be **Reachable**.
- **Hub VM → Storage private endpoint**: `udatech-starter-vm (10.0.0.4)` → `Storage PE (10.30.3.4:443)` should be **Reachable**.

### Expected routing/segmentation behavior

- **Next hop to Sales ILB** should show **VirtualNetworkPeering** (routing over hub↔sales peering).
- **Next hop to Storage PE** should show **PrivateEndpoint**.

## Evidence to capture (recommended)

Use **Cloud Shell** (Bash) and take screenshots of the command outputs.

### 1) Ensure agent is installed (prerequisite)

```bash
RG=udacity-project-starter
az vm extension show -g $RG --vm-name udatech-starter-vm --name NetworkWatcherAgentLinux -o table
```

Expected: `ProvisioningState` = `Succeeded`.

### 2) Connectivity tests (Network Watcher)

```bash
RG=udacity-project-starter

az network watcher test-connectivity \
  --source-resource udatech-starter-vm \
  --resource-group $RG \
  --dest-address 10.20.1.4 \
  --dest-port 80

az network watcher test-connectivity \
  --source-resource udatech-starter-vm \
  --resource-group $RG \
  --dest-address 10.30.3.4 \
  --dest-port 443
```

Expected: `connectionStatus` = `Reachable`.

### 3) Next hop (route validation)

```bash
RG=udacity-project-starter

az network watcher show-next-hop \
  --resource-group $RG \
  --vm udatech-starter-vm \
  --source-ip 10.0.0.4 \
  --dest-ip 10.20.1.4

az network watcher show-next-hop \
  --resource-group $RG \
  --vm udatech-starter-vm \
  --source-ip 10.0.0.4 \
  --dest-ip 10.30.3.4
```

Expected:

- `10.20.1.4` → `nextHopType` = `VirtualNetworkPeering`
- `10.30.3.4` → `nextHopType` = `PrivateEndpoint`

## Where to store screenshots in this repo

- `docs/screenshots/rubric-08-network-watcher/`

