# MkDocs
Repositorio para la práctica MkDocs de IAW

Antonio Jesus Galvez Rodriguez

Durante esta práctica vamos a ver cómo podemos configurar un workflow en GitHub Actions para publicar el sitio web que hemos creado en GitHub Actions.

## 1. `mkdocs.yml`
Para crear la estructura de archivos del proyecto MkDocs podemos hacer uso del comando `new`.

```bash
docker run --rm -it -p 8000:8000 -u $(id -u):$(id -g) -v "$PWD":/docs squidfunk/mkdocs-material new .
```

Dentro del archivo configuraremos el nombre del sitio web que estamos creando (`site_name`), los enlaces a las páginas que van a aparecer en el menú de navegación (`nav`) y el theme que vamos a utilizar (`theme`).

```yml
site_name: IAW

nav:
    - Principal: index.md
    - Acerca de: about.md

theme:
    name: material
    palette:
        primary: blue
```
Esta será la configuración que aparecerá después cuando accedamos a la página web.

## 2. Creación de un workflow de CI/CD en GitHub Actions para pubicar un sitio web en GitHub Pages
Para crear un workflow en GitHub Actions podemos hacerlo de dos formas:

Desde la sección `Actions` -> `New workflow`.
Creando un archivo YAML dentro del directorio `.github/workflows`, en nuestro repositorio.
En nuestro caso, vamos a crear un archivo con el nombre `build-push-mkdocs.yaml`.

Dentro del archivo debemos poner el siguiente contenido.

```yml
name: build-push-mkdocs

# Eventos que desescandenan el workflow
on:
  push:
    branches: ["main"]

  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:

  # Job para crear la documentación de mkdocs
  build:
    # Indicamos que este job se ejecutará en una máquina virtual con la última versión de ubuntu
    runs-on: ubuntu-latest
    
    # Definimos los pasos de este job
    steps:
      - name: Clone repository
        uses: actions/checkout@v4

      - name: Install Python3
        uses: actions/setup-python@v4
        with:
          python-version: 3.x

      - name: Install Mkdocs
        run: |
          pip install mkdocs
          pip install mkdocs-material 

      - name: Build MkDocs
        run: |
          mkdocs build

      - name: Push the documentation in a branch
        uses: s0/git-publish-subdir-action@develop
        env:
          REPO: self
          BRANCH: gh-pages # The branch name where you want to push the assets
          FOLDER: site # The directory where your assets are generated
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # GitHub will automatically add this - you don't need to bother getting a token
          MESSAGE: "Build: ({sha}) {msg}" # The commit message
```

Antes de ejecutarlo deberemos configurar los permisos que tendrá el `GITHUB_TOKEN` cuando se ejecute el workflow en este repositorio.

Para configurar el repositorio seleccionamos: `Settings` -> `Actions` -> `General`.
Buscamos la sección Workflow permissions y seleccionamos la opción `Read and write permissions`.

![](/img/Captura%20de%20pantalla%202025-03-07%20162716.png)

Ahora al ejecutarlo este debe de ser el resultado:
![](/img/Screenshot_20250307_140630.png)

Y una cosa a tener en cuenta es que para poder publicar un sitio web en GitHub Pages, debemos poner el repositorio en público y dentro del apartado `Settings` -> `Pages` y poner el branch creado despues de ejecutar el `Workflow`, en nuestro caso `gh-pages`.
![](/img/Captura%20de%20pantalla%202025-03-07%20163013.png)

Por último, despues de hacer todo esto ya podemos acceder a la pagina web que hemos publicado desde GitHub
![](/img/Screenshot_20250307_142118.png)