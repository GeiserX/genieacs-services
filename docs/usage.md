# Usage

Day-to-day commands for the four services, updating the files, the two ways to run the Supervisord config, and which tag goes with which GenieACS version. Installing is in [Getting started](getting-started.md); the paths each file expects are in [Configuration](configuration.md).

## Systemd

| Service | What it is | Port |
|---|---|---|
| `genieacs-cwmp` | the TR-069 endpoint devices inform to | 7547 |
| `genieacs-nbi` | the northbound REST API | 7557 |
| `genieacs-fs` | the file server for firmware and config files | 7567 |
| `genieacs-ui` | the web UI | 3000 |

The ports are GenieACS's defaults; [the environment file](configuration.md#the-environment-file) can change them.

```bash
systemctl status genieacs-cwmp               # one service
journalctl -f -u genieacs-cwmp               # follow its process log
sudo systemctl restart genieacs-nbi          # after changing genieacs.env
sudo systemctl disable --now genieacs-ui     # stop it, and not at boot
```

Process logs go to the journal. Access logs go where the `GENIEACS_*_ACCESS_LOG_FILE` settings point, `/var/log/genieacs/` with the file from Getting started. A service that crashes stays down; see [Troubleshooting](troubleshooting.md#a-service-shows-failed-after-a-crash).

## Updating the files

```bash
cd genieacs-services && git pull
sudo cp genieacs-*.service /etc/systemd/system/ && sudo systemctl daemon-reload
sudo systemctl restart genieacs-cwmp genieacs-nbi genieacs-fs genieacs-ui
```

## Supervisord

`supervisord.conf` runs the same four services under Supervisord and restarts any that exits. It starts them from `/opt/genieacs/dist/bin/` (on an npm install, see [A different install path](configuration.md#a-different-install-path)) and writes each one's output to `/var/log/genieacs/genieacs-<service>.log`, so that directory must exist and belong to `genieacs`.

### As its own Supervisord

This is how the [genieacs-container](https://github.com/GeiserX/genieacs-container) image runs it: the file is Supervisord's only config. Supervisord switches to `genieacs` and stays in the foreground, so run it under something that keeps it alive, such as a container or a systemd unit of your own.

```bash
sudo supervisord -c /path/to/genieacs-services/supervisord.conf
```

### Under the system Supervisord

Do not copy the file unchanged into `/etc/supervisor/conf.d/`. Its `[supervisord]` section merges into the system one: a reload starts the four services as root, and the next start of `supervisor.service` fails with a `PermissionError` (see [Troubleshooting](troubleshooting.md#supervisor-fails-to-start-with-permissionerror)).

Remove the first three lines and give each program the user instead. The second `sed` also switches every program to its `run_with_env.sh` line, so the services read `/opt/genieacs/genieacs.env` the way the systemd units do:

```bash
sudo install -m 755 run_with_env.sh /usr/local/bin/run_with_env.sh
sed -e '1,3d' -e 's/^autorestart=true$/autorestart=true\nuser=genieacs/' supervisord.conf \
  | sed -e 's/^;command=/command=/' -e '/^command=\/opt/d' \
  | sudo tee /etc/supervisor/conf.d/genieacs.conf >/dev/null
cd / && sudo supervisorctl reread && sudo supervisorctl update && sudo supervisorctl status
```

`supervisorctl status` should list the four as `RUNNING`, and every GenieACS process should run as `genieacs`.

## run_with_env.sh

```bash
run_with_env.sh <env-file> <command>
```

It exports every variable in the env file, then runs the command. The Supervisord config's commented `command=` lines call it from `/usr/local/bin/`. The file must be valid bash; see [the environment file](configuration.md#the-environment-file).

## Versions

Git tags name the GenieACS version the files target:

| Tag | Notes |
|---|---|
| `1.2.0` | GenieACS 1.2.0-beta; the units run from `/opt/genieacs/dist/bin/` |
| `1.2.8` | its `genieacs-fs.service` starts `genieacs-cwmp` by mistake |
| `1.2.13` to `1.2.16` | one commit; same `genieacs-fs.service` mistake |
| `main` | the fixed `genieacs-fs.service`; no tag yet |

[genieacs-container](https://github.com/GeiserX/genieacs-container) builds its image from tag `1.2.16` and uses only `supervisord.conf` and `run_with_env.sh` from it, so the fs-unit mistake does not reach the image. The `1.1` branch holds the files for GenieACS 1.1.
