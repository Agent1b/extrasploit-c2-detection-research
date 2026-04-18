# 🔬 Extrasploit — Custom RAT Proof of Concept Research

> ⚠️ **DISCLAIMER:** This project was developed strictly for educational and research purposes. No malicious use is intended or endorsed. The tool was tested exclusively in an isolated lab environment with no connection to external networks. This repository contains **documentation only** — no source code or binaries are provided.

---

## 📋 Project Overview

A custom-built Remote Access Tool (RAT) developed from scratch as a proof-of-concept to study:
- Command & Control (C2) communication architectures with encrypted protocols
- Evasion techniques against Windows 11 security features
- Persistence mechanisms and their detection opportunities
- Cross-language development challenges (Python ↔ C#)
- Real-time remote desktop streaming over encrypted channels

**Goal:** Understand how real-world threat actors build and operate C2 infrastructure in order to develop better detection strategies and defensive controls.

**Key Achievement:** Fully functional C2 framework with 20+ commands, encrypted communications, remote desktop with input control, and cross-compiled from Linux — all built from scratch without using existing frameworks like Metasploit or Cobalt Strike.

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    ATTACKER — Kali Linux                         │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │              Extrasploit Controller (Python 3.13)       │     │
│  │                                                         │     │
│  │  main.py ──► server.py ──► commands.py                  │     │
│  │  (CLI)       (TCP+AES)    (dispatch 20+ commands)       │     │
│  │     │            │              │                        │     │
│  │  crypto.py   builder.py   desktop_viewer.py             │     │
│  │  (AES-256)   (cross-      (Tkinter remote               │     │
│  │              compile)      desktop + input)              │     │
│  └────────────────────────────────────────────────────────┘     │
│                              │                                   │
└──────────────────────────────┼───────────────────────────────────┘
                               │
                    ┌──────────┴──────────┐
                    │  AES-256-CBC        │
                    │  4-byte length      │
                    │  prefix framing     │
                    │  JSON payloads      │
                    │  Fresh IV per msg   │
                    └──────────┬──────────┘
                               │
┌──────────────────────────────┼───────────────────────────────────┐
│                    TARGET — Windows 11                            │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │              Zeus Agent (C# .NET 6.0 — single .exe)     │     │
│  │                                                         │     │
│  │  Agent.cs ──► Commands.cs ──► Feature Modules:          │     │
│  │  (connect,    (dispatch       ScreenCapture.cs           │     │
│  │  reconnect,   dictionary      DesktopStream.cs           │     │
│  │  encrypt)     26 handlers)    Keylogger.cs               │     │
│  │                               AudioCapture.cs            │     │
│  │  Crypto.cs    Logger.cs       WebcamCapture.cs           │     │
│  │  (AES-256)    (file-only,     FileOperations.cs          │     │
│  │               no console)     ShellExecutor.cs           │     │
│  │                               Persistence.cs             │     │
│  │  Program.cs                   RegistryEditor.cs          │     │
│  │  (FreeConsole                 ProcessManager.cs           │     │
│  │  + silent)                                               │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

**Components:**
- **C2 Server (Python):** Async TCP listener using asyncio, handles multiple concurrent agent sessions, routes 26+ message types, manages request-response with futures and fire-and-forget events
- **Agent/Implant (C#):** Single-file .NET 6.0 executable (~67MB) cross-compiled from Linux, communicates over AES-256-CBC encrypted TCP, auto-reconnects on disconnect, runs completely invisible on Windows 11
- **Console/Operator Interface (Python):** Interactive CLI built with prompt_toolkit, tab completion for all commands, session management with stable IDs, real-time remote desktop viewer

---

## ⚙️ Features Implemented

| Feature | Description | Technique | Status |
|---------|-------------|-----------|--------|
| **Encrypted C2 Comms** | All traffic AES-256-CBC with fresh IV per message | Custom protocol with 4-byte length prefix, cross-language crypto (Python ↔ C#) | ✅ Implemented |
| **Remote Desktop** | Live screen streaming with full mouse + keyboard control | GDI+ CopyFromScreen → JPEG → base64 → TCP stream, Win32 SetCursorPos/mouse_event/keybd_event | ✅ Implemented |
| **Interactive Shell** | Execute commands on target | Process.Start with redirected stdin/stdout | ✅ Implemented |
| **Screenshot Capture** | On-demand target screen capture | Graphics.CopyFromScreen → PNG → base64 | ✅ Implemented |
| **Webcam Capture** | Take photos from target webcam | AForge.Video.DirectShow | ✅ Implemented |
| **Keylogger** | Record keystrokes with low-level hooks | SetWindowsHookEx (WH_KEYBOARD_LL) on dedicated STA thread | ✅ Implemented |
| **Audio Capture** | Record from target microphone | NAudio WaveInEvent → WAV → base64 | ✅ Implemented |
| **File Transfer** | Upload/download files bidirectionally | Base64 encoding over encrypted channel | ✅ Implemented |
| **Registry Editor** | Query and modify Windows registry remotely | Microsoft.Win32 RegistryKey API | ✅ Implemented |
| **Persistence** | Survive reboots, auto-start on login | HKCU Run key + full file copy to AppData | ✅ Implemented |
| **Process Manager** | List and kill remote processes | Process.GetProcesses / Process.Kill | ✅ Implemented |
| **Directory Browsing** | ls, pwd, cd, cat on target filesystem | System.IO Directory/File operations | ✅ Implemented |
| **Network Scanner** | Controller-side port scanner with banner grabbing | Async socket scanning with semaphore-bounded concurrency | ✅ Implemented |
| **Silent Execution** | Zero visible UI on Windows 11 | PE header patching (subsystem CONSOLE→GUI) + FreeConsole() | ✅ Implemented |
| **Auto-Reconnection** | Agent reconnects if connection drops | 10-second retry loop with fresh TcpClient per attempt | ✅ Implemented |
| **Cross-Compilation** | Build Windows agent from Kali Linux | dotnet publish with 9 flags + PE header patching | ✅ Implemented |
| **Session Management** | Handle multiple agents, reuse IDs on reconnect | Same IP reuses session ID, futures cancelled on disconnect | ✅ Implemented |
| **Anti-VM Detection** | Detect sandbox/VM environments | Registry, MAC, WMI, hardware checks with scoring system | ⬚ Planned |
| **Code Obfuscation** | Make agent harder to reverse-engineer | String encryption + ConfuserEx post-build | ⬚ Planned |
| **Domain Fronting** | Hide C2 behind legitimate CDN | HTTPS with CDN routing by Host header | ⬚ Planned |
| **Privilege Escalation** | Elevate agent privileges | UAC bypass (fodhelper, eventvwr) + token impersonation | ⬚ Planned |
| **Lateral Movement** | Spread within network | PsExec-style, WMI, pass-the-hash | ⬚ Planned |

---

## 🔒 Evasion Techniques Studied

### 1. PE Header Patching for Silent Execution
**What:** Windows 11's new Terminal application intercepts ALL processes with PE subsystem = CONSOLE before Main() even runs, creating a visible window that can't be hidden by code.  
**How I implemented it:** Post-build Python script reads the PE header, locates the subsystem field at PE_OFFSET + 0x5C, and overwrites the value from 3 (CONSOLE) to 2 (WINDOWS_GUI). This is the OS-level control — Windows never creates a console window.  
**Detection method:** Static analysis of PE headers reveals a GUI subsystem with no actual GUI — suspicious for a 67MB executable. Sysmon Event ID 1 (Process Creation) still logs the execution even without a visible window.

### 2. AES-256-CBC Encrypted Communications
**What:** All C2 traffic is encrypted with AES-256-CBC using a pre-shared key, making content inspection impossible without the key.  
**How I implemented it:** Shared 256-bit key stored in config (hex encoded). Each message gets a random 16-byte IV prepended to the ciphertext. Length-prefix framing (4-byte LE) ensures complete message delivery over TCP. Identical encryption implementations in Python (pycryptodome) and C# (System.Security.Cryptography).  
**Detection method:** While content is invisible, traffic patterns are detectable: regular heartbeat intervals (30 seconds), consistent 4-byte headers before encrypted blobs, persistent long-lived TCP connections to non-standard ports. IDS rules can flag high-entropy payloads with regular timing.

### 3. Console-Free Agent with File-Only Logging
**What:** The agent produces zero visible output — no console window, no error dialogs, no tray icons.  
**How I implemented it:** FreeConsole() as first line of Main(), zero Console.WriteLine calls throughout the codebase, Logger class writes to file only (and only when running from AppData install directory — not during initial execution). Combined with PE header patching and OutputType=WinExe.  
**Detection method:** Task Manager shows the process (no rootkit), EDR tools see the process creation, and the agent leaves a log file in AppData that's a clear IOC if the install path is known.

### 4. Persistence via Registry Run Key
**What:** Agent survives reboots by registering in the Windows startup registry.  
**How I implemented it:** Copies ALL build files (not just .exe — .NET needs runtime config) to `%LOCALAPPDATA%\WindowsSystemUpdate\`, then adds a Run key at `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`. Uses per-file try/catch during copy to handle locked files. Uses AppDomain.CurrentDomain.BaseDirectory for paths (not relative — relative paths resolve to system32 when started from Run key).  
**Detection method:** Sysmon Event ID 13 (Registry Value Set) detects the Run key creation. Autoruns (Sysinternals) lists all startup entries. The AppData folder name "WindowsSystemUpdate" is a suspicious non-Microsoft folder in a standard location.

### 5. Auto-Reconnection with Session Persistence
**What:** If the connection drops, the agent automatically reconnects and the controller reuses the same session ID.  
**How I implemented it:** Agent's RunAsync creates a NEW TcpClient for each attempt (disposed sockets can't reconnect), retries every 10 seconds indefinitely. Finally block closes all resources + stops background features. Controller detects same-IP reconnection and reuses the old session ID, removing the stale session first. All pending futures are cancelled on disconnect to prevent hanging awaits.  
**Detection method:** Regular reconnection attempts at fixed intervals (10 seconds) are a strong C2 beaconing indicator. Network flow analysis shows repeated TCP SYN packets to the same destination at consistent intervals.

---

## 🛡️ Detection & Defense — Blue Team Perspective

This is the most important section. Building the RAT taught me how to **detect and defend** against these techniques:

| Attack Technique | MITRE ATT&CK ID | Detection Method | Tools |
|-----------------|------------------|------------------|-------|
| Encrypted Reverse Shell | T1573.001 (Encrypted Channel: Symmetric) | Detect persistent outbound TCP to non-standard ports with high-entropy payloads | Zeek, Suricata, Wireshark |
| Registry Run Key Persistence | T1547.001 (Boot/Logon Autostart: Registry Run Keys) | Monitor registry modifications to Run/RunOnce keys | Sysmon (Event ID 13), Autoruns |
| C2 Beaconing | T1071.001 (Application Layer Protocol: Web) | Detect regular-interval callbacks (30-second heartbeat pattern) | Network flow analysis, RITA |
| Screen Capture | T1113 (Screen Capture) | Monitor for GDI+ CopyFromScreen API calls at unusual frequency | EDR behavioral detection |
| Keylogging | T1056.001 (Input Capture: Keylogging) | Detect SetWindowsHookEx calls for WH_KEYBOARD_LL | Sysmon (Event ID 8), EDR hook detection |
| File Exfiltration | T1041 (Exfiltration Over C2 Channel) | Monitor for large base64-encoded data transfers over encrypted channel | DLP, Network volume analysis |
| Audio Capture | T1123 (Audio Capture) | Monitor for microphone access by non-standard applications | Windows privacy settings, EDR |
| Video Capture | T1125 (Video Capture) | Monitor for webcam access by unsigned executables | Windows privacy settings, EDR |
| Process Discovery | T1057 (Process Discovery) | Correlate process enumeration with unauthorized programs | Behavioral analysis |
| Ingress Tool Transfer | T1105 (Ingress Tool Transfer) | Monitor for file writes to AppData by processes started from unusual locations | Sysmon (Event ID 11), file integrity monitoring |

### Recommended Defensive Controls

1. **Network Segmentation** — Restrict outbound connections from workstations to only required destinations and ports
2. **EDR with Behavioral Detection** — Signature-based AV misses custom tools; behavioral analysis detects the actions (hook installation, screen capture, registry modification)
3. **Sysmon Deployment** — Events 1 (Process Create), 3 (Network Connect), 7 (Image Load), 8 (CreateRemoteThread), 11 (FileCreate), 13 (RegistryEvent) cover all IOCs
4. **Network Traffic Analysis** — Tools like RITA detect beaconing patterns even in encrypted traffic
5. **Application Whitelisting** — Prevent execution of unsigned/unknown executables
6. **Registry Monitoring** — Alert on any modification to known persistence locations
7. **Least Privilege** — Standard user accounts limit persistence and lateral movement options

---

## 🧪 Lab Environment

```
┌────────────────────────────────────────────────┐
│              VMware Workstation                  │
│                                                  │
│  ┌──────────────────┐  ┌──────────────────────┐│
│  │   Kali Linux      │  │    Windows 11 Pro    ││
│  │   (Controller)    │  │    (Target Agent)    ││
│  │                   │  │                      ││
│  │                   │  │                      ││
│  │                   │  │                      ││
│  │                   │  │                      ││
│  │  Python 3.13      │  │  .NET Runtime 6.0   ││
│  │  prompt_toolkit   │  │  Agent.exe (67MB)   ││
│  │  pycryptodome     │  │  Silent execution   ││
│  │  Pillow (viewer)  │  │                      ││
│  └──────────────────┘  └──────────────────────┘│
│                                                  │
│  Network: Host-only (NO internet access)         │
│  Snapshots: Clean state restored between tests   │
│  AV: Windows Defender enabled during testing     │
└────────────────────────────────────────────────┘
```

- **Attacker Machine:** Kali Linux 2024.x, Python 3.13, .NET SDK 6.0 (for cross-compilation)
- **Target Machine:** Windows 11 Pro 25H2, .NET Runtime 6.0, Windows Defender enabled
- **Network:** VMware host-only adapter — fully isolated, no internet access
- **Virtualization:** VMware Workstation with snapshots for clean test states

---

## 📸 Screenshots & Demo

<!-- SCREENSHOTS TO TAKE — see instructions below each entry -->

| Screenshot | Description |
|-----------|-------------|
| ![Controller CLI](screenshots/01_controller_cli.png) | Extrasploit controller with interactive CLI, session management, and tab completion |
| ![Agent Connect](screenshots/02_agent_connect.png) | Agent connecting back to controller — encrypted handshake established |
| ![Remote Desktop](screenshots/03_remote_desktop.png) | Live remote desktop streaming with mouse and keyboard control active |
| ![Screenshot Capture](screenshots/04_screenshot.png) | On-demand screenshot capture from the target machine saved to disk |
| ![Keylogger](screenshots/05_keylogger.png) | Keylogger capture showing intercepted keystrokes from the target |
| ![Network Scanner](screenshots/06_scanner.png) | Built-in port scanner with service detection and banner grabbing |
| ![Build System](screenshots/07_build.png) | Cross-compiling agent from Kali Linux → Windows single-file .exe |
| ![Silent Agent](screenshots/08_silent_agent.png) | Agent running in Task Manager with NO visible window on Windows 11 |
| ![Persistence](screenshots/09_persistence.png) | Registry-based persistence — Run key visible in regedit |
| ![Sessions](screenshots/10_sessions.png) | Multiple agent sessions with reconnection and status indicators |
| ![Wireshark](screenshots/11_wireshark.png) | Encrypted C2 traffic captured in Wireshark showing length-prefix framing |
| ![File Transfer](screenshots/12_file_transfer.png) | Bidirectional file transfer — download from target to controller |

---

## 📚 What I Learned

### Technical Insights

1. **Cross-language cryptography is harder than expected** — Python and C# handle key derivation, padding, and encoding differently. The critical lesson: hex key strings must be decoded with `bytes.fromhex()` / `Convert.FromHexString()`, never encoded as UTF-8. This single mistake produces a completely different (wrong) AES key.

2. **TCP is a stream, not a message protocol** — Without explicit message framing (4-byte length prefix), messages fragment unpredictably. This is the most common source of "mysterious" network bugs and is fundamental to any reliable C2 architecture.

3. **Windows 11 changed the silent execution game** — The new Terminal app creates a console window before Main() runs. Traditional techniques (ShowWindow, FreeConsole) are too late. PE header patching at the binary level is now required — a technique that works at the OS level before any application code executes.

4. **GUI + async = threading discipline** — Running a real-time desktop viewer alongside an async network stack requires strict threading boundaries. Tkinter must own its thread, asyncio owns another, and thread-safe queues are the only safe bridge. Getting the close/cleanup order wrong causes crashes that are nearly impossible to debug without understanding the threading model.

5. **Fire-and-forget vs request-response is a fundamental protocol design decision** — Mouse/keyboard events at 30+ events/second cannot use futures (the latency is unacceptable). The distinction between commands (request-response with futures) and events (fire-and-forget, return null) is essential for responsive C2 operation.

6. **Defense in depth is real** — No single control stops a custom RAT. It takes network monitoring + endpoint detection + behavioral analysis + registry monitoring + application control working together. Building the RAT revealed exactly which combination of controls would have stopped each phase of the attack chain.

### Offensive Security Insights

7. **Custom tools bypass signature-based detection** — Windows Defender never flagged the agent because it has no known signature. This demonstrates why behavioral detection and EDR solutions are critical — signatures only catch known threats.

8. **Persistence is trivially easy, detection is the real challenge** — Adding a registry Run key is one line of code. Detecting it reliably across an enterprise with thousands of legitimate startup entries requires sophisticated filtering and baselining.

9. **The network never lies** — Even with encrypted traffic, the C2 channel has detectable patterns: regular heartbeat intervals, persistent connections, consistent packet structure. Network traffic analysis is the most reliable detection layer for custom C2 tools.

---

## 🔮 Future Research

Areas for continued exploration:

| Topic | MITRE ATT&CK | Research Value |
|-------|--------------|----------------|
| DNS Tunneling | T1071.004 | Ultra-covert C2 — most firewalls allow DNS |
| Domain Fronting | T1090.004 | Hiding C2 behind legitimate CDN infrastructure |
| In-Memory Execution | T1620 | Load .NET assemblies without touching disk |
| Process Injection | T1055 | Migrate agent into legitimate processes |
| AMSI Bypass | T1562.001 | Circumvent PowerShell security features |
| Staged Payloads | T1104 | Minimal loader → encrypted second stage |
| Anti-VM Detection | T1497 | Sandbox evasion with scoring system |
| Pass-the-Hash | T1550.002 | Lateral movement with NTLM hashes |

---

## 🔗 Related Projects

- [Custom Payload Research](link-to-payload-repo) — Companion payload development and AV evasion analysis
- [Home Security Lab](link-to-homelab-repo) — The isolated VMware environment where all testing was conducted

---

## 📬 Contact

**Michael Baazov**  
[LinkedIn](https://www.linkedin.com/in/michael-baazov-87417823b/) | [GitHub](https://github.com/Agent1b) | michbz@proton.me

---

*This project is part of my ongoing cybersecurity research. Understanding how attackers operate is the first step to building effective defenses. Every technique documented here includes its corresponding detection method — because offense informs defense.*
