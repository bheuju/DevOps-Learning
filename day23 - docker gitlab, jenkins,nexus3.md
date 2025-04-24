# Nginx configurations for gitlab, jenkins, nexus3

```nginx
➜  ~ sudo cat /etc/nginx/conf.d/gitlab.conf
server {
  add_header       X-Served-By $host;
  location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Scheme $scheme;
        proxy_set_header X-Forwarded-Proto  $scheme;
        proxy_set_header X-Forwarded-For    $remote_addr;
        proxy_set_header X-Real-IP          $remote_addr;
        proxy_pass       http://127.0.0.1:8079$request_uri;
  }
  listen 80;
  server_name gitlab.dev.me;
  access_log /var/log/gitlab_access.log;
  error_log /var/log/gitlab_error.log warn;
}
```

```nginx
➜  ~ sudo cat /etc/nginx/conf.d/jenkins.conf
server {
  add_header       X-Served-By $host;
  location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Scheme $scheme;
        proxy_set_header X-Forwarded-Proto  $scheme;
        proxy_set_header X-Forwarded-For    $remote_addr;
        proxy_set_header X-Real-IP          $remote_addr;
        proxy_pass       http://127.0.0.1:8080$request_uri;
  }
  listen 80;
  server_name jenkins.dev.me;
  access_log /var/log/jenkins_access.log;
  error_log /var/log/jenkins_error.log warn;
}
```

```nginx
➜  ~ sudo cat /etc/nginx/conf.d/nexus.conf
server {
  add_header       X-Served-By $host;
  location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Scheme $scheme;
        proxy_set_header X-Forwarded-Proto  $scheme;
        proxy_set_header X-Forwarded-For    $remote_addr;
        proxy_set_header X-Real-IP          $remote_addr;
        proxy_pass       http://127.0.0.1:8081$request_uri;
  }
  listen 80;
  server_name nexus.dev.me;
  access_log /var/log/nexus_access.log;
  error_log /var/log/nexus_error.log warn;
}
```
