# Configuration

The files have no settings of their own. They fix where GenieACS lives, which user runs it, and where its settings come from. GenieACS's own settings live in the environment file.

## What the files expect

| | systemd units | `supervisord.conf` |
|---|---|---|
| Binaries | `/opt/genieacs/bin/genieacs-<service>` | `/opt/genieacs/dist/bin/genieacs-<service>` |
| User | `genieacs` (`User=`) | `genieacs` (`user=` in `[supervisord]`) |
| Settings | `/opt/genieacs/genieacs.env` (`EnvironmentFile=`) | the process environment, or `/opt/genieacs/genieacs.env` through `run_with_env.sh` on the commented `command=` lines |
| Working directory | not set | `/opt/genieacs` |
| Process log | the journal | `/var/log/genieacs/genieacs-<service>.log` |
| After a crash | stays down (no `Restart=`) | restarted (`autorestart=true`) |

`/opt/genieacs/bin/` is where `npm install -g --prefix /opt/genieacs` puts the binaries. `/opt/genieacs/dist/bin/` is where a build from source (`npm run build` in `/opt/genieacs`) puts them, which is how the [genieacs-container](https://github.com/GeiserX/genieacs-container) image builds GenieACS.

## A different install path

For the systemd units, change `ExecStart` before copying them. After a plain `sudo npm install -g genieacs` the binaries are in `/usr/bin/`:

```bash
sed -i 's|/opt/genieacs/bin/|/usr/bin/|' genieacs-*.service
```

For `supervisord.conf` on an npm install under `/opt/genieacs`, link the directory it expects to the one npm made:

```bash
sudo mkdir -p /opt/genieacs/dist && sudo ln -s /opt/genieacs/bin /opt/genieacs/dist/bin
```

## The environment file

Every GenieACS setting is an environment variable with the `GENIEACS_` prefix; the full list is GenieACS's [Environment variables](https://docs.genieacs.com/en/latest/environment-variables.html) page. The ones people change first:

| Variable | Default | Change it when |
|---|---|---|
| `GENIEACS_MONGODB_CONNECTION_URL` | `mongodb://127.0.0.1/genieacs` | MongoDB runs on another host or needs a password |
| `GENIEACS_UI_JWT_SECRET` | unset | always: it signs the UI's login cookies ([Getting started](getting-started.md#install-genieacs-where-the-units-expect-it) generates one) |
| `GENIEACS_CWMP_PORT`, `GENIEACS_NBI_PORT`, `GENIEACS_FS_PORT`, `GENIEACS_UI_PORT` | `7547`, `7557`, `7567`, `3000` | a port is taken |
| `GENIEACS_CWMP_ACCESS_LOG_FILE` and the `NBI`, `FS`, `UI` equivalents | unset: access lines go to the process log | you want access logs in their own files |
| `GENIEACS_EXT_DIR` | `<install dir>/config/ext` | you use extension scripts |

Write one `NAME=value` per line and put a value that contains spaces in double quotes. systemd accepts either form, but `run_with_env.sh` reads the file with bash, where `NAME=two words` sets nothing and logs `words: command not found`.

Keep the file owned by `genieacs` with mode `600`. It holds the JWT secret, and may hold a MongoDB password. systemd reads it as root before switching user, but under Supervisord `run_with_env.sh` runs as `genieacs` and must be able to read it.

After changing it, restart the services that read it:

```bash
sudo systemctl restart genieacs-cwmp genieacs-nbi genieacs-fs genieacs-ui
```
