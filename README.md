# Opencode Unlimited

A Windows desktop controller for **OpenCode** that keeps you working by routing OpenCode through **Cloudflare WARP IPv6** and automatically rotating your `/64` block whenever the free usage limit is hit.

![Windows](https://img.shields.io/badge/platform-Windows%2010%2F11-0078D4)
![Build](https://img.shields.io/badge/build-prebuilt%20.exe-success)
![License](https://img.shields.io/badge/license-MIT-blue)

> Unofficial redistribution of [ShadowySphinX/Opencode-Unlimited-Limit-Bypass](https://github.com/ShadowySphinX/Opencode-Unlimited-Limit-Bypass-). See [Credits](#credits).

---

## What it does

OpenCode's free limit is tracked against your IPv6 `/64` block and the Cloudflare edge you connect through. Opencode Unlimited:

- routes OpenCode traffic through **WARP IPv6** (TUN mode),
- pins a specific **Cloudflare edge**, with one-click re-roll,
- watches `opencode.log` in the background (**Zen Guard**),
- grabs a fresh WARP `/64` in **~12 seconds** when a limit hit appears,
- **fails closed** if WARP drops, so your real IP never leaks.

---

## Features

| Feature | Description |
|---|---|
| **WARP Keeper** | Reconnects WARP automatically if the tunnel drops. |
| **Zen Guard** | Tails `opencode.log` and triggers a `/64` rotation on limit errors. |
| **Desktop App Proxy** | Wires the OpenCode Desktop daemon to the local proxy. |
| **Pinned Edge** | Locks a Cloudflare edge for stable latency; re-roll anytime. |
| **Firewall Kill-Switch** | Blocks `opencode-cli.exe` from using your direct connection. |
| **Install / Repair System** | One click sets up background tasks, TUN mode, and `PATH`. |
| **Health Check** | `OpencodeUnlimited-Avatar.exe --doctor` for diagnostics. |

---

## Requirements

- **OS:** Windows 10 or 11, 64-bit
- **OpenCode:** Desktop app or CLI — [opencode.ai](https://opencode.ai)
- **Cloudflare WARP:** [1.1.1.1](https://one.one.one.one/) installed and registered
- **WebView2 Runtime:** [download](https://go.microsoft.com/fwlink/p/?LinkId=2124703)
  (preinstalled on most Windows 10/11; required on stripped/LTSC builds)

**Python is not required.** This repository ships a prebuilt executable; source code and build scripts are not included here.

---

## Getting started

1. Download `OpencodeUnlimited-Avatar.exe` from this repository (or the **Releases** tab).
2. Windows flags the unsigned binary, so unblock it first:

   ```powershell
   Unblock-File .\OpencodeUnlimited-Avatar.exe
   ```

   Or right-click → **Properties** → tick **Unblock** → **Apply**.
3. Run the executable. If SmartScreen appears, choose **More info** → **Run anyway**.
4. Click **Install / Repair System** (bottom left) and give it a few seconds.
5. Confirm the status indicators are green:
   - badge reads `PROTECTED • WARP IPv6`
   - public `/64` card shows `CF Verified`
   - **WARP Keeper**, **Zen Guard**, and **Desktop App Proxy** are all **ON**
6. Optional: enable **Firewall Kill-Switch** and accept the UAC prompt.

---

## Daily use

- **Desktop app** — click **Launch OpenCode Desktop**. Existing chats and projects are untouched.
- **CLI** — click **Launch OpenCode CLI**, or run `opencode` in any terminal.

### When you hit "Free usage limit reached"

1. Leave everything open. *(If it doesn't recover, close OpenCode and reopen it.)*
2. Wait ~10–15 seconds — Zen Guard detects the error and rotates WARP. The badge switches to `ROTATING /64...`.
3. Re-send your message.

Need a new IP/edge manually? Use **Rotate /64 Now (Hop)** or **Re-roll** on the Pinned Edge card.

---

## Troubleshooting

**Blank or black window on launch**

Missing WebView2. Install the [Evergreen Runtime](https://go.microsoft.com/fwlink/p/?LinkId=2124703) and reopen the app. Common on AtlasOS, ReviOS, and LTSC installs.

**No internet unless the VPN is connected**

The Firewall Kill-Switch is blocking `opencode-cli.exe`. Toggle it **OFF** in the app, or disconnect via the WARP toggle.

**Something else**

Run the built-in diagnostic:

```powershell
.\OpencodeUnlimited-Avatar.exe --doctor
```

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
original work.

## License

MIT License. See [LICENSE](LICENSE) for details.
