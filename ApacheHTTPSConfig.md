## Enable SSL in Apache (OSX)

The following will guide you through the process of enabling SSL on a Apache webserver
- The instructions have been verified with OSX El Capitan (10.11.2) running **Apache 2.4.16**
- The instructions assume you already have a basic Apache configuration enabled on OSX, if this is not the case feel free to consult Gist: "[Enable Apache HTTP server (OSX)](http://)"

#### Apache SSL Configuration

Create a directory within `/etc/apache2/` using **Terminal**.app: `sudo mkdir /etc/apache2/ssl`
Next, generate two host keys:
```
cd /usr/local/etc/httpd
mkdir ssl
cd ssl
sudo openssl genrsa -out /usr/local/etc/httpd/ssl/server.key 2048
sudo openssl genrsa -out /usr/local/etc/httpd/ssl/localhost.key 2048
sudo openssl rsa -in /usr/local/etc/httpd/ssl/localhost.key -out /usr/local/etc/httpd/ssl/localhost.key.rsa
```

Create a configuration file using **Terminal**.app: `sudo touch /usr/local/etc/httpd/ssl/localhost.conf`
Edit the newly created configuration file and add the following:
```
[req]
default_bits = 1024
distinguished_name = req_distinguished_name
req_extensions = v3_req

[req_distinguished_name]

[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
DNS.2 = *.localhost
```

Generate the required Certificate Requests using **Terminal**.app:
```
sudo openssl req -new -key /usr/local/etc/httpd/ssl/server.key -subj "/C=/ST=/L=/O=/CN=/emailAddress=/" -out /usr/local/etc/httpd/ssl/server.csr
sudo openssl req -new -key /usr/local/etc/httpd/ssl/localhost.key.rsa -subj "/C=/ST=/L=/O=/CN=localhost/" -out /usr/local/etc/httpd/ssl/localhost.csr -config /usr/local/etc/httpd/ssl/localhost.conf
```
**Note**: Complete the values `C= ST= L= O= CN=` to reflect your own organizational structure, where:
* `C=` eq. Country: The two-letter ISO abbreviation for your country.
* `ST=` eq. State or Province: The state or province where your organization is legally located.
* `L=` eq. City or Locality: The city where your organization is legally located.
* `O=` eq. Organization: he exact legal name of your organization.
* `CN=` eq. Common Name: The fully qualified domain name for your web server


Use the Certificate Requests to sign the SSL Certificates using **Terminal**.app:
```
sudo openssl x509 -req -days 365 -in /usr/local/etc/httpd/ssl/server.csr -signkey /usr/local/etc/httpd/ssl/server.key -out /usr/local/etc/httpd/ssl/server.crt
sudo openssl x509 -req -extensions v3_req -days 365 -in /usr/local/etc/httpd/ssl/localhost.csr -signkey /usr/local/etc/httpd/ssl/localhost.key.rsa -out /usr/local/etc/httpd/ssl/localhost.crt -extfile /usr/local/etc/httpd/ssl/localhost.conf
```

Add the SSL Certificate to **Keychain Access**.
```
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain /usr/local/etc/httpd/ssl/localhost.crt
```

#### Apache Configuration
Edit the Apache main configuration file `/usr/local/etc/httpd/httpd.conf` and enable the required modules to support SSL :
```
LoadModule socache_shmcb_module libexec/apache2/mod_socache_shmcb.so
LoadModule ssl_module libexec/apache2/mod_ssl.so
```

Enable Secure (SSL/TLS) connections
```
Include /usr/local/etc/httpd/extra/httpd-ssl.conf
```

#### Apache Virtual Host Configuration
Edit the Virtual Hosts file `/usr/local/etc/httpd/extra/httpd-vhosts.conf` and add the SSL Directive at the end of the file:
```
<VirtualHost *:443>
    ServerName localhost
    DocumentRoot "/users/macsi/Sites/dupak-jfd"

    SSLEngine on
    SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
    SSLCertificateFile /usr/local/etc/httpd/ssl/localhost.crt
    SSLCertificateKeyFile /usr/local/etc/httpd/ssl/localhost.key

    <Directory "/users/macsi/Sites/dupak-jfd">
        Options Indexes FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
</VirtualHost>
```

Finally restart Apache using **Terminal**.app : `sudo apachectl restart`
Open Safari and visit [https://localhost](https://localhost) to verify your configuration.
