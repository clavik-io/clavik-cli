# clavik

Command-line client for the [Clavik vault](https://clavik.io).

`clavik` manages secrets, encryption keys, folders and access from a terminal or
a CI job: store and retrieve credentials, encrypt and decrypt payloads, sign and
verify data, inspect the audit log, and query compliance and security policy.

This repository distributes **release binaries**. The source is not published
here.

## Install

Download the binary for your platform from the
[latest release](https://github.com/clavik-io/clavik-cli/releases/latest).

```bash
# macOS (Apple silicon)
curl -sSL -o clavik https://github.com/clavik-io/clavik-cli/releases/latest/download/clavik_v1.0.0_darwin_arm64
chmod +x clavik && sudo mv clavik /usr/local/bin/
```

```bash
# Linux (x86-64)
curl -sSL -o clavik https://github.com/clavik-io/clavik-cli/releases/latest/download/clavik_v1.0.0_linux_amd64
chmod +x clavik && sudo mv clavik /usr/local/bin/
```

Builds are provided for:

| Platform | Architectures |
| --- | --- |
| macOS | arm64 (Apple silicon), amd64 (Intel) |
| Linux | amd64, arm64, arm (32-bit) |
| Windows | amd64, arm64 |

They are statically linked and have no runtime dependencies.

**Verify what you downloaded.** Every release ships a `checksums.txt`:

```bash
curl -sSL -O https://github.com/clavik-io/clavik-cli/releases/latest/download/checksums.txt
shasum -a 256 -c checksums.txt --ignore-missing
```

On macOS, Gatekeeper will quarantine an unsigned download. Clear it with
`xattr -d com.apple.quarantine /usr/local/bin/clavik`.

## Configure

A profile holds an endpoint, a tenant and one credential. Settings resolve in
this order: flags, then environment variables, then the selected profile, then
defaults.

```bash
clavik config set --profile prod \
  --endpoint https://api.clavik.io \
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
