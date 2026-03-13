# AXIS---official

<div align="center">

<br>

```
 █████╗ ██╗  ██╗██╗███████╗
██╔══██╗╚██╗██╔╝██║██╔════╝
███████║ ╚███╔╝ ██║███████╗
██╔══██║ ██╔██╗ ██║╚════██║
██║  ██║██╔╝ ██╗██║███████║
╚═╝  ╚═╝╚═╝  ╚═╝╚═╝╚══════╝
```

### *A local AI agent shell where you are the architect of everything.*

<br>

![Status](https://img.shields.io/badge/status-coming%20soon-blueviolet?style=for-the-badge&labelColor=0d1117)
![Platform](https://img.shields.io/badge/platform-Windows%2010%2F11-informational?style=for-the-badge&logo=windows&logoColor=white&labelColor=0d1117)
![Python](https://img.shields.io/badge/python-3.13-blue?style=for-the-badge&logo=python&logoColor=white&labelColor=0d1117)
![LLM](https://img.shields.io/badge/backend-Ollama-orange?style=for-the-badge&labelColor=0d1117)
![License](https://img.shields.io/badge/license-see%20repo-lightgrey?style=for-the-badge&labelColor=0d1117)

<br>

> **AXIS is not released yet.** The repository currently holds a placeholder.<br>
> No release date. No promises. It ships when it's ready — and not a day before.<br>
> **Star or watch the repo** to be notified when it drops.

<br>

---

</div>

## What is AXIS?

AXIS is a local AI agent shell — a middleware layer that gives a locally-running LLM (via [Ollama](https://ollama.ai)) the ability to operate your machine in a structured, audited, and strictly permission-governed way. The model can read files, run commands, manage data, and pursue multi-step goals autonomously — entirely on your hardware, with no cloud dependency, no external API, and no data ever leaving your machine.

The key word is *structured*. AXIS is not a wrapper that hands an AI a shell prompt and hopes for the best. Every capability the model has access to is something *you* explicitly defined in a JSON command library. Every action it wants to take goes through a deterministic validation pipeline. Every high-risk operation surfaces a permission prompt with the agent's stated reasoning, and you approve or reject it in real time. The shell itself is a strict parser and executor — it does not guess intent, it does not infer behavior, it does not improvise. It follows schemas.

This matters because a lot of existing local agent tools get this badly wrong. They optimize for impressiveness — "look what the model can do" — and treat safety as a config flag, a softly-worded system prompt, or a vague disclaimer. The model ends up with broad, unchecked access and the user has no meaningful visibility into what it's actually doing. AXIS was built to never be that. <strong><em style="font-size:large">Security is a first-class architectural constraint here, not an afterthought bolted on at the end.</em></strong>

<br>

---

## The Design Philosophy

**Security by architecture, not by prompt.** The model is not told "please don't touch system files." It structurally *cannot* touch them unless a command exists in your library that allows it, and that command passes ACL checks against your configured restricted paths. There is no jailbreak path here because there is no open terminal — only a typed, validated, schema-governed command surface.

**You define what the model can do.** Command libraries are JSON files you write and own. Want the model to be able to call a custom CLI tool? Write a library entry. Want it to edit files in a specific folder? Add that command with the appropriate permission tier. Want to revoke access to something entirely? Remove the entry, or add a path to your restricted list. <strong>The agent's complete capability surface is a configuration artifact you control</strong> — not decisions baked into the shell's source code.

**Everything meaningful is configurable.** Token limits, context window percentages, retry counts, per-command timeouts, permission polling intervals, snapshot behavior, storage quotas, playground size limits, session archival destinations — all of it lives in `variables.json`. Nothing meaningful is hardcoded. You tune the system to your hardware and your risk tolerance without touching a single line of Python.

**The shell is not an AI.** The middleman between the model and your system is a deterministic Python process. It parses model output with strict regex, validates command shape against a schema, checks permissions, handles timeouts, and routes execution. If something doesn't match the schema, it fails explicitly and tells the model exactly why — giving it the chance to self-correct. The shell does not interpret intent. It does not make judgment calls.

**Full local sovereignty.** Your goals, your file contents, your session history, your audit logs — none of it ever leaves your machine. Ollama runs locally. The shell runs locally. The only network call AXIS ever makes is to `localhost` and to any web commands you configure within a command library.

<br>

---

## How It Works

```
┌────────────────────────────────────────────────────────────────────┐
│                           You (Operator)                           │
│          Define goal → Attach files → Approve / reject actions     │
└─────────────────────────────┬──────────────────────────────────────┘
                              │
                     ┌────────▼────────┐
                     │  Interaction    │  meta-shell.bat
                     │  Handler        │  Mode selection, goal acquisition,
                     └────────┬────────┘  file attachment handoff
                              │
                     ┌────────▼────────┐
                     │   Core Loop     │  core/main.py
                     │   (main.py)     │  Session lifecycle, context assembly,
                     └────────┬────────┘  snapshot management, recovery
          ┌────────────────────┼────────────────────────┐
          │                    │                        │
 ┌────────▼───────┐   ┌────────▼────────┐   ┌──────────▼─────────┐
 │  Local LLM     │   │  Command        │   │  Audit Engine      │
 │  via Ollama    │   │  Library        │   │  SQLite + WAL      │
 │  (gemma3, etc.)│   │  .lib.json      │   │  model_io/shell_io │
 └────────┬───────┘   └────────┬────────┘   └────────────────────┘
          │                    │
          │          ┌─────────▼──────────────────────────────────┐
          │          │         Validation Pipeline                 │
          │          │  output_gate → command_validator            │
          │          │  → path_security → permission protocol      │
          └─────────►└─────────┬──────────────────────────────────┘
                               │
                     ┌─────────▼───────────┐
                     │  CST Handler        │  Placeholder resolution,
                     │  + Dispatcher       │  Base64 staging, routing
                     └─────────┬───────────┘
                               │
                     ┌─────────▼──────────────────────────────────┐
                     │        PowerShell Executor                  │
                     │  Runs command · CEFIP integrity check       │
                     │  Returns structured result to Core Loop     │
                     └────────────────────────────────────────────┘
```

The model outputs command blocks using a custom `cmnd` fence syntax. The shell's output gate validates fence balance and enforces the per-turn block cap. The command validator checks each block against your library schema. The permission protocol applies the appropriate tier — auto-approve, notify-only, or interactive prompt. The CST handler resolves argument placeholders, Base64-encodes all content fields into temp files so nothing ever touches the command line raw, and hands the final translated command to the PowerShell executor. After execution, CEFIP (the Cross-Executor File Integrity Protocol) validates the result before it is returned. The result is injected back into the model's context as structured text, and the loop continues toward the goal.

<br>

---

## Customization — The Whole Point

<strong>AXIS was built with customizability as a first-order design requirement.</strong> This is not a tool with a handful of knobs. Virtually every behavioral contract in the system is configurable, and the parts that are not configurable are that way for explicitly-documented safety reasons.

### Command Libraries — You Define What the Model Can Do

Every command available to the agent lives in the `.lib.json` files you author. The library system is extensible without modifying any Python source code. Each command entry defines its argument slots and typed placeholders, the translated PowerShell command template with `$>><placeholder><<$` slot syntax, the execution mode (permission tier 0–3), whether the command is allowed on restricted paths, an optional file extension allowlist, a per-command timeout in milliseconds, a max retry count, and a file effect profile. Libraries are hot-reloadable mid-session — edit a file and the shell picks up the change on the next command dispatch with no restart. The startup preflight validates every library file against the strict JSON schema before the agent loop starts, so broken entries fail loudly at boot rather than silently at runtime.

```jsonc
// Example: a minimal READ command entry
{
  "command": "READ",
  "arguments": ["filepath"],
  "executionMode": 0,
  "permission_requirement": false,
  "restrictedPaths": false,
  "allowedExtensions": [".txt", ".md", ".json", ".py"],
  "timeoutMs": 10000,
  "maxRetries": 2,
  "fileEffectProfile": "read",
  "backend": {
    "shell": "powershell",
    "translatedCommand": "Get-Content -Path $>><filepath><<$ -Raw -Encoding UTF8"
  }
}
```

Want the model to be able to call a CLI tool you use daily? Write a library entry for it. Want to restrict it to only read from a specific directory? Set it in `restrictedPaths`. Want it to ask for explicit approval before writing anything? Set `executionMode: 1` on all write commands. The library is your policy document, and it lives outside the shell's source code on purpose.

### `variables.json` — The Master Configuration

`variables.json` is the single file governing the system's runtime behavior. There is no environment variable spaghetti, no scattered INI files, no magic constants buried in source. If a behavior is tunable, it lives here. A partial picture of what you control:

<br>

<table>
<tr><th>Category</th><th>What you configure</th></tr>
<tr>
<td><strong>Model</strong></td>
<td>Model name, context window limit, history window percentage, summarization threshold, max output characters, server URL and port, request timeout</td>
</tr>
<tr>
<td><strong>Session paths</strong></td>
<td>Storage folder, agent playground, reasoning folder, all 12 fully-resolved dynamic runtime paths</td>
</tr>
<tr>
<td><strong>Permission protocol</strong></td>
<td>Polling interval, watchdog check interval (1–60 s, validated at startup), no-response timeout, max retry counts</td>
</tr>
<tr>
<td><strong>Execution safety</strong></td>
<td>Restricted path list, Base64 max payload bytes, decode chunk size, max consecutive retry-exhaustion events before halt</td>
</tr>
<tr>
<td><strong>Context management</strong></td>
<td>Summarization prompt path, max summarization retries (separate limit from syntax retry limit), max end-summary characters</td>
</tr>
<tr>
<td><strong>Storage &amp; cleanup</strong></td>
<td>Playground quota in MB, cleanup target percentage, archive subdirectory, cleanup batch size, archive-counts-toward-quota toggle, verify-archive-before-delete toggle</td>
</tr>
<tr>
<td><strong>Audit</strong></td>
<td>Max entries per SQLite partition before rolling to a new file</td>
</tr>
<tr>
<td><strong>Recovery</strong></td>
<td>Storage cleanup retry on startup toggle, max cleanup retries per startup, unlinked artifact policy (ignore / warn / block), watchdog auto-heal toggle</td>
</tr>
<tr>
<td><strong>Ollama startup</strong></td>
<td>Connect retry count and delay, auto-start command, auto-start retry count</td>
</tr>
<tr>
<td><strong>7-Zip</strong></td>
<td>Absolute path to 7z.exe for Archiver mode</td>
</tr>
</table>

<br>

Fields that can be changed mid-session (live-reloaded on next use, no restart needed) versus fields that require a restart are documented explicitly in the codebase. If you edit a non-reloadable field mid-session, the shell logs the event and continues on the old value — it does not crash, and it does not silently apply a half-baked configuration.

### System Prompts — Plain Markdown You Write

The agent's behavioral instructions live in two files: `system_prompt_prefix.md` (injected before your goal) and `system_prompt_suffix.md` (injected after). These are plain markdown files you write and own. You control exactly what the model is instructed to do, what command format it uses, what reasoning style it adopts, and how it handles edge cases. Like libraries, both files are hot-reloadable mid-session. The startup preflight runs a Semantic Dry-Run Harness against them before the agent loop starts — if required placeholder tokens like `<MAX_CMND_BLOCKS_PER_TURN>` are missing, it fails at boot rather than producing a silent misbehaving session.

<br>

---

## Permission Tiers

Every command in your library is assigned an execution mode. The shell enforces this at the validation layer — it is not advisory.

<br>

<table>
<tr><th>Mode</th><th>Name</th><th>Behavior</th></tr>
<tr>
<td><strong>0</strong></td>
<td>Auto-approve</td>
<td>Executes immediately with no user interaction. Appropriate for safe, read-only operations like listing a directory or reading a file from the storage zone.</td>
</tr>
<tr>
<td><strong>1</strong></td>
<td>Notify</td>
<td>Executes automatically but emits a Windows toast notification so you stay aware without being blocked.</td>
</tr>
<tr>
<td><strong>2</strong></td>
<td>Interactive prompt</td>
<td>The shell surfaces the command and the agent's stated reasoning in a prompt window. Execution is blocked until you respond Y or N. The full request/response cycle uses SHA-256 integrity checks, a monotonic sequence number, atomic file operations, and a watchdog heartbeat check during the blocking wait.</td>
</tr>
<tr>
<td><strong>3</strong></td>
<td>Strict prompt</td>
<td>Same as mode 2 but with additional policy enforcement for the most sensitive operations.</td>
</tr>
</table>

<br>

The permission IPC protocol is engineered to be robust against file corruption, race conditions, and watchdog death. It uses AXIS-Canonical-JSON (ACJ v1.0) — a custom deterministic canonicalization algorithm — for checksum computation, with SHA-256 verification on the response payload before it is accepted. The response file is deleted atomically immediately after reading to prevent stale response reuse.

<br>

---

## Run Modes

AXIS supports three session modes, selected at launch via `meta-shell.bat`.

<br>

<table>
<tr><th>Mode</th><th>How it runs</th><th>Use case</th></tr>
<tr>
<td><strong>Verbose</strong></td>
<td>Attached to your terminal. All shell output is visible. Permission prompts appear inline.</td>
<td>Active work, debugging, watching what the agent is doing step by step.</td>
</tr>
<tr>
<td><strong>Background</strong></td>
<td>Detaches from the terminal after goal acquisition. Runs as a background process. Status updates and permission requests are delivered as Windows toast notifications, which spawn a temporary console window for interaction when needed.</td>
<td>Long-running autonomous tasks where you don't want to babysit a terminal.</td>
</tr>
<tr>
<td><strong>Archiver</strong></td>
<td>Same as Background, but at session end the shell compresses the full session log bundle — model I/O, shell I/O, state, storage — using 7-Zip and delivers it to a destination path you specify at launch. Storage cleanup is conditional on successful archival; if compression fails, nothing is deleted.</td>
<td>Audit-conscious workflows where you want a portable, compressed record of every action the agent took.</td>
</tr>
</table>

<br>

---

## Reliability & Recovery

Autonomous agents fail. Processes get killed, machines restart, context windows fill up. AXIS is engineered to handle all of these without losing session state.

**Context management.** The shell tracks estimated token counts for the history window, current tool output, and system prompt overhead using a content-aware Waterfall Heuristic (different estimation ratios for JSON, CSV, and prose). When the history approaches the configured summarization threshold — a percentage of the context window limit you set in `variables.json` — it invokes a dedicated summarizer call using a prompt you authored. The summarized history replaces the raw exchange history in the running context. The summarization retry limit is a separate configurable field from the command syntax retry limit, giving you independent control over both.

**Crash recovery.** At the end of every turn, the shell writes the exact message list it fed to the model to `state/snapshot_curr.json` using an atomic write strategy: write to a `.tmp` file, validate, rename into place. A rolling backup (`snapshot_prev.json`) is maintained in parallel. On next startup, if a stale session is detected, you get an interactive prompt: `[W] WIPE`, `[R] RECOVER`, or `[A] ABORT`. Recovery restores the last clean snapshot and injects a `SYSTEM: PREVIOUS ATTEMPT FAILED` message into the context so the model can re-evaluate its goal without losing its history.

**Semantic Reconstruction.** If both snapshots are gone or corrupted, the shell can rebuild session context entirely from the SQLite audit logs — iterating through split database partitions, synthesizing summaries of inputs, outputs, and reasoning entries, and reconstructing a context snapshot targeting 50% of the history window, leaving breathing room before the next summarization cycle triggers.

**File integrity on every write.** Every write operation through the PowerShell executor is subject to the Cross-Executor File Integrity Protocol (CEFIP). Deterministic known-target writes use Stage-Validate-Commit: write to a `.tmp` file, validate it, atomically commit. Non-deterministic writes use Observe-Complete-Validate. Mutating commands that cannot prove write coverage fail closed rather than proceeding with an uncertain result.

<br>

---

## Audit Trail

Every model input, model output, executed command, and command result is logged to SQLite in `logs/`. The audit engine uses WAL mode for durability and auto-splits into new database partitions when one reaches the `auditMaxEntriesPerDb` limit you configure. Corrupted partition files are moved to a `corrupted/` quarantine subdirectory rather than deleted, and they persist there until a Clean Slate is run. The `-history` flag on `meta-shell.bat` lets you browse the last session registry entry without starting a new agent session.

<br>

---

## Security Model

**Restricted paths.** The `restrictedPaths` array in `variables.json` is an ACL you own. Commands flagged as `restrictedPaths: false` are blocked by the validator if their target path resolves inside this list. Path normalization runs in `core/path_security.py` before any ACL check — canonical resolution, case-insensitive comparison, trailing slash normalization, and UNC path handling — so there is no path-trick bypass.

**File attachment isolation.** Files you attach to a session are copied into a session-scoped `storage/` directory before the agent loop starts. The agent accesses them through the `STORAGE_IO` command set using paths relative to the storage root. It never sees absolute paths on your filesystem. The storage folder is a declared Safe Zone — it bypasses global `restrictedPaths` checks because files were placed there by explicit user action, not autonomous behavior.

**No raw shell access.** The model outputs structured `cmnd` blocks. The output gate validates fence balance and enforces a per-turn block cap. The command validator checks shape and policy. The CST handler Base64-encodes all content arguments into temp files so the final translated command string never contains raw user-provided text — only resolved paths and temp file references. This eliminates command injection as an attack surface at the architecture level.

**Single instance enforcement.** AXIS acquires a startup lock file at launch, validated by PID and a process start token to handle PID reuse. A second instance cannot start while one is running. Cross-host lock detection is also handled — a lock file from a different machine is never silently overwritten.

<br>

---

## Tech Stack

<div align="center">

| Layer | Technology | Notes |
|:------|:-----------|:------|
| Language | Python 3.13 (x64) | MVP. Production target is C/Assembly. |
| LLM Runtime | Ollama | Local inference, HTTP REST at localhost |
| Default Model | `gemma3:4b-int-qat` | ~3 GB. Any Ollama model configurable. |
| Command Execution | PowerShell 5.1 / 7.x | Windows only for MVP |
| Persistence | SQLite 3.38+ with WAL mode | Split partition audit logs |
| Notifications | Windows Runtime Toast API | Background and Archiver modes, Build 15063+ |
| Compression | 7-Zip 22.01+ | Archiver mode only |
| Target OS | Windows 10 (Build 15063+) / Windows 11 | x64 required |

</div>

<br>

The MVP is implemented in Python for development speed, testability, and architectural clarity before the production rewrite. The production target is a compiled C/Assembly binary — the Python codebase is intended to be a behavioral specification as much as it is a functional tool. The architecture document, performance tuning guide, and production build documentation will live in `docs/` when the repository goes public.

<br>

---

## Project Structure

<details>
<summary><strong>Expand to see the full file tree</strong></summary>

<br>

```
AXIS/
├── core/
│   ├── main.py                  # Core loop — session lifecycle, agent orchestration
│   ├── interaction_handler.py   # Unified I/O layer, mode selection, goal acquisition
│   ├── agent.py                 # Model interface, context assembly, turn management
│   ├── library/                 # Library command validation pipeline
│   ├── superuser/               # Internal command validation (system-initiated path)
│   ├── dispatcher.py            # Executor routing by shell type and command tag
│   ├── cst_handler.py           # Placeholder resolution, Base64 staging, delimiter conversion
│   ├── executors/               # Shell-specific execution (PowerShell)
│   ├── model_output_gate.py     # First parser gate — fence validation, block extraction, cap
│   ├── path_security.py         # Canonical path normalization and ACL checks
│   ├── crypto_utils.py          # ACJ v1.0 canonicalization, SHA-256 utilities
│   ├── tokenizer.py             # Waterfall heuristic token estimation (JSON/CSV/Prose)
│   ├── preflight.py             # Startup preflight — 30+ validation checks before loop start
│   ├── registry_store.py        # Session registry atomic write and restore
│   ├── audit_engine.py          # SQLite logging, partition management, WAL, quarantine
│   ├── recovery.py              # Snapshot restore, Semantic Reconstruction from audit logs
│   ├── signal_handler.py        # Process reaping on termination signals (Grim Reaper)
│   ├── custom.py                # Custom behavioral commands (END, ALERT, REASON)
│   └── ...
│
├── cmd/
│   └── libs/                    # Your command libraries (.lib.json files live here)
│
├── config/
│   └── variables.json           # Master configuration (copied from templates/ on setup)
│
├── prompts/
│   ├── system_prompt_prefix.md  # Behavioral instructions injected before the user goal
│   ├── system_prompt_suffix.md  # Flow control instructions injected after the user goal
│   ├── summarizer.md            # Context summarization prompt
│   └── ...                      # Retry and error handling prompts
│
├── templates/                   # Reference configuration examples (not runtime artifacts)
│
├── docs/
│   ├── ARCHITECTURE.md
│   ├── LIBRARY_AUTHORING.md     # Guide for writing and extending command libraries
│   └── VARIABLES_GUIDE.md       # Guide for configuring variables.json
│
├── logs/                        # Session audit logs (SQLite partitions), session registry
├── state/                       # Context snapshots, temp payload staging files
├── storage/                     # Session-scoped file attachment safe zone
├── agent_playground/            # Agent scratchpad with configurable size quota
├── reasoning/                   # Model reasoning capture artifacts (.rsn files)
├── tests/                       # 320 test cases, 10 modules, 7 CI validation gates
└── meta-shell.bat               # Primary entry point
```

</details>

<br>

---

## Test Suite

The release is gated on 320 test cases across 10 modules and 7 CI validation gates. Coverage spans startup preflight, session lifecycle, the permission protocol, the audit engine, crash recovery, the tokenizer, path security, and the library loader, with integration tests for the Ollama client and the PowerShell executor. Every merge requires 100% pass on all gating items with a traceability matrix linking requirements to implementation artifacts and CI evidence.

<br>

---

<div align="center">
<div style="font-size: xxx-large; font-family: 'Varela Round', sans-serif; font-weight: 300;">
  Coming Soon . . .
</div>
<br>

AXIS is in active development. There is no release date and none will be announced in advance.<br>
It will appear in this repository when it's ready — not before.

<br>

If this is something you'd use, **star or watch the repo** and you'll be notified the moment it lands.

<br>

[![Star this repo](https://img.shields.io/github/stars/SANIUL-blackdragon/AXIS---official?style=for-the-badge&logo=github&labelColor=0d1117&color=blueviolet)](https://github.com/SANIUL-blackdragon/AXIS---official/stargazers)

<br>

---

# Contact

Questions, feedback, or ideas before the release? Reach out directly.

<br>

[![Email](https://img.shields.io/badge/mdalifsaniul%40gmail.com-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:mdalifsaniul@gmail.com)
&nbsp;&nbsp;
[![GitHub](https://img.shields.io/badge/SANIUL--blackdragon-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/SANIUL-blackdragon)

<br>

---

<sub>No cloud. No API keys. No data sent to a server you don't control. Your machine, your model, your rules.</sub>

<br>

</div>
