# Liman Agent Notes

This repository develops thin helper scripts around Lima.

## Canonical Behavior

- `there` behavior is specified in `.agents/specs/there/spec.md`.
- `there help` prints a compact agent-friendly command reference suitable for downstream project `AGENTS.md` files.
- Keep code-facing text in English.

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
