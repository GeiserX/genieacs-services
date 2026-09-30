<p align="center">
  <img src="docs/images/banner.svg" alt="GenieACS Services" width="900"/>
</p>

<h1 align="center">GenieACS Services</h1>

<p align="center">
  <a href="https://github.com/GeiserX/genieacs-services/tags"><img src="https://img.shields.io/github/v/tag/GeiserX/genieacs-services?style=flat-square" alt="Tag"/></a>
  <a href="https://github.com/GeiserX/genieacs-services/blob/main/LICENSE"><img src="https://img.shields.io/github/license/GeiserX/genieacs-services?style=flat-square" alt="License"/></a>
  <a href="https://github.com/GeiserX/genieacs-services/stargazers"><img src="https://img.shields.io/github/stars/GeiserX/genieacs-services?style=flat-square&logo=github" alt="GitHub Stars"/></a>
</p>

<p align="center"><strong>Systemd and Supervisord service files for GenieACS.</strong></p>

If you would rather run GenieACS in Docker or Kubernetes, use [genieacs-container](https://github.com/GeiserX/genieacs-container); these files are for bare-metal installs.

## Quick start

On a host where GenieACS is installed under `/opt/genieacs` and a `genieacs` user exists:

```bash
git clone https://github.com/GeiserX/genieacs-services && cd genieacs-services
sudo cp genieacs-*.service /etc/systemd/system/ && sudo systemctl daemon-reload
sudo systemctl enable --now genieacs-cwmp genieacs-nbi genieacs-fs genieacs-ui
```

Follow the logs with `journalctl -f -u genieacs-cwmp`. Installing GenieACS where the units look for it, and checking that the four services answer, is in [Getting started](https://geiserx.github.io/genieacs-services/getting-started/). The Supervisord config needs one change before it goes under your distribution's Supervisord. [Usage](https://geiserx.github.io/genieacs-services/usage/#under-the-system-supervisord) has it.

## Documentation

The full documentation is at [geiserx.github.io/genieacs-services](https://geiserx.github.io/genieacs-services/).

- [Getting started](https://geiserx.github.io/genieacs-services/getting-started/): install GenieACS where the units look for it, copy the units, check that the four services answer
- [Usage](https://geiserx.github.io/genieacs-services/usage/): day-to-day systemd commands, updating the files, the two ways to run the Supervisord config, which tag targets which GenieACS version
- [Configuration](https://geiserx.github.io/genieacs-services/configuration/): the paths and user each file expects, a different install path, the GenieACS environment file
- [How it works](https://geiserx.github.io/genieacs-services/how-it-works/): what each line of the units and the Supervisord config does, and how the units differ from GenieACS's guide
- [Troubleshooting](https://geiserx.github.io/genieacs-services/troubleshooting/): symptoms seen on a real install, their cause and the fix

## Related projects

Part of the GenieACS family: [genieacs-container](https://github.com/GeiserX/genieacs-container), [genieacs-ansible](https://github.com/GeiserX/genieacs-ansible), [genieacs-mcp](https://github.com/GeiserX/genieacs-mcp), [genieacs-ha](https://github.com/GeiserX/genieacs-ha), [genieacs-sim-container](https://github.com/GeiserX/genieacs-sim-container). The full list is in [genieacs-container's related projects](https://github.com/GeiserX/genieacs-container/blob/main/docs/related.md).

## License

[GPL-3.0-or-later](LICENSE)
