`.blueprint/07-ai-context/GETTING_STARTED.md`
# Repository engineering quickstart

## Before code

Review [../DOCUMENT_AUTHORITY.md](../DOCUMENT_AUTHORITY.md),
[../00-product/PROJECT_COVERAGE.md](../00-product/PROJECT_COVERAGE.md), and
Revision 1.4 ([../BASELINE-V1.md](../BASELINE-V1.md),
[../BASELINE-V2.md](../BASELINE-V2.md), [../BASELINE-V3.md](../BASELINE-V3.md)).
When ready to freeze, commit the reviewed documentation if desired and record
owner acceptance in `.execution/baseline-acceptance.md`. Implementation tasks
(`T001` onward) remain unstarted until that owner decision.

## Ubuntu toolchain

```bash
sudo apt update
sudo apt install -y build-essential curl git ca-certificates
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs -o /tmp/rustup-init.sh
sh /tmp/rustup-init.sh
source "$HOME/.cargo/env"
rustup component add rustfmt clippy
rustc --version
cargo --version
cc --version
```

If `rustup` already exists, inspect its installed toolchains instead of
reinstalling. `T001` pins the selected stable Rust version. LLVM is only needed
at `M05` (`T051`); do not choose an arbitrary system LLVM major now. Benchmark
model API access comes at `T034` and `M06`.

## Self-contained task workflow

Each task file (`04-tasks/T001-workspace-skeleton.md` through `T074`, plus
`V2-TASKS.md` and `V3-TASKS.md`) is self-contained. After recording owner
baseline acceptance in `.execution/baseline-acceptance.md` and marking `T001`
`READY` in `.execution/task-status.md`, instruct the agent:
`"Kerjakan T001 sesuai blueprint dan catat bukti."`
