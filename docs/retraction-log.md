# Retraction Log

Corrections made during the analysis, in order. Each entry shows what was said, what replaced it, and why.

---

## 1. Downloader concurrency

- **Was:** the C++ downloader fetches all 7 files in parallel.
- **Now:** at most 3 at a time.
- **Why:** the pcap, the Joe Sandbox thread log and a `cmp r13d, 3` check in the code all show a limit of 3.

## 2. "Debug" runtime DLLs

- **Was:** one of the staged DLLs is very likely a Debug build.
- **Now:** all are release builds. `ucrtbased.dll` is Microsoft-signed release `ucrtbase.dll` 10.0.18362.815, renamed. `vcruntime140D.dll` is very likely release `vcruntime140.dll` 14.28.29115, renamed.
- **Why:** `websockets.dll` and the OpenSSL DLLs import the release `VCRUNTIME140.dll`, and the signature and version data match release builds.

## 3. Triage "TCP connection opened"

- **Was:** in the Oct 15 Triage run, the staging TCP connection opened but no HTTP reply came back.
- **Now:** the Triage runs only show that no content came back from upstream.
- **Why:** Triage puts a transparent TCP proxy in front of the sandbox. Every server showed TTL 62 and the same ~43.6 ms handshake. A Triage run cannot show whether the real server accepted the connection.

## 4. Staging certificate as a pivot

- **Was:** pivot on the staging server's `alma9.ispot.cc` certificate and its JARM.
- **Now:** EXCLUDED as a hosting-template artifact.
- **Why:** the identical certificate (same serial and dates) is on an unrelated InterServer customer, and it was seen in another InterServer block too.

## 5. C2 IP reassignment

- **Was:** likely the C2 IP was released and reassigned to the operator.
- **Now:** roughly even chance either way. Superseded by entry 6.
- **Why:** FOFA showed an established Windows Server profile (IIS, MSMQ, RDP, SMB, WinRM), which a fresh rental rarely has.

## 6. C2 hosting

- **Was:** roughly even chance between "rented" and "legacy server".
- **Now:** likely (55–80%) the C2 runs on a legitimate business server, either hijacked or reassigned.
- **Why:** port 8080 serves the Microsoft Dynamics NAV default page. The FOFA data is undated, so this stays at "likely" until dated host names are found.
