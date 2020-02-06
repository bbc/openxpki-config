
Add TLS Server Certificate for the OpenXPKI Web UI and EST Server

* Create a TLS certificate for the machines hostname
* Copy TLS Certificate Bundle (Server Cert and Intermediates) to `openxpki-config/contrib/https/<filename>.crt`
* Copy Certificate private key to `openxpki-config/contrib/https/<filename>.pem`
* These will be copied to `/etc/apache2/ssl.crt/openxpki.crt` when the docker container is started
