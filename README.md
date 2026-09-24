# AIOZ AI CLI

`ai-cli` is the command-line tool for running and managing an AIOZ AI Node. It creates a wallet, sets a storage cap, starts the node, and exposes status, logs, rewards, stats, and diagnostics.

Production **v1.0.0** supports Linux amd64, Windows amd64, macOS Apple Silicon (arm64), and macOS x86_64. Each archive bundles a matching node runtime and keytool. Linux ARM64 and FreeBSD amd64 are not in this release. Linux ARM 32-bit is unsupported.

Download one of the four archives below. [Release v1.0.0](https://github.com/DiseAgon/os-pack-dl/releases/tag/v1.0.0).

## Requirements

- Windows 10 64-bit (amd64) or later
- Ubuntu 20.04 64-bit (amd64) or later
- macOS 12+ 64-bit, Apple Silicon (arm64) or x86_64 (amd64)

`start` prints a **card**, then **streams logs**. Other commands print indented JSON. Failures print `{"error": "…"}` on stdout (exit ≠ 0). Success JSON includes `"error": null`.

Windows PowerShell: type `.\ai-cli.exe` (the `.\` is required).

Linux and macOS: `./ai-cli` or `ai-cli` if it is on `PATH`.

## Install

Download the archive for your OS, extract it, and rename the inner file to `ai-cli` or `ai-cli.exe`.

`version` prints JSON. `commit` is the git SHA of this build, not the version tag.

```json
{
  "built": "2026-09-24T05:24:01Z",
  "commit": "27930ba2dd77e32b49fc4a1da360f10fab8ab8a7",
  "error": null,
  "version": "1.0.0"
}
```

### Windows

Work in **your** profile folder. PowerShell as that user, not Administrator.

```powershell
cd $env:USERPROFILE
curl.exe -fLO https://github.com/DiseAgon/os-pack-dl/releases/download/v1.0.0/aioz-ai-cli-windows-amd64-1.0.0.zip
Expand-Archive -LiteralPath aioz-ai-cli-windows-amd64-1.0.0.zip -DestinationPath . -Force
Move-Item -Force .\aioz-ai-cli-windows-amd64.exe .\ai-cli.exe
.\ai-cli.exe version
```

`--save-priv-key privkey.json` writes into the current folder. `Access is denied` means that folder is not yours — `cd $env:USERPROFILE` and retry. Data is under `%LOCALAPPDATA%\aioz\ai-cli\`. The first `start` may show a Windows Firewall prompt; allow it for private networks.

### Linux amd64

Run `uname -m` first. Continue only when it prints `x86_64`. This release has no Linux ARM64 archive.

```bash
uname -m
curl -fLO https://github.com/DiseAgon/os-pack-dl/releases/download/v1.0.0/aioz-ai-cli-linux-amd64-1.0.0.tar.gz
tar -xzf aioz-ai-cli-linux-amd64-1.0.0.tar.gz
mv aioz-ai-cli-linux-amd64 ai-cli
./ai-cli version
```

### macOS

Pick the archive that matches `uname -m`:

| `uname -m` | Chip | Archive |
|------------|------|---------|
| `arm64` | Apple Silicon (M1–M4) | `aioz-ai-cli-darwin-arm64-1.0.0.tar.gz` |
| `x86_64` | x86_64 | `aioz-ai-cli-darwin-x86_64-1.0.0.tar.gz` |

Apple Silicon:

```bash
curl -fLO https://github.com/DiseAgon/os-pack-dl/releases/download/v1.0.0/aioz-ai-cli-darwin-arm64-1.0.0.tar.gz
tar -xzf aioz-ai-cli-darwin-arm64-1.0.0.tar.gz
mv aioz-ai-cli-darwin-arm64 ai-cli
./ai-cli version
```

x86_64:

```bash
curl -fLO https://github.com/DiseAgon/os-pack-dl/releases/download/v1.0.0/aioz-ai-cli-darwin-x86_64-1.0.0.tar.gz
tar -xzf aioz-ai-cli-darwin-x86_64-1.0.0.tar.gz
mv aioz-ai-cli-darwin-x86_64 ai-cli
./ai-cli version
```

Do not use the x86_64 archive on Apple Silicon.

## First run

### 1. Create a key

Writes a new private-key JSON and prints the mnemonic once.

**Windows**

```powershell
.\ai-cli.exe keytool new --save-priv-key privkey.json
```

**Linux and macOS**

```bash
./ai-cli keytool new --save-priv-key privkey.json
```

`--save-priv-key` writes the private key JSON (mode `0600`). Store the mnemonic now; it is not shown again. This does not create a data folder.

```json
{
  "address": "aioz1…",
  "address_evm": "0xAbc0…def1",
  "error": null,
  "mnemonic": "twelve words …",
  "priv_key_file": "privkey.json"
}
```

Treat `privkey.json` and the mnemonic as **wallet secrets**. Use a **dedicated key for each node**. Keep an offline backup. Never paste them into websites, chats, or support tickets.

To restore an existing wallet, put the 12 or 24 words in a file:

**Windows**

```powershell
.\ai-cli.exe keytool recover --mnemonic-file words.txt --save-priv-key privkey.json
```

**Linux and macOS**

```bash
./ai-cli keytool recover --mnemonic-file words.txt --save-priv-key privkey.json
```

### 2. Set a storage limit

Required **before** `start`. The value must be **greater than 2 GB**. There is no 2 GB default. Bare `storage` prints help.

**Windows**

```powershell
.\ai-cli.exe storage limit 10 --priv-key-file privkey.json
```

**Linux and macOS**

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
| Linux | `~/.local/share/aioz/ai-cli/ai-nodes/<uuid>/` |
| Windows | `%LOCALAPPDATA%\aioz\ai-cli\ai-nodes\<uuid>\` |
| macOS | `~/Library/Application Support/aioz/ai-cli/ai-nodes/<uuid>/` |

### 3. Start

Starts this wallet's node. Prints a card, then streams logs. Ctrl+C stops **this wallet** only. There is no `stop` command. `--priv-key-file` is required.

**Windows**

```powershell
.\ai-cli.exe start --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli start --priv-key-file privkey.json
```

```
╭─ node ───────────────────────────────────────────────────────────────╮
│                                                                      │
│  Status            running                                           │
│  CLI               1.0.0                                             │
│  PID               12345                                             │
│  Home              ~/.local/share/aioz/ai-cli/ai-nodes/<uuid>/       │
│  Storage           10 GB                                             │
│  EVM               0xAbc0…def1                                       │
│                                                                      │
╰──────────────────────────────────────────────────────────────────────╯
```

Logs then stream in the same terminal.

Failures (missing key, no storage limit, extra args) print `{"error": "…"}` on stdout.

Log paths:

| OS | `log_path` |
|----|------------|
| Linux | `~/.local/state/aioz/ai-cli/logs/<uuid>/ai.log` |
| Windows | `%LOCALAPPDATA%\aioz\ai-cli\logs\<uuid>\ai.log` |
| macOS | `~/Library/Logs/aioz/ai-cli/<uuid>/ai.log` |

## Commands

Wallet commands take `--priv-key-file` unless noted.

### Status

Whether this wallet's node process is running on this machine.

**Windows**

```powershell
.\ai-cli.exe status --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli status --priv-key-file privkey.json
```

```json
{
  "evm_address": "0xAbc0…def1",
  "home": "~/.local/share/aioz/ai-cli/ai-nodes/<uuid>",
  "log_path": "~/.local/state/aioz/ai-cli/logs/<uuid>/ai.log",
  "other_running": 0,
  "running": false
}
```

`ai-cli status --all` lists every indexed home (`homes`, `nodes`, `running`). No `--priv-key-file`.

### Storage show

Storage cap and usage for this wallet (byte strings).

**Windows**

```powershell
.\ai-cli.exe storage show --priv-key-file privkey.json
```

**Linux and macOS**

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

### Logs

Snapshot of `ai.log` (last `--bytes`, default 32 KiB), not a live follow. Secrets in the file are redacted.

**Windows**

```powershell
.\ai-cli.exe logs --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli logs --priv-key-file privkey.json
```

```json
{
  "home": "~/.local/share/aioz/ai-cli/ai-nodes/<uuid>",
  "log": "… redacted snapshot …",
  "path": "~/.local/state/aioz/ai-cli/logs/<uuid>/ai.log",
  "wallet": "0xAbc0…def1"
}
```

Live logs: leave `start` attached, or tail `path` / `log_path` on disk.

### Reward balance

Spendable and earned AIOZ. Works with the node off. Denom is `attoaioz`.

**Windows**

```powershell
.\ai-cli.exe reward balance --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli reward balance --priv-key-file privkey.json
```

```json
{
  "earned": {
    "amount": "0",
    "denom": "attoaioz",
    "aioz": "0"
  },
  "earned_count": 0,
  "error": null,
  "spendable": {
    "amount": "0",
    "denom": "attoaioz",
    "aioz": "0"
  }
}
```

### Withdraw

Send rewards to a MetaMask `0x` on AIOZ. `--amount` is in AIOZ. Minimum **0.01 AIOZ**. `--yes` skips the confirm prompt.

**Windows**

```powershell
.\ai-cli.exe reward withdraw --address 0xAbc0…def1 --amount 1 --priv-key-file privkey.json --yes
```

**Linux and macOS**

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

Download the latest signed CLI and replace this binary. Most commands also check GitHub in the background.

**Windows**

```powershell
.\ai-cli.exe update
```

**Linux and macOS**

```bash
./ai-cli update
```

```json
{
  "skipped": false,
  "newer": false,
  "current": "1.0.0",
  "current_commit": "27930ba2dd77e32b49fc4a1da360f10fab8ab8a7",
  "remote": "1.0.0",
  "remote_commit": "27930ba2dd77e32b49fc4a1da360f10fab8ab8a7",
  "note": "CLI is up to date"
}
```

`update --check-only` prints status and does not download. `update --stop` stops every node on this machine first, then replaces the binary.

### Stats

Hub snapshot for this wallet: `status`, `wallet_address`, and `ai_tasks`.

**Windows**

```powershell
.\ai-cli.exe stats --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli stats --priv-key-file privkey.json
```

```json
{
  "ai_tasks": [],
  "error": null,
  "status": "standby",
  "wallet_address": "0xAbc0…def1"
}
```

`status` values:

| Value | Meaning |
|-------|---------|
| `standby` | Active and idle (ready for work). |
| `computing` | Active and processing a task. |
| `initiating` | Logging in and registering with the system; not ready yet. |
| `coming_soon` | Went offline within the last 2 minutes. |
| `offline` | Not active. |

If the hub is unreachable, stdout is `{"error": "…"}` and the process exits non-zero.

### Doctor

Checks disk, runtime, wallet, GPU, and this wallet's log.

**Windows**

```powershell
.\ai-cli.exe doctor --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli doctor --priv-key-file privkey.json
```

```json
{
  "ok": true,
  "checks": [
    {"name": "os", "ok": true, "detail": "linux/amd64"},
    {"name": "home", "ok": true, "detail": "~/.local/share/aioz/ai-cli/ai-nodes/<uuid>"},
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

Prints `address_evm` and `address`. Never prints the private key.

**Windows**

```powershell
.\ai-cli.exe keytool show --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli keytool show --priv-key-file privkey.json
```

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `command not found` | Run `./ai-cli` (Linux/macOS) or `.\ai-cli.exe` (Windows) from the folder you extracted. |
| macOS “cannot be opened because the developer cannot be verified” | `xattr -dr com.apple.quarantine ./ai-cli` then run it again. |
| `Access is denied` on Windows | `cd $env:USERPROFILE` — do not run as Administrator in a protected folder. |
| `set a storage limit before start` | `storage limit N --priv-key-file privkey.json` with **N > 2**. |
| `storage must be greater than 2 GB` | Same: N must be **greater than 2**. |
| `storage limit must be a number of GB` | Pass a number (`10`), not `10GB` or `abc`. |
| `mnemonic has N words` | Recover needs **12 or 24** words. |
| `need mnemonic words or --mnemonic-file` | Quote the phrase or pass `--mnemonic-file`. |
| `start does not take arguments` | Only flags (`--priv-key-file`). |
| `--priv-key-file is required` | Pass the JSON you created with `keytool new`. |
| `wallet_address already running` | Ctrl+C that wallet's `start`. One live node per key. |
| Inner archive file is `aioz-ai-cli-darwin-arm64`, not `ai-cli` | `mv` it as in the install steps. |
| x86_64 binary on Apple Silicon (or the reverse) | Match `uname -m` to the table above. |

## Security

- Keep `privkey.json` and mnemonic files private. Do not commit them or paste them into chat, tickets, or websites.
- Prefer `keytool recover --mnemonic-file` so the words are not stored in shell history.
- Never put `privkey.json` contents on the command line.
- `logs` redacts secrets it recognizes; do not publish the log file on disk.
- One live node per wallet. Ctrl+C on `start` stops this wallet only.
