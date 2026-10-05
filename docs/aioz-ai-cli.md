# AIOZ AI CLI

The command-line tool for running and managing an AIOZ AI Node. This guide matches production **v1.1.0** published at [GitHub](https://github.com/DiseAgon/os-pack-dl/releases/tag/v1.1.0). It bundles Ainode 2.41 for Linux amd64, Windows amd64, macOS arm64, and macOS x86_64.

## What is AIOZ AI CLI?

AIOZ AI CLI creates a wallet, sets a storage cap, starts the node, and exposes status, logs, rewards, stats, and diagnostics. The node runtime and keytool are bundled in the CLI binary.

`start` prints a status card and streams logs. Other commands print indented JSON. Failures print `{"error": "…"}` on stdout (non-zero exit). Success JSON omits the error field. For JSON from `start`: `ai-cli --json start --priv-key-file privkey.json`.

There is no `stop` command. Ctrl+C on attached `start` stops this wallet only.

## Requirements

- Windows 10 64-bit (amd64) or later
- Linux x86_64 (amd64)
- macOS 12 or later, Apple Silicon (arm64) or x86_64 (amd64)

## Getting started

### Windows

Download and extract the published v1.1.0 archive. The scripts below are written for Windows PowerShell.

Work in **your** profile folder. Run PowerShell as that user, not Administrator:

```
cd $env:USERPROFILE
curl.exe -fLO https://github.com/DiseAgon/os-pack-dl/releases/download/v1.1.0/aioz-ai-cli-windows-amd64-1.1.0.zip
Expand-Archive -LiteralPath aioz-ai-cli-windows-amd64-1.1.0.zip -DestinationPath . -Force
Move-Item -Force .\aioz-ai-cli-windows-amd64.exe .\ai-cli.exe
```

Verify the installation

```
.\ai-cli.exe version
```

The output should be the version of AIOZ AI CLI. The `built` value below is from the Windows amd64 archive; the other platform builds have their own timestamps.

```
{
  "built": "2026-10-05T03:27:39Z",
  "commit": "7fab2348b770774f214c557fcbf46a8b9ffb1d4f",
  "version": "1.1.0"
}
```

On PowerShell, type `.\ai-cli.exe` (the `.\` is required; `ai-cli.exe` alone is not found).

`--save-priv-key privkey.json` writes into the current folder. `Access is denied` means that folder is not yours — `cd $env:USERPROFILE` and retry. Node data is under `%LOCALAPPDATA%\aioz\ai-cli\`. The first `start` may show a Windows Firewall prompt; allow it for private networks.

Generate a new mnemonic phrase and private key

```
.\ai-cli.exe keytool new --save-priv-key privkey.json
```

`keytool new` requires `--save-priv-key`. Keep this secret file in your user profile; `priv_key_file` is its absolute path. `-P` or `--password` adds `priv_armor`. Example: `.\ai-cli.exe keytool new -P 12345678 --save-priv-key privkey.json`.

Response

```
{
  "address": "aioz1…",
  "address_hex": "0xAbc0…def1",
  "mnemonic": "word1 word2 … word12",
  "priv_key": "{\"@type\":\"/ethermint.crypto.v1.ethsecp256k1.PrivKey\",\"key\":\"…\"}",
  "priv_key_file": "<absolute-path>",
  "pub_key": "{\"@type\":\"/ethermint.crypto.v1.ethsecp256k1.PubKey\",\"key\":\"…\"}"
}
```

**IMPORTANT NOTE**

- Treat your `privkey.json` and mnemonic phrase as **wallet secrets**. Anyone who has them can **control your node rewards**.
- Use a **dedicated key for each node**; **do not reuse** the key from your main wallet, exchange account, or other nodes.
- Store backups in an **offline, encrypted location** such as encrypted USB drives, air-gapped devices, or hardware security modules (HSM).
- **Never paste** your mnemonic or private key into websites, chats, or support tickets.

Set a storage limit. This is **required before `start`**. The value must be **greater than 2 GB**. There is no 2 GB default:

```
.\ai-cli.exe storage limit 10 --priv-key-file privkey.json
```

Response

```
{
  "success": true,
  "state": "confirmation",
  "message": "Storage allocation updated.",
  "storage_limit": "10000000000",
  "storage_used": "0"
}
```

You can run the same command later to raise the cap. It is applied on the next `start`.

Start the node

```
.\ai-cli.exe start --priv-key-file privkey.json
```

`--priv-key-file` the private key file which the node starts with.

Default `start` prints a card, then streams logs. Ctrl+C stops **this wallet only**.

Without `--home`, data for this wallet is `%LOCALAPPDATA%\aioz\ai-cli\ai-nodes\<uuid>\`.

### macOS and Linux

Download the published v1.1.0 archive for your operating system and CPU.

For macOS - ARM64 (`uname -m` is `arm64`)

```
curl -fLO https://github.com/DiseAgon/os-pack-dl/releases/download/v1.1.0/aioz-ai-cli-darwin-arm64-1.1.0.tar.gz
tar -xzf aioz-ai-cli-darwin-arm64-1.1.0.tar.gz
mv aioz-ai-cli-darwin-arm64 ai-cli
xattr -dr com.apple.quarantine ./ai-cli
```

For macOS - x86_64 (`uname -m` is `x86_64`)

```
curl -fLO https://github.com/DiseAgon/os-pack-dl/releases/download/v1.1.0/aioz-ai-cli-darwin-x86_64-1.1.0.tar.gz
tar -xzf aioz-ai-cli-darwin-x86_64-1.1.0.tar.gz
mv aioz-ai-cli-darwin-x86_64 ai-cli
xattr -dr com.apple.quarantine ./ai-cli
```

Do not use the x86_64 archive on Apple Silicon.

For Linux, pick the archive that matches `uname -m`:

| `uname -m` | CPU | Archive |
|------------|-----|---------|
| `x86_64` | amd64 | `aioz-ai-cli-linux-amd64-1.1.0.tar.gz` |

x86_64:

```
curl -fLO https://github.com/DiseAgon/os-pack-dl/releases/download/v1.1.0/aioz-ai-cli-linux-amd64-1.1.0.tar.gz
tar -xzf aioz-ai-cli-linux-amd64-1.1.0.tar.gz
mv aioz-ai-cli-linux-amd64 ai-cli
```


The archive contains the CLI, node runtime, and keytool. Run it as `./ai-cli`; no Go toolchain or separate payload download is needed. Linux ARM64 and FreeBSD are not included in v1.1.0.

Verify the installation

```
./ai-cli version
```

The output should be the version of AIOZ AI CLI.

Generate a new mnemonic phrase and private key

```
./ai-cli keytool new --save-priv-key privkey.json
```

`keytool new` requires `--save-priv-key`. The file is mode 0600 and `priv_key_file` is the absolute path. `-P` or `--password` adds `priv_armor`. Example: `ai-cli keytool new -P 12345678 --save-priv-key privkey.json`.

Response

```
{
  "address": "aioz1…",
  "address_hex": "0xAbc0…def1",
  "mnemonic": "word1 word2 … word12",
  "priv_key": "{\"@type\":\"/ethermint.crypto.v1.ethsecp256k1.PrivKey\",\"key\":\"…\"}",
  "priv_key_file": "<absolute-path>",
  "pub_key": "{\"@type\":\"/ethermint.crypto.v1.ethsecp256k1.PubKey\",\"key\":\"…\"}"
}
```

**IMPORTANT NOTE**

- Treat your `privkey.json` and mnemonic phrase as **wallet secrets**. Anyone who has them can **control your node rewards**.
- Use a **dedicated key for each node**; **do not reuse** the key from your main wallet, exchange account, or other nodes.
- Store backups in an **offline, encrypted location** such as encrypted USB drives, air-gapped devices, or hardware security modules (HSM).
- **Never paste** your mnemonic or private key into websites, chats, or support tickets.

Set a storage limit. This is **required before `start`**. The value must be **greater than 2 GB**. There is no 2 GB default:

```
./ai-cli storage limit 10 --priv-key-file privkey.json
```

Start the node

```
./ai-cli start --priv-key-file privkey.json
```

`--priv-key-file` the private key file which the node starts with.

Default `start` prints a card, then streams logs. Ctrl+C stops **this wallet only**.

Without `--home`, data for this wallet is Linux `~/.local/share/aioz/ai-cli/ai-nodes/<uuid>/` or macOS `~/Library/Application Support/aioz/ai-cli/ai-nodes/<uuid>/`.


## Usage

### Start the node

Set a storage limit greater than 2 GB for this wallet before the first start. Then keep the terminal open while the node runs:

For Windows

```
.\ai-cli.exe storage limit 10 --priv-key-file privkey.json
.\ai-cli.exe start --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli storage limit 10 --priv-key-file privkey.json
./ai-cli start --priv-key-file privkey.json
```

The default output is a status card followed by live logs. A Linux example (the home path wraps within the card):

```text
╭─ node ───────────────────────────────────────────────────────────────╮
│                                                                      │
│  Status            running                                           │
│  CLI               1.1.0                                             │
│  Update            CLI is up to date                                 │
│  PID               12345                                             │
│  Home              /home/user/.local/share/aioz/ai-cli/ai-nodes/     │
│                    <uuid>/                                           │
│  Storage           10 GB                                             │
│  EVM               0xAbc0…def1                                       │
│                                                                      │
│  ⚠                 streaming logs; Ctrl+C to stop the node           │
│                                                                      │
╰──────────────────────────────────────────────────────────────────────╯
```

Ctrl+C stops this wallet's node. There is no `stop` command. Use [`--json`](#json-start) for a machine-readable start response.

### View node status

For Windows

```
.\ai-cli.exe status --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli status --priv-key-file privkey.json
```

Response

```
{
  "address_hex": "0xAbc0…def1",
  "home": "~/.local/share/aioz/ai-cli/ai-nodes/<uuid>",
  "log_path": "~/.local/state/aioz/ai-cli/logs/<uuid>/ai.log",
  "running": true,
  "pid": 12345,
  "state": "success",
  "cpu": { "cores": 8, "message": "Node is running." },
  "storage": {
    "state": "success",
    "percent": 83,
    "used": "487000000000",
    "limit": "581000000000",
    "message": "Storage in use."
  }
}
```

`state` is `off`, `start`, `success`, or `alert`. `cpu.percent` is omitted until this process can be measured. `gpu` is present only when `nvidia-smi` returns a card, and only while the node is not `off`. `storage.state` is `alert` when used storage has reached the limit.

`status --all` returns one local machine sample and a `wallets` array. It does not need `--priv-key-file` and does not call the hub. Each indexed wallet remains in the array while stopped. A live process that is not in the index is added with `state` `alert`.

```json
{
  "sample": {
    "sampled_at": "2026-10-02T07:34:00Z",
    "cpu_percent": 6.0,
    "load_average": [0.42, 0.38, 0.31],
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

Host `sample` values are measured on each invocation. Wallet CPU and GPU values sum the node process and its descendants. `storage` uses the local limit in `settings.json` and scans `storage_dir` on each invocation; `measured_at` records when the scan completed. The CLI does not cache this response. The UI can retain it and decide when to request a fresh sample. The `doctor` response also includes a `machine` object for relatively stable CPU, memory, GPU, disk, filesystem, and network-adapter details; call it once when the UI opens rather than polling it.

List every indexed home on this machine (`--priv-key-file` is not required):

For Windows

```
.\ai-cli.exe status --all
```

For macOS and Linux

```
./ai-cli status --all
```

`status --all` reads the index (`instances.json` on Linux: `~/.config/aioz/ai-cli/`). Deleting only `~/.local/share/aioz/ai-cli/ai-nodes/` does not clear the list.

### Clear a node home

`clear` stops the node if it is running, then deletes that home and its index entry. The private key file is kept.

For Windows

```
.\ai-cli.exe clear --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli clear --priv-key-file privkey.json
```

Delete every indexed home on this machine. `clear --all` prints a warning and continues only after `y`. Without a terminal, pass `--yes`. It deletes each home's data directory, pid file, and log. The private key file is kept.

For Windows

```
.\ai-cli.exe clear --all
```

For macOS and Linux

```
./ai-cli clear --all
```

`clear` with no flag fails (`--priv-key-file is required`). Do not pass `--priv-key-file` and `--all` together.

Response

```
{
  "cleared": 1,
  "running_stopped": 1,
  "priv_key_kept": true
}
```

### Set storage limit

For Windows

```
.\ai-cli.exe storage limit 10 --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli storage limit 10 --priv-key-file privkey.json
```

This command sets the total storage limit in GB. The value must be **greater than 2 GB**. You cannot start until a limit is set. You can raise the limit later; it applies on the next `start`.

Response

```
{
  "success": true,
  "state": "confirmation",
  "message": "Storage allocation updated.",
  "storage_limit": "10000000000",
  "storage_used": "0"
}
```

### View storage

For Windows

```
.\ai-cli.exe storage show --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli storage show --priv-key-file privkey.json
```

Response

```
{
  "storage_limit": "10000000000",
  "storage_used": "0"
}
```

`storage_limit` and `storage_used` are byte strings. Bare `storage` prints help.

### View reward

For Windows

```
.\ai-cli.exe reward balance --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli reward balance --priv-key-file privkey.json
```

Works with the node off. Amounts are integer `attoaioz` strings. Each `*_aioz` field is the same amount in AIOZ, for display.

Response

```
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

`total_earned` is the AI reward since the beginning. `total_withdrawn` is what has already been sent. `balance` is what is left to withdraw.

### Reward by task

For Windows

```
.\ai-cli.exe reward task --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli reward task --priv-key-file privkey.json
```

Response

```
{
  "total": {
    "denom": "attoaioz",
    "amount": "5641168955315402692",
    "aioz": "5.641168955315402692"
  },
  "task_types": {
    "inference": { "amount": "1410292238828850673", "count": 1, "aioz": "1.410292238828850673" },
    "training": { "amount": "1410292238828850673", "count": 1, "aioz": "1.410292238828850673" },
    "evaluation": { "amount": "1410292238828850673", "count": 1, "aioz": "1.410292238828850673" },
    "challenge": { "amount": "1410292238828850673", "count": 1, "aioz": "1.410292238828850673" }
  }
}
```

A task type is present only when the node reports it. `count` is how many tasks of that type earned the `amount`.

### Withdraw history

For Windows

```
.\ai-cli.exe reward withdraw history --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli reward withdraw history --priv-key-file privkey.json
```

Response

```
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

`timestamp` is unix seconds from the node. `time` is the same instant in UTC.

### Withdraw reward

For Windows

```
.\ai-cli.exe reward withdraw --address 0xAbc0…def1 --amount 1 --priv-key-file privkey.json --yes
```

For macOS and Linux

```
./ai-cli reward withdraw --address 0xAbc0…def1 --amount 1 --priv-key-file privkey.json --yes
```

`--address` the address to withdraw the reward to. The reward is transferred on the **AIOZ Chain**. Use a MetaMask `0x` address.

`--amount` the amount of reward to withdraw, in AIOZ. Minimum withdrawal is **0.01 AIOZ**.

`--priv-key-file` the private key file that the node started with.

`--yes` skips the confirm prompt.

Response

```
{
  "txid": "2604F553…944D59"
}
```

### Recover private key from mnemonic phrase

You can recover with the original **12-word phrase** (a 24-word phrase also works). Keep the words in their original order in a private file:

Use `--mnemonic-file` so the phrase stays out of shell history:

For Windows

```
.\ai-cli.exe keytool recover --mnemonic-file words.txt --save-priv-key privkey.json
```

For macOS and Linux

```
./ai-cli keytool recover --mnemonic-file words.txt --save-priv-key privkey.json
```

The response contains the recovered addresses and key JSON. It does not echo the mnemonic.

```
{
  "address": "aioz1…",
  "address_hex": "0xAbc0…def1",
  "priv_key": "{\"@type\":\"/ethermint.crypto.v1.ethsecp256k1.PrivKey\",\"key\":\"…\"}",
  "priv_key_file": "<absolute-path>",
  "pub_key": "{\"@type\":\"/ethermint.crypto.v1.ethsecp256k1.PubKey\",\"key\":\"…\"}"
}
```

Wrong word count:

```
{
  "error": "mnemonic has 11 words (too few); need 12 or 24"
}
```

No words and no file: `{"error": "need mnemonic words or --mnemonic-file"}`.

### Keytool usage

`keytool` has seven commands:

- `new` runs the keytool sidecar and prints wallet JSON (`address`, `address_hex`, `pub_key`, `priv_key`, `mnemonic`). `--save-priv-key FILE` is required. `-P` / `--password` adds `priv_armor`. Default mnemonic length is 12 words; `--mnemonic-word 24` selects 24.
- `recover` restores a wallet through the same sidecar from a 12- or 24-word `--mnemonic-file FILE`.
- `encrypt [priv-key]` runs the sidecar and prints `address`, `address_hex`, and `priv_armor`. `-P` / `--password` is required to encrypt; the sidecar reads it from a file, not from its own arguments.
- `decrypt [priv-armor]` runs the sidecar and prints `address`, `address_hex`, `pub_key`, and `priv_key`. Pass `--` before an armor string that starts with `-`.
- `sign [priv-armor] [data]` runs the sidecar and prints `signature`. The data is the only sidecar argument; the armor and password stay in files.
- `show --priv-key-file FILE` prints the wallet addresses without printing the private key. After a node has started, `keytool show --home PATH` can read the address recorded in that node home without the key file.
- `export --priv-key-file FILE --out FILE` copies the private key to another file. The key is never printed to stdout; the destination file is also a wallet secret.

To create a wallet with a 24-word mnemonic:

For Windows

```
.\ai-cli.exe keytool new --mnemonic-word 24 --save-priv-key privkey.json
```

For macOS and Linux

```
./ai-cli keytool new --mnemonic-word 24 --save-priv-key privkey.json
```

To view the addresses for a private key file:

For Windows

```
.\ai-cli.exe keytool show --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli keytool show --priv-key-file privkey.json
```

Response (addresses are placeholders):

```json
{
  "address": "aioz1…",
  "address_hex": "0xAbc0…def1"
}
```

To copy a private key file:

For Windows

```
.\ai-cli.exe keytool export --priv-key-file privkey.json --out backup-privkey.json
```

For macOS and Linux

```
./ai-cli keytool export --priv-key-file privkey.json --out backup-privkey.json
```

Response:

```json
{
  "out": "backup-privkey.json"
}
```

`new` and `recover` refuse an existing output file unless you pass `--force`, which replaces that file. `export` replaces an existing `--out` file. Use `ai-cli keytool <command> --help` for the flags of each command.

### Encrypt, decrypt, and sign

These commands use the bundled keytool. The CLI accepts key material as positional arguments; avoid storing real secrets in shell history. The values below are placeholders.

For Windows:

```powershell
.\ai-cli.exe keytool encrypt -P <password> '<priv-key-json>'
.\ai-cli.exe keytool decrypt -P <password> -- '<encrypted-armor>'
.\ai-cli.exe keytool sign -P <password> -- '<encrypted-armor>' 'message'
```

For macOS and Linux:

```bash
./ai-cli keytool encrypt -P <password> '<priv-key-json>'
./ai-cli keytool decrypt -P <password> -- '<encrypted-armor>'
./ai-cli keytool sign -P <password> -- '<encrypted-armor>' 'message'
```

`encrypt` returns:

```json
{"address": "aioz1…", "address_hex": "0xAbc0…def1", "priv_armor": "<encrypted-armor>"}
```

`decrypt` returns:

```json
{"address": "aioz1…", "address_hex": "0xAbc0…def1", "priv_key": "<private-key-json>", "pub_key": "<public-key-json>"}
```

`sign` returns:

```json
{"signature": "<base64-signature>"}
```

### Errors

Every failed command prints JSON on stdout and exits non-zero. That includes `start` and `keytool`.

```
{
  "error": "--priv-key-file is required"
}
```

Wrong input names the actual mistake:

- missing `--priv-key-file` on `start`, `status`, `storage`, or `reward`
- recover: not 12 or 24 words, or both the phrase and `--mnemonic-file`
- `storage limit abc` / `10GB` → `storage limit must be a number of GB` (not the 2 GB floor)
- `start extra` → `start does not take arguments`
- `start` before `storage limit` → set a storage limit first
- One live node per wallet; Ctrl+C on `start` stops this wallet only

### Update

For Windows

```
.\ai-cli.exe update
```

For macOS and Linux

```
./ai-cli update
```

Downloads the matching archive for this OS from the latest release. `update --check-only` prints status and does not download. `update --stop` stops every node on this machine first, then replaces the binary.

### Stats

For Windows

```
.\ai-cli.exe stats --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli stats --priv-key-file privkey.json
```

Response

```
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

### Logs

For Windows

```
.\ai-cli.exe logs --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli logs --priv-key-file privkey.json
```

Prints a snapshot of this wallet's `ai.log` (redacted on screen). Live logs stream from attached `start`.

### Doctor

For Windows

```
.\ai-cli.exe doctor --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli doctor --priv-key-file privkey.json
```

Response

```
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
    "memory": {"total_bytes": 101202948096, "swap_total_bytes": 8589934592},
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
      {"name": "sda", "model": "Samsung SSD 860 EVO", "size_bytes": 1000204886016, "rotational": false}
    ],
    "nics": [{"name": "enp5s0", "speed_mbps": 1000, "mtu": 1500}],
    "filesystems": [
      {"mount": "/", "total_bytes": 980122034176, "used_bytes": 326507765760, "available_bytes": 603751333888}
    ]
  }
}
```

`machine` contains relatively stable hardware details for a later UI. Call `doctor` when opening the machine page or when the operator requests a fresh hardware check; do not poll it alongside `status --all`.

### JSON start

For Windows

```
.\ai-cli.exe --json start --priv-key-file privkey.json
```

For macOS and Linux

```
./ai-cli --json start --priv-key-file privkey.json
```

Example output from production v1.1.0 (paths, PID, and wallet address are placeholders). `data_dir` is the node home. `update.current` is `1.1.0` and `note` is `CLI is up to date` when the signed manifest is reachable:

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

When you stop the attached command with Ctrl+C, it prints a second JSON object:

```json
{
  "running": 0,
  "status": "stopped"
}
```

Logs are not streamed in JSON mode unless you also pass `--follow`; those logs go to stderr.
