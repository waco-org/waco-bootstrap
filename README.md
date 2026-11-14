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
git clone https://github.com/waco-org/waco-bootstrap.git ~/.waco
cd ~/.waco
./waco-bootstrap          # git, uv, the waco account and its ansible-core
$EDITOR settings/type.yml settings/machine.yml
git commit -am "My settings"
./waco-provision          # install the collection and apply it to this machine, as waco
```

- `waco-bootstrap` installs git and the few system packages Ansible's modules need. It installs
  uv from the distro where uv is packaged (Fedora, EPEL 10) and from its GitHub release
  otherwise (checked against its SHA-256). Then it sets up the `waco` account (below) with
  ansible-core and its own copy of your clone. Running it again is harmless.
- `waco-provision` runs as `waco`: started from your clone, it hands over with
  `sudo -iu waco waco-provision`, which asks for your password; you can also run that command
  yourself. It refreshes `waco`'s copy from what is **committed** in your clone, installs the
  collection into `.ansible/` in that copy, then runs `provision.yml` on this machine. Options:
  - `-s <ref>`: a git ref of `waco-org/waco` (default: `main`, until the collection is on
    Galaxy, which then becomes the default);
  - `-s git+<url>[,<ref>]`: another git repository;
  - `-s <path>`: a local checkout of the collection, which `waco` must be able to read;
  - `-s galaxy`: Ansible Galaxy, once the collection is published;
  - `-n`: install and check only, without running anything.

  Anything after `--` goes to `ansible-playbook`, e.g. `-- --check --diff`.

### The `waco` account

Waco runs as `waco`, a system account kept apart from the people who use the machine:

- its home is `/var/lib/waco`, holding ansible-core, the collection and its copy of your clone;
- its password is locked and it has no SSH key, so the only way in is `sudo -iu waco`, which
  asks for your password;
- it has passwordless sudo (`/etc/sudoers.d/waco`), which Ansible uses for the tasks that need
  root. Having got in already cost a password.

`waco`'s copy of your clone is refreshed at every run: your account packs the committed state
into a git bundle (`waco` runs `sudo -u <you> git bundle create`), and `waco` fetches it.
Uncommitted changes are not applied, and your account can't change `waco`'s files. If you
move your clone, run `./waco-bootstrap` again from the new place.

### After the first run

Your clone now holds your settings, and its `origin` is still this public repository. Push it
to a git server of your choice as a **private** repository: settings name a machine's users,
printers and so on.

```sh
git remote rename origin upstream          # to merge later improvements
git remote add origin <your private repository>
git push -u origin HEAD
```

To keep more than one set of settings, use branches (e.g. one per distro or per machine) or
separate clones; see "Keeping your settings in git" below.

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
`local/tasks.yml`. They run after Waco. Whatever collections they need go in
`local/requirements.yml`.

### Keeping your settings in git

Each clone or branch holds one set of settings. To keep more than one, use branches or
clones, not extra files:

- **One machine:** commit your changes to both files in your own clone.
- **Several machines of one kind:** a branch for the type that changes only `type.yml`, plus
  a branch or clone per machine, made from it, that changes only `machine.yml` and `local/`.
  Each level touches its own files, so merging upstream into the type and the type into the
  machines shouldn't conflict.

Settings name a machine's users, printers and so on: keep them in a private repository. A
GitHub fork of this repository is public.

Testing
-------

CI runs `waco-bootstrap` and `waco-provision -n` in a fresh container of each supported distro
and checks the `waco` account (`tests/in-container`). To run it locally:

```sh
podman run --rm -v "$PWD:/src:ro,z" quay.io/rockylinux/rockylinux:10 /src/tests/in-container
```

License
-------

GPL-3.0, see [LICENSE](LICENSE).
