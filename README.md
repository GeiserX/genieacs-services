<p align="center">
  <img src="docs/images/banner.svg" alt="GenieACS Services banner" width="900"/>
</p>

# Systemd/Supervisord service files for GenieACS

[![GitHub Stars](https://img.shields.io/github/stars/GeiserX/genieacs-services?style=flat-square&logo=github)](https://github.com/GeiserX/genieacs-services/stargazers) [![GitHub Forks](https://img.shields.io/github/forks/GeiserX/genieacs-services?style=flat-square&logo=github)](https://github.com/GeiserX/genieacs-services/network/members) [![License](https://img.shields.io/github/license/GeiserX/genieacs-services?style=flat-square)](https://github.com/GeiserX/genieacs-services/blob/master/LICENSE)

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
