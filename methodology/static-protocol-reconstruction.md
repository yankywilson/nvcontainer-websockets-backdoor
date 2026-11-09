# Static Protocol Reconstruction

How the C2 protocol was rebuilt from `nvcontainer.exe` with **no live traffic**. No capture of the backdoor talking to its C2 exists in any public sandbox run, because the backdoor was never staged with a working C2.

---

## 1. Ground truth from the library

The backdoor uses stock libwebsockets 4.3.99, so the library's own headers fix the meaning of every number in the callback.

| Constant | Source | Value used |
|---|---|---|
| Callback reasons | `lws-callbacks.h` (v4.3-stable) | 1 connection error · 3 connected · 8 receive · 9 pong received · 10 ready to send · 75 closed |
| Write modes | `lws-write.h` | `LWS_WRITE_TEXT` = 0, `LWS_WRITE_BINARY` = 1 |
| Header room before each buffer | `lws-write.h` | `LWS_PRE` = 16 bytes |
| Protocol table layout | `lws-protocols-plugins.h` | name, callback, per-session size, receive buffer size, … |
| Client handshake headers | `lib/roles/http/client/client-http.c`, `lib/roles/ws/client-ws.c` | Pragma, Cache-Control, Host, Origin, Upgrade, Key, Protocol, Version |

---

## 2. Find the callback

1. The imports show `websockets.dll`, with `lws_create_context`, `lws_client_connect_via_info`, `lws_write`, `lws_callback_on_writable` and others imported by name.
2. A pointer to the string `ws-protocol` sits at `0x140041aa0`, which is the protocol table entry.
3. Reading that entry gives **callback `0x140006740`**, receive buffer 65,536 bytes, no per-session data.

## 3. Decode the reason dispatch

The callback subtracts 1 from the reason, checks it against 74, and jumps through a byte table and a dword table (`0x140007840` / `0x140007824`). Decoding both tables gives **6 handlers**. The other 69 reasons fall to a default that returns 0.

## 4. Enumerate every send

Every reference to the `lws_write` import (`0x140031350`) was listed: **11 call sites**. Each was traced to its buffer, length and write mode:

| # | Site | Message | Mode |
|---|---|---|---|
| 1 | `0x1400029e9` | `[LAL_P]…` hello | text |
| 2 | `0x140002cf5` | `[LS]` / `[LD]` / `[LE]` | text |
| 3 | `0x1400033d4` | `[PREPARE_*]` status | text |
| 4–6 | `0x1400064e8` / `6618` / `66c8` | `[START]` / `[FILENAME]` / `[FILESIZE]` | text |
| 7 | `0x14000688e` | `[PONG]]…` | text |
| 8 | `0x14000712b` | `[RRRRRR]` / `[VVVVVV]` / `[FFFFFF]` replies | text |
| 9 | `0x140007344` | `[HB]` | text |
| 10 | `0x140007445` | file chunk (≤ 64 KB) | **binary** |
| 11 | `0x140007516` | `[END]` | text |

## 5. Read the receive chain

The receive handler formats the incoming frame into a buffer and runs a **single linear chain** of prefix compares, in this order:

`[HB_OK]` → `[PING]` → `[CAL]` → `[DIS]` → `[CONFIRMED]` → `[CON]` → `[RESUME_OFFSET]` → `PPPP ` → `DDDD ` → exactly `1` → exactly `2` → `cd `

Nothing else is compared, which is why the command map is assessed as **highly likely** complete.

## 6. Field order: registers, not guesses

For every formatted message, the format arguments were read from the Windows x64 calling convention: the 4th argument in `r9`, then the stack from `[rsp+0x20]` up. Two results that differ from a naive read:

- **`[FFFFFF]%s|%s`** → listing first, path second.
- **`[LAL_P]` with 9 fields** → MAC, user, computer, OS, IP, uptime, connection type, adapter, VPN.

## 7. Emulate what is easy to get wrong

The PONG builder copies from the receive buffer at offset 5, which looks like an off-by-one. The path from `0x14000682f` to the `lws_write` call was emulated in **Unicorn**, with the receive buffer seeded and no network.

| Seeded input | Captured `lws_write` buffer |
|---|---|
| `[PING]` | `[PONG]]` (7 bytes, text) |
| `[PING]abc123` | `[PONG]]abc123` |

## 8. Timers from the main loop

The service loop around `lws_service` holds every constant that sets the timing: heartbeat at 5 s or more, liveness at 15 s or 120 s, connect timeout 12 s, and backoff caps of 5 and 12 s, plus a jitter bit taken from the process ID.

## 9. What static analysis could not settle

- The `Origin` header format. It depends on a libwebsockets context option that was not traced.
- Whether `[DIS]` makes the implant dormant until relogon.
- Anything the **server** does beyond what the client's parser accepts.
