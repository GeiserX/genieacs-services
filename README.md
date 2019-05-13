# Systemd/Supervisord service files for GenieACS

Recommended way to deploy GenieACS, instructions here: https://github.com/genieacs/genieacs/wiki/Docker-Installation-with-Docker-Compose 

## Instructions for Systemd:

    cp genieacs-cwmp.service /etc/systemd/system/
    systemctl enable genieacs-cwmp.service
    
    cp genieacs-nbi.service /etc/systemd/system/
    systemctl enable genieacs-nbi.service
    
    cp genieacs-fs.service /etc/systemd/system/
    systemctl enable genieacs-fs.service
    
    cp genieacs-gui.service /etc/systemd/system/
    systemctl enable genieacs-gui.service

In order to see & follow the logs: 

    journalctl -f -u genieacs-X.service

## Instructions for Supervisord:

Just copy the `supervisord.conf` file to `/etc/supervisor/conf.d/`
