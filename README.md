# Opencode Unlimited v2.4.0

An automatic rate-limit rotator, multi-egress proxy router, and autonomous session auto-resume controller for **OpenCode Desktop** and **OpenCode CLI**.

![Windows](https://img.shields.io/badge/platform-Windows%2010%2F11-0078D4)
![Linux](https://img.shields.io/badge/platform-Linux%20%2F%20VPS-FCC624?logo=linux&logoColor=black)
![Build](https://img.shields.io/badge/build-prebuilt%20.exe-success)
![License](https://img.shields.io/badge/license-MIT-blue)

> Unofficial redistribution of [ShadowySphinX/Opencode-Unlimited-Limit-Bypass](https://github.com/ShadowySphinX/Opencode-Unlimited-Limit-Bypass-). All credit for the original implementation goes to **[@ShadowySphinX](https://github.com/ShadowySphinX)**. See [Credits](#credits).

---

## What it does

OpenCode enforces free-tier message limits against your inbound public IP (`/64` prefix on IPv6, `/32` on IPv4) and Cloudflare edge affinity. Opencode Unlimited:

- watches `opencode.log` in real time (**Zen Guard**),
- catches quota / rate-limit errors the moment they appear,
- cycles your network identity (fresh WARP IPv6 `/64`, Tor circuit, clean endpoint, or proxy-pool node),
- **automatically sends "continue"** to your running session so generation resumes without you being at the keyboard,
- **fails closed** if the tunnel drops, so your real IP never leaks.

---

## What's new in v2.4.0

| Feature | Description |
|---|---|
| **Auto-Resume (walk-away mode)** | After a rotation, sends `continue` to the active OpenCode session via keystroke injection and session DB tracking. |
| **WARP self-healing** | Detects `CF_HAPPY_EYEBALLS_MITM_FAILURE` on launch and repairs WARP keys in ~3 seconds. One-click **Fix WARP (Reset Keys)** button. |
| **Clean endpoints** | Ping-tests 27 unblocked WireGuard endpoints and locks the lowest-latency one. Auto-switches protocol for `:2408` nodes. |
| **Tor mode** | Starts portable Tor and sends `SIGNAL NEWNYM` for a fresh exit node in under a second. |
| **Proxy pool** | Custom SOCKS5/HTTP pools with automatic round-robin cycling on limit hits. |
| **Broadened detection** | Multi-pattern engine catches `QuotaExceeded`, `Free usage limit reached`, `rate_limit_exceeded`, `429 Too Many Requests`, and more. |
| **Isolated app-only mode** | Only OpenCode routes through the local proxy by default. Games, Discord, and browsers stay on your normal connection. |
| **VPS safe** | Strictly whitelists SSH and never hijacks default routes on cloud servers. |

---

## Requirements

- **Windows 10/11 64-bit** (desktop app) **or** a supported Linux distro for headless/VPS use
- **OpenCode:** Desktop app or CLI — [opencode.ai](https://opencode.ai)
- **Cloudflare WARP:** [1.1.1.1](https://one.one.one.one/) installed and registered (WARP modes only)
- **WebView2 Runtime:** [download](https://go.microsoft.com/fwlink/p/?LinkId=2124703)
  (preinstalled on most Windows 10/11; required on stripped/LTSC builds)

**Python is not required on Windows** — the repository ships a prebuilt executable. Building from source requires Python 3.10+.

---

## Quick start (Windows)

1. Download or clone this repository, then unblock the unsigned binary:

   ```powershell
   Unblock-File .\OpencodeUnlimited-Multi.exe
   ```

   Or right-click the file → **Properties** → tick **Unblock** → **Apply**.

2. Run **`OpencodeUnlimited-Multi.exe`**. If SmartScreen appears, choose **More info** → **Run anyway**.
3. Confirm the left-panel switches are **ON** (defaults):
   - **WARP Keeper**
   - **Zen Guard**
   - **Desktop App Proxy**
   - **Auto-Resume on Bypass**
4. Optionally click **Install / Repair System** (bottom left) to set up background tasks, TUN mode, and `PATH`.
5. Use OpenCode normally. On a limit hit, the app rotates your identity and auto-resumes the session.

### Daily use

- **Desktop app** — click **Launch OpenCode Desktop**. Existing chats and projects are untouched.
- **CLI** — click **Launch OpenCode CLI**, or run `opencode` in any terminal.

### When you hit a rate limit

1. Leave OpenCode and this controller running.
2. Zen Guard rotates your identity (~10–12 s for WARP, under 1 s for Tor/proxy modes) and sends `continue` automatically.
3. If auto-resume is off, re-send your message manually.

Need a new IP/edge on demand? Use **Rotate /64 Now (Hop)**, **Re-roll** on the Pinned Edge card, or **Fix WARP (Reset Keys)**.

---

## Egress modes

| Mode | Public identifier | Reset mechanism | Duration |
|---|---|---|---|
| **WARP (isolated)** | Cloudflare IPv6 `/64` | New SOCKS5 client on port 40000; fresh `/64` | ~10–12 s |
| **Clean endpoint** | Clean WireGuard IP + `/64` | Bind to an unblocked WireGuard node | ~10–12 s |
| **Tor network** | Tor exit IPv4 | `SIGNAL NEWNYM` → fresh 3-hop circuit | < 1 s |
| **Custom proxy pool** | Proxy server public IP | Advance to next node round-robin | Instant |
| **WARP (full TUN)** | Cloudflare IPv6 `/64` | Re-register client on virtual adapter | ~10–12 s |

Routing overview:

```
[ OpenCode Desktop / CLI ]
            │  HTTP / HTTPS via service.json
            ▼
   [ 127.0.0.1:41987 (Zen Proxy) ]
            │
   ┌────────┴──────────────────────────────────────────────┐
   ├── WARP isolated  → 127.0.0.1:40000 SOCKS5 → CF edge    │
   ├── WARP full TUN  → Wintun adapter → system-wide VPN    │
   ├── Tor            → 127.0.0.1:9050 → fresh circuit      │
   ├── Proxy pool     → node #1 → node #2 → ...             │
   └── Clean endpoint → WireGuard IP:2408 → unblocked edge  │
```

---

## Restricted networks (Russia, Iran, China, etc.)

If your ISP blocks Cloudflare WARP UDP traffic:

1. **Clean endpoints** — open the **Clean Endpoint** tab, click **Ping Test**, pick the lowest-latency node, then **Apply Endpoint**.
2. **Tor** — open the **Tor Network** tab and click **Start Portable Tor**. No WARP required.
3. **Custom proxy** — open the **Custom Proxy** tab, paste your Clash/V2Ray/Xray/sing-box address (`127.0.0.1:7890`, etc.), choose SOCKS5 or HTTP, then **Apply & Route**. Paste multiple proxies (comma, semicolon, or newline separated) to enable round-robin cycling.

---

## Linux / VPS setup

Run the automated installer:

```bash
bash install.sh
```

- Supports Debian, Ubuntu, Fedora, Arch Linux, NixOS, and openSUSE.
- Configures systemd units `opencode-unlimited-keeper` and `opencode-unlimited-guard`.
- Whitelists SSH port 22 and uses isolated proxy mode, so you never get locked out.

Uninstall with:

```bash
bash uninstall.sh
```

---

## Troubleshooting

**`CF_HAPPY_EYEBALLS_MITM_FAILURE` / WARP blocked**

Stale WARP keys or a protocol mismatch. The app self-heals on launch; you can also click **Fix WARP (Reset Keys)**. Switching between WireGuard (`:2408` endpoints) and MASQUE (official Anycast) is automatic.

**`ECONNREFUSED: Unable to connect`**

OpenCode is pointed at `127.0.0.1:41987`, but `OpencodeUnlimited-Multi.exe` is not running. Keep it open in the background.

**`opencode-cli.exe not found`**

Click **Install / Repair System**; the app scans for the binary and rewires `PATH`.

**Game latency / Discord voice drops during rotation**

WARP is in Full System TUN mode. Switch to **App Only (Isolated)** in the **WARP (IPv6)** tab.

**WARP stuck on "VERIFYING IPv6"**

System clock drift. Sync the clock: Windows Settings → **Time & Language** → **Date & Time** → **Sync now**.

**Blank or black window on launch**

Missing WebView2. Install the [Evergreen Runtime](https://go.microsoft.com/fwlink/p/?LinkId=2124703) and reopen. Common on AtlasOS, ReviOS, and LTSC installs.

**Diagnostics**

```powershell
.\OpencodeUnlimited-Multi.exe --doctor
```

```bash
bash scripts/doctor.sh
```

---

## Building from source

```bash
git clone https://github.com/ShadowySphinX/Opencode-Unlimited-Limit-Bypass-.git
cd Opencode-Unlimited-Limit-Bypass-
pip install -r requirements.txt
```

Build the standalone Windows executable:

```powershell
powershell -ExecutionPolicy Bypass -File .\build_multi_exe.ps1
```

Source lives in `src/` (`avatar_ui.py`, `zen_guard.py`, `zen_proxy.py`, `warp_keeper.py`, `tor_manager.py`, `auto_resume.py`, `proxy_chain.py`, `clean_endpoint.py`, `doctor.py`, and the UI in `src/ui/index.html`).

---

## Disclaimer

This project is provided for educational and local-development purposes only. It modifies network routing and system firewall/task configuration on your own machine. Use it at your own risk and in line with the terms of service of any service you connect to. The authors accept no liability for account actions, data loss, or system changes resulting from its use.

---

## Credits

This project builds on the original **Opencode Unlimited** work by
**[@ShadowySphinX](https://github.com/ShadowySphinX)** —
[ShadowySphinX/Opencode-Unlimited-Limit-Bypass](https://github.com/ShadowySphinX/Opencode-Unlimited-Limit-Bypass-).

All credit for the original implementation goes to the upstream author. This
repository is an unofficial redistribution and does not claim authorship of the
original work. Please do not rebrand the project or remove the original credits.

## License

MIT License. See [LICENSE](LICENSE) for details.
