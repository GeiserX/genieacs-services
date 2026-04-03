<p align="center">
  <img src="docs/images/banner.svg" alt="GenieACS Services banner" width="900"/>
</p>

<h1 align="center">GenieACS Services</h1>

<p align="center">
  <a href="https://github.com/GeiserX/genieacs-services/stargazers"><img src="https://img.shields.io/github/stars/GeiserX/genieacs-services?style=flat-square&logo=github" alt="GitHub Stars"/></a>
  <a href="https://github.com/GeiserX/genieacs-services/network/members"><img src="https://img.shields.io/github/forks/GeiserX/genieacs-services?style=flat-square&logo=github" alt="GitHub Forks"/></a>
  <a href="https://github.com/GeiserX/genieacs-services/blob/master/LICENSE"><img src="https://img.shields.io/github/license/GeiserX/genieacs-services?style=flat-square" alt="License"/></a>
</p>

<p align="center"><strong>Systemd and Supervisord service files for GenieACS.</strong></p>

---

Recommended way to deploy GenieACS, instructions here: https://github.com/genieacs/genieacs/wiki/Docker-Installation-with-Docker-Compose

## Instructions for Systemd:

    cp genieacs-cwmp.service /etc/systemd/system/
    systemctl enable genieacs-cwmp.service
    
    cp genieacs-nbi.service /etc/systemd/system/
    systemctl enable genieacs-nbi.service
    
    cp genieacs-fs.service /etc/systemd/system/
    systemctl enable genieacs-fs.service
    
    cp genieacs-ui.service /etc/systemd/system/
    systemctl enable genieacs-ui.service

In order to see & follow the logs: 

    journalctl -f -u genieacs-X.service

## Instructions for Supervisord:

Just copy the `supervisord.conf` file to `/etc/supervisor/conf.d/`

## GenieACS Ecosystem

This project is part of a broader set of tools for working with GenieACS:

| Project | Type | Description |
|---------|------|-------------|
| [genieacs-docker](https://github.com/GeiserX/genieacs-docker) | Docker + Helm | Production-ready multi-arch Docker image and Helm chart |
| [genieacs-ansible](https://github.com/GeiserX/genieacs-ansible) | Ansible Collection | Dynamic inventory plugin and device management modules |
| [genieacs-mcp](https://github.com/GeiserX/genieacs-mcp) | MCP Server | AI-assisted device management via Model Context Protocol |
| [genieacs-ha](https://github.com/GeiserX/genieacs-ha) | HA Integration | Home Assistant integration for TR-069 monitoring |
| [n8n-nodes-genieacs](https://github.com/GeiserX/n8n-nodes-genieacs) | n8n Node | Workflow automation for GenieACS |
| [genieacs-sim-docker](https://github.com/GeiserX/genieacs-sim-docker) | Simulator | Docker-based GenieACS simulator for testing |
