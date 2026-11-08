# Local tasks

Tasks that only make sense on one machine, such as mounting its data disks, go in
`local/tasks.yml`. `provision.yml` runs them after Waco, if the file exists. Files they need,
such as a printer's PPD, can sit next to them.
