docker container top nginx

## Lets make a django container using docker

### Attempt 1

```dockerfile
FROM ubuntu:24.04
RUN apt -y update && apt -y install python3 python3-pip python3-venv
COPY ./app /srv/app
WORKDIR /srv/app

RUN python3 -m venv venv
RUN ./venv/bin/activate
RUN pip install -r requirements.txt

CMD ["python3", "manage.py", "runserver", "0.0.0.0:8010"]
```

> docker build -t django .

### Attempt 2

```dockerfile
FROM ubuntu:24.04
RUN apt -y update && apt -y install python3 python3-pip python3-venv
COPY ./app /srv/app
WORKDIR /srv/app

RUN python3 -m venv venv
ENV PATH="/srv/app/venv/bin:$PATH"      # need to add this because using ubuntu image
RUN pip install -r requirements.txt

EXPOSE 8010

CMD ["python3", "manage.py", "runserver", "0.0.0.0:8010"]
```

_Note: Everytime code changes, container must be rebuilt_

---

**_Why is new container large size ?_**

```bash
docker history <image_id>
---------------------------------------------------------------------------------------------------
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
ba3af7599c9e   8 minutes ago    /bin/sh -c #(nop)  ENV PATH=/srv/app/venv/bi…   0B
a8abb4e7ff40   8 minutes ago    /bin/sh -c python3 -m venv venv                 14.3kB
eaf0387b0c36   8 minutes ago    /bin/sh -c #(nop) WORKDIR /srv/app              0B
5eb4a26535e2   8 minutes ago    /bin/sh -c #(nop) COPY dir:088510d581bfd3084…   55.5MB
e5172a76e2dc   22 minutes ago   /bin/sh -c apt -y update && apt -y install p…   477MB               #===> Can see this is adding 477 MB
602eb6fb314b   12 days ago      /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  #===> Normal till this point
<missing>      12 days ago      /bin/sh -c #(nop) ADD file:1d7c45546e94b90e9…   78.1MB              # .
<missing>      12 days ago      /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B                  # .
<missing>      12 days ago      /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B
<missing>      12 days ago      /bin/sh -c #(nop)  ARG LAUNCHPAD_BUILD_ARCH     0B
<missing>      12 days ago      /bin/sh -c #(nop)  ARG RELEASE                  0B
---------------------------------------------------------------------------------------------------
```

## Multi stage container

Helps to decrease container size

- use apt update in a temporary container
- app in another container
- final build **image** is using app only

https://docs.docker.com/build/building/multi-stage/

```bash
### Example multi-stage
## Stage 1
FROM python:3.12-slim AS compile-image
RUN apt -y update
COPY ./app /srv/app
WORKDIR /srv/app

RUN python3 -m venv venv              # might need to change venv to venv1 (because venv already exists)
# ENV PATH="/srv/app/venv/bin:$PATH"  # dont need env file as using python image
RUN pip install -r requirements.txt

## Stage 2
FROM python:3.12-slim AS build-image
COPY --from=compile-image /srv/app /srv/app
WORKDIR /srv/app
EXPOSE 8010
CMD ["python3", "manage.py", "runserver", "0.0.0.0:8010"]
```

---

MySQL container

- also should persist data
