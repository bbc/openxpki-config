# NMOS Docker Setup

Guide on how to setup an instance of OpenXPKI, to provide TLS Certificates using EST, in accordance with BCP-003-03.

Install docker on ubuntu
1. Install Docker
    * https://docs.docker.com/install/linux/docker-ce/ubuntu/
2. Install Docker Compose
    * https://docs.docker.com/compose/install/

Create instance of OpenXPKI, using Docker
1. Clone OpenXPKI Docker repo
```
git clone git@github.com:openxpki/openxpki-docker.git
```
2. Clone NMOS config
```
cd openxpki-docker
git clone git@github.com:bbc/openxpki-config.git --branch=nmos
```
3. Add TLS certificate for OpenXPKI webUI and EST server
    * Create a TLS certificate for the machines hostname
    * Copy TLS Certificate Bundle (Server Cert and Intermediates) to `openxpki-config/contrib/https/<filename>.crt`
    * Copy Certificate private key to `openxpki-config/contrib/https/<filename>.pem`
    * These will be copied to `/etc/apache2/ssl.crt/openxpki.crt` when the docker container is started
4. Add CA to config folder
    * Copy the Root CA and Issuing Certificates along with their private keys to `openxpki-config/ca/nmos/`
    * If the private keys are encrypted, add a `*.pass` file containing the password
5. Create self signed certificate for data vault
6. Create symbolic link to data vault and issuing ca key
```
cd openxpki-config/ca/
ln -s nmos/OpenXPKI_DataVault.key vault-1.pem
cd nmos
ln -s OpenXPKI_Issuing_CA.key ca-signer-1.pem
```
7. The directory `openxpki-config/ca/nmos/` should look like this
```
-r--r--r--  1 OpenXPKI_DataVault.crt
-r--r-----  1 OpenXPKI_DataVault.key
-r--------  1 OpenXPKI_DataVault.pass
-rwxrwxrwx  1 OpenXPKI_Issuing_CA.crt
-rw-r--r--  1 OpenXPKI_Issuing_CA.key
-r--------  1 OpenXPKI_Issuing_CA.pass
-rwxrwxrwx  1 OpenXPKI_Root_CA.crt
-rw-r--r--  1 OpenXPKI_Root_CA.key
-r--------  1 OpenXPKI_Root_CA.pass
-rw-r--r--  1 README.md
lrwxrwxrwx  1 ca-signer-1.pem -> OpenXPKI_Issuing_CA.key
```
5. Create container instance
```
sudo docker-compose up
```
6. Configure CA, by importing Certificates into certificate database
```
sudo docker-compose exec openxpki-client /bin/bash
openxpkiadm certificate import  --file /etc/openxpki/ca/nmos/OpenXPKI_Root_CA.crt
openxpkiadm certificate import  --file /etc/openxpki/ca/nmos/OpenXPKI_Issuing_CA.crt --realm nmos --token certsign
openxpkiadm certificate import  --file /etc/openxpki/ca/nmos/OpenXPKI_DataVault.crt
openxpkiadm alias --realm nmos --token datasafe --identifier `openxpkiadm certificate id --file /etc/openxpki/ca/nmos/OpenXPKI_DataVault.crt`
```
7. If import successful database the database should look like this
```
$ openxpkiadm alias --realm nmos
=== functional token ===
scep (scep):
Alias     : scep-1
Identifier: YsBNZ7JYTbx89F_-Z4jn_RPFFWo
NotBefore : 2015-01-30 20:44:40
NotAfter  : 2016-01-30 20:44:40

vault (datasafe):
Alias     : vault-1
Identifier: lZILS1l6Km5aIGS6pA7P7azAJic
NotBefore : 2015-01-30 20:44:40
NotAfter  : 2016-01-30 20:44:40

ca-signer (certsign):
Alias     : ca-signer-1
Identifier: Sw_IY7AdoGUp28F_cFEdhbtI9pE
NotBefore : 2015-01-30 20:44:40
NotAfter  : 2018-01-29 20:44:40

=== root ca ===
current root ca:
Alias     : root-1
Identifier: fVrqJAlpotPaisOAsnxa9cglXCc
NotBefore : 2015-01-30 20:44:39
NotAfter  : 2020-01-30 20:44:39

upcoming root ca:
  not set
```
8. Restart container to load new config
```
sudo docker-compose restart
```

**Adding Externally Trusted CA, for Client verification**

1. Append certificates to the `openxpki-config/externallyTrustedCAs/external.ca.certs.pem`
2. Restart `openxpki-client` for changes to be imported
```
sudo docker-compose restart openxpki-client
```


**Bug fixes, should be fixed when docker image updated**

1. Change permissions of OpenXPKI logs folder `/var/log/openxpki/`, as EST endpoint cannot start without creating a log file
```
sudo docker-compose exec openxpki-client /bin/bash
chown -R www-data /var/log/openxpki
chgrp -R openxpki /var/log/openxpki
```
2. Enable re-enroll endpoint
```
sudo docker-compose exec openxpki-client /bin/bash
cp /etc/openxpki/est.fcgi /usr/lib/cgi-bin/est.fcgi
```
OpenXPKI is now ready to accept EST requests

**Certificates showing as offline in Web UI**

If the CA is showing as Offline is Web UI, do the following to diagnose the problem
1. Log into container and check the `catchall.log` for errors
2. If OpenXPKI failed to open cert check the file permissions in `/etc/openxpki/ca`
    * Can be changed to `openxpki`
```
chgrp -R openxpki /etc/openxpki/ca
```
3. Check password for certificate private key in `/etc/openxpki/config.d/realm/nmos/crypto.yaml`
