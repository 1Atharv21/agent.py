# 🤖 Qwen Local Desktop Control Agent

A production-grade, secure local desktop agent designed to manage the lifecycle (**OPEN**, **CLOSE**, **STATUS**, **CONFIGURE**) of your locally installed Qwen application across **Windows**, **macOS**, and **Linux**.

---

## 🌟 Architecture & Security Model

```text
User Input ("Open Qwen", "Close Qwen")
       │
       ▼
 ┌──────────────┐
 │ Intent Parser│ ──► (Local LLM API or Deterministic Rules)
 └──────┬───────┘
        │ Strict Allowlist Mapping
        ▼
 ┌─────────────────┐
 │ Action Allowlist│ (OPEN_QWEN, CLOSE_QWEN, STATUS_QWEN, CONFIGURE_QWEN)
 └──────┬──────────┘
        │
        ▼
 ┌─────────────────┐
 │ Qwen Controller │
 └──────┬──────────┘
        │
        ▼
 ┌─────────────────┐
 │ Process Manager │ ──► (OS Process Control via psutil & Popen)
 └─────────────────┘
```

### 🔒 Security Principles
1. **No Arbitrary Shell Command Execution**: Language models or user inputs **never** generate raw shell execution strings. Intent parser outputs are mapped strictly to pre-validated allowlisted enum actions (`OPEN_QWEN`, `CLOSE_QWEN`, `STATUS_QWEN`, `CONFIGURE_QWEN`).
2. **Safe Subprocess Spawning**: Applications are started with `shell=False` to prevent shell injection attacks.
3. **Targeted Process Control**: Process matching checks exact executable paths and process names to ensure unrelated system processes are never touched or killed.
4. **Path Sanitization**: Executable paths are validated for existence, regular file properties, and checked against a system executable blacklist before execution.
5. **Data Privacy**: Structured logs sanitize passwords, API keys, and sensitive fields automatically.

---

## 🚀 Quick Start & Installation

### 1. Prerequisites
- **Python 3.8+** installed on your system.

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

---

## 💻 Usage Instructions

### Command Line Interface (CLI)

#### Natural Language Commands
```bash
# Open / Launch Qwen
python -m agent.main "Open Qwen"
python -m agent.main "Launch Qwen"
python -m agent.main "Start Qwen"

# Close / Quit Qwen
python -m agent.main "Close Qwen"
python -m agent.main "Quit Qwen"
python -m agent.main "Stop Qwen"

# Check Status
python -m agent.main "Is Qwen running?"
```

#### CLI Action Flags
```bash
# Direct command flags
python -m agent.main --open
python -m agent.main --close
python -m agent.main --status
python -m agent.main --config-path "C:\Program Files\Qwen\Qwen.exe"
```

#### Interactive Terminal Mode
Launch an interactive session where you can type multiple commands:
```bash
python -m agent.main --interactive
```

#### Desktop Graphical Interface (GUI)
Launch the built-in Tkinter desktop GUI:
```bash
python -m agent.main --gui
```
*(The GUI displays real-time status badges, path configuration dialogs, control buttons, and log streams).*

---

## ⚙️ Configuration (`qwen_agent_config.json`)

The agent automatically discovers common Qwen application installation paths upon first run. You can also manually configure the executable path in `qwen_agent_config.json`:

```json
{
    "qwen_executable_path": "C:\\Program Files\\Qwen\\Qwen.exe",
    "process_name": "Qwen.exe",
    "graceful_timeout": 5.0,
    "startup_timeout": 5.0,
    "qwen_api_url": "http://localhost:11434/api/generate",
    "qwen_model": "qwen2.5",
    "log_file": "qwen_agent.log"
}
```

### Environment Variables
- `QWEN_API_URL`: Optional local endpoint for LLM intent parsing (e.g. `http://localhost:11434/api/generate`).
- `QWEN_MODEL`: Optional local model identifier (default: `qwen2.5`).

---

## 🐧 OS Considerations

### Windows
- Default executable candidate paths:
  - `%LocalAppData%\Programs\Qwen\Qwen.exe`
  - `C:\Program Files\Qwen\Qwen.exe`
- Graceful shutdown sends WM_CLOSE signal via `taskkill /PID <pid>`, with fallback to `/F` force termination after timeout.

### macOS
- Default application bundle paths:
  - `/Applications/Qwen.app`
  - `~/Applications/Qwen.app`
- Launched via application bundle or direct binary invocation.
- Process control uses POSIX `SIGTERM` followed by `SIGKILL` if timeout expires.

### Linux
- Default paths:
  - `/usr/bin/qwen`
  - `/usr/local/bin/qwen`
  - `~/.local/bin/qwen`
  - `~/Applications/qwen.AppImage`
- Requires executable permissions (`chmod +x`).

---

## 🧪 Running Automated Tests

Run the full unit test suite with `pytest`:
```bash
pytest agent/tests -v
```

 Or run tests using Python's standard `unittest`:
```bash
python -m unittest discover -s agent/tests -p "test_*.py"
```

All process management unit tests use mocks to ensure no real desktop applications are launched or terminated during testing.

---

## 🛠️ Troubleshooting

| Issue | Root Cause | Solution |
| :--- | :--- | :--- |
| `Qwen executable path is not configured` | Auto-detection did not find Qwen in default directories. | Run `python -m agent.main --config-path "/path/to/qwen"` or set path in GUI. |
| `Permission denied` on Linux/macOS | File lacks executable bit permissions. | Run `chmod +x /path/to/qwen`. |
| `Process started but failed startup verification` | Application takes longer than 5s to show up in process list. | Increase `"startup_timeout"` in `qwen_agent_config.json`. |
| `Refusing to execute dangerous system binary` | Attempted to set path to system executable like `cmd.exe` or `bash`. | Configure path to the legitimate Qwen executable. |

---

## 📁 Project Structure

```text
agent/
├── __init__.py
├── main.py              # CLI & Application Entrypoint
├── intent_parser.py     # Natural Language Intent Parser & LLM Fallback
├── qwen_controller.py   # Core Application Lifecycle Controller
├── process_manager.py   # Cross-platform Process Manager
├── config.py            # Configuration Manager & OS Path Auto-Detector
├── security.py          # Action Allowlist & Path Security Validator
├── logger.py            # Structured JSON & Sanitized Logging
├── gui.py               # Tkinter Desktop GUI Interface
├── requirements.txt     # Python Dependencies
├── README.md            # Documentation
└── tests/
    ├── __init__.py
    ├── test_intent_parser.py
    ├── test_process_manager.py
    ├── test_qwen_controller.py
    └── test_security.py
```
