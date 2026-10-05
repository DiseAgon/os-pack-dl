# v1.1.0

AIOZ AI CLI v1.1.0 bundles Ainode 2.41 and a matching keytool for each supported platform.

## Highlights

### New features

- `keytool encrypt`, `keytool decrypt`, and `keytool sign` support private-key armor and signing. The commands return the relevant wallet address, key, armor, or signature fields in JSON.
- `reward balance` reports total earned, total withdrawn, and remaining balance; `reward task` groups earnings by task type; `reward withdraw history` lists previous withdrawals.
- `status` reports node state, CPU/GPU use, and measured storage. `status --all` adds a fresh local machine sample and every indexed wallet, including stopped wallets. `doctor` adds hardware details for machine diagnostics.
- `clear --priv-key-file` removes one node home; `clear --all` removes every indexed home after confirmation. Private-key files are retained. `storage limit` confirms the updated allocation in JSON.

### Bug fixes

- Recovering a wallet from a mnemonic no longer reports an error after successfully writing the private-key file. Recovery JSON does not echo the mnemonic.
- `keytool new` and `keytool recover` now honor `--force` when replacing an existing output file.
- Improved node startup on Windows systems with NVIDIA drivers.
- Fixed false update prompts caused by stale release information.

## Supported platforms

- Linux amd64 (`x86_64`)
- Windows amd64
- macOS Apple Silicon (arm64)
- macOS Intel (x86_64)

Linux ARM64 and FreeBSD packages are temporarily omitted from v1.1.0. Choose the archive matching your OS and CPU. Each archive contains one production CLI executable. The signed `manifest.json` contains SHA256 checksums for the four archives and the source commit `7fab2348b770774f214c557fcbf46a8b9ffb1d4f`.

## Documentation

Install steps and complete example JSON outputs: [README](./README.md) and the [operator guide](./docs/aioz-ai-cli.md).
