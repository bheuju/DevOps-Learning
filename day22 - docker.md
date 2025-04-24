# Docker

`sudo apt install -y docker.io docker-compose-v2`

Add user to group

```bash
whoami
sudo usermod -aG docker <username>

# This allows the <username> user to run Docker commands without using sudo each time
sudo docker ps
docker ps

# list groups of <username>
groups <username>
```

---

### Docker Commands

```bash
docker ps
docker search hello-world   # docker hub

docker run hello-world
docker ps
docker ps -a  # show all ps (not only running)
-------------------------------------------------------------------------------------------------------
> CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
> f7bc96061e9e   hello-world   "/hello"   3 seconds ago   Exited (0) 2 seconds ago             ntc1
-------------------------------------------------------------------------------------------------------
docker run --name ntc1 hello-world
-------------------------------------------------------------------------------------------------------
> CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
> f7bc96061e9e   hello-world   "/hello"   3 seconds ago   Exited (0) 2 seconds ago             ntc1
> 01b7c15b2e74   hello-world   "/hello"   2 minutes ago   Exited (0) 2 minutes ago             cool_turing
-------------------------------------------------------------------------------------------------------

docker logs ntc1

docker run -it ubuntu bash
# stop running container
docker container stop <id/name>

# remove single container
docker container rm <id/name>

# remove all containers at once
docker ps -a
# display only first (delimited by tab)
docker ps -a | awk '{ print $1}'
> CONTAINER
> f7bc96061e9e
> 01b7c15b2e74
# remove all containers output by previous
docker container rm $(docker ps -a | awk '{ print $1}')

# docker image remove
docker image rm <name?>

```

## Docker network

```bash
# list
docker network list
# inspect network
docker network inspect <network id>
# create new network
docker network create test1

docker volume list

```

```bash
sudo ls /var/lib/docker
-------------------------------------------------------------------------------------------------------
> buildkit  containers  engine-id  image  network  overlay2  plugins  runtimes  swarm  tmp  volumes
-------------------------------------------------------------------------------------------------------
# where containers are created
sudo ls /var/lib/docker/containers

```

Enter interactive

```
docker exec -it <container name/id> bash
```

## Setup following

sonatype/nexus3 - https://hub.docker.com/r/sonatype/nexus3
jenkins - https://github.com/jenkinsci/docker
gitlab - https://medium.com/@BuildWithLal/dockerized-gitlab-how-to-easily-set-up-your-own-gitlab-server-9a925be09c59

```bash
docker pull sonatype/nexus


#############################
#      Sonatype Nexus       #
#############################
docker pull sonatype/nexus
docker volume create nexus data

docker run -d --name nexus-data sonatype/nexus echo "data-only container for Nexus"
docker run -d -p 8081:8081 --name nexus --volumes-from nexus-data sonatype/nexus

### Class ------------
docker volume create --name nexus-data
docker run -d -p 8081:8081 -p 8082:8082 --restart=always --name nexus -v nexus-data:/nexus-data sonatype/nexus3



#############################
#          Jenkins          #
#############################
docker pull jenkins/jenkins
docker run -d -v jenkins_home:/var/jenkins_home -p 8080:8080 -p 50000:50000 --restart=on-failure jenkins/jenkins

### Class ------------
docker run -p 8080:8080 -p 50000:50000 -d --name jenkins --restart=always --dns 1.1.1.1 --dns 8.8.8.8 jenkins/jenkins:lts-jdk17


#############################
#          Gitlab          #
#############################
docker pull gitlab/gitlab-ce
docker image ls | grep gitlab-ce
docker run -d -p 8443:443 -p 8079:80 -p 2222:22 \
           --restart=always \
           -v ./gitlab/config:/etc/gitlab \
           -v ./gitlab/data:/var/opt/gitlab \
      gitlab/gitlab-ce
docker exec -it 6befb2ecb255 cat /etc/gitlab/initial_root_password

### Class ------------
sudo mkdir -p /srv/gitlab
export GITLAB_HOME=/srv/gitlab
sudo docker run --detach \
  --env GITLAB_ROOT_PASSWORD="SkhB4mHWFRONshW" \
  --hostname gitlab.devops.com \
  --env GITLAB_OMNIBUS_CONFIG="external_url 'http://gitlab.devops.com'" \
  --publish 8443:443 --publish 8079:80 --publish 2222:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  --shm-size 256m \
  gitlab/gitlab-ce:latest
```

### Extend hard disk volume

```bash
lsblk
growpart --help
growpart /dev/sda 3
lsblk
df -h
vgs
lvs
lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
tail -n 4 /etc/fstab
resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
# xfs_growfs if fs
```
