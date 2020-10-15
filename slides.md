---
title: Automatizando releases con commitizen y github actions
theme: black
revealOptions:
  transition: "convex"
---

# Commitizen

Automatizando releases con commitizen y github actions

[![github logo](./assets/github.png)](https://github.com/commitizen-tools/commitizen)

v0.4.0

---

## About me

Santiago

origen: Argentina 🇦🇷 <!-- .element: class="fragment" data-fragment-index="0" -->

actual: Amsterdam 🇳🇱 <!-- .element: class="fragment" data-fragment-index="0" -->

🐍 python, web, js, stuff <!-- .element: class="fragment" data-fragment-index="0" -->

---

## Agenda

1. intro
2. _Release workflow_
3. semver
4. _conventional commits_
5. configurar _commitizen_
6. incrementar version automaticamente
7. actualizar archivos con la nueva version
8. generar _changelog_ 📂 automaticamente

---

## Introducción

`commitizen` es una herramienta escrita en python para recolectar los cambios
recientes y generar la version correcta de semver automaticamente.

---

Un solo comando

```bash
cz bump --changelog
```

---

## Release workflow

🧑 Merge PR

⬇️ <!-- .element: class="fragment" data-fragment-index="0" -->

> 🤖 Bump version and generate changelog <!-- .element: class="fragment" data-fragment-index="0" -->

⬇️ <!-- .element: class="fragment" data-fragment-index="1" -->

🤖 Push al branch principal <!-- .element: class="fragment" data-fragment-index="1" -->

⬇️ <!-- .element: class="fragment" data-fragment-index="2" -->

🤖 Trigger deploy/release para el nuevo tag <!-- .element: class="fragment" data-fragment-index="2" -->

---

## Sin miedo a SEMVER

Una versión semver esta cargada de información <!-- .element: class="fragment" data-fragment-index="1" -->

[semver.org](https://semver.org/)

---

## MAJOR

⬇️ <!-- .element: class="fragment" data-fragment-index="0" -->

BREAKING CHANGES ⚠️ <!-- .element: class="fragment" data-fragment-index="0" -->

---

### MINOR

⬇️ <!-- .element: class="fragment" data-fragment-index="0" -->

Cosas nuevas 🎉 <!-- .element: class="fragment" data-fragment-index="0" -->

backwards compatible -> es "confiable" instalarla

---

#### PATCH

⬇️ <!-- .element: class="fragment" data-fragment-index="0" -->

Seguridad 🔒 y bug fixes 🐛 <!-- .element: class="fragment" data-fragment-index="0" -->

backwards compatible -> es "confiable" instalarla

---

MAJOR.MINOR.PATCH

## 2.23.3

Deja que commitizen genere la version por vos <!-- .element: class="fragment" data-fragment-index="0" -->

---

## Conventional commits

Escribir los mensajes de los commits de manera **estandarizada**

Facil de leer por humanos 🧑 y maquinas 🤖 <!-- .element: class="fragment" data-fragment-index="0" -->

[conventionalcommits.org](https://www.conventionalcommits.org/)

---

### Tips para commits

- Si aplico este commit se introducirá... (If applied this commit will...)
- Imperativo
- Cortita y al pie

---

## Ejemplos

```text
feat: add adapter for different banks
fix: vulnerability in the login session
```

---

### Extra change_types

```text
build  # ignorado por commitizen
docs  # ignorado por commitizen
tests  # ignorado por commitizen
refactor  # parseado por commitizen
```

---

## Breaking change

```text
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

---

## Objetivo del live coding

Con cada nuevo cambio, generar tag y changelog; y publicar los cambios

---

## Configuracion de commitizen

En un archivo `.cz.toml` o `pyproject.toml`

```toml
[tool.commitizen]
version = "0.1.0"  # should be your current semver version
```

`cz init` <!-- .element: class="fragment" data-fragment-index="0" -->

---

## Actualizar archivos con la nueva version

```toml
[tool.commitizen]
version = "0.1.0"  # This should be your current semver version
version_files = [
    "src/__version__.py:version",
    "README.md"
]
```

---

## Generar _changelog_ 📂 automaticamente

En el `yaml` con nuestra github action agregamos `--commitizen`
Quedando:

```bash
cz bump --yes --changelog
```

---

## Incrementar la version automaticamente

Creamos un archivo `yaml` dentro de `.github/workflows/`

```yaml
on:
  push:
    branches:
      - main

jobs:
  build:
    if: "!contains(github.event.head_commit.message, 'bump')"
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.x']
    steps:
    - uses: actions/checkout@v2
      with:
        token: '${{ secrets.PERSONAL_ACCESS_TOKEN }}'
        fetch-depth: 0
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v1
      with:
        python-version: ${{ matrix.python-version }}
    - name: Install dependencies
      run: |
        python --version
        python -m pip install -U commitizen
    - name: Configure repo
      run: |
        git config --local user.email "action@github.com"
        git config --local user.name "GitHub Action"
    - name: Create bump
      run: |
        cz bump --yes --changelog
        git tag
    - name: Push changes
      uses: Woile/github-push-action@master
      with:
        github_token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
        tags: "true"
        branch: main
```

---

## Ejemplos de GH actions

```bash
.
├── bump-version.yml
├── publish-docs.yaml
├── pr-checks.yml
└── publish-package.yaml
```

---

## Usando commitizen-action

```yaml
name: Bump version

on:
  push:
    branches:
      - main

jobs:
  bump_version:
    if: "!startsWith(github.event.head_commit.message, 'bump:')"
    runs-on: ubuntu-latest
    name: "Bump version and create changelog with commitizen"
    steps:
      - name: Check out
        uses: actions/checkout@v2
        with:
          fetch-depth: 0
          token: "${{ secrets.PERSONAL_ACCESS_TOKEN }}"
      - name: Create bump and changelog
        uses: commitizen-tools/commitizen-action@master
        with:
          github_token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
          branch: main
```

[commitizen-action](https://github.com/commitizen-tools/commitizen-action)

---

Publish package + Documentation

```yaml
name: Build and Deploy
on:
  push:
    tags:
      - "*"
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout 🛎️
        uses: actions/checkout@v2.3.1
        with:
          persist-credentials: false

      - name: Install and Build
        run: |
          npm install
          npm run build

      - name: Deploy 🚀
        uses: JamesIves/github-pages-deploy-action@3.6.2
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          BRANCH: gh-pages
          FOLDER: build
          CLEAN: true
```

---

![thanks](./assets/thanks.gif)

email: santiwilly@gmail.com

twitter: [@santiwilly](twitter.com/santiwilly)

github: [woile](github.com/Woile)

[github.com/commitizen-tools/commitizen](https://github.com/commitizen-tools/commitizen)
