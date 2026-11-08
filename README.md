waco-bootstrap
==============

Set up a newly installed Linux machine with [Waco](https://github.com/waco-org/waco), the
`waco_org.waco` Ansible collection. This repository holds two scripts that install what Waco
needs and run it, plus suggested settings for you to change.

> **Work in progress: not ready for use**, like the collection itself.

Supported distros: the ones Waco supports, i.e. Rocky Linux 10 and 9, CentOS Stream 10 and 9,
Fedora 44 and 43, Ubuntu 26.04, 24.04 and 22.04 LTS.

Usage
-----

As your own user (one that can use `sudo`), not as root:

```sh
git clone https://github.com/waco-org/waco-bootstrap.git
cd waco-bootstrap
./waco-bootstrap          # git, uv and ansible-core
$EDITOR settings/type.yml settings/machine.yml
./waco-provision          # install the collection and apply it to this machine
```

- `waco-bootstrap` installs git and the few system packages Ansible's modules need. It installs
  uv from the distro where uv is packaged (Fedora, EPEL 10) and from its GitHub release
  otherwise (checked against its SHA-256), then ansible-core for your user with
  `uv tool install`. Running it again is harmless.
- `waco-provision` installs the collection into `.ansible/` in this clone, then runs
  `provision.yml` on this machine. It asks for your sudo password. Options:
  - `-s <ref>`: a git ref of `waco-org/waco` (default: `main`);
  - `-s git+<url>[,<ref>]`: another git repository;
  - `-s <path>`: a local checkout of the collection;
  - `-s galaxy`: Ansible Galaxy, once the collection is published;
  - `-n`: install and check only, without running anything.

  Anything after `--` goes to `ansible-playbook`, e.g. `-- --check --diff`.

Settings
--------

Settings come in two levels, with the same variable names in both:

- `settings/type.yml` is for the **kind of machine**, e.g. "home desktop" or "Ubuntu work
  laptop". Upstream it holds Waco's suggested defaults.
- `settings/machine.yml` is for **this machine**. Upstream it is empty.

They are merged before Waco runs:
- a single value in `machine.yml` replaces the type's;
- a list is added to the type's, without duplicates;
- a dictionary is merged key by key.

A machine can add to its type's lists but can't take anything out of them, so a type should
only list what every machine of that kind wants.

Tasks that only make sense on one machine, such as mounting its data disks, go in
`local/tasks.yml`. They run after Waco.

### Keeping your settings in git

Each clone or branch holds one set of settings. To keep more than one, use branches or
clones, not extra files:

- **One machine:** commit your changes to both files in your own clone.
- **Several machines of one kind:** a branch for the type that changes only `type.yml`, plus
  a branch or clone per machine, made from it, that changes only `machine.yml` and `local/`.
  Each level touches its own files, so merging upstream into the type and the type into the
  machines shouldn't conflict.

Settings name a machine's users, printers and so on. Keep them in a private repository if that
matters to you. A GitHub fork of this repository is public.

Testing
-------

CI runs `waco-bootstrap` and `waco-provision -n` in a fresh container of each supported distro
(`tests/in-container`). To run it locally:

```sh
podman run --rm -v "$PWD:/src:ro,z" quay.io/rockylinux/rockylinux:10 /src/tests/in-container
```

License
-------

GPL-3.0, see [LICENSE](LICENSE).
