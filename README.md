# AIOZ AI CLI

`ai-cli` is the command-line tool for running and managing an AIOZ AI Node. It creates a wallet, sets a storage cap, starts the node, and exposes status, logs, rewards, stats, and diagnostics.

Production **v1.1.0** supports Linux amd64, Windows amd64, macOS Apple Silicon (arm64), and macOS x86_64. Each archive bundles the CLI, the Ainode 2.41 runtime, and a matching keytool. Linux ARM64 and FreeBSD packages are temporarily unavailable in v1.1.0.

Download the archive for your OS below. [Release v1.1.0](https://github.com/DiseAgon/os-pack-dl/releases/tag/v1.1.0).

## Requirements

- Windows 10 64-bit (amd64) or later
- Linux x86_64 (amd64)
- macOS 12+ 64-bit, Apple Silicon (arm64) or x86_64 (amd64)

`start` prints a **card**, then **streams logs**. Other commands print indented JSON. Failures print `{"error": "…"}` on stdout (exit ≠ 0). Success JSON omits the `error` field.

Windows PowerShell: type `.\ai-cli.exe` (the `.\` is required).

Linux and macOS: `./ai-cli` or `ai-cli` if it is on `PATH`.

## Install

Download the archive for your OS, extract it, and rename the inner file to `ai-cli` or `ai-cli.exe`.

`version` prints JSON. `commit` is the source git SHA, not the version tag. This sample is the Linux amd64 build; `built` differs across platform archives.

```json
{
  "built": "2026-10-05T03:27:36Z",
  "commit": "7fab2348b770774f214c557fcbf46a8b9ffb1d4f",
  "version": "1.1.0"
}
```

### Windows

Work in **your** profile folder. PowerShell as that user, not Administrator.

```powershell
cd $env:USERPROFILE
curl.exe -fLO https://github.com/DiseAgon/os-pack-dl/releases/download/v1.1.0/aioz-ai-cli-windows-amd64-1.1.0.zip
Expand-Archive -LiteralPath aioz-ai-cli-windows-amd64-1.1.0.zip -DestinationPath . -Force
Move-Item -Force .\aioz-ai-cli-windows-amd64.exe .\ai-cli.exe
.\ai-cli.exe version
```

`--save-priv-key privkey.json` writes into the current folder. `Access is denied` means that folder is not yours — `cd $env:USERPROFILE` and retry. Data is under `%LOCALAPPDATA%\aioz\ai-cli\`. The first `start` may show a Windows Firewall prompt; allow it for private networks.

### Linux

v1.1.0 is available for `x86_64` (amd64). Check `uname -m` before downloading:

```bash
uname -m
curl -fLO https://github.com/DiseAgon/os-pack-dl/releases/download/v1.1.0/aioz-ai-cli-linux-amd64-1.1.0.tar.gz
tar -xzf aioz-ai-cli-linux-amd64-1.1.0.tar.gz
mv aioz-ai-cli-linux-amd64 ai-cli
./ai-cli version
```

Linux ARM64 is temporarily unavailable in v1.1.0.

### macOS

Pick the archive that matches `uname -m`:

| `uname -m` | Chip | Archive |
|------------|------|---------|
| `arm64` | Apple Silicon (M1–M4) | `aioz-ai-cli-darwin-arm64-1.1.0.tar.gz` |
| `x86_64` | x86_64 | `aioz-ai-cli-darwin-x86_64-1.1.0.tar.gz` |

Apple Silicon:

```bash
curl -fLO https://github.com/DiseAgon/os-pack-dl/releases/download/v1.1.0/aioz-ai-cli-darwin-arm64-1.1.0.tar.gz
tar -xzf aioz-ai-cli-darwin-arm64-1.1.0.tar.gz
mv aioz-ai-cli-darwin-arm64 ai-cli
./ai-cli version
```

x86_64:

```bash
curl -fLO https://github.com/DiseAgon/os-pack-dl/releases/download/v1.1.0/aioz-ai-cli-darwin-x86_64-1.1.0.tar.gz
tar -xzf aioz-ai-cli-darwin-x86_64-1.1.0.tar.gz
mv aioz-ai-cli-darwin-x86_64 ai-cli
./ai-cli version
```

Match the macOS archive to `uname -m`. FreeBSD is temporarily unavailable in v1.1.0.

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

`--save-priv-key` writes the private key JSON. On Linux and macOS its file mode is `0600`; on Windows, keep it in your user profile. Store the mnemonic now; it is not shown again. This does not create a data folder.

```json
{
  "address": "aioz1…",
  "address_hex": "0xAbc0…def1",
  "mnemonic": "word1 word2 … word12",
  "priv_key": "{\"@type\":\"/ethermint.crypto.v1.ethsecp256k1.PrivKey\",\"key\":\"…\"}",
  "priv_key_file": "<absolute-path>",
  "pub_key": "{\"@type\":\"/ethermint.crypto.v1.ethsecp256k1.PubKey\",\"key\":\"…\"}"
}
```

Treat `privkey.json` and the mnemonic as **wallet secrets**. Use a **dedicated key for each node**. Keep an offline backup. Never paste them into websites, chats, or support tickets.

`new` uses 12 words. For 24 words:

**Windows**

```powershell
.\ai-cli.exe keytool new --mnemonic-word 24 --save-priv-key privkey.json
```

**Linux and macOS**

```bash
./ai-cli keytool new --mnemonic-word 24 --save-priv-key privkey.json
```

`new` refuses an existing output file unless you pass `--force`. Next, [set a storage limit](#set-a-storage-limit), then [start the node](#start-the-node). Recover is under [Recover a key](#recover-a-key).

## Commands

Wallet commands take `--priv-key-file` unless noted.

### Set a storage limit

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
  "success": true,
  "state": "confirmation",
  "message": "Storage allocation updated.",
  "storage_limit": "10000000000",
  "storage_used": "0"
}
```

`N` is decimal GB (`10` → `10000000000` bytes on Linux). Raise the cap by running the same command again; it applies on the next `start`.

Without `--home`, this wallet gets a UUID folder:

| OS | Home |
|----|------|
| Linux | `~/.local/share/aioz/ai-cli/ai-nodes/<uuid>/` |
| Windows | `%LOCALAPPDATA%\aioz\ai-cli\ai-nodes\<uuid>\` |
| macOS | `~/Library/Application Support/aioz/ai-cli/ai-nodes/<uuid>/` |

### Start the node

Starts this wallet's node. Prints a card, then streams logs. Ctrl+C stops **this wallet** only. There is no `stop` command. `--priv-key-file` is required.

**Windows**

```powershell
.\ai-cli.exe start --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli start --priv-key-file privkey.json
```

Response:

```
╭─ node ───────────────────────────────────────────────────────────────╮
│                                                                      │
│  Status            running                                           │
│  CLI               1.1.0                                             │
│  Update            CLI is up to date                                 │
│  PID               12345                                             │
│  Home              ~/.local/share/aioz/ai-cli/ai-nodes/<uuid>/       │
│  Storage           10 GB                                             │
│  EVM               0xAbc0…def1                                       │
│                                                                      │
│  ⚠                 streaming logs; Ctrl+C to stop the node           │
│                                                                      │
╰──────────────────────────────────────────────────────────────────────╯
...
```

Logs then stream in the same terminal. The `...` means the command is still running.

Response:

Example output from production v1.1.0. Paths, PID, and the wallet address are placeholders. `data_dir` is the node home. `update.note` is `CLI is up to date` when the signed manifest is reachable. The command keeps running after this object. `ai-cli --json start` does not stream logs unless you also pass `--follow` (logs then go to stderr).

**Windows**

```powershell
.\ai-cli.exe --json start --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli --json start --priv-key-file privkey.json
```

```json
{
  "data_dir": "<node-home>",
  "evm_address": "0xAbc0…def1",
  "home": "<node-home>",
  "log_path": "<log-path>",
  "pid": 12345,
  "pid_path": "<pid-path>",
  "running": true,
  "storage_bytes": 10000000000,
  "update": {
    "skipped": false,
    "newer": false,
    "current": "1.1.0",
    "current_commit": "7fab2348b770774f214c557fcbf46a8b9ffb1d4f",
    "remote": "1.1.0",
    "remote_commit": "7fab2348b770774f214c557fcbf46a8b9ffb1d4f",
    "note": "CLI is up to date"
  }
}
```

Ctrl+C prints a second object: `{"running": 0, "status": "stopped"}`.

Failures (missing key, no storage limit, extra args) print `{"error": "…"}` on stdout.

Log paths:

| OS | `log_path` |
|----|------------|
| Linux | `~/.local/state/aioz/ai-cli/logs/<uuid>/ai.log` |
| Windows | `%LOCALAPPDATA%\aioz\ai-cli\logs\<uuid>\ai.log` |
| macOS | `~/Library/Logs/aioz/ai-cli/<uuid>/ai.log` |

### Clear a node

Stops the selected node and deletes its home, index entry, PID file, and log. The private-key file is kept.

**Windows**

```powershell
.\ai-cli.exe clear --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli clear --priv-key-file privkey.json
```

```json
{"cleared": 1, "running_stopped": 1, "priv_key_kept": true}
```

To clear every indexed home, use `clear --all`. It asks for confirmation; add `--yes` in unattended sessions.

**Windows**

```powershell
.\ai-cli.exe clear --all --yes
```

**Linux and macOS**

```bash
./ai-cli clear --all --yes
```

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

Total earned, total withdrawn, and remaining AIOZ. Works with the node off. Integer amounts use `attoaioz`; `*_aioz` fields are for display.

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
  "balances": {
    "denom": "attoaioz",
    "total_earned": "5641168955315402692",
    "total_withdrawn": "10000000000000000",
    "balance": "5631168955315402692",
    "total_earned_aioz": "5.641168955315402692",
    "total_withdrawn_aioz": "0.01",
    "balance_aioz": "5.631168955315402692"
  }
}
```

### Reward by task

AI rewards grouped by task type. Types appear only when reported by the node.

**Windows**

```powershell
.\ai-cli.exe reward task --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli reward task --priv-key-file privkey.json
```

```json
{
  "total": {
    "denom": "attoaioz",
    "amount": "5641168955315402692",
    "aioz": "5.641168955315402692"
  },
  "task_types": {
    "inference": {
      "amount": "1410292238828850673",
      "count": 1,
      "aioz": "1.410292238828850673"
    },
    "training": {
      "amount": "1410292238828850673",
      "count": 1,
      "aioz": "1.410292238828850673"
    },
    "evaluation": {
      "amount": "1410292238828850673",
      "count": 1,
      "aioz": "1.410292238828850673"
    },
    "challenge": {
      "amount": "1410292238828850673",
      "count": 1,
      "aioz": "1.410292238828850673"
    }
  }
}
```

### Withdraw history

Previously sent withdrawals, with UTC time and transaction hash.

**Windows**

```powershell
.\ai-cli.exe reward withdraw history --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli reward withdraw history --priv-key-file privkey.json
```

```json
{
  "total": {
    "denom": "attoaioz",
    "amount": "10000000000000000",
    "aioz": "0.01"
  },
  "count": 1,
  "items": [
    {
      "amount": "10000000000000000",
      "aioz": "0.01",
      "timestamp": 1788504168.5052338,
      "time": "2026-09-04T06:42:48.505233765Z",
      "tx_hash": "…"
    }
  ]
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
  "current": "1.1.0",
  "current_commit": "7fab2348b770774f214c557fcbf46a8b9ffb1d4f",
  "remote": "1.1.0",
  "remote_commit": "7fab2348b770774f214c557fcbf46a8b9ffb1d4f",
  "note": "CLI is up to date"
}
```

`update --check-only` prints status and does not download. `update --stop` stops every node on this machine first, then replaces the binary.

### Status

This wallet’s local state, process CPU and GPU use, and measured storage.

**Windows**

```powershell
.\ai-cli.exe status --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli status --priv-key-file privkey.json
```

Response:

```json
{
  "address_hex": "0xAbc0…def1",
  "home": "~/.local/share/aioz/ai-cli/ai-nodes/<uuid>",
  "log_path": "~/.local/state/aioz/ai-cli/logs/<uuid>/ai.log",
  "running": true,
  "pid": 12345,
  "state": "success",
  "cpu": {
    "cores": 8,
    "message": "Node is running."
  },
  "storage": {
    "state": "success",
    "percent": 83,
    "used": "487000000000",
    "limit": "581000000000",
    "message": "Storage in use."
  }
}
```

`status --all` samples the local machine and lists every indexed wallet, including stopped nodes. No `--priv-key-file`.

**Windows**

```powershell
.\ai-cli.exe status --all
```

**Linux and macOS**

```bash
./ai-cli status --all
```

```json
{
  "sample": {
    "sampled_at": "2026-10-02T07:34:00Z",
    "cpu_percent": 6.0,
    "load_average": [
      0.42,
      0.38,
      0.31
    ],
    "memory_available_bytes": 87001346048,
    "gpus": [
      {
        "index": 0,
        "name": "NVIDIA TITAN V",
        "percent": 10,
        "vram_total_bytes": 12884901888,
        "vram_used_bytes": 1223688192
      }
    ],
    "filesystems": [
      {
        "mount": "/",
        "total_bytes": 980122034176,
        "used_bytes": 326507765760,
        "available_bytes": 603751333888
      }
    ]
  },
  "wallets": [
    {
      "home": "<uuid-1>",
      "path": "<node-home-1>",
      "state": "success",
      "running": true,
      "pid": 3471060,
      "cpu_percent": 3.5,
      "memory_rss_bytes": 1800000000,
      "gpu_percent": 8,
      "gpu_vram_bytes": 2684354560,
      "storage": {
        "state": "success",
        "limit_bytes": 10000000000,
        "used_bytes": 1200000000,
        "remain_bytes": 8800000000,
        "remain_percent": 88.0,
        "measured_at": "2026-10-02T07:34:00Z"
      }
    },
    {
      "home": "<uuid-2>",
      "path": "<node-home-2>",
      "state": "off",
      "running": false,
      "cpu_percent": 0,
      "gpu_percent": 0,
      "gpu_vram_bytes": 0
    }
  ]
}
```

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

Checks disk, runtime, wallet, GPU, and this wallet’s log. The `machine` object reports relatively stable hardware details; `status --all` gives a fresh host sample on each invocation.

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
    {
      "name": "os",
      "ok": true,
      "detail": "linux/amd64"
    },
    {
      "name": "home",
      "ok": true,
      "detail": "~/.local/share/aioz/ai-cli/ai-nodes/<uuid>"
    },
    {
      "name": "disk",
      "ok": true,
      "detail": "100 GB free"
    },
    {
      "name": "runtime",
      "ok": true,
      "detail": "ok"
    },
    {
      "name": "wallet",
      "ok": true,
      "detail": "0xAbc0…def1"
    },
    {
      "name": "log",
      "ok": true,
      "detail": "~/.local/state/aioz/ai-cli/logs/<uuid>/ai.log"
    },
    {
      "name": "identity",
      "ok": true,
      "detail": "credential is --priv-key-file"
    },
    {
      "name": "gpu",
      "ok": true,
      "detail": "NVIDIA, 8 GB"
    }
  ],
  "machine": {
    "os": "linux",
    "os_name": "Ubuntu 24.04 LTS",
    "kernel": "6.8.0-generic",
    "architecture": "amd64",
    "uptime_seconds": 720000,
    "cpu": {
      "model": "AMD Ryzen Threadripper 1950X",
      "sockets": 1,
      "cores": 16,
      "threads": 32,
      "min_mhz": 2200,
      "max_mhz": 3400
    },
    "memory": {
      "total_bytes": 101202948096,
      "swap_total_bytes": 8589934592
    },
    "gpus": [
      {
        "index": 0,
        "name": "NVIDIA TITAN V",
        "vram_bytes": 12884901888,
        "compute_capability": "7.0",
        "driver_version": "580.178.04",
        "cuda_version": "13.0"
      }
    ],
    "disks": [
      {
        "name": "sda",
        "model": "Samsung SSD 860 EVO",
        "size_bytes": 1000204886016,
        "rotational": false
      }
    ],
    "nics": [
      {
        "name": "enp5s0",
        "speed_mbps": 1000,
        "mtu": 1500
      }
    ],
    "filesystems": [
      {
        "mount": "/",
        "total_bytes": 980122034176,
        "used_bytes": 326507765760,
        "available_bytes": 603751333888
      }
    ]
  }
}
```

### Encrypt, decrypt, and sign

`keytool encrypt`, `decrypt`, and `sign` use the bundled keytool. These CLI commands accept the key material as positional arguments. Keep private material out of shell history when using them.

Encrypt a private-key JSON string with `-P`:

**Windows**

```powershell
.\ai-cli.exe keytool encrypt -P <password> '<priv-key-json>'
```

**Linux and macOS**

```bash
./ai-cli keytool encrypt -P <password> '<priv-key-json>'
```

```json
{"address": "aioz1…", "address_hex": "0xAbc0…def1", "priv_armor": "<encrypted-armor>"}
```

Decrypt that armor (the `--` separates flags from the armor):

**Windows**

```powershell
.\ai-cli.exe keytool decrypt -P <password> -- '<encrypted-armor>'
```

**Linux and macOS**

```bash
./ai-cli keytool decrypt -P <password> -- '<encrypted-armor>'
```

```json
{"address": "aioz1…", "address_hex": "0xAbc0…def1", "priv_key": "<private-key-json>", "pub_key": "<public-key-json>"}
```

Sign data with the encrypted armor:

**Windows**

```powershell
.\ai-cli.exe keytool sign -P <password> -- '<encrypted-armor>' 'message'
```

**Linux and macOS**

```bash
./ai-cli keytool sign -P <password> -- '<encrypted-armor>' 'message'
```

```json
{"signature": "<base64-signature>"}
```

### Show address

Prints `address_hex` and `address`. Never prints the private key.

**Windows**

```powershell
.\ai-cli.exe keytool show --priv-key-file privkey.json
```

**Linux and macOS**

```bash
./ai-cli keytool show --priv-key-file privkey.json
```

```json
{
  "address": "aioz1…",
  "address_hex": "0xAbc0…def1"
}
```

### Export a key copy

Copies `--priv-key-file` to `--out`. The key is never printed. An existing `--out` file is replaced.

**Windows**

```powershell
.\ai-cli.exe keytool export --priv-key-file privkey.json --out backup-privkey.json
```

**Linux and macOS**

```bash
./ai-cli keytool export --priv-key-file privkey.json --out backup-privkey.json
```

```json
{
  "out": "backup-privkey.json"
}
```

### Recover a key

A 12-word phrase is the usual case. A 24-word phrase also works. Put the original words in a private text file and use `--mnemonic-file`.

**Windows**

```powershell
.\ai-cli.exe keytool recover --mnemonic-file words.txt --save-priv-key privkey.json
```

**Linux and macOS**

```bash
./ai-cli keytool recover --mnemonic-file words.txt --save-priv-key privkey.json
```

`recover` refuses an existing output file unless you pass `--force`. It returns the recovered addresses and key JSON, but does not echo the mnemonic.

```json
{
  "address": "aioz1…",
  "address_hex": "0xAbc0…def1",
  "priv_key": "{\"@type\":\"/ethermint.crypto.v1.ethsecp256k1.PrivKey\",\"key\":\"…\"}",
  "priv_key_file": "<absolute-path>",
  "pub_key": "{\"@type\":\"/ethermint.crypto.v1.ethsecp256k1.PubKey\",\"key\":\"…\"}"
}
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
| `need mnemonic words or --mnemonic-file` | Pass the private phrase file with `--mnemonic-file`. |
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

## Documentation

The [operator guide](./docs/aioz-ai-cli.md) includes installation, all v1.1.0 commands, and complete JSON examples. See the [release notes](./RELEASE.md) for new features and bug fixes.
