Today we run django app using `.env` file to connect to database

```ini
# .env
DATABASE_NAME=
DATABASE_USER=
DATABASE_PASSWORD=
DATABASE_HOST=
DATABASE_PORT=
ALLOWED_HOSTS=localhost,127.0.0.1
```

`pip install python-dotenv`

Edit: `settings.py`

```python
import os
from dotenv import load_dotenv

load_dotenv()

ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS').split(',')

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': os.getenv('DATABASE_NAME'),
        'USER': os.getenv('DATABASE_USER'),
        'PASSWORD': os.getenv('DATABASE_PASSWORD'),
        'HOST': os.getenv('DATABASE_HOST'),
        'PORT': os.getenv('DATABASE_PORT'),
    }
}
```

```dockerfile
# Stage 1
FROM python:3.12-slim AS compile-image
RUN apt -y update && apt -y install pkg-config libmariadb-dev gcc g++
COPY ./ /srv/app
WORKDIR /srv/app

RUN python3 -m venv venv1
ENV PATH="/srv/app/venv1/bin:$PATH"     # need?
RUN pip install -r requirements.txt

# Stage 2
FROM python:3.12-slim AS build-image
RUN apt update && apt install -y libmariadb3
COPY --from=compile-image /srv/app /srv/app
WORKDIR /srv/app
ENV PATH="/srv/app/venv1/bin:$PATH"     # need?

EXPOSE 8000

CMD ["python3", "manage.py", "runserver", "0.0.0.0:8000"]
```
