# Troubleshooting

Each entry except the 203/EXEC one was reproduced on a real install: Ubuntu 24.04, GenieACS 1.2.16, MongoDB 8.0, Supervisor 4.2.5. The 203/EXEC entry comes from systemd.exec(5).

## The file server never listens on port 7567

`genieacs-fs` shows `active`, nothing answers on 7567, and `journalctl -u genieacs-fs` repeats:

```text
[ERROR] Uncaught exception; ... exceptionMessage="bind EADDRINUSE :::7547"
[ERROR] Worker died; ... exitCode=0
```

**Cause:** the unit came from a tag between `1.2.8` and `1.2.16`. In those, `genieacs-fs.service` runs `genieacs-cwmp`, which cannot bind the port the real `genieacs-cwmp` already holds. `main` has the fix.

**Fix:** copy the unit from `main` and restart it.

```bash
grep ExecStart /etc/systemd/system/genieacs-fs.service   # wrong: .../genieacs-cwmp
sudo cp genieacs-fs.service /etc/systemd/system/ && sudo systemctl daemon-reload
sudo systemctl restart genieacs-fs
```

## A service shows failed after a crash

`systemctl status genieacs-nbi` says `failed` and its port does not answer.

**Cause:** the units set no `Restart=`, so systemd leaves a crashed service down.

**Fix:** read why it stopped with `journalctl -u genieacs-nbi -n 50`, then `sudo systemctl start genieacs-nbi`.

## Workers die with MongoServerSelectionError

The unit says `active`, but about 30 seconds after it starts the journal shows:

```text
[ERROR] Uncaught exception; ... exceptionName="MongoServerSelectionError" exceptionMessage="connect ECONNREFUSED 127.0.0.1:27017"
[ERROR] Worker died; ... exitCode=1
```

**Cause:** MongoDB is not running, or `GENIEACS_MONGODB_CONNECTION_URL` points at the wrong place. The units do not wait for MongoDB.

**Fix:** start MongoDB (`sudo systemctl start mongod` with MongoDB's own packages), check the URL in [the environment file](configuration.md#the-environment-file), then restart the four services.

## A service fails with status 203/EXEC

`systemctl status` shows `code=exited, status=203/EXEC`.

**Cause:** systemd could not run the binary in `ExecStart`, which is `/opt/genieacs/bin/genieacs-<service>` in these units. GenieACS is installed somewhere else, or not at all.

**Fix:** [A different install path](configuration.md#a-different-install-path).

## Supervisor fails to start with PermissionError

After copying `supervisord.conf` into `/etc/supervisor/conf.d/`, `supervisor.service` keeps restarting and its journal shows:

```text
PermissionError: [Errno 13] Permission denied: '/var/log/supervisor/supervisord.log'
supervisor.service: Failed with result 'exit-code'.
```

**Cause:** the file's `[supervisord]` section merges into the system config, so Supervisord switches to `genieacs` before it opens its own log.

**Fix:** replace the copy with the version in [Under the system Supervisord](usage.md#under-the-system-supervisord).

## supervisorctl says the ini file has no supervisorctl section

`supervisorctl status` answers `Error: .ini file does not include supervisorctl section`.

**Cause:** you ran it from the clone of this repository. supervisorctl reads `./supervisord.conf` before `/etc/supervisor/supervisord.conf`.

**Fix:** run it from another directory, or name the config: `sudo supervisorctl -c /etc/supervisor/supervisord.conf status`.

## A setting is empty under Supervisord

A service started through `run_with_env.sh` ignores a setting, and its log in `/var/log/genieacs/` shows `<word>: command not found`.

**Cause:** a value with spaces and no quotes in `genieacs.env`. The script reads the file with bash.

**Fix:** put the value in double quotes, then `sudo supervisorctl -c /etc/supervisor/supervisord.conf restart all`.

## Reporting a bug

Open an [issue](https://github.com/GeiserX/genieacs-services/issues) with:

- the distribution and its version, and systemd or Supervisord;
- the GenieACS version (`journalctl -u genieacs-cwmp | grep starting` shows it) and how it was installed;
- the tag or commit of these files you copied;
- the output of `systemctl status <service>` and `journalctl -u <service> -n 50`.

Never paste `genieacs.env`: it holds the UI's JWT secret and may hold a MongoDB password.
