```bash
docker tag sdfsasd nexus.dev.me:8082/mariadb  # tag image
docker image list                             # list image
docker push nexus.dev.me:8082/mariadb         #
```

```
docker login -u admin -p redhat nexus.dev.me:8082
```

cat .bash_logout
comment them

### add gitlab secure usename password variables

Settings > CI/CD > variables

```yml
stages:
  - clone

workflow:
  rules
  - when: always

variables:
  VERSION: "10.0"

djangoapp:
  tags:
    - local
  stage: clone
  script:
    - echo "${username} and ${password}"
    - docker login -u ${username} -p ${password} nexus.dev.me:8082
    - cd /home/bishal/django
    - git pull origin master
    - docker build -t app:${VERSION} .
    - docker tag app:${VERSION} nexus.dev.me:8082/app
    - docker push nexus.dev.me

```
