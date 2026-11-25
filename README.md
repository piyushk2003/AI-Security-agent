# AI Security Agent

An AI-assisted security monitor for Windows machines. It scans the host for risky activity, streams the results to a live web dashboard, uses a large language model to explain the threats it finds, and offers one-click fixes for the most common problems.

## Description

AI Security Agent is a local security monitoring tool built with Python and FastAPI. It watches a computer from the inside and looks for the kinds of signs that often appear before or during an attack: unexpected services listening on the network, unknown or suspicious processes, connections to ports that malware commonly uses, newly connected USB drives, and files that look like ransomware or that have been dropped into sensitive folders.

The backend runs a set of independent scanners in parallel. The port scanner uses psutil to list open listening ports and flags dangerous services such as Telnet, FTP, and exposed remote-desktop ports. The process scanner checks running programs against high-risk and medium-risk name lists and notes scripting hosts such as PowerShell, wscript, and cscript when they are active. The network scanner looks at established outbound connections and marks those that reach suspicious remote ports or are started from shells such as cmd.exe, PowerShell, or bash. The USB scanner detects newly mounted drives and reports their size and usage. The filesystem watcher uses the watchdog library to monitor chosen folders in real time and raises warnings for suspicious extensions and filenames, with danger-level alerts for known ransomware extensions.

All scan results are collected into one report and pushed to the browser over a WebSocket connection, so the dashboard updates without refreshing. When threats change, the Groq API (running Llama 3.3 70B) receives a summary of the findings and returns a plain-language analysis with risk levels and recommended actions. This keeps the report readable for someone who is not a security specialist.

The auto-fixer can act on what the scanners find. It can close a dangerous port, terminate a suspicious process, delete a flagged file, disable the Telnet service, or start a Windows Defender quick scan. Fixes can be applied one at a time from the dashboard or in bulk. Some actions need administrator rights, and the code checks for this before attempting them.

A separate demo script simulates safe, harmless threats so the detections can be shown without running real malware. The project is intended for learning, lab work, and personal machine monitoring. It is not a replacement for a commercial endpoint protection product.

## Features

- Live dashboard with real-time updates over WebSocket
- Port, process, network connection, USB, and filesystem scanners
- Ransomware-style file extension and keyword detection
- AI-generated threat analysis through the Groq API
- Guided one-click and bulk fixes, with administrator checks
- Demo threat simulator for safe demonstrations

## Project structure

```
AI-Security-agent/
├── backend/
│   ├── main.py            # FastAPI app, scan loop, WebSocket and fix endpoints
│   ├── ai_analyzer.py     # Groq-based threat analysis
│   ├── auto_fixer.py      # Remediation actions
│   ├── demo_threats.py    # Safe threat simulator for demos
│   ├── check_models.py    # Lists available Google GenAI models
│   ├── requirements.txt
│   └── scanners/          # ports, processes, network, usb, filesystem
└── frontend/
    └── index.html         # Dashboard
```

## Setup

1. Install the dependencies:
   ```bash
   cd backend
   pip install -r requirements.txt
   ```
2. Create `backend/.env` and add your keys. Never commit this file.
   ```
   GROQ_API_KEY=your_groq_key
   GOOGLE_API_KEY=your_google_key   # only needed for check_models.py
   ```
3. Start the server:
   ```bash
   uvicorn main:app --reload
   ```
4. Open http://localhost:8000 in your browser.

## Safety notes

- The fix endpoints can terminate processes, delete files, and change system services. Run the agent only on machines you own or are authorised to manage, and review each suggested fix before applying it.
- The server is designed for local use. Do not expose it to a network without adding authentication.
- Demo threats are harmless, but the scanners will still flag them, so expect alerts while the demo runs.

## Built with

Python, FastAPI, Uvicorn, psutil, watchdog, Groq API (Llama 3.3 70B), and vanilla HTML/CSS/JavaScript.
