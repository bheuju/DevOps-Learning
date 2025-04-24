## docker pass env file

```bash
docker run --env-file <env-filename> -p 8000:8000 django
```

https://hub.docker.com/_/mariadb

```bash
docker run --name mysql --network django -v /home/:/var/lib/mysql:Z -e MARIADB_ROOT_PASSWORD=pass -d mariadb:latest
```

docker run --env-file <env-filename> --network django -p 8000:8000 django

hostname in network

1. create mysql docker dontainer
2. docker network create

# Nexus

repo > create - docker hosted
http port: 8082
anonymous: off

https://help.sonatype.com/en/pushing-images.html

```bash
# we need to create a tag before pushing to nexus
docker tag <imageId or imageName> <nexus-hostname>:<repository-port>/<image>:<tag>
# docker tag d99b15185dfc nexus.dev.me:8082/django:v1.0.0
docker push <nexus-hostname>:<repository-port>/<image>:<tag>
# docker push nexus.dev.me:8082/django:v1.0.0
```

nexus will try to connect to https
so, in nexus - config insecure-registries

```json
# /etc/docker/daemon.json
{
  "insecure-registries" : [
      "nexus.dev.me:8082",
      "192.168.22.98:8082"
    ]
}
```

```bash
systemctl restart docker
```

nexus
privileges > users > admin > role anonymous

settings > blob stores > create

file
name: docker

settings > security > realm - add docker bearer token

repo > create
name
http : 8082
enable anonymous # disable anonymous - was causing error in login
enable v1

docker login nexus.dev.me:8082

> admin # nexus login
> password

add ip (192.168.22.98) in insecure registries as well

docker image list
