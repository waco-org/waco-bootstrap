# Local tasks

Tasks that only make sense on one machine, such as mounting its data disks, go in
`local/tasks.yml`. `provision.yml` runs them after Waco, if the file exists. Files they need,
such as a printer's PPD, can sit next to them.

Collections or roles those tasks need, such as `ansible.posix` for `ansible.posix.mount`, go in
`local/requirements.yml`. `waco-provision` installs them along with the collection.
