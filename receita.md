# Receita Django
## Criando/Ativando ambiente virtual
```shell
$ mkdir e_jango
$ cd e_jango
$ python3 -m venv .e_jango
$ source .e_jango/bin/activate
```
## Instalando o Django
```shell
$ pip install django
$ django-admin --version
```
## Criando um projeto
```shell
$ django-admin startproject meuprj
$ cd meuprj
```
## Criando um aplicativo de catálogo
```shell
$ python3 manage.py startapp catalog
$ cd meuprj
```
## Registrando o app
Arquivo: `meuprj/settings.py`
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'catalog.apps.CatalogConfig',
]```
