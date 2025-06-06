# Notes on NixOS

## General

### force renew SSL certificates
after credentials changing, files corruption, etc.
```bash
sudo systemctl clean --what=state acme-<domain>.service
sudo systemctl start acme-<domain>.service
```
