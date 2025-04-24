## docker pass env file

docker run --env-file <env-filename> -p 8000:8000 django

https://hub.docker.com/_/mariadb

```
docker run --name mysql --network django -v /home/:/var/lib/mysql:Z -e MARIADB_ROOT_PASSWORD=pass -d mariadb:latest
```

docker run --env-file <env-filename> --network django -p 8000:8000 django

hostname in network

1. create mysql docker dontainer
2. docker network create

# Nexus

repo > select recepie > creare - docker hosted
http port: 8082

https://help.sonatype.com/en/pushing-images.html

```
docker tag <imageId or imageName> <nexus-hostname>:<repository-port>/<image>:<tag>
# docker tag d99b15185dfc nexus.dev.me:8082/django:v1.0.0
docker push <nexus-hostname>:<repository-port>/<image>:<tag>
# docker push nexus.dev.me:8082/django:v1.0.0
```

nexus will try to connect to https
in nexus - config insecure-registries

```json
# /etc/docker/daemon.json
{
  "insecure-registries" : [
      "nexus.dev.me:8082",
      "192.168.22.98:8082"
    ]
}
```

systemctl restart docker

nexus
privileges > users > admin > role anonymous

settings > blob stores > create

file
name: docker

settings > security > realm - add docker bearer token

repo > create
name
http : 8082
enable anonymous
enable v1

docker login nexus.dev.me:8082

> admin # nexus login
> Bishal@123

add ip (192.168.22.98) in insecure registries as well

docker image list
