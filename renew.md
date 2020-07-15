```bash
docker run --rm -it --name certbot \
-v "/docker-volumes/etc/letsencrypt:/etc/letsencrypt" \
-v "/docker-volumes/var/lib/letsencrypt:/var/lib/letsencrypt" \
-v "/docker-volumes/data/letsencrypt:/data/letsencrypt" \
-v "/docker-volumes/var/log/letsencrypt:/var/log/letsencrypt" \
certbot/certbot renew --webroot -w /data/letsencrypt --quiet && docker kill --signal=HUP loadbalancer
```

docker run --rm -it --name certbot \
-v "/docker-volumes/etc/letsencrypt:/etc/letsencrypt" \
-v "/docker-volumes/var/lib/letsencrypt:/var/lib/letsencrypt" \
-v "/docker-volumes/data/letsencrypt:/data/letsencrypt" \
-v "/docker-volumes/var/log/letsencrypt:/var/log/letsencrypt" \
certbot/certbot renew --webroot -w /data/letsencrypt --quiet && docker kill --signal=HUP loadbalancer

sudo docker run -it --rm \
-v /docker-volumes/etc/letsencrypt:/etc/letsencrypt \
-v /docker-volumes/var/lib/letsencrypt:/var/lib/letsencrypt \
-v $PWD/html:/data/letsencrypt \
-v /docker-volumes/var/log/letsencrypt:/var/log/letsencrypt \
certbot/certbot \
certonly --webroot \
--email leon.sczepansky@gmail.com --agree-tos --no-eff-email \
--webroot-path=/data/letsencrypt \
-d magicpony.de -d www.magicpony.de