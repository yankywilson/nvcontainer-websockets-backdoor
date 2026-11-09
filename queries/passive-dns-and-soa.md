# Passive DNS and SOA Lookups

Passive and third-party lookups only. None of them sends traffic to the operator's hosts. A public resolver query reaches the provider's authoritative DNS (InterServer), not the operator.

Values are defanged here. Refang them before running.

---

## Passive DNS by IP (AlienVault OTX)

```
https://otx.alienvault.com/api/v1/indicators/IPv4/153.75.247[.]94/passive_dns
https://otx.alienvault.com/api/v1/indicators/IPv4/173.214.174[.]40/passive_dns
```

**Expected:** `nibibomo[.]com` (first seen 2026-05-08) on the staging IP. `xionamiconm[.]com` (2026-05-05), `www.xionamiconm[.]com` and the prior tenant's hostname (2023) on the C2 IP.

## Reverse IP and host search (HackerTarget)

```
https://api.hackertarget.com/reverseiplookup/?q=173.214.174[.]40
https://api.hackertarget.com/hostsearch/?q=nibibomo[.]com
https://api.hackertarget.com/hostsearch/?q=xionamiconm[.]com
```

## Current records and SOA serials (Google public DNS, JSON)

```
https://dns.google/resolve?name=xionamiconm[.]com&type=SOA
https://dns.google/resolve?name=nibibomo[.]com&type=SOA
https://dns.google/resolve?name=www.xionamiconm[.]com&type=A
https://dns.google/resolve?name=nibibomo[.]com&type=MX
```

**Reading SOA serials:** InterServer writes `YYYYMMDDnn`, so the serial is the date of the last zone change.

**Control:** before trusting a serial date, query 2–3 unrelated zones that use the same nameservers. Different serials mean there was no provider-wide bump.

## Certificate history (crt.sh)

```
https://crt.sh/?q=xionamiconm[.]com
https://crt.sh/?q=nibibomo[.]com
https://crt.sh/?q=<prior-tenant hostname>      (redacted in this repo)
```

**Expected:** no certificates for either operator domain, and a 2015 → 2025 issuance chain for the prior tenant's hostname.

## WHOIS

Any RDAP or WHOIS client against `xionamiconm[.]com` and `nibibomo[.]com`. Compare the registrar (Tucows), the reseller (InterServer), the nameservers, the creation dates and the registrant state.
