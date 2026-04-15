# Honeypot Blocklist

Public threat-intel list derived from an ICS/SCADA-themed honeypot exposed on the open internet. Every listed IP has interacted with the honeypot unsolicited and does not belong to any research scanner I could identify (Shodan, Censys, Shadowserver, BinaryEdge, Onyphe and others are filtered out).

The lists use the de-facto standard format: **one IP per line** with `#`-prefixed header comments. Compatible with `ipset`, `iptables`, `nftables`, `fail2ban`, Cloudflare IP Access Rules, pfSense/OPNsense URL tables, nginx, Caddy and anything that speaks FireHOL/Spamhaus-style plain text.

## Lists

### [`high.txt`](high.txt)
IPs that actively tried to compromise or manipulate the honeypot. Examples: successful or repeated SSH brute-force (≥20 attempts), VNC authentication attempts, HMI admin login attempts, control-plane interactions like setpoint changes or alarm acknowledgements, active Modbus/S7 function calls, CVE probes against environment and backup endpoints. **High confidence** that the activity is malicious.

### [`medium.txt`](medium.txt)
IPs that only scanned, probed or anonymously loaded pages. TCP handshakes on Modbus/S7/VNC with no further interaction, `GET` requests to the login page, passive reconnaissance. **Medium confidence** - could be research or opportunistic scanning. Use with care.

## Usage

### iptables / ipset
```bash
ipset create honeypot_high hash:ip
curl -s https://raw.githubusercontent.com/MuteBefehl/honeypot-blocklist/main/high.txt \
  | grep -vE '^(#|$)' \
  | xargs -I {} ipset add honeypot_high {} 2>/dev/null

iptables -I INPUT -m set --match-set honeypot_high src -j DROP
```

### nftables
```bash
nft add set inet filter honeypot_high { type ipv4_addr\; }
curl -s https://raw.githubusercontent.com/MuteBefehl/honeypot-blocklist/main/high.txt \
  | grep -vE '^(#|$)' \
  | while read ip; do nft add element inet filter honeypot_high { $ip }; done
nft add rule inet filter input ip saddr @honeypot_high drop
```

### nginx
```bash
curl -s https://raw.githubusercontent.com/MuteBefehl/honeypot-blocklist/main/high.txt \
  | grep -vE '^(#|$)' \
  | sed 's|^|deny |; s|$|;|' > /etc/nginx/honeypot-blocklist.conf

# then in your server block:
# include /etc/nginx/honeypot-blocklist.conf;
```

### pfSense / OPNsense
Firewall → Aliases → IP → URL Table, paste one of the raw URLs above as the source.

### Cloudflare
Use the IP Access Rules API, or create a WAF Custom Rule with an `ip.src in { ... }` match list.

## Updates

The lists are periodically regenerated from fresh honeypot events. IPs that stay inactive for a few weeks may disappear in future updates (residential/CGNAT recycling).

## No warranties

This is a best-effort threat-intel dump from a single honeypot, not a commercial feed. False positives are possible - especially in `medium.txt`. Review whether the list fits your use case before deploying it in production.

## Reporting false positives

If you believe an IP shouldn't be listed (for example because it belongs to a new research scanner or a subnet was mis-identified), open an issue with the IP and a short justification.

## License

The lists themselves are Public Domain / CC0 - use them, redistribute them, combine them with your own feeds, no attribution required. Scripts and README are MIT.
