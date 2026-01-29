## Don’t upload anything that includes:

- Public IPs, VPN keys, API keys
- Passwords
- Full firewall exports with secrets
- Public IP addresses (WAN IP, ISP details)
- Domain name you own, DDNS hostname, Cloudflare zone names
- VPN details (WireGuard/OpenVPN keys, PSKs, server endpoint)
- Port forwards, exposed management ports (especially 443/8443/8006/22/3389)
- Full pfSense backups/exports (XML) — they often include sensitive material
- Usernames + any auth hints (RADIUS secrets, LDAP bind strings, etc.)
  
## Only upload internal RFC1918 subnets like 10.x/172.16-31/192.168.

## Sanitization Method
- Documentation uses **representative subnets** (not my real home IP plan).
- Screenshots are reviewed to remove WAN/public IPs, DDNS/domains, VPN details, and secrets.
- No raw pfSense backups/exports are uploaded.
