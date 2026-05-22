# There

A [Lima](https://lima-vm.io/) wrapper to start or enter the project-local Ubuntu instance. The current directory is
mounted writable as `/here` (à la [Vagrant](https://developer.hashicorp.com/vagrant)'s `/vagrant` style).

```bash
there
```

Run a command inside the instance with `/here` as the working directory.

```bash
there run pwd
```

Stop, destroy, or prune Lima state explicitly.

```bash
there status
there doctor
there stop
there destroy
there prune
```

Run the shell test suite without starting real Lima instances.

```bash
tests/there
```
