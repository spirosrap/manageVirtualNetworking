# Validation & Test Evidence (Azure Networking)

Use this document to record the validation steps and outputs/screenshots for the “Validate and Test Network Architecture” requirement.

## 1) Load balancer distribution test (Sales ILB)

- **Resource**: Internal Load Balancer `lb-sales-ilb`
- **Frontend IP**: `10.20.1.4`
- **Backend VMs**: `sales-web-1`, `sales-web-2`

### Test

From `udatech-starter-vm` (via Bastion SSH), curl the ILB multiple times:

```bash
for i in {1..10}; do curl -s http://10.20.1.4/; done
```

### Expected result

Responses should show both backends over multiple requests (e.g., `hello from sales-web-1` and `hello from sales-web-2`).

## 2) Private endpoint DNS & connectivity test (Storage Blob)

- **Storage account**: `udatechsa4c79f445`
- **Private endpoint subnet**: `vnet-devops/snet-devops-pe` (`10.30.3.0/24`)
- **Private DNS zone**: `privatelink.blob.core.windows.net`

### Test

From `udatech-starter-vm` (via Bastion SSH):

```bash
nslookup udatechsa4c79f445.blob.core.windows.net
curl -I https://udatechsa4c79f445.blob.core.windows.net/
```

### Expected result

- `nslookup` returns an address in `10.30.3.0/24` (private endpoint IP).
- `curl -I` returns an HTTP response (even `400/403` is acceptable for connectivity proof).

## 3) Network Watcher checks (run from Cloud Shell)

> Network Watcher provides “ground truth” troubleshooting: route/NSG evaluation and connection results.

Record:

- Connection troubleshoot (VM → ILB, VM → Storage private endpoint)
- IP flow verify (selected NIC + destination/port)
- Next hop (validate routing/peering behavior)

### Captured results (Cloud Shell)

#### Install agent (prerequisite)

- VM extension: `NetworkWatcherAgentLinux` → **Succeeded**

#### Connection troubleshoot: VM → Sales ILB (HTTP/80)

- Source: `udatech-starter-vm` (private IP `10.0.0.4`)
- Destination: `10.20.1.4:80` (Sales ILB)
- Result: **Reachable**
- Observations: Hop path included hub→sales peering and ILB backend IPs `10.20.1.5` and `10.20.1.6`.

#### Connection troubleshoot: VM → Storage private endpoint (HTTPS/443)

- Source: `udatech-starter-vm` (private IP `10.0.0.4`)
- Destination: `10.30.3.4:443` (Storage Blob private endpoint)
- Result: **Reachable**
- Observations: Destination identified as **PrivateEndpoint**.

#### Next hop (routing validation)

- VM → Sales ILB (`10.20.1.4`): `nextHopType` = **VirtualNetworkPeering** (`routeTableId` = `System Route`)
- VM → Storage private endpoint (`10.30.3.4`): `nextHopType` = **PrivateEndpoint** (`routeTableId` = `System Route`)

