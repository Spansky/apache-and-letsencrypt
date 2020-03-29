# Apache in a Docker

## Temporary Apache

The temporary Apache has the only purpose for exposing a temporary website over http / port 80. For preparing it you need those three files:

<details><summary>index.html (click to see the content)</summary>

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Temporary Webserver</title>
    <style>
        body {
            margin: 0;
            width: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-content: center;
            height: 100vh;
            align-items: center;
        }
    </style>
</head>
<body>
    <h1>This is the webserver for the Acme-Challenge</h1>
</body>
</html>
```
</details>

<details><summary>Dockerfile (click to see the content)</summary>

```Dockerfile
FROM alpine:latest
LABEL author="Leon Sczepansky"
ENV server_name=localhost
RUN apk add --no-cache apache2
RUN rm -rf /var/www/localhost/cgi-bin/
CMD exec /usr/sbin/httpd -D FOREGROUND -f /etc/apache2/httpd.conf
```
</details>

<details><summary>Configfile httpd.conf (click to see the content)</summary>

```plaintext
ServerRoot /var/www

LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
LoadModule authn_file_module modules/mod_authn_file.so
LoadModule authn_core_module modules/mod_authn_core.so
LoadModule authz_host_module modules/mod_authz_host.so
LoadModule authz_groupfile_module modules/mod_authz_groupfile.so
LoadModule authz_user_module modules/mod_authz_user.so
LoadModule authz_core_module modules/mod_authz_core.so
LoadModule access_compat_module modules/mod_access_compat.so
LoadModule auth_basic_module modules/mod_auth_basic.so
LoadModule reqtimeout_module modules/mod_reqtimeout.so
LoadModule filter_module modules/mod_filter.so
LoadModule mime_module modules/mod_mime.so
LoadModule log_config_module modules/mod_log_config.so
LoadModule env_module modules/mod_env.so
LoadModule headers_module modules/mod_headers.so
LoadModule setenvif_module modules/mod_setenvif.so
LoadModule version_module modules/mod_version.so
LoadModule unixd_module modules/mod_unixd.so
LoadModule status_module modules/mod_status.so
LoadModule autoindex_module modules/mod_autoindex.so
LoadModule dir_module modules/mod_dir.so
LoadModule alias_module modules/mod_alias.so
LoadModule negotiation_module modules/mod_negotiation.so
LoadModule rewrite_module modules/mod_rewrite.so

Listen 80

<IfModule unixd_module>
    User apache
    Group apache
</IfModule>

ServerName $server_name
ServerAdmin leon.sczepansky@example.org
ServerTokens Prod
ServerSignature Off

DocumentRoot  "/var/www/localhost/htdocs"

<Directory /.well-known/acme-challenge>
        Allow from all
</Directory>

<Directory "/var/www/localhost/htdocs">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

ErrorLog                                logs/error.log
LogLevel info
```
</details>

The folder tree should look like this:
```plaintext
├── letsencrypt
│   ├── docker-compose.yml
│   ├── Dockerfile
│   ├── html
│   │   └── index.html
│   └── httpd.conf
└── README.md
```


* Build it via `docker build -t lets-encrypt-apache .`
* Testrun it via
    ```bash
    docker run --rm -p 80:80 --name le-apache \
    -v $PWD/html/:/var/www/localhost/htdocs/ \
    -v $PWD/httpd.conf:/etc/apache2/httpd.conf \
    -e 'server_name=magicpony.de' \
    lets-encrypt-apache:latest
    ```

It is also quite handy to use `docker-compose` with following `docker-compose.yml` file:

```plaintext
version: '3.7'
services:
  le-apache:
    container_name: 'le-apache'
    image: lets-encrypt-apache:latest
    ports:
      - "80:80"
    volumes:
      - ./httpd.conf:/etc/apache2/httpd.conf
      - ./html:/var/www/localhost/htdocs/
    networks:
      - docker-network
networks:
  docker-network:
    driver: bridge
```

* Testrun it via `docker-compose up -d` 

![testserver](testserver-apache-port-80acme-challenge.pngx)


## Run Test Certification

The following commmand downloads and runs the official docker hub certbot-image. It assumes that you are in 

```bash
docker run -it --rm \
-v /docker-volumes2/etc/letsencrypt:/etc/letsencrypt \
-v /docker-volumes2/var/lib/letsencrypt:/var/lib/letsencrypt \
-v "/home/leon/apache-and-letsencrypt/letsencrypt/html:/data/letsencrypt" \
-v /docker-volumes2/var/log/letsencrypt:/var/log/letsencrypt \
certbot/certbot \
certonly --webroot \
--register-unsafely-without-email --agree-tos \
--webroot-path=/data/letsencrypt \
--staging \
-d magicpony.de -d www.magicpony.de
```




### 