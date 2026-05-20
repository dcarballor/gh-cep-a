# CEP Introducción Devops A
## 1. Workflow CI para el proyecto de frontend

Lo primero que hacemos es crear el archivo ci-front.yaml con el siguiente contenido:

```
name: CI-front

on:
  pull_request:
    branches: [ main ]
    paths: ['hangman-front/**']
  push:
    branches: [ main ]
    paths: [ 'hangman-front/**' ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v6
      - name: Setup Node.js version
        uses: actions/setup-node@v6
        with:
          node-version: 18
      - name: Build
        working-directory: ./hangman-front
        run: |
          npm ci
          npm run build --if-present

  test:
    runs-on: ubuntu-22.04
    needs: build

    steps:
      - name: Checkout
        uses: actions/checkout@v6
      - name: Setup Node.js version
        uses: actions/setup-node@v6
        with:
          node-version: 18
      - name: Unit tests
        working-directory: ./hangman-front
        run: |
          npm ci
          npm run test 

```
En el apartado on se pide que el workflow se dispare cuando haya pull-request y push en el proyecto hangman-front.

En el apartado jobs se indica qué se debe realizar en el caso de que el workflow se dispare: hacer la build y el test.

En el primer lanzamiento del workflow (para lo que hay que tocar algo del proyecto) el test falla porque estaba así preparado para que se viera el job de test.

Ahí teníamos un fallo en un fichero determinado porque una lista tenía un array de diferente tamaño al esperado.

Después de corregir el fichero, volvemos a lanzar el worklow y observamos que tanto el build como el test han dado el resultado esperado.

## 2. Workflow CD para el proyecto de frontend

Creamos un fichero cd-front.yaml en .guthub/workflows con el siguiente contenido:
```
name: Despliegue continuo front

on:
  workflow_dispatch:

jobs:
  buildAndPushImage:
    runs-on: ubuntu-latest

    steps: 
      - name: Checkout
        uses: actions/checkout@v6
      - name: Login into GitHub Container Registry
        uses: docker/login-action@v4
        with:
          username: dcarballodocker
          password: ${{ secrets.PASSWORD }}
      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v4
      - name: Build and push Docker Image
        uses: docker/build-push-action@v7
        with:
          context: ./hangman-front
          push: true
          tags: dcarballodocker/hangman-front:latest
          file: ./hangman-front/Dockerfile

```

Hacemos push y luego pull request. Al mergear, nos vamos a la pestaña Actions para poder lanzar el workflow manualmente.
Como se puede ver en mi repo, tras varios errores por los logins y usuarios y contraseñas de Docker Hub, creo que ha finalizado por fin tal como se esperaba.


