# Rebel Media gitconfig

Configure git in a GitHub Actions job to commit and tag as a bot identity, with
every commit GPG-signed.

[![Lint](https://github.com/rebelmediausa/gitconfig/actions/workflows/lint.yml/badge.svg?branch=develop)](https://github.com/rebelmediausa/gitconfig/actions/workflows/lint.yml)
[![CodeQL](https://github.com/rebelmediausa/gitconfig/actions/workflows/codeql.yml/badge.svg?branch=develop)](https://github.com/rebelmediausa/gitconfig/actions/workflows/codeql.yml)
[![OpenSSF Scorecard](https://api.scorecard.dev/projects/github.com/rebelmediausa/gitconfig/badge)](https://scorecard.dev/viewer/?uri=github.com/rebelmediausa/gitconfig)

Rebel Media release workflows use it to create signed release tags, so a
release carries the same verifiable identity as the commits it points at.

## What it does

1. Imports the private key into the runner's GPG keyring, non-interactively
   (`gpg --batch`).
2. Sets `user.name`, `user.email` and `user.signingkey` for the repository in
   the working directory.
3. Turns on `commit.gpgsign`, so every `git commit` afterwards is signed.

It changes only the repository's own git config, not `--global`. Run it after
`actions/checkout`, in the same job that commits or tags.

## Usage

```yaml
jobs:
  tag:
    runs-on: ubuntu-latest
    environment: tagging          # keeps the key away from other branches
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@<sha> # v7.0.1
        with:
          fetch-depth: 0
          token: ${{ secrets.BOT_TOKEN }}

      - uses: rebelmediausa/gitconfig@<sha> # v1.0.1
        with:
          git_user_name: Rebel Bot
          git_user_email: bot@rebelmedia.io
          git_gpg_key: ${{ secrets.GPG_KEY }}
          git_gpg_key_id: ${{ secrets.GPG_KEY_ID }}

      - run: |
          git tag -s "v1.2.3" -m "Release 1.2.3"
          git push origin "v1.2.3"
```

`git tag -s` signs the tag; `commit.gpgsign` covers commits made after this
step.

## Inputs

| Input | Required | Description |
|---|---|---|
| `git_user_name` | yes | Committer and author name, e.g. `Rebel Bot`. Spaces are fine: every input reaches git quoted. |
| `git_user_email` | yes | Committer and author email. Must match an email on the GPG key's user ID, or GitHub will not show the signature as verified. |
| `git_gpg_key` | yes | The **base64-encoded** armoured private key. Use a secret. |
| `git_gpg_key_id` | yes | The key's ID or fingerprint, used as `user.signingkey`. Use a secret. |

## Preparing the key

Use a dedicated signing key for the bot, not a personal one, and give it no
passphrase: a runner cannot answer a passphrase prompt.

```bash
# Create the bot key (ed25519, signing only).
gpg --batch --passphrase '' --quick-generate-key "Rebel Bot <bot@rebelmedia.io>" ed25519 sign 2y

# Its ID.
gpg --list-secret-keys --keyid-format long bot@rebelmedia.io

# The value for GPG_KEY: the armoured private key, base64-encoded on one line.
gpg --armor --export-secret-keys <KEY_ID> | base64 | tr -d '\n'

# Add the public key to the bot's GitHub account (Settings -> SSH and GPG keys)
# so its signatures show as Verified.
gpg --armor --export <KEY_ID>
```

Store the secrets in an **environment** limited to the branch that releases,
for example `tagging` allowing only `master`, rather than as repository
secrets. Then a workflow running on any other branch cannot read the key.

```bash
gh secret set GPG_KEY    --env tagging --repo <owner>/<repo>
gh secret set GPG_KEY_ID --env tagging --repo <owner>/<repo>
```

## Versioning

Releases follow semver. `v1` always points at the newest `v1.x.y` release, but
a moving tag is the supply-chain risk that SHA pinning removes. **Pin to a
commit SHA** with the version as a comment (`@<sha> # v1.0.1`), and let
Dependabot raise PRs for new releases. Notes are in [CHANGELOG.md](CHANGELOG.md)
and on the [releases page](https://github.com/rebelmediausa/gitconfig/releases).

## Security

This action handles a private signing key, so:

- The key and every other input reach the shell only through quoted environment
  variables, never by interpolation. CI rejects `${{ inputs.* }}` inside a
  `run:` block.
- Every third-party action in this repository is pinned to a commit SHA, and CI
  rejects an unpinned one.
- `develop` and `master` accept changes only by pull request, with passing
  checks and signed commits. Force pushes are blocked and nobody can bypass
  these rules.
- Version tags (`vX.Y.Z`) can never be moved or deleted once created, and
  releases are immutable: a published release's tag and files cannot change.
  The major tag (`v1`) cannot be deleted; only the release workflow moves it.
- Secret scanning with push protection, Dependabot, CodeQL and OpenSSF Scorecard
  are enabled.

Report vulnerabilities privately; see [SECURITY.md](SECURITY.md).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). In short: `feature/*` branches into
`develop`, signed and signed-off Conventional Commits, and only `develop` is
merged into `master`.

## License

Apache-2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
