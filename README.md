# fs-crates: registry index `freesynergy`

Git index registry of FreeSynergy in the cargo registry format. It lets the repos of FreeSynergy depend on each other by version range (`fs-types = { version = "~0.0.N", registry = "freesynergy" }`) instead of re-tagging every consumer on each release.

Written with AI tools (Claude) and reviewed by a second AI model; a human reads and merges this pull request. Mit KI-Werkzeugen (Claude) geschrieben, von einem zweiten KI-Modell geprüft; ein Mensch liest und merged diesen Pull Request.

## What is here
- `config.json`: the registry configuration (`dl` points to `raw.githubusercontent.com`; no `api`, because nobody runs `cargo publish` against this registry).
- The index itself: one file per crate in the cargo index layout (`fs/-t/fs-types`, one JSON line per version). **The index starts empty.** A registry index is valid with only `config.json` at the root; crates appear when the workflow enters them.
- `crates/<crate>/<crate>-<version>.crate`: the packaged crates (also created by the workflow).
- `allowed-sources.txt`: which repos may put crates in (first line: `FreeSynergy-Net/fs-types`, first version 0.0.7, packages `fs-types fs-conformance fs-records`).
- `.github/workflows/index.yml`: the PULL workflow, every 15 minutes and by hand (`Run workflow`). It reads the allow list, takes new annotated release tags, packages, checks (semver gate against the newest version in this registry) and enters them. Nothing is written by hand. The pin of fs-release is a full commit SHA, set by hand.
- `.github/CODEOWNERS`: allow list, workflow and `config.json` belong to `@LordKalEl` (the owner account). A team is created only when there are further maintainers.

Details, test evidence and the click list: `docs/crate-index.md` in `FreeSynergy-Net/fs-release` (ADR 0093 in `FreeSynergy-Net/fs-documentation`).

## Trust model (honest)
Schützt vor VERSEHEN, nicht vor jemandem, der Workflow/Toolchain/Skripte ändern kann. In English: this registry protects against MISTAKES (a wrong tag, a replayed or moved tag, a changed `.crate`: cargo refuses it because the checksum of the index does not match). It does NOT protect against someone who can change the workflow, the toolchain or the scripts: whoever can write to this repo can swap a crate and its checksum together. A lock file protects only a consumer that already has it; a fresh consumer without a lock accepts a swapped version. The trust set is: the people who can write here or change the workflow or the allow list, and the repos on the allow list (their build code runs while packaging).

**Initial phase: not protected until the click list is done** (branch protection of `main`, CODEOWNERS made effective, bot without write right, Actions settings). Until then treat the registry as unprotected.

## Licence
The `LICENSE` of this repo (Apache-2.0) covers the index, the workflow and the files written here. The `.crate` files carry the licence of their source: `fs-types`, `fs-conformance` and `fs-records` are `MIT OR Apache-2.0` (see `LICENSE-MIT` and `LICENSE-APACHE` in `FreeSynergy-Net/fs-types`, `license = "MIT OR Apache-2.0"` in its `Cargo.toml`).

## Using it (consumers)
In `.cargo/config.toml` of your repo:
```toml
[registries.freesynergy]
index = "https://github.com/FreeSynergy-Net/fs-crates"
```
Then `fs-types = { version = "~0.0.7", registry = "freesynergy" }`. Keep `Cargo.lock` and build with `--locked`. The index URL must be spelled exactly like this everywhere.

## Rules
- A bad tag (lightweight, off `main`, semver red, moved) makes every run red and blocks all LATER tags of all source repos. Way out: delete the tag in the source repo, or raise the first version of that line in `allowed-sources.txt` by a reviewed pull request.
- GitHub switches scheduled workflows off after 60 days without activity in the repo. No calendar reminder is kept for this. Check after each release that its index line appeared; if it did not, Actions -> index -> `Enable workflow` (if it is off), then `Run workflow`. The automatic blocker in the release flow for a missing index line is not wired in yet.
