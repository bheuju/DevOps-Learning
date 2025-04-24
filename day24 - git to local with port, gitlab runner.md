mattermost - alternative to trello, slack

# Config ssh key setup for new server - specify port

> ~/.ssh/config

```bash
# Bishal Heuju - Local Gitlab
Host gitlab.dev.me
  HostName gitlab.dev.me
  Port 2222
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa_nopass
```

## Gitlab CI

setup gitlab runner

Change gitlab-runner service to run runner as service (default may have problems)

```ini
# sudo cat /etc/systemd/system/gitlab-runner.service

[Unit]
Description=GitLab Runner
ConditionFileIsExecutable=/usr/local/bin/gitlab-runner

After=network.target

[Service]
StartLimitInterval=5
StartLimitBurst=10
ExecStart=/usr/local/bin/gitlab-runner "run" "--config" "/etc/gitlab-runner/config.toml" "--working-directory" "/home/bishal" "--service" "gitlab-runner" "--user" "bishal"
Restart=always

RestartSec=120
EnvironmentFile=-/etc/sysconfig/gitlab-runner

[Install]
WantedBy=multi-user.target
```

Gitlab CI file

```yml
# add in root directory: .gitlab-ci.yml
stages:
  - clone
djangoapp:
  tags:
    - local
  stage: clone
  script:
    - cd /var/www
    - git pull origin
```

---

# Container

```bash
docker pull nginx
docker image list

docker run --name webserver -p 80:80 nginx
docker rm webserver
docker run -d --name webserver -p 80:80 nginx

docker exec -it webserver bash

# then change contents of nginx html, then check page

# docker pull nginx:1.18.0  # specify version
# docker run -d --name webserver1 -p 80:80 nginx

docker stop webserver
docker start webserver

/var/lib/docker/containers # place where containers are stored


mkdir myweb
nano index.html

docker run -d --name webserver -v /home/bishal/myweb:/usr/share/nginx/html -p 80:80 nginx # copy contents of myweb to nginx/html
# then change contents of nginx html, then check page
```

```bash
mkdir conf
mdkir app

# myweb/app/index.html
-------------------------
I am index. Testing.
-------------------------
# myweb/conf/default.conf
-------------------------
server {
  listen 80;
  server_name _;
  root /var/www/html;
  index index.html;
}
-------------------------

# copy contents of myweb/conf to /etc/nginx/conf.d
# copy contents of myweb/app to /etc/nginx/conf.d
docker run -d --name webserver \
  -v /home/bishal/myweb/conf:/etc/nginx/conf.d \
  -v /home/bishal/myweb/app:/var/www/html \
  -p 80:80 nginx

```

---

### Own image

```
docker run -d --name webserver \
  -v /home/bishal/myweb/conf:/etc/nginx/conf.d \
  -v /home/bishal/myweb/app:/var/www/html \
  -p 80:80 nginx
```

We want to not do this by creating our own image

```
|--- myweb
|    |--- app
|    |--- conf

```

```dockerfile
# myweb/Dockerfile
FROM nginx:1.10.1-alpine
COPY ./app /var/www/html
COPY ./conf /etc/nginx/conf.d
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```bash
docker build -t nginxm .                        # build image - auto add 'latest' tag
docker build -t nginxm:v1.0.0 .                 # build image - version v1.0.0
docker run -d --name webserver -p 80:80 nginxm  # run new image
```
