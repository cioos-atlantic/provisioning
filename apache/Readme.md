# Apache reverse proxy server configuration

#### File path
 Be sure that the following two files are placed in the right Apache configuration folder.
 
 `/etc/httpd/conf/httpd.conf`
 
 `/etc/httpd/conf/httpd-le-ssl.conf`
 
#### Note: These require that a valid SSL (https) certificate is loaded in the file.

* Load the /new_server versions for a fresh VM install.
    * These do not include the certificates in the ssl Apache conf.
* Load the /live_server_fix to easily fix any corrupted files.
 
```
## SSL directives
SSLCertificateFile /etc/letsencrypt/live/software-test.cioos-atlantic.ca/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/software-test.cioos-atlantic.ca/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
SSLCertificateChainFile /etc/letsencrypt/live/software-test.cioos-atlantic.ca/chain.pem
```
