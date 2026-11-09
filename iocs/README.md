# IOC Inventory

Indicators for the nvcontainer WebSockets backdoor toolset. **Read the context notes before acting on any of them.**

## Files

| File | Contents | Durability |
|---|---|---|
| [`file-hashes.csv`](file-hashes.csv) | Downloaders, backdoor, staged library, status marker | Long-lived. Binary artifacts |
| [`network-indicators.csv`](network-indicators.csv) | Operator domains, staging and C2 IPs, staging URLs, C2 endpoint, JA3 | Moderate. The staging web ports closed around Nov 1–8, 2026 |
| [`host-artifacts.csv`](host-artifacts.csv) | Mutexes, paths, registry, archive patterns, protocol strings, UA | Long-lived per build |
| [`excluded-artifacts.csv`](excluded-artifacts.csv) | Hosting-template and stock-library artifacts | **Do not pivot or block on these** |

## Column conventions

- **`value`** holds raw values for tool ingestion. **`defanged`** (network file) holds safe-to-paste values. URLs are defanged only.
- **`tag`**: `NOVEL` · `CORROBORATED` · `PUBLISHED` · `EXCLUDED`, as defined in the main README.

## Context notes

- **173.214.174[.]40 (C2)** is assessed as **likely a legitimate business server**. Other parties may depend on it, and a third-party hostname still resolves to it.
- **The staging URLs** use `.jpg` / `.png` names for DLLs, a CA bundle and the backdoor. The table of requested names to saved names is in [`../docs/technical-analysis.md`](../docs/technical-analysis.md).
- **The stock DLL hashes** (OpenSSL, Microsoft runtime, CA bundle) are listed only in `excluded-artifacts.csv`. They appear in unrelated software.
- **`[PONG]]` and `[LAL_P]`** travel inside TLS. They are only visible where TLS is decrypted.

## Confidence

| Indicator group | Confidence | Note |
|---|---|---|
| Sample hashes | High | From public samples, verified locally |
| Operator domains | High | WHOIS + passive DNS + sample config |
| Staging IP | High | Hard-coded in both downloaders |
| C2 IP as operator-controlled infrastructure | Moderate | Used by the operator. Ownership of the server is unresolved |
| Host artifacts | High | Read directly from the binaries |
