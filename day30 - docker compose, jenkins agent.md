### how name resolution happens in docker?

eg: DATABASE_HOST=mysql

# Let's Revise

1. create django app
2. connect database
3. backup database as \*.sql and backup

```
|
|--- app
|--- backup
|--- Dockerfile
|--- docker-compose.yml
```

## dockercompose

```yaml
# docker-compose.yml
version: "3.9"
services:
  mysql:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_DATABASE: django
      MYSQL_USER: bishal
      MARIADB_PASSWORD: redhat
      MYSQL_ROOT_PASSWORD: redhat
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./backup:/docker-entrypoint-initdb.d

  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - mysql
    environment:
      DB_NAME: django
      DB_USER: django
      DB_PASSWORD: redhat
      DB_HOST: db
      DB_PORT: 3306

volumes:
  mysql_data:
networks:
  djangonew:
    driver: bridge
```

Jenkins agent

```bash
curl -sO http://192.168.22.98:8080/jnlpJars/agent.jar
java -jar agent.jar -url http://192.168.22.98:8080/ -secret 53d1d2511bbf37efe01d9f5c6383515049b2bc0a60415665599e609f8079d47f -name jenkinsagent -webSocket -workDir "/home/bishal/jenkins-agent"
```
