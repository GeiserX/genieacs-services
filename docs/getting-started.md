# Getting started

These files start GenieACS; they do not install it. This page installs GenieACS 1.2.16 where the systemd units look for it, starts the four services, and shows what a working install looks like. Every command was run on a fresh Ubuntu 24.04 host with Node.js 18 and MongoDB 8.0.

## Before you start

- A Linux host with systemd.
- Node.js 12.13 or later and MongoDB 3.6 or later, running. GenieACS's [installation guide](https://docs.genieacs.com/en/latest/installation-guide.html) links the install instructions for both.
- `git`, to fetch these files.

## Install GenieACS where the units expect it

The units run `/opt/genieacs/bin/genieacs-<service>` as the user `genieacs`. Installing from npm with `--prefix /opt/genieacs` puts the four binaries there:

```bash
sudo npm install -g --prefix /opt/genieacs genieacs@1.2.16
sudo useradd --system --no-create-home --user-group genieacs
sudo mkdir -p /opt/genieacs/ext /var/log/genieacs
sudo chown genieacs:genieacs /opt/genieacs/ext /var/log/genieacs
```

Then write the environment file the units read. These are the lines GenieACS's guide uses; the `node` command appends a random secret that signs the UI's login cookies:

```bash
sudo tee /opt/genieacs/genieacs.env >/dev/null <<'EOF'
GENIEACS_CWMP_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-cwmp-access.log
GENIEACS_NBI_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-nbi-access.log
GENIEACS_FS_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-fs-access.log
GENIEACS_UI_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-ui-access.log
GENIEACS_DEBUG_FILE=/var/log/genieacs/genieacs-debug.yaml
NODE_OPTIONS=--enable-source-maps
GENIEACS_EXT_DIR=/opt/genieacs/ext
EOF
node -e "console.log(\"GENIEACS_UI_JWT_SECRET=\" + require('crypto').randomBytes(128).toString('hex'))" | sudo tee -a /opt/genieacs/genieacs.env >/dev/null
sudo chown genieacs:genieacs /opt/genieacs/genieacs.env
sudo chmod 600 /opt/genieacs/genieacs.env
```

If GenieACS is already installed somewhere else, for example in `/usr/bin` after a plain `sudo npm install -g genieacs`, see [A different install path](configuration.md#a-different-install-path). [The environment file](configuration.md#the-environment-file) lists every setting the file can hold.

## Install the systemd units

```bash
git clone https://github.com/GeiserX/genieacs-services && cd genieacs-services
sudo cp genieacs-*.service /etc/systemd/system/ && sudo systemctl daemon-reload
sudo systemctl enable --now genieacs-cwmp genieacs-nbi genieacs-fs genieacs-ui
```

Take the files from `main`. The tags `1.2.8` to `1.2.16` ship a `genieacs-fs.service` that starts `genieacs-cwmp` by mistake; see [Troubleshooting](troubleshooting.md#the-file-server-never-listens-on-port-7567).

## Check that it works

```console
$ sudo systemctl enable --now genieacs-cwmp genieacs-nbi genieacs-fs genieacs-ui
Created symlink /etc/systemd/system/default.target.wants/genieacs-cwmp.service → /etc/systemd/system/genieacs-cwmp.service.
Created symlink /etc/systemd/system/default.target.wants/genieacs-nbi.service → /etc/systemd/system/genieacs-nbi.service.
Created symlink /etc/systemd/system/default.target.wants/genieacs-fs.service → /etc/systemd/system/genieacs-fs.service.
Created symlink /etc/systemd/system/default.target.wants/genieacs-ui.service → /etc/systemd/system/genieacs-ui.service.
$ systemctl is-active genieacs-cwmp genieacs-nbi genieacs-fs genieacs-ui
active
active
active
active
$ journalctl -u genieacs-cwmp -n 3 -o cat
2026-09-30T18:40:31.093Z [INFO] genieacs-cwmp starting; pid=10048 version="1.2.16+26032938e9"
2026-09-30T18:40:33.025Z [INFO] Worker listening; pid=10089 address="::" port=7547
2026-09-30T18:40:33.035Z [INFO] Worker listening; pid=10090 address="::" port=7547
$ curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:7557/devices
200
$ curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:3000/
200
```

It works when all four are `active`, the journal shows `Worker listening` (one line per CPU core), and ports 7557 and 3000 answer 200. Then open `http://<this host>:3000/` in a browser, and point a device's ACS URL at `http://<this host>:7547/`. GenieACS's [documentation](https://docs.genieacs.com/) covers presets, provisions and the rest.

## With Supervisord instead

The Supervisord config needs one change before it goes under your distribution's Supervisord. Both ways to run it are in [Usage](usage.md#supervisord).
