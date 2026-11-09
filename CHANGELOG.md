# Changelog

All notable changes to this analysis are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project follows analyst-research versioning rather than software semver.

## [1.0.0] — 2026-11-08

### Initial public release

First public release of the nvcontainer WebSockets backdoor analysis.

#### Added

- Technical analysis of the C++ downloader, the .NET 8 downloader and the backdoor (`docs/technical-analysis.md`)
- Full C2 protocol specification with a session diagram (`docs/c2-protocol.md`)
- Infrastructure intelligence and C2 hosting assessment (`docs/infrastructure.md`)
- Dated timeline, May → Nov 2026 (`docs/timeline.md`)
- Retraction log with six corrections (`docs/retraction-log.md`)
- Sanitization and redaction notes (`docs/disclosure.md`)
- Methodology for static protocol reconstruction, the passive pivot chain and template exclusion
- Reproducible passive-DNS, SOA and scan-engine lookups (`queries/`)
- IOC inventory: file hashes, network indicators, host artifacts, excluded artifacts
- STIX 2.1 bundle
- Architecture, kill-chain and protocol diagrams (`assets/architecture-diagram.md`)

#### Key findings in this release

- C2 protocol rebuilt statically. All 11 send paths mapped. The `[PONG]]` reply bug confirmed by emulation
- .NET downloader documented: ping gate, cleanup bug, leftover artifacts
- 5-day infrastructure build in early May 2026. Staging domain `nibibomo[.]com` identified
- C2 zone edited Sep 28, 2026, the day before the first sample sighting
- C2 IP assessed as likely a legitimate business Windows server (Dynamics NAV, MSMQ)

#### Sanitization applied

- A third-party hostname that resolves to the C2 IP is redacted
- Indicators are defanged in all prose
- No attribution is made
