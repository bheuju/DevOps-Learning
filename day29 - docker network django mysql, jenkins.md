## create django container

docker run --name django

## create mysql container

docker run --name mysql

## connect and run them

???

---

# Jenkins

Jenkinsfile

Plugins

- gitlab
- sonarqube
- owasp
- zaproxy
- kubernetes

Freestyle project

```bash
# for jenkins with git ssh
# this will allow the use of git ssh url
docker run --name jenkins -d \
  -v jenkins_home:/var/jenkins_home \
  -v ~/.ssh/id_rsa:/var/jenkins_home/.ssh/id_rsa:ro \
  -v ~/.ssh/known_hosts:/var/jenkins_home/.ssh/known_hosts:ro \
  -v ~/.ssh/config:/var/jenkins_home/.ssh/config:ro \
  -p 8080:8080 -p 50000:50000 \
  --restart=on-failure \
  --add-host=gitlab.dev.me:192.168.22.98 \
  jenkins/jenkins
```
