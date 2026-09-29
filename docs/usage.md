# Usage

This repository holds service definitions for the four GenieACS services, for two process managers.
None of them installs GenieACS itself; follow the
[GenieACS installation guide](https://docs.genieacs.com/en/latest/installation-guide.html) first.

## Systemd units

| File | Runs |
|---|---|
| `genieacs-cwmp.service` | `/opt/genieacs/bin/genieacs-cwmp`, the TR-069 endpoint devices inform to (port 7547) |
| `genieacs-nbi.service` | `/opt/genieacs/bin/genieacs-nbi`, the northbound REST API (port 7557) |
| `genieacs-fs.service` | `/opt/genieacs/bin/genieacs-fs`, the file server for firmware and config files (port 7567) |
| `genieacs-ui.service` | `/opt/genieacs/bin/genieacs-ui`, the web UI (port 3000) |

Every unit expects:

- a system user `genieacs` (`User=genieacs`);
- the GenieACS binaries in `/opt/genieacs/bin/`;
- an environment file at `/opt/genieacs/genieacs.env` with the `GENIEACS_*` settings, such as
  `GENIEACS_MONGODB_CONNECTION_URL` and `GENIEACS_UI_JWT_SECRET`.

If your install differs, for example binaries in `/usr/bin` after `npm install -g genieacs`, edit
`ExecStart` before copying the files. Install and start them as in the README's quick start; each
service logs to the journal:

```bash
journalctl -f -u genieacs-cwmp
```

## Supervisord

`supervisord.conf` runs the same four services as the `genieacs` user from `/opt/genieacs/dist/bin/`,
the layout a source build (`npm run build`) produces. This is the file the
[genieacs-container](https://github.com/GeiserX/genieacs-container) image uses. Copy it to
`/etc/supervisor/conf.d/` and reload Supervisord. Each service writes its output to
`/var/log/genieacs/genieacs-<service>.log`, so create `/var/log/genieacs` owned by `genieacs` first.

The file reads its settings from the process environment. To load them from a file instead, each
program has a commented `command=` line that goes through `run_with_env.sh`.

## run_with_env.sh

`run_with_env.sh <env-file> <command...>` exports every variable in the env file, then runs the
command. It lets a Supervisord program read `/opt/genieacs/genieacs.env` the way the systemd units do.

## Versions

Git tags name the GenieACS version the files target: `1.2.8`, `1.2.13`, `1.2.14`, `1.2.15`, `1.2.16`.
genieacs-container builds its image from the `1.2.16` tag.
