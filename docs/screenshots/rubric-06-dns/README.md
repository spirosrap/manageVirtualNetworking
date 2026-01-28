# Rubric 6 screenshots — DNS & name resolution

Put Rubric 6 screenshots in this folder.

## Suggested set

1. **Private DNS records (Cloud Shell)**
   - `az network private-dns record-set a list ...` showing the `udatechsa4c79f445` A record in `privatelink.blob.core.windows.net`
2. **Cross-VNet resolution test (VM via Bastion)**
   - `nslookup udatechsa4c79f445.blob.core.windows.net` showing a `10.30.3.x` address
   - (Optional) `curl -I https://udatechsa4c79f445.blob.core.windows.net/` showing an HTTP response


