# LogicSec C2 Framework | Hot-Loaded PSX Modules

> *Lightweight, PowerShell-based C2 with a built-in web dashboard & hot-loadable PSX modules.*

---

## ⚠️ OPSEC & Source Code Notice

To preserve the operational security of custom C2 infrastructure and avoid signatured detection, the full source code for this project is **not** publicly available.

This repository serves as a technical showcase of the architecture, capabilities, and red-team use cases.

---

## 💡 Why LogicSec C2?

Traditional C2 frameworks are either:

- **Heavy & complex** — requiring extensive setup, agents, and infrastructure.
- **Heavily signatured** — EDR/AV products detect out-of-the-box payloads.
- **Rigid** — adding new post-exploitation modules often requires recompiling or deep C++ knowledge.

**LogicSec C2** was built to solve these problems. It provides:

- A **lightweight PowerShell TeamServer** that runs on any Windows machine.
- A **full-featured web dashboard** for console access, payload generation, and module management.
- **Hot-loadable PSX modules** — write new post-exploitation capabilities in plain PowerShell, drop them into a folder, and dispatch them instantly to any beacon.
- **Stealth-aware architecture** — uses native HTTP.SYS for listener registration, supports variable obfuscation, and keeps the beacon small.

---

## 🏗️ Architecture Overview

```text
┌─────────────────┐       ┌─────────────────────────────────┐
│   Dashboard     │       │          TeamServer             │
│  (Embedded UI)  │◄──────┤  (augsrv.ps1 - HttpListener)  │
│   HTML/CSS/JS   │       │                                 │
└─────────────────┘       └───────────────┬─────────────────┘
                                          │
                                ┌─────────▼─────────┐
                                │    PSX Modules    │
                                │    (.psx files)   │
                                │   Hot-loaded via  │
                                │    #META headers  │
                                └─────────┬─────────┘
                                          │
                                ┌─────────▼─────────┐
                                │     Beacons       │
                                │      (PS1)        │
                                │ Polling / Task    │
                                │ Execution Loop    │
                                └───────────────────┘
```

### Components

- **TeamServer** (`augsrv.ps1`) — a single PowerShell script that starts an `HttpListener`, serves the dashboard, and handles all REST API calls.
- **Beacon** — a generated PowerShell script that registers itself, polls for tasks, executes them, and posts results back with jitter.
- **Dashboard** — a self-contained single-page application (SPA) that lets you monitor beacons, run commands, dispatch modules, and generate new payloads.
- **PSX Module System** — the heart of the framework. Modules are `.psx` files (plain PowerShell with metadata headers) that are dynamically loaded and dispatched to beacons as base64-encoded scripts.

---

## 🚀 Key Features

| Feature | Description |
|---|---|
| **Asynchronous TeamServer** | Non-blocking HTTP listener that handles multiple beacons concurrently. |
| **Real-Time Dashboard** | Live beacon status (active/slow/dead), command history, and per-beacon consoles. |
| **PSX Module System** | Write post-exploitation modules in PowerShell, add `#META` headers for metadata, and hot-load them without restarting the server. |
| **Parameterized Modules** | Define `#META params: Target, Ports` — the dashboard prompts for values when dispatching. |
| **Payload Generator** | Build custom beacons with variable obfuscation and selectable output formats (Raw PS1, IEX cradle, EncodedCommand). |
| **Built-In Recon Commands** | Execute native commands like `tasklist`, `ipconfig`, `netstat`, and `systeminfo` via `shell` tasks. |
| **Module Manager** | Create, edit, unload, and reload modules directly from the dashboard. |
| **Stealth Options** | Variable name obfuscation, `-ExecutionPolicy Bypass`, and `-WindowStyle Hidden` for execution. |

---

## 🧩 PSX Module System

The **PSX (PowerShell Extensions)** system is what makes this C2 truly extensible.

### File Format

- Files **must** end with `.psx` and reside in the `modules/` folder.
- The filename stem becomes the module name in the UI.

For example:

```text
portscanner.psx → portscanner
```

### Metadata Header (`#META`)

Add these comment lines at the top of your module. They are **stripped before dispatch** and never reach the beacon.

```powershell
#META description: Scan TCP ports on a target subnet
#META category: Recon
#META author: LogicSec
#META params: Target, Ports, Threads, Timeout
```

### Supported Categories

Categories affect the appearance of module buttons in the dashboard:

| Category | UI Color |
|---|---|
| `Recon` | Cyan |
| `Persistence` | Orange |
| `Exfil` | Red |
| `Lateral` | Light blue |
| `Custom` | Default green |

### Parameterized Dispatch

When you click a module button in the console:

1. The dashboard parses the `params` field.
2. A modal opens with an input field for each parameter.
3. The supplied values are injected as PowerShell variables (`$Target`, `$Ports`, etc.).
4. The module code is executed on the selected beacon.

### Sample Module

```powershell
#META description: Pings a host and returns latency
#META category: Recon
#META params: TargetHost, Count

$target = if ($TargetHost) { $TargetHost } else { "localhost" }
$count  = if ($Count) { [int]$Count } else { 4 }

Test-Connection -ComputerName $target -Count $count | Out-String
```

### Hot Reload

- Drop a new `.psx` file into the `modules/` folder.
- Click **RELOAD** in the **MODULES** tab.
- The server re-reads the directory without restarting.
- Modules already dispatched to beacons are unaffected.

---

## 🖥️ Dashboard Walkthrough

### 1. Console View

- Select a beacon from the left sidebar.
- Run arbitrary PowerShell via the input box.
- Click a **PSX module** button to dispatch a module.
- View real-time command history and output.
- Supports screenshots via base64-encoded images.

### 2. Payload Generator

- Configure the callback address, port, sleep interval, and output format.
- Toggle variable obfuscation.
- Generate a one-liner or download the `.ps1` payload.

### 3. Modules Tab

#### Loaded Modules

Browse all loaded PSX modules, view their metadata and code, and dispatch them directly.

#### New Module

Create modules directly in the browser and save them to disk. Modules are automatically reloaded after saving.

### 4. Docs

Built-in reference documentation for:

- PSX module format
- Metadata syntax
- Parameterized modules
- Module development best practices

---

## 📸 Screenshots

> *Below are demonstration screenshots of the dashboard in action.*

### Dashboard — Beacon Check-In & Console

![Dashboard — Beacon Check-In & Console](https://screenshot%25202026-09-06%2520185240.png/)

### Payload Generator

![Payload Generator](https://screenshot%25202026-09-06%2520184950.png/)

### PSX Module Manager

![PSX Module Manager](https://screenshot%25202026-09-06%2520185009.png/)

### Parameterized Module Dispatch

![Parameterized Module Dispatch](https://screenshot%25202026-09-06%2520185258.png/)

### TeamServer Startup

![TeamServer Startup](https://screenshot%25202026-09-06%2520184644.png/)

---

## 🔧 Deployment & Usage

### Prerequisites

- Windows OS with **PowerShell 5.1+**
- Administrator privileges recommended for port binding
- Alternatively, use a high port (`>1024`)

### Starting the Server

```powershell
# Default port 9999, modules folder = .\modules
.\augsrv.ps1

# Custom port and module directory
.\augsrv.ps1 -Port 8443 -ModuleDir "C:\c2\my_modules"
```

The server will:

1. Create the module directory if it doesn't exist.
2. Load all `.psx` files.
3. Start the `HttpListener` on `http://localhost:<Port>/`.
4. Open the dashboard automatically, or allow navigation to the configured listener URL.

### Generating a Beacon

1. Open the dashboard and navigate to the **PAYLOAD_GEN** tab.
2. Set the callback address.
3. Choose the desired output format.
4. Click **GENERATE PAYLOAD**.
5. Deploy the generated payload in your authorized test environment.

### Tasking a Beacon

- Once a beacon checks in, select it from the sidebar.
- Type a PowerShell command into the input box and click **EXECUTE**.
- Alternatively, click a PSX module button to dispatch the module.

---

## 🛡️ OPSEC Considerations

- **No persistent disk footprint** — modules are loaded into memory and dispatched as base64.
- **Customizable obfuscation** — variable names can be randomized in generated payloads.
- **Low-signal egress** — HTTPS support through a reverse proxy such as ngrok/stunnel, combined with sleep and jitter.
- **Hot-loading** — modules can be added or removed without restarting the TeamServer, avoiding additional process creation events on the TeamServer side.

---

## 🤝 Contributing & Collaboration

This project is maintained as a **closed-source technology demonstrator**.

For collaboration, authorized pentesting engagements, or private licensing, please reach out via [GitHub](https://github.com/LogicSecActivated).

---

## 📜 License & Credits

- Authored and maintained by **LogicSec**.
- All rights reserved.
- Unauthorized redistribution of core evasion primitives and the C2 framework is strictly prohibited.
