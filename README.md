# clavik

Command-line client for the [Clavik vault](https://clavik.io).

`clavik` manages secrets, encryption keys, folders and access from a terminal or
a CI job: store and retrieve credentials, encrypt and decrypt payloads, sign and
verify data, inspect the audit log, and query compliance and security policy.

This repository distributes **release binaries**. The source is not published
here.

## Install

```bash
brew install clavik-io/tap/clavik
```

Homebrew works on macOS and Linux. Upgrade with `brew upgrade clavik`.

> Installed with Homebrew before v1.0.1? Earlier releases were a formula, which
> has been replaced by a cask. Switch once:
> `brew uninstall clavik && brew install --cask clavik-io/tap/clavik`

To download instead, pick the archive for your platform from the
[latest release](https://github.com/clavik-io/clavik-cli/releases/latest):

| Platform | Archive |
| --- | --- |
| macOS, Apple silicon | `clavik_<version>_darwin_arm64.tar.gz` |
| macOS, Intel | `clavik_<version>_darwin_amd64.tar.gz` |
| Linux, x86-64 | `clavik_<version>_linux_amd64.tar.gz` |
| Linux, arm64 | `clavik_<version>_linux_arm64.tar.gz` |
| Linux, 32-bit ARM (ARMv7) | `clavik_<version>_linux_armv7.tar.gz` |
| Windows, x86-64 | `clavik_<version>_windows_amd64.zip` |
| Windows, arm64 | `clavik_<version>_windows_arm64.zip` |

For example, on Linux x86-64:

```bash
VERSION=$(curl -sSL https://api.github.com/repos/clavik-io/clavik-cli/releases/latest \
  | grep -o '"tag_name": *"[^"]*"' | cut -d'"' -f4)
curl -sSLO "https://github.com/clavik-io/clavik-cli/releases/download/${VERSION}/clavik_${VERSION}_linux_amd64.tar.gz"
curl -sSLO "https://github.com/clavik-io/clavik-cli/releases/download/${VERSION}/checksums.txt"
shasum -a 256 -c checksums.txt --ignore-missing
tar -xzf "clavik_${VERSION}_linux_amd64.tar.gz" clavik && sudo mv clavik /usr/local/bin/
```

Each archive holds the `clavik` binary and the license. The binaries are
statically linked and have no runtime dependencies. The ARMv7 build does not run
on ARMv6 boards (Raspberry Pi 1, Pi Zero W).

On macOS, a binary downloaded through a browser is quarantined by Gatekeeper,
because it is not signed or notarized. Clear it with
`xattr -d com.apple.quarantine /usr/local/bin/clavik`. Downloads made with
`curl`, and installs through Homebrew, are not affected.

v1.0.0 predates this layout and shipped bare binaries rather than archives.

## Configure

A profile holds an endpoint, a tenant and one credential. Settings resolve in
this order: flags, then environment variables, then the selected profile, then
defaults.

```bash
clavik config set --profile prod \
  --endpoint https://portal.clavik.de \
  --tenant my-account \
  --api-key vault_...

clavik config use prod
clavik config list          # never prints credentials
```

Three authentication modes are supported:

| Mode | Flags |
| --- | --- |
| API key | `--api-key` |
| OAuth2 client credentials | `--client-id`, `--client-secret`, `--token-url` |
| Pre-issued bearer token | `--bearer-token` |

Every setting has an environment variable — `CLAVIK_ENDPOINT`,
`CLAVIK_TENANT_KEY`, `CLAVIK_API_KEY`, `CLAVIK_PROFILE` and so on — which is
usually what you want in CI, so no credential is written to disk.

`--endpoint` is the host of your Clavik deployment — `https://portal.clavik.de`
for the hosted service. `https://portal.clavik.de/api/v1` also works;
`https://portal.clavik.de/api` does not.

Profiles live in `~/.clavik/config.yaml`, written at mode `0600`. Credentials
are stored in plaintext, as with the AWS CLI; the file's permissions are what
protect them. Prefer environment variables on shared machines.

### A note on API key permissions

An API key's effective permission is its **own scopes intersected with the
scopes its groups' roles grant**. A key created without scopes, or belonging to
no group, will authenticate successfully and then be refused by every route.
That looks like a broken key and is a configuration gap — check both sides.

Scopes are fixed when a key is created. A key minted without them cannot be
repaired, only replaced.

## Use

```bash
clavik health                             # is the API serving?
clavik secrets list
clavik secrets get my-app/db-password
clavik folders list
clavik keys list

clavik encrypt my-key --in payload.txt > payload.enc
clavik decrypt my-key --in payload.enc

clavik sign my-signing-key --in release.tar.gz > release.sig
clavik verify my-signing-key --in release.tar.gz --signature "$(cat release.sig)"

clavik activity list                      # audit log
clavik compliance report
```

Secrets and folders can be addressed by path or by ID; pass `--id` to skip path
resolution when you already hold an ID.

### Output

`-o table` (default), `-o json` or `-o yaml`. JSON and YAML are stable and meant
for scripting; the table format is for humans and may change.

```bash
clavik secrets list -o json | jq '.items[].name'
```

### Exit codes

Scripts can branch on these rather than parsing messages:

| Code | Meaning |
| --- | --- |
| 0 | success |
| 1 | generic failure |
| 2 | usage error — the command line was malformed |
| 3 | ambiguous reference |
| 4 | not found |
| 5 | authentication or authorization failure |
| 6 | validation error — the input was unacceptable |
| 7 | server error |

`2` and `6` are deliberately distinct: `2` means the command could not be
parsed, `6` means it parsed and the server rejected what it contained.

## Support

Report issues to your Clavik contact. This repository is for distribution and
does not track source issues.

## License

[Apache License 2.0](LICENSE).
