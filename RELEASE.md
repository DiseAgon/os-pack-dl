# v1.1.0

Production release of AIOZ AI CLI with the newly supplied Ainode 2.41 payloads. The CLI, node runtime, and matching keytool are bundled in each platform archive.

## Highlights

### New features

- `keytool encrypt`, `keytool decrypt`, and `keytool sign` support private-key armor and signing. Keytool JSON now uses the DePIN fields `address`, `address_hex`, `pub_key`, `priv_key`, and `priv_armor` where applicable.
- `reward balance` reports total earned, total withdrawn, and remaining balance; `reward task` groups earnings by task type; `reward withdraw history` lists previous withdrawals.
- `status` reports node state, CPU/GPU use, and measured storage. `status --all` adds a fresh local machine sample and every indexed wallet, including stopped wallets. `doctor` adds hardware details for machine diagnostics.
- `clear --priv-key-file` removes one node home; `clear --all` removes every indexed home after confirmation. Private-key files are retained. `storage limit` confirms the updated allocation in JSON.
- Production starts Ainode with dynamic `-wd`, `-dc`, and `--cli-run`; one-shot Ainode queries use `--cli` before the command flag. The production build omits the demo hub/mode arguments and hidden development flags.

### Bug fixes

- `keytool recover` succeeds when the sidecar omits optional secrets; it writes the recovered private-key file without a misleading error. Recovery JSON does not echo the mnemonic.
- `keytool new` and `keytool recover` now honor `--force` when replacing an existing output file.
- Windows runtime launch preserves NVIDIA driver paths.
- Update checks and manifest downloads refresh the GitHub latest-release redirect, preventing a stale release from being offered as an update.

## Supported platforms

- Linux amd64 (`x86_64`)
- Windows amd64
- macOS Apple Silicon (arm64)
- macOS Intel (x86_64)

Linux ARM64 and FreeBSD packages are temporarily omitted from v1.1.0. Choose the archive matching your OS and CPU. Each archive contains one production CLI executable. The signed `manifest.json` contains SHA256 checksums for the four archives and the source commit `7fab2348b770774f214c557fcbf46a8b9ffb1d4f`.

## Documentation

Install steps and complete example JSON outputs: [README](./README.md) and the [operator guide](./docs/aioz-ai-cli.md). The CLI source remains on internal GitLab.
