# Disclosure and Sanitization

## What was redacted

| Item | Treatment | Why |
|---|---|---|
| A business hostname that still resolves to the C2 IP | Shown as `[REDACTED prior tenant]` | The business may be a victim, either of a hijacked server or of a stale DNS record. Naming it publicly could cause harm before it is notified |
| Victim organizations | Not named. Targeting is described only as "likely Iraq" | Matches the original public report. No victim data was available |

## What was kept

- **Hashes, operator domains and server IPs.** The operator domains are `xionamiconm[.]com` and `nibibomo[.]com`. The servers are the staging IP and the C2 IP.
- **The C2 IP is published** even though it may be a legitimate business server, because it is already public in the original report. It carries a note against blocking decisions that affect third parties without context.

## Defanging

All domains, IPs and URLs in the Markdown docs are defanged (`[.]`, `hxxp`). The CSV files carry raw values in the `value` column for tool ingestion, with defanged URLs.

## Notification status

| Party | Status |
|---|---|
| Prior tenant of the C2 IP | Notification at the analyst's discretion. Identity withheld here |
| Hosting provider (InterServer) | No report filed from this research |
| Original reporter (dmpdump) | Credited as prior work |

## Sources of data

Everything here comes from the analyst's own offline analysis of public samples, public sandbox reports, and public passive data: passive DNS, CT logs, public DNS resolution, and scan-engine host records. No private or incident-response data is included.
