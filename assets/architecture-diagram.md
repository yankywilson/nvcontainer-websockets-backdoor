# Architecture Diagrams

All diagrams render natively on GitHub (Mermaid).

---

## 1. Kill chain (as observed)

```mermaid
flowchart LR
    subgraph D["Delivery (vector unknown)"]
        A1["document.exe<br/>C++ · compiled 2026-09-13"]
        A2["Dper.dll<br/>.NET 8 single-file"]
    end
    subgraph S["Staging · 153.75.247[.]94"]
        F["7 files as .jpg/.png<br/>libcrypto · libssl · ucrtbased<br/>vcruntime140D · websockets<br/>cacert · nvcontainer"]
    end
    subgraph H["Host"]
        I["%APPDATA%\\Microsoft\\Protect\\NV\\<br/>or \\Microssoft\\Cache\\"]
        B["nvcontainer.exe<br/>Run key MyClientApp"]
    end
    subgraph C["C2 · www.xionamiconm[.]com:8050"]
        W["WSS /ws/<br/>ws-protocol"]
    end
    A1 -- "HEAD ×7, GET ×3 at a time" --> F
    A2 -- "ping gate → HEAD ×7, GET ×3" --> F
    F --> I --> B
    B <== "recon · exfil · download" ==> W
```

---

## 2. Infrastructure topology

```mermaid
flowchart TB
    subgraph REG["Registration · Tucows via InterServer reseller · InterServer DNS (cdns1-3)"]
        D1["xionamiconm[.]com<br/>created 2026-05-03<br/>zone edited 2026-09-28"]
        D2["nibibomo[.]com<br/>created 2026-05-06<br/>zone unchanged since"]
    end
    subgraph AS["AS19318 · InterServer"]
        C2[("C2 · 173.214.174[.]40<br/>vps3379646<br/>Windows · IIS · MSMQ<br/>Dynamics NAV :8080")]
        ST[("Staging · 153.75.247[.]94<br/>Ubuntu Apache (May→Sep)<br/>Webuzo / Alma 9 (Oct→)<br/>hostname C2")]
    end
    PT["[REDACTED prior tenant]<br/>TLS certs 2015 → 2026-03"]
    D1 -- "A · from 2026-05-05" --> C2
    D2 -- "A · from 2026-05-08" --> ST
    PT -. "A record still live" .-> C2

    classDef op fill:#3b0d0d,stroke:#ff6b6b,color:#fff
    classDef third fill:#2b2b2b,stroke:#adb5bd,color:#fff,stroke-dasharray: 4 3
    class D1,D2,ST op
    class PT third
```

---

## 3. Backdoor state machine (connection layer)

```mermaid
stateDiagram-v2
    [*] --> Dialing
    Dialing --> Connected: TLS + WS upgrade OK
    Dialing --> Backoff: no connect in 12 s
    Backoff --> Dialing: 2–5 s (cap 12 s) + PID-parity jitter
    Connected --> Connected: [HB] every ≥5 s / any server frame
    Connected --> Uploading: PPPP → [RESUME_OFFSET]
    Uploading --> Connected: [END] → [CONFIRMED] / [CAL]
    Connected --> Backoff: server silent 15 s
    Uploading --> Backoff: server silent 120 s
    Connected --> Stopped: [DIS] (code 1000)
    Stopped --> [*]
```

---

## 4. Upload handshake

```mermaid
sequenceDiagram
    participant C2
    participant I as nvcontainer.exe
    C2->>I: PPPP <path>
    I-->>C2: [PREPARE_PROGRESS]queued → scanning → zipping | raw-stream
    I->>C2: [START]
    Note right of I: 500 ms
    I->>C2: [FILENAME]V_C_1A2B3C4D_Documents.zip
    Note right of I: 500 ms
    I->>C2: [FILESIZE]<bytes>
    C2->>I: [RESUME_OFFSET]<n>
    loop from offset n
        I->>C2: binary frame ≤ 64 KB
    end
    I->>C2: [END]
    C2->>I: [CONFIRMED]
    Note over I: delete zip + .vortexstamp
```

*The filename in the upload diagram is illustrative. It shows the `V_<drive>_<8hex>_<name>.zip` pattern.*
