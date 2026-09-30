# How it works

GenieACS is four Node.js services that share one MongoDB database and nothing else: `genieacs-cwmp` talks TR-069 to devices, `genieacs-nbi` is the REST API for scripts, `genieacs-fs` serves firmware and config files to devices, and `genieacs-ui` is the web interface. Each starts a primary process that forks workers, one per CPU core by default. This repository only decides how those four are started and kept running.

## The systemd units

The four units are the ones in GenieACS's [installation guide](https://docs.genieacs.com/en/latest/installation-guide.html), with two lines changed:

| Line | Here | GenieACS's guide |
|---|---|---|
| `ExecStart` | `/opt/genieacs/bin/genieacs-<service>` | `/usr/bin/genieacs-<service>` |
| `KillMode` | `process` | not set (systemd's default, `control-group`) |

What each line does:

- `After=network.target` orders the start after the network is up. Nothing orders it after MongoDB, so a service can start while MongoDB is still down; its workers then die and the unit stays `active` (see [Troubleshooting](troubleshooting.md#workers-die-with-mongoserverselectionerror)).
- `User=genieacs` runs the service without root.
- `EnvironmentFile=/opt/genieacs/genieacs.env` loads the settings. systemd reads the file as root, before switching user.
- `KillMode=process` sends the stop signal to the primary process only; the primary takes its workers down with it, so `systemctl stop` leaves nothing running.
- No `Restart=` line: a service that crashes is marked `failed` and stays down until someone starts it.
- `WantedBy=default.target`: `systemctl enable` links the unit into `default.target.wants`, so it starts at boot.

## The Supervisord config

`supervisord.conf` was written for a container. Its `[supervisord]` section sets `user=genieacs`, so Supervisord itself drops to that user, and `nodaemon=true`, so it stays in the foreground as the container's main process. Each of the four `[program:...]` sections runs one service from `/opt/genieacs/dist/bin/` in `/opt/genieacs`, writes its output to `/var/log/genieacs/genieacs-<service>.log`, and restarts it when it exits (`autorestart=true`).

[genieacs-container](https://github.com/GeiserX/genieacs-container) clones tag `1.2.16` of this repository, copies the file to `/etc/supervisor/conf.d/genieacs.conf` and runs `supervisord -c` on that file alone. That is the use it was written for. Under a distribution's Supervisord the `[supervisord]` section is a problem: [Usage](usage.md#under-the-system-supervisord) says why and what to change.

## run_with_env.sh

Ten lines: turn on `allexport`, `source` the env file, turn it off, run the command. It gives a Supervisord program the same settings file the systemd units read. It joins the command's arguments into one string and splits it on spaces again, so it suits what the config passes it: a binary path with no arguments.
