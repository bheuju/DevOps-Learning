### Gihub actions example

```yml
name: Django CI

on:
  push:
    branches: ["dev"]
  pull_request:
    branches: ["master"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Hello world check
        run: |
          echo "Hello world!"
          hostname
```

```yml
name: Django CI

on:
  push:
    branches: ["dev"]
  pull_request:
    branches: ["master"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Hello world check
        run: |
          echo "Hello world!"
          hostname
          cd /home/bishal/django-demo
          git checkout dev
          git pull origin dev
```
