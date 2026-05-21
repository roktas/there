# Liman

[Lima](https://lima-vm.io/) Nuts & Bolts.

### `there`

Start or enter the project-local Ubuntu instance. The current directory is mounted writable as `/here`.

```bash
there
```

Run a command inside the instance with `/here` as the working directory.

```bash
there run pwd
```

Stop, destroy, or prune Lima state explicitly.

```bash
there stop
there destroy
there prune
```

Run the shell test suite without starting real Lima instances.

```bash
tests/there
```
