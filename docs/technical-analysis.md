# Technical Analysis — Reverse Engineering

> Covers the C++ downloader, the .NET downloader and the backdoor. The C2 protocol has its own document: [c2-protocol.md](c2-protocol.md).

All work was static. The only execution was offline emulation of single code paths with Unicorn, with no network.

---

## 1. Samples

| Role | File | SHA-256 |
|---|---|---|
| C++ downloader | `document.exe` | `b6c49943afd87215dbd560583fc0c464f3dcf819278557953a33a811b4880f36` |
| .NET downloader (managed core) | `Dper.dll` | `7a0ce15051cf82cfb107d94dd4a16d55ee18783e2577b434d268799db8f843c9` |
| .NET single-file bundle (VT run) | — | `d172138487a2e66398c7474e24849dbf19157bb30823de7df98556943de7f74f` |
| Backdoor | `nvcontainer.exe` | `7bdd7d62f28d859e8dc63a962e56b4fbf72135fde0e92bebf36e2a4fde628f27` |

The staged support files are listed in [../iocs/file-hashes.csv](../iocs/file-hashes.csv).

**Note:** `Dper.dll` behaves the same as the VT run of bundle `d1721384…`: same path, mutex and `.dat` files. That it is the bundle's core is **not hash-verified**.

---

## 2. C++ downloader (`document.exe`)

| Item | Value |
|---|---|
| MD5 | `a4850780bb5754288935f4a6cc4b2c49` |
| Compiled | 2026-09-13 11:27:11 UTC |
| PDB | `L:\1-The-Last-Versions\sendingto-costumers-version2\TAHVILAT\mehr\DP-version 2.1.0\Dpr-Vor-Cpp -final - Copy\x64\Release\Dp.pdb` |
| Strings | XOR `0x5A` |
| API resolution | FNV-1a hashing (`0x811c9dc5` / `0x1000193`) |
| Network | WinHTTP: HEAD for size, then GET with `Range` resume (HTTP 206) |
| Concurrency | Max 3 downloads at once |
| Mutex | `Global\NvCDIr_U54321` |
| Install path | `%APPDATA%\Microsoft\Protect\NV\` |

**PDB note:** the folder names look like a versioned project ("DP-version 2.1.0", "Dpr-Vor-Cpp"). Some words read as Persian transliterations ("TAHVILAT", "mehr"). This is recorded as a **language artifact only, not attribution.**

---

## 3. .NET downloader (`Dper.dll`)

| Item | Value |
|---|---|
| Assembly | `Dper.dll` v1.0.0, .NET 8, Release, 23,552 bytes |
| Build | Deterministic: the timestamp is a hash. No PDB path |
| Obfuscation | None. Plaintext strings, debug console logs left in |
| Mutex | `Global\Nvidia_Container_Downloader_Unique_ID_54321` |
| Install path | `C:\Users\<user>\AppData\Roaming\Microssoft\Cache\nvcontainer.exe` (typo hard-coded) |
| Status marker | `%TEMP%\nvidia_download_status.dat` = `COMPLETED` |
| Resume file | `%TEMP%\nvidia_download_progress.dat` (`name=size` lines) |
| Crash file | `%TEMP%\Nvidia container_error.txt` |
| Persistence | None. The backdoor handles persistence |

### Run order

1. Hides its console. Blocks close, logoff and shutdown events until the download completes.
2. Waits a random 1–10 s. It repeats this between every stage.
3. If the status file reads `COMPLETED` and the exe exists, it just launches the exe and stops.
4. Sends a HEAD for all 7 files at once.
5. **Ping gate:** pings 8.8.8.8 with a 2 s timeout. It needs a round trip under 500 ms.
   - If that fails, it retries every 5 s for up to 3 hours.
   - After that, it retries every 5 s **with no limit** until any reply comes back.
   - **Effect:** a sandbox that blocks ICMP never sees a payload GET.
6. Sends GETs, **at most 3 at a time**. Up to 5 tries per file, with Range resume.
7. Outer loop: up to 50 rounds. The wait between rounds is min(5 s × round, 60 s).
8. Writes `COMPLETED`, then runs `nvcontainer.exe` with no arguments.

### HTTP fingerprint

- UA: `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36`. It is cut off there, with no `(KHTML, like Gecko)`.
- `Accept-Encoding: gzip, deflate`. No `Accept` header and no `Connection` header.

### Cleanup bug

After a successful run, cleanup deletes `download_status.dat`, but the real file is `nvidia_download_status.dat`. **The status marker survives on disk**, which gives defenders a host artifact.

Cleanup also deletes files this build never creates:
- `HKCU\SOFTWARE\Nvidia containerDownloader`
- `%TEMP%\Nvidia_container_*.tmp`
- `C:\ProgramData\error_log.txt`
- `...\Microssoft\Cache\download.log`

These are likely left over from an earlier build and can serve as hunting leads.

### Code matched to a sandbox capture (Triage `261016-lrbefscx8s`)

| Wire | Code |
|---|---|
| 7 HEADs at once | Parallel HEAD for all files |
| One ICMP echo to 8.8.8.8 at +42.7 s, reply received | Ping gate passed |
| GETs start at +47.7 s | Random 1–10 s delay |
| GETs only for libcrypto, libssl and ucrtbased | The first 3 files in the table, and the 3-download limit |

---

## 4. The two downloaders compared

| | C++ | .NET |
|---|---|---|
| Staging | 153.75.247[.]94 | Same |
| File table | 7 files, `.jpg`/`.png` disguise | Same files, same order |
| Mutex | `Global\NvCDIr_U54321` | `Global\Nvidia_Container_Downloader_Unique_ID_54321` |
| Install folder | `Roaming\Microsoft\Protect\NV\` | `Roaming\Microssoft\Cache\` |
| Strings | XOR 0x5A, API hashing | Plaintext |
| Parallel GETs | 3 | 3 |
| Ping gate | None found | Yes |

**Staged file table** (requested as / saved as):

| Requested | Saved |
|---|---|
| `/libcrypto-3-x64.jpg` | `libcrypto-3-x64.dll` |
| `/libssl-3-x64.jpg` | `libssl-3-x64.dll` |
| `/ucrtbased.jpg` | `ucrtbased.dll` |
| `/vcruntime140D.jpg` | `vcruntime140D.dll` |
| `/websockets.jpg` | `websockets.dll` |
| `/cacert.jpg` | `cacert.pem` |
| `/nvcontainer.png` | `nvcontainer.exe` |

Despite the names, `ucrtbased.dll` and `vcruntime140D.dll` are **release** runtimes that were renamed, not debug builds (see retraction 2).

---

## 5. Backdoor (`nvcontainer.exe`)

| Item | Value |
|---|---|
| MD5 | `5e427e93694cf8ed07ba04d5ae7437c7` |
| Imphash | `fcfc49049b543c5b5d608be8ff48e901` |
| Build | `/Brepro`: the timestamp is a hash. Build tag `v6` |
| Disguise | Fake NVIDIA version info |
| Network library | `websockets.dll` (libwebsockets 4.3.99, stock OpenSSL 3.1.2 DLLs) |
| Persistence | `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` value `MyClientApp` |
| Activation flag | `HKCU\SOFTWARE\Microsoft\MyApp` `Activated` = 1 |
| Mutex | `MyUniqueClientAppMutex` |
| Archive names | `V_<drive>_<8hex>_<name>.zip`, plus a `.vortexstamp` sidecar |
| Archiving | `tar.exe -a -caf`, falling back to PowerShell `-EncodedCommand` (ZipFile), plus `cmd /c mklink /J` |

**Commands:** recon, exfiltration and download only. **There is no command-execution command.** The full protocol is in [c2-protocol.md](c2-protocol.md).

**Debug strings:** many log strings in the binary are mangled, with characters dropped. They are only partly readable and carry no protocol content.
