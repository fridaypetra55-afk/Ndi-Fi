# Contributing to Ndi-Fi

Thanks for helping build Ndi-Fi. This guide covers contribution rules, workflows,
and review expectations across the repository, including our participation in the
[Stellar Wave Program on Drips](https://www.drips.network/wave/stellar).

> 🐣 **New contributor?** Start with our comprehensive [**Contributor Onboarding Guide**](docs/ONBOARDING.md) for step-by-step environment setup, starter task ladders, and curated good first issues!
> 🖥️ **Running the full stack locally?** See [**docs/LOCAL_STACK.md**](docs/LOCAL_STACK.md) for anvil, Stellar/Soroban, frontend, and web UI setup in one place.

## Ground rules

- **Do not start work on an issue until a maintainer assigns it to you. Comment on the issue to express interest.**
- Every PR must include `Closes #<issue>` in its description so Drips Wave can link it.
- Keep PRs to one issue. Do not change files outside `stellar/` or `docs/stellar/` unless the issue says so. In particular, do not touch the Solidity code in `src/`, `test/` or `script/`.
- Report security problems privately to the maintainers, not in a public issue.

## Quick start

Run the bootstrap script from the repository root to install submodules, build both Foundry and Soroban contracts, and run all tests in one shot:

```bash
./bootstrap.sh
```

The script checks for missing tools ([Foundry](https://book.getfoundry.sh/getting-started/installation), [Rust/rustup](https://rustup.rs), [Node.js ≥ 18](https://nodejs.org), [Stellar CLI 28.x](https://github.com/stellar/stellar-cli/releases)) and tells you what to install before it proceeds.

## Setup

1. **Rust.** Install [rustup](https://rustup.rs). The toolchain version, the `wasm32v1-none` target, `rustfmt` and `clippy` are pinned in `stellar/rust-toolchain.toml`, so this installs everything:

   ```bash
   cd stellar
   rustup toolchain install
   ```

   If you manage toolchains yourself: `rustup target add wasm32v1-none`.

2. **Stellar CLI** (version 28.x). Pick one:

   ```bash
   cargo install --locked stellar-cli
   # or a prebuilt binary from https://github.com/stellar/stellar-cli/releases
   ```

   Check with `stellar --version`.

3. **Build and test**

   ```bash
   cd stellar
   cargo test
   stellar contract build   # writes target/wasm32v1-none/release/*.wasm
   ```

## Branches

Branch from `main` and name the branch by type and issue number:

- `feat/<issue>-short-name`, for example `feat/42-staking-topups`
- `fix/<issue>-short-name`
- `docs/<issue>-short-name`
- `test/<issue>-short-name`

## Commits

Use [Conventional Commits](https://www.conventionalcommits.org):

```
feat(staking): allow topping up an existing stake
fix(lending): round shares down on deposit
docs(stellar): add error-code reference
test(token): cover burn_from
```

Scopes: `token`, `lending`, `staking`, `stellar` (workspace, CI, scripts), `docs`, `frontend`, `ci`.

PR titles follow the same format and are enforced by CI. See [Commit and PR title format](#commit-and-pr-title-format) below.

## Pre-commit hooks (optional, recommended)

Install [pre-commit](https://pre-commit.com) to catch formatting and lint issues
locally before CI:

```bash
pip install pre-commit  # or pipx/brew
pre-commit install
pre-commit run --all-files  # optional
```

Hooks are configured in `.pre-commit-config.yaml` (Foundry `forge fmt`, Stellar
`cargo fmt`, Prettier, ESLint, and basic hygiene checks). Hooks that require
tools you don't have installed are skipped. See [DEVELOPMENT.md](DEVELOPMENT.md)
for details.

## Before you open a PR

Run these in `stellar/`. CI runs the same checks.

```bash
cargo fmt --all --check
cargo clippy --all-targets -- -D warnings
cargo test
stellar contract build
```

For Solidity changes, run the Foundry checks used in CI:

```bash
forge fmt --check
forge build --sizes
forge test -vvv
forge doc --build
```

Document every public/external function, event and public state variable with NatSpec
(`@notice`/`@dev`, plus `@param` and `@return`) so the generated contract reference stays
complete. `forge doc --build` writes the browsable API reference to
`docs/reference/solidity/`; see [docs/SOLIDITY_API.md](docs/SOLIDITY_API.md).

The root `.gitignore` ignores Foundry's build outputs in `docs/` while tracking markdown
guides in `docs/` and `docs/stellar/`. See [docs/ONBOARDING.md](docs/ONBOARDING.md) for full
setup and first-issue guidance.

Contract code rules:

- Every state-changing function that acts for a user calls `user.require_auth()`; admin functions use `#[only_owner]`.
- Errors go in the contract's `#[contracterror]` enum with an explicit code. No `panic!` with strings.
- Use instance storage for config, persistent storage for per-user data, and extend TTLs.
- Use `checked_*` math on `i128`. No floats.
- Emit an event for every state change users or indexers care about.
- Every public function has a `///` doc comment.

## Linting GitHub Actions workflows locally

[actionlint](https://github.com/rhysd/actionlint) catches misconfigured workflows before they reach CI.

Install it once:

```bash
# macOS
brew install actionlint

# Linux — download the binary
bash <(curl -s https://raw.githubusercontent.com/rhysd/actionlint/main/scripts/download-actionlint.bash)
```

Then run it from the repository root:

```bash
actionlint
```

CI runs the same check on every change to `.github/workflows/**`.

## Pull requests

Fill in the PR template: `Closes #<issue>`, what changed, how you tested it,
and the checklist. A maintainer reviews it. Please respond to review comments
within the Wave cycle.
