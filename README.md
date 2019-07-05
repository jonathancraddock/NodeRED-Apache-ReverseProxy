# NodeRED with an Apache Reverse Proxy

I found it quite difficult to find good setup guides for anyone wanting to put NodeRED behind an Apache reverse proxy. The following config certainly works, but with the disclaimer that it probably does require some further fine-tuning.

In particular, several otherwise very helpful guides don't mention *web sockets*, and a couple that do include that info don't remind you to enable proxy_wstunnel. (Yes, it sounds obvious now, but I didn't find the errors very informative!)

## Apache Modules

After installing Apache, I enabled the following modules:

```bash
sudo a2enmod ...
```

* proxy
* proxy_http
* proxy_balancer
* lbmethod_byrequests
* **proxy_wstunnel**
* xml2enc
* headers
* ssl

## Apache Config

The conf below is working ok. I'd like to enable TLSv1.3, but at he time of writing the Apache2 version in Ubuntu 18.04 LTS doesn't support it. Also worth noting that the choice of cipher suites below may prevent some out-of-date browsers from accessing the page.

```bash
<IfModule mod_ssl.c>
<VirtualHost *:443>

ServerAdmin webmaster@localhost
ServerName nodered.example.com

	SSLEngine on
	SSLHonorCipherOrder on
	SSLCompression off

	SSLOpenSSLConfCmd Protocol "-ALL, TLSv1.2"
    SSLOpenSSLConfCmd DHParameters "/etc/ssl/certs/dhparam.pem"

	SSLProtocol -all +TLSv1.2 
	# +TLSv1.3
	
	SSLOptions +FakeBasicAuth +ExportCertData +StrictRequire

	SSLCipherSuite ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384

	Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
	Header always set X-Frame-Options SAMEORIGIN
	Header always set X-Content-Type-Options nosniff

	ProxyPreserveHost On

	ProxyPass               /comms        ws://localhost:1880/comms
	ProxyPassReverse        /comms        ws://localhost:1880/comms
	ProxyPass               /             http://127.0.0.1:1880/
	ProxyPassReverse        /             http://127.0.0.1:1880/


	SSLCertificateFile /etc/letsencrypt/live/nodered.example.com/fullchain.pem
	SSLCertificateKeyFile /etc/letsencrypt/live/nodered.example.com/privkey.pem
	Include /etc/letsencrypt/options-ssl-apache.conf

</VirtualHost>
</IfModule>
```
