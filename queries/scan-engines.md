# Scan-Engine Lookups

Host records only: passive, from each engine's existing scan data. Do not run active scans against these hosts.

> Values in this file are **not defanged**, so the queries run as-is.

---

## Host pages

| Engine | Lookup | What it showed (Nov 2026) |
|---|---|---|
| FOFA | `ip="173.214.174.40"` | Windows Server: SMB, RPC, RDP, WinRM, SSH, IIS, Dynamics NAV on 8080, MSMQ, TLS on 8871 (undated export) |
| FOFA | `ip="153.75.247.94"` | Webuzo stack. A Sep 17 record shows Ubuntu Apache and `nibibomo.com` |
| ODIN | `https://search.odin.io/hosts/173.214.174.40/raw-data/json` | Reverse DNS `vps3379646.trouble-free.net`, SMB 445 live 2026-10-12 |
| Shodan | `https://www.shodan.io/host/173.214.174.40` | No record (404) |
| Shodan | `https://www.shodan.io/host/153.75.247.94` | Webuzo, Exim hostname `C2`, cert `alma9.ispot.cc` |
| Censys | `173.214.174.40` | 0 services, possible blocklist entry. Forward DNS lists both C2 names and the prior tenant's name |

## Pivots left open (not run)

| Question | Query |
|---|---|
| Other servers with hostname `C2` | Shodan: `"220-C2 ESMTP Exim"`, then `asn:AS19318 "220-C1 ESMTP Exim"` and `"220-C3 ESMTP Exim"` |
| Is the May 4 Ubuntu build date unique? | Shodan: `"29af-650fed4d6e93f"` |
| Is the staging SSH key unique? | Search the SHA-256 `798ad809d1291343474a939048fe4922325514751a247252b24f1a760bbddf74` in Censys / FOFA |
| Who held the C2 VPS before May 2026? | Historical reverse DNS for 173.214.174.40 (InterServer VPS ID `vps3379646`) |

## Template control

Before pivoting on any staging-server artifact, compare it with a known unrelated host on the same InterServer Webuzo / AlmaLinux 9 template. See [../methodology/template-exclusions.md](../methodology/template-exclusions.md).
