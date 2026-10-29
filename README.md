# DEUSPROOF Notarize — GitHub Action

Record when your work existed, from CI. This action timestamps your work
(RFC 3161) and anchors it to **Bitcoin** on a public, append-only ledger — a
permanent record of *when*, verifiable by anyone without trusting us. It proves
when, not who. Free, no account, no API key.

Two modes:

- **`content`** (default) — records the file's text on the public ledger and
  scores it (AAS). For work you intend to make public.
- **`hash`** — sends **only the SHA-256**; nothing is uploaded. A proof of
  existence for **private repos and proprietary code**. Hash a file, or the
  commit itself.

> In `content` mode the file's **text is recorded on a public permanent ledger** —
> point it only at content you intend to make public. In `hash` mode nothing but
> the fingerprint ever leaves the runner.

## Usage

### Private repos — notarize every commit's fingerprint (hash mode)

Add `.github/workflows/notarize.yml`:

```yaml
name: Notarize commit
on: [push]

jobs:
  notarize:
    runs-on: ubuntu-latest
    steps:
      - uses: Abracadabrastartup/deusproof-notarize@v1
        with:
          mode: hash                     # only the SHA-256 is sent
          handle: my-repo                # who to credit
          # no `file:` → the commit itself is notarized
```

Notarize a specific (private) build artifact instead of the commit:

```yaml
      - uses: Abracadabrastartup/deusproof-notarize@v1
        with:
          mode: hash
          file: dist/model.bin           # only its SHA-256 is sent
```

### Public creative work — record it on the ledger (content mode)

```yaml
name: Notarize on release
on:
  release:
    types: [published]

jobs:
  notarize:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Abracadabrastartup/deusproof-notarize@v1
        with:
          file: creations/latest.md      # the file to notarize (content goes public)
          handle: my-agent               # who to credit
          model: gpt-4                   # optional
```

The certificate URL appears in the job summary and as an output:

```yaml
      - uses: Abracadabrastartup/deusproof-notarize@v1
        id: proof
        with:
          file: creations/latest.md
      - run: echo "Proof: ${{ steps.proof.outputs.verify-url }}"
```

## Inputs

| Input | Required | Default | What it does |
|-------|----------|---------|--------------|
| `mode` | no | `content` | `content` records the file text (public) and scores it; `hash` sends only the SHA-256 (private-safe). |
| `file` | in content mode | — | File to notarize. Content mode: its text goes on the public ledger. Hash mode: only its SHA-256 is sent; omit to notarize the commit. |
| `handle` | no | repo owner | Author/agent handle to credit. |
| `model` | no | — | The AI model that produced the work. |
| `fail-on-error` | no | `false` | If `false`, a notarization problem warns but never fails your build. |
| `api-base` | no | `https://deusproof.com` | Override only for testing. |

## Outputs

- `verify-url` — permanent public certificate URL.
- `certificate-id` — the certificate id.
- `output-hash` — the SHA-256 that was notarized (hash mode).

## Verify a hash-mode proof later

The commit recipe matches the [git hook](https://github.com/Abracadabrastartup/deusproof/tree/master/sdk/git):

```sh
printf 'deusproof-commit:%s' "<commit sha>" | sha256sum   # → the notarized digest
```

For a file: `sha256sum yourfile`. Then look it up at
`https://deusproof.com/check` or `GET /api/verify/by-hash/<digest>`.

## What it proves (honestly)

DEUSPROOF records **existence and time**. In `content` mode: the exact bytes,
scored (AAS 0–100). In `hash` mode: only the fingerprint — proof the content
existed, with **nothing disclosed and no authorship score**. Either way it is
timestamped (RFC 3161), sealed on a hash-chained ledger and anchored to Bitcoin
(OpenTimestamps — the anchor settles over hours). A certificate proves the
content existed at that moment; it does not, by itself, adjudicate authorship.
Verify any certificate independently — the `.ots` Bitcoin proof needs nothing
from DEUSPROOF.

Learn more: https://deusproof.com · Agents: https://deusproof.com/skill.md
