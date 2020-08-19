# NMOS Docker Setup

Guide on how to setup an instance of OpenXPKI, to provide TLS Certificates using EST, in accordance with BCP-003-03.

Install docker on ubuntu
1. Install Docker
    * https://docs.docker.com/install/linux/docker-ce/ubuntu/
2. Install Docker Compose
    * https://docs.docker.com/compose/install/

Create instance of OpenXPKI, using Docker
1. Clone OpenXPKI Docker repo (https://github.com/bbc/openxpki-docker)
```
git clone https://github.com/bbc/openxpki-docker
```
2. Clone NMOS config
```
cd openxpki-docker
git clone git@github.com:bbc/openxpki-config.git --branch=nmos
```
3. Add a TLS certificate for OpenXPKI webUI and EST server for your domain
    * Create a TLS certificate for the machine's hostname
    * Copy TLS Certificate Bundle (Server certificate and any intermediate certificates) to `openxpki-config/contrib/https/<filename>.crt`
    * Copy Certificate private key to `openxpki-config/contrib/https/<filename>.pem`
4. Add CA to config folder
    * Copy the Root CA and Issuing Certificate along with their private keys to `openxpki-config/ca/nmos/` with the following names, the suffix -XX must contain only numbers and is used as generation identifier on import.
    * `root(-XX).crt` for root certificates `ca-signer(-XX).crt` for signer certificates `vault(-XX).crt`
    * `root(-XX).pem` for root key `ca-signer(-XX).pem` for signer key `vault(-XX).pem`
5. Restart container to load new config
```
sudo docker compose up
```
6. Load CA
```
sudo docker-compose exec openxpki-server /usr/bin/setup-cert.sh
```

**Adding Externally Trusted CA, for Client verification**

1. Append certificates to the `openxpki-config/externallyTrustedCAs/external.ca.certs.pem`
2. Restart `openxpki-client` for changes to be imported
```
sudo docker-compose restart openxpki-client
```


**Bug fixes, should be fixed when docker image updated**

1. Change permissions of OpenXPKI logs folder `/var/log/openxpki/`, as EST endpoint cannot start without creating a log file, this only need to be done the first time the log volume is created
```
sudo docker-compose exec openxpki-client /bin/bash
chown -R www-data /var/log/openxpki
chgrp -R openxpki /var/log/openxpki
```

**Certificates showing as offline in Web UI**

If the CA is showing as Offline is Web UI, do the following to diagnose the problem
1. Log into container and check the `catchall.log` for errors
2. If OpenXPKI failed to open cert check the file permissions in `/etc/openxpki/ca`
    * Can be changed to `openxpki`
```
chgrp -R openxpki /etc/openxpki/ca
```
3. Check password for certificate private key in `/etc/openxpki/config.d/realm/nmos/crypto.yaml`
