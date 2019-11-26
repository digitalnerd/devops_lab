# Deploy Testlink 

## Run Deploy Testlink

$ ansible-playbook --tags "deploy,php56" deploy_testlink.yml

### Modify the custom TestLink configuration file:
Use the vi text editor to open this configuration file:

```bash
sudo vi /var/www/html/testlink-code-1.9.16/custom_config.inc.php
```

Find the following lines:

```bash
// $tlCfg->log_path = '/var/testlink-ga-testlink-code/logs/'; /* unix example */
// $g_repositoryPath = '/var/testlink-ga-testlink-code/upload_area/';  /* unix example */
```

Replace them with:

```bash
$tlCfg->log_path = '/var/www/html/testlink-code-1.9.16/logs/';
$g_repositoryPath = '/var/www/html/testlink-code-1.9.16/upload_area/';
```

Save and quit:

```bash
:wq!
```

## Post-install Testlink

### Security measures after the installation:
For security purposes, you should restrict the apache user's permissions after the installation:

```bash
sudo chown -R root:root /var/www/html/testlink-code-1.9.16
sudo chown -R apache:apache /var/www/html/testlink-code-1.9.16/{gui,logs,upload_area}
sudo systemctl restart httpd.service
```

Additionally, you should remove the `/var/www/html/testlink-code-1.9.16/install` directory:

```bash
sudo rm -rf /var/www/html/testlink-code-1.9.16/install
```

##  Set up a self-signed SSL certificate

### Install Mod SSL
```bash
sudo yum install mod_ssl -y 
```

### Create a New Certificate
```bash
sudo mkdir /etc/ssl/private
sudo chmod 700 /etc/ssl/private
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
...
Country Name (2 letter code) [XX]:RU
State or Province Name (full name) []:SPB
Locality Name (eg, city) [Default City]:SPB 
Organization Name (eg, company) [Default Company Ltd]:T-SYSTEMS
Organizational Unit Name (eg, section) []:FAS
Common Name (eg, your name or your server's hostname) []:testlink.local
Email Address []:konstantin.levin@t-systems.com
```

```bash
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
cat /etc/ssl/certs/dhparam.pem | sudo tee -a /etc/ssl/certs/apache-selfsigned.crt

```

### Set Up the Certificate
```bash
sudo vim /etc/httpd/conf.d/ssl.conf
...
<VirtualHost _default_:443>
. . .
DocumentRoot "/var/www/html/testlink-code-1.9.16"
ServerName 10.220.177.99:443

. . .
# SSLProtocol all -SSLv2
. . .
# SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5:!SEED:!IDEA

...
SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
...
```

**(Recommended) Modify the Unencrypted Virtual Host File to Redirect to HTTPS**
```bash
<VirtualHost *:80>
        ServerName www.example.com
        Redirect "/" "https://www.example.com/"
</VirtualHost>
```

### Activate the Certificate
```bash
sudo apachectl configtest
Syntax OK
```

```bash
sudo systemctl restart httpd.service
```