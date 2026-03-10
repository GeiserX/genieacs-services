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
