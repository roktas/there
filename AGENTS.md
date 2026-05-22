# Liman Agent Notes

This repository develops thin helper scripts around Lima.

## Design Principles

- Preserve Lima semantics unless there is a strong, explicit reason to do otherwise.
- Do not add wrappers for operations that `limactl` already makes easy.
- When behavior needs detailed control, prefer exposing or documenting the relevant `limactl` usage instead of hiding it
  behind a new abstraction.
- Assume users have practical Lima knowledge.
- Avoid persistent state, generated configuration files, and repo-local caches unless they are clearly necessary.

## Canonical Behavior

- `there` behavior is specified in `.agents/specs/there/spec.md`.
- `there help` prints a compact agent-friendly command reference suitable for downstream project `AGENTS.md` files.
- Keep code-facing text in English.

## Skills

- Always load the `lang-bash` skill before writing or editing any Bash code — every time, without exception.
- Always load the `dev-commits` skill before preparing any commit message — every time, without exception.

## Validation

Run before committing changes to Bash scripts:

```bash
shellcheck there tests/there
tests/there
```

The test suite fakes `limactl` and `uname`; it must not start real Lima instances.

## Upstream References

- Lima docs: https://lima-vm.io/docs/
- Lima configuration guide: https://lima-vm.io/docs/config/
- Lima VZ driver notes: https://lima-vm.io/docs/config/vmtype/vz/
