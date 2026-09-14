# Host CLI

Linux amd64, Windows amd64, macOS Apple Silicon (arm64), and macOS Intel (amd64). This GitHub repository is **download + version-check only**. It is not the source tree. Only the **latest** release is kept.

`ai-cli` runs a node on your machine, takes AI tasks, and earns HOST rewards. The node runtime and keytool are **bundled inside the binary** and extracted on first use.

## Requirements

- Windows 10 64-bit (amd64) or later
- Ubuntu 20.04 64-bit (amd64) or later
- macOS 12+ 64-bit, Apple Silicon (arm64) or Intel (amd64)

You do **not** need a `.env` or a hub URL. Those are baked into the binary.

## Output

| Command | stdout |
|---------|--------|
| **`start`** | Human **card**, then **live logs**. Ctrl+C stops **this wallet**. |
| **`ai-cli --json start`** | One JSON object. Does **not** stream logs unless you also pass **`--follow`** (logs then go to **stderr**). |
| **Everything else** (`version`, `storage`, `reward`, `status`, `logs`, `update`, `doctor`, …) | Indented JSON. You do **not** need `--json`. |
| **Help** | Human. |
| **Errors** | One text line on **stderr**. Exit code ≠ 0. |

Windows PowerShell: type `.\ai-cli.exe` (the `.\` is required). Linux and macOS: `./ai-cli` or `ai-cli` if it is on `PATH`.

## Install

Current release: **0.30** (`v0.30.0-demo`).

### Windows

Work in **your** profile folder. PowerShell as that user, not Administrator:

```powershell
cd $env:USERPROFILE
irm https://github.com/DiseAgon/os-pack-dl/releases/latest/download/install.ps1 | iex
```

Or the zip:

```powershell
cd $env:USERPROFILE
curl.exe -LO https://github.com/DiseAgon/os-pack-dl/releases/latest/download/aioz-ai-cli-windows-amd64-0.30.zip
Expand-Archive -Path aioz-ai-cli-windows-amd64-0.30.zip -DestinationPath .
ren aioz-ai-cli-windows-amd64.exe ai-cli.exe
.\ai-cli.exe version
```

`--save-priv-key privkey.json` writes into the current folder. `Access is denied` means that folder is not yours — `cd $env:USERPROFILE` and retry. Data is under `%LOCALAPPDATA%\AIOZ\ai-cli\`. The first `start` may show a Windows Firewall prompt; allow it for private networks.

### Linux

```bash
curl -fsSL https://github.com/DiseAgon/os-pack-dl/releases/latest/download/install.sh | bash
```

Or the archive:

```bash
curl -LO https://github.com/DiseAgon/os-pack-dl/releases/latest/download/aioz-ai-cli-linux-amd64-0.30.tar.gz
tar -xzf aioz-ai-cli-linux-amd64-0.30.tar.gz
mv aioz-ai-cli-linux-amd64 ai-cli
./ai-cli version
```

### macOS

`install.sh` picks Apple Silicon vs Intel from `uname -m`:

```bash
curl -fsSL https://github.com/DiseAgon/os-pack-dl/releases/latest/download/install.sh | bash
```

| `uname -m` | Chip | Archive | Inner file |
|------------|------|---------|------------|
| `arm64` | Apple Silicon (M1–M4) | `aioz-ai-cli-darwin-arm64-0.30.tar.gz` | `aioz-ai-cli-darwin-arm64` |
| `x86_64` | Intel | `aioz-ai-cli-darwin-amd64-0.30.tar.gz` | `aioz-ai-cli-darwin-amd64` |

Apple Silicon, manual:

```bash
curl -LO https://github.com/DiseAgon/os-pack-dl/releases/latest/download/aioz-ai-cli-darwin-arm64-0.30.tar.gz
tar -xzf aioz-ai-cli-darwin-arm64-0.30.tar.gz
mv aioz-ai-cli-darwin-arm64 ai-cli
xattr -dr com.apple.quarantine ./ai-cli
./ai-cli version
```

Intel: same steps with `aioz-ai-cli-darwin-amd64-0.30.tar.gz` / `aioz-ai-cli-darwin-amd64`. Do not use the Intel archive on Apple Silicon.

`version`:

```json
{
  "built": "2026-09-14T03:06:01Z",
  "commit": "v0.30.0-demo",
  "version": "0.30"
}
```

## First run

### 1. Create a key

```bash
./ai-cli keytool new --save-priv-key privkey.json
```

Windows: `.\ai-cli.exe keytool new --save-priv-key privkey.json`.

`--save-priv-key` writes the private key JSON (mode `0600`). Store the mnemonic now; it is not shown again. This does not create a data folder.

```json
{
  "address": "…",
  "address_evm": "0xAbc0…def1",
  "mnemonic": "twelve words …",
  "priv_key_file": "privkey.json"
}
```

Treat `privkey.json` and the mnemonic as **wallet secrets**. Use a **dedicated key for each node**. Keep an offline backup. Never paste them into websites, chats, or support tickets.

Recover later from a file (do not put the words on the command line):

```bash
./ai-cli keytool recover --mnemonic-file words.txt --save-priv-key privkey.json
```

### 2. Set a storage limit

Required **before** `start`. The value must be **greater than 2 GB**. There is no 2 GB default. Bare `storage` prints help.

```bash
./ai-cli storage limit 10 --priv-key-file privkey.json
```

```json
{
  "error": null,
  "success": true
}
```

`N` is decimal GB (`10` → `10000000000` bytes on Linux). Raise the cap by running the same command again; it applies on the next `start`.

Without `--home`, this wallet gets a UUID folder:

| OS | Home |
|----|------|
| Linux | `~/.local/share/aioz/ai-nodes/<uuid>/` |
| Windows | `%LOCALAPPDATA%\AIOZ\ai-cli\ai-nodes\<uuid>\` |
| macOS | `~/Library/Application Support/AIOZ/ai-cli/ai-nodes/<uuid>/` |

### 3. Start

```bash
./ai-cli start --priv-key-file privkey.json
```

`--priv-key-file` is required. Default `start` prints a **card**, then **streams logs**. Ctrl+C stops **this wallet** only. There is no `stop` command.

JSON start (no live logs unless `--follow`):

```bash
./ai-cli --json start --priv-key-file privkey.json
```

```json
{
  "data_dir": "~/.local/share/aioz/ai-nodes/<uuid>",
  "evm_address": "0xAbc0…def1",
  "home": "~/.local/share/aioz/ai-nodes/<uuid>",
  "log_path": "~/.local/state/aioz/ai-cli/logs/<uuid>/ai.log",
  "pid": 12345,
  "pid_path": "~/.local/state/aioz/ai-cli/<uuid>/node.pid",
  "running": true,
  "storage_bytes": 10000000000,
  "update": {
    "skipped": false,
    "newer": false,
    "current": "0.30",
    "current_commit": "v0.30.0-demo",
    "remote": "0.30",
    "remote_commit": "v0.30.0-demo",
    "note": "CLI is up to date"
  }
}
```

On default UUID homes, **Home** and the data dir are the same path; the human card then omits **Dir**. JSON still has `data_dir`.

Ctrl+C then prints a **second** JSON object (`--json`). `running` here is a **count** of other node processes still live (not a boolean):

```json
{
  "running": 0,
  "status": "stopped"
}
```

If this wallet is already running, the first object is only `running`, `pid`, `pid_path`, plus `update`.

Log paths:

| OS | `log_path` |
|----|------------|
| Linux | `~/.local/state/aioz/ai-cli/logs/<uuid>/ai.log` |
| Windows | `%LOCALAPPDATA%\AIOZ\ai-cli\logs\<uuid>\ai.log` |
| macOS | `~/Library/Logs/AIOZ/ai-cli/<uuid>/ai.log` |

## Commands

Wallet commands take `--priv-key-file` unless noted.

### Status

```bash
./ai-cli status --priv-key-file privkey.json
```

```json
{
  "evm_address": "0xAbc0…def1",
  "home": "~/.local/share/aioz/ai-nodes/<uuid>",
  "log_path": "~/.local/state/aioz/ai-cli/logs/<uuid>/ai.log",
  "other_running": 0,
  "running": false
}
```

`ai-cli status --all` lists every indexed home (`homes`, `nodes`, `running`). No `--priv-key-file`.

### Storage show

```bash
./ai-cli storage show --priv-key-file privkey.json
```

```json
{
  "error": null,
  "storage_limit": "10000000000",
  "storage_used": "0"
}
```

`storage_limit` / `storage_used` are **byte strings**. Before the first successful `start`, used is `0` (the node is not registered yet).

### Logs

`logs` is a **snapshot** (last `--bytes`, default 32 KiB), not a live follow. Secrets in the file are redacted.

```bash
./ai-cli logs --priv-key-file privkey.json
```

```json
{
  "home": "~/.local/share/aioz/ai-nodes/<uuid>",
  "log": "… redacted snapshot …",
  "path": "~/.local/state/aioz/ai-cli/logs/<uuid>/ai.log",
  "wallet": "0xAbc0…def1"
}
```

Live logs: leave `start` attached, or tail `path` / `log_path` on disk.

### Reward balance

Works with the node off.

```bash
./ai-cli reward balance --priv-key-file privkey.json
```

```json
{
  "earned": {
    "amount": "0",
    "denom": "attohost",
    "host": "0"
  },
  "earned_count": 0,
  "error": null,
  "spendable": {
    "amount": "0",
    "denom": "attohost",
    "host": "0"
  }
}
```

### Withdraw

`--address` is a MetaMask `0x` on HOST Chain. `--amount` is in HOST. Minimum **0.01 HOST**. `--yes` skips the confirm prompt.

```bash
./ai-cli reward withdraw --address 0xAbc0…def1 --amount 1 --priv-key-file privkey.json --yes
```

```json
{
  "error": null,
  "txid": "2604F553…944D59"
}
```

### Update

Most commands also check GitHub and may replace the binary when a newer signed release exists. To do that explicitly:

```bash
./ai-cli update
```

```json
{
  "skipped": false,
  "newer": false,
  "current": "0.30",
  "current_commit": "v0.30.0-demo",
  "remote": "0.30",
  "remote_commit": "v0.30.0-demo",
  "note": "CLI is up to date"
}
```

`update --check-only` prints status and does not download. `update --stop` stops every node on this machine first, then replaces the binary.

### Stats

Hub snapshot (`wallet_address` is the hub field name):

```bash
./ai-cli stats --priv-key-file privkey.json
```

```json
{
  "ai_tasks": [],
  "error": null,
  "status": "Online",
  "wallet_address": "0xAbc0…def1"
}
```

If the hub is unreachable, this command prints an error on **stderr** instead of JSON.

### Doctor

```bash
./ai-cli doctor --priv-key-file privkey.json
```

```json
{
  "ok": true,
  "checks": [
    {"name": "os", "ok": true, "detail": "linux/amd64"},
    {"name": "home", "ok": true, "detail": "~/.local/share/aioz/ai-nodes/<uuid>"},
    {"name": "disk", "ok": true, "detail": "100 GB free"},
    {"name": "runtime", "ok": true, "detail": "ok"},
    {"name": "wallet", "ok": true, "detail": "0xAbc0…def1"},
    {"name": "log", "ok": true, "detail": "~/.local/state/aioz/ai-cli/logs/<uuid>/ai.log"},
    {"name": "identity", "ok": true, "detail": "credential is --priv-key-file"},
    {"name": "gpu", "ok": true, "detail": "NVIDIA, 8 GB"}
  ]
}
```

### Show address

```bash
./ai-cli keytool show --priv-key-file privkey.json
```

Prints `address_evm` and `address`. Never prints the private key.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `command not found` after install | Open a new terminal, or `source ~/.bashrc` / `~/.zshrc`. Binary is `~/.local/bin/ai-cli`. |
| `sha256sum: command not found` | You ran an old installer on macOS. Use **0.30+** `install.sh` (it uses `openssl`). |
| macOS “cannot be opened because the developer cannot be verified” | `xattr -dr com.apple.quarantine ./ai-cli` then run it again. |
| `Access is denied` on Windows | `cd $env:USERPROFILE` — do not run as Administrator in a protected folder. |
| `set a storage limit before start` | `storage limit N --priv-key-file privkey.json` with **N > 2**. |
| `storage must be greater than 2 GB` | Same: N must be **greater than 2**. |
| `--priv-key-file is required` | Pass the JSON you created with `keytool new`. |
| `wallet_address already running` | Ctrl+C that wallet's `start`. One live node per key. |
| Inner archive file is `aioz-ai-cli-darwin-arm64`, not `ai-cli` | `mv` it as in the install steps. |
| Intel binary on Apple Silicon (or the reverse) | Match `uname -m` to the table above. |

## Security

- Never put the mnemonic or `privkey.json` on the command line except as a **file path**.
- Never paste keys into chat, tickets, or websites.
- `logs` redacts secrets it recognizes; the file on disk may still contain runtime noise — do not publish it.
- This repo keeps **only the latest** release. A bad build is replaced by the next signed version; there is no tag rollback on GitHub.
