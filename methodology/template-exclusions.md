# Template Exclusions

**Why this matters:** cheap hosting is built from templates. Anything that ships with a template is shared by thousands of unrelated customers, and pivoting on it leads to innocent servers. Every candidate pivot on the staging server was tested against a **control host**: an unrelated InterServer customer on the same template (identity withheld: WordPress sites, Exim hostname `wp-server`).

---

## Results

| Candidate pivot | Staging server | Control host | Verdict |
|---|---|---|---|
| TLS cert `CN=alma9.ispot.cc`, serial `07:73:5a:a4:…:66:ce`, valid 2026-05-31 → 2027-05-31 | Present | **Identical serial** | **EXCLUDED**: template |
| JARM `3fd3fd15d3fd3fd00042d42d000000a0d14c559c674ff8857d4e511e7563d1` | Present | Same stack | **EXCLUDED** |
| Webuzo default page, Last-Modified 2026-05-31 16:34:37 | Present | Template | **EXCLUDED**: written 36 s after the template cert |
| Server timezone (-0400, then -0500 after Nov 1) | Present | Same | **EXCLUDED**: provider default |
| Exim / Dovecot / BIND / MariaDB / ports 2002–2005 | Present | Same | **EXCLUDED**: Webuzo default install |
| DNS zone layout (`www`, `mail`, `MX 25 mail.<domain>`, no TXT) | Present on both operator domains | Identical on an unrelated zone | **EXCLUDED**: InterServer DNS default |
| Nameserver `cdns1.interserver.net` | Present | 13,108 domains | **EXCLUDED** |
| **Exim / BIND hostname `C2`** | Present | `wp-server` | **KEPT**: set per server |
| **SSH host key** `798ad809…bddf74` (SHA-256) | Present | Different ECDSA key | **KEPT (provisional)**: staging key type not confirmed |

---

## Also excluded

| Artifact | Why |
|---|---|
| EHLO names (`xmy8qzdypg5pvu.org` …) and `224.x.x.x` addresses in banners | These are the scan engines' own identities |
| Stock OpenSSL and Microsoft runtime DLL hashes | Shared with unrelated software and malware |
| The C2 IP's earlier certificates as an operator pivot | They belong to the prior tenant (2015–2026) |
