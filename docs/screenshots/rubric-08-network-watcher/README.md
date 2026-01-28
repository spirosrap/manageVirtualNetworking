# Rubric 8 screenshots — Network Watcher validation

Put Rubric 8 screenshots in this folder.

## Recommended set (Cloud Shell outputs)

1. `NetworkWatcherAgentLinux` extension status = Succeeded
2. `az network watcher test-connectivity` to:
   - `10.20.1.4:80` (Sales ILB) → Reachable
   - `10.30.3.4:443` (Storage private endpoint) → Reachable
3. `az network watcher show-next-hop` to:
   - `10.20.1.4` → VirtualNetworkPeering
   - `10.30.3.4` → PrivateEndpoint

See `docs/RUBRIC_08_NETWORK_WATCHER_VALIDATION.md` for the exact commands and expected values.

