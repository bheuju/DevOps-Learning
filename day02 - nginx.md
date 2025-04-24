# Nginx Config

We can move the default from /sites-available to /conf.d but extension must be \*.conf
`/etc/nginx$ cat nginx.conf | grep include`

```
include /etc/nginx/modules-enabled/*.conf;
        include /etc/nginx/mime.types;
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
```

```nginx
server {
        listen 80 default_server;
        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;
        server_name dev.test;
}
```

Add to /drivers/etc/host

```
192.168.22.100 dev.test
192.168.22.100 dev1.test
192.168.22.100 dev2.test
192.168.22.100 dev3.test
```

`sudo nginx -t`

`bishal@bishal:/etc/nginx/conf.d$ cat *`

```nginx
# dev
server {
        listen 80;
        root /var/www/html/dev;
        index index.html;
        server_name dev.test;

        access_log /var/log/nginx/devops_access.log
        error_log /var/log/nginx/devops_error.log
}
# dev1
server {
        listen 80;
        root /var/www/html/dev1;
        index index.html;
        server_name dev1.test;
}
# dev2
server {
        listen 80;
        root /var/www/html/dev2;
        index index.html;
        server_name dev2.test;
}

```

### Logging files in nginx configuration

```
access_log /var/log/nginx/devops_access.log
error_log /var/log/nginx/devops_error.log
```
