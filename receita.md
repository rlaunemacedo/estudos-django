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
$ cd catalog
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
]
```
## Especificando o banco de dados
Arquivo: `meuprj/settings.py`
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```
## Outras configurações
Arquivo: `meuprj/settings.py`
```python
LANGUAGE_CODE = 'pt-br'

TIME_ZONE = 'America/Fortaleza'
```
## Conectando os mapeadores de URL
Arquivo: `meuprj/urls.py`
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('catalog/', include('catalog.urls')),
]
```



