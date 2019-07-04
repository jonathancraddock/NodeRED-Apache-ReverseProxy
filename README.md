# NodeRED with an Apache Reverse Proxy

I found it quite difficult to find good setup guides for anyone wanting to put NodeRED behind an Apache reverse proxy. The following config certainly works, but with the disclaimer that it probably does require some further fine-tuning.

## Apache Modules

After installing Apache, I enabled the following modules:

```bash
sudo a2enmod ...
```

* proxy
* proxy_http
* proxy_balancer
* lbmethod_byrequests
* proxy_wstunnel
* xml2enc
* headers
* ssl

## Apache Config

The conf below is working ok. I'd like to enable TLSv1.3, but at he time of writing the Apache2 version in Ubuntu 18.04 LTS doesn't support it.

```bash
<IfModule mod_ssl.c>
<VirtualHost *:443>

  ServerAdmin webmaster@localhost
  ServerName example.com

  SSLEngine on

  SSLOpenSSLConfCmd Protocol "-ALL, TLSv1.2"
  SSLOpenSSLConfCmd SignatureAlgorithms RSA+SHA384:ECDSA+SHA256
  SSLProtocol -all +TLSv1.2
#                                 +TLSv1.3
  SSLHonorCipherOrder on
  SSLCompression off

  SSLOptions +FakeBasicAuth +ExportCertData +StrictRequire
  SSLCipherSuite "HIGH:!aNULL:!MD5:!3DES:!CAMELLIA:!AES128"

  Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
  Header always set X-Frame-Options SAMEORIGIN
  Header always set X-Content-Type-Options nosniff

  ProxyPreserveHost On

	ProxyPass               /comms        ws://localhost:1880/comms
	ProxyPassReverse        /comms        ws://localhost:1880/comms
	ProxyPass               /             http://127.0.0.1:1880/
	ProxyPassReverse        /             http://127.0.0.1:1880/

  SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
  Include /etc/letsencrypt/options-ssl-apache.conf

</VirtualHost>
</IfModule>
```
