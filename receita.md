# MINHA RECEITA DJANGO
## Criando/Ativando Ambiente Virtual
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
## Criando um Projeto
```shell
$ django-admin startproject meuprj
$ cd meuprj
```
## Criando um Aplicativo
```shell
$ python3 manage.py startapp catalog
$ cd catalog
```
## Configurações
### Registrando o app
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
### Especificando o banco de dados
Arquivo: `meuprj/settings.py`
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```
### Outras configurações
Arquivo: `meuprj/settings.py`
```python
LANGUAGE_CODE = 'pt-br'

TIME_ZONE = 'America/Fortaleza'
```
## Definindo as Rotas
### Conectando os mapeadores de URL
Arquivo: `meuprj/urls.py`
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('catalog/', include('catalog.urls')),
]
```
### Mudando a raiz do site de 127.0.0.1:8000 para 127.0.0.1:8000/catalog/
Arquivo: `meuprj/urls.py`
```python
from django.contrib import admin
from django.urls import path, include
from django.views.generic import RedirectView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('catalog/', include('catalog.urls')),
    path('', RedirectView.as_view(url='/catalog/')),
]
```
### Habilitando arquivo estáticos (opcional)
Arquivo: `meuprj/urls.py`
```python
from django.contrib import admin
from django.urls import path, include
from django.views.generic import RedirectView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('catalog/', include('catalog.urls')),
    path('', RedirectView.as_view(url='/catalog/')),
]

## Use static() to add url mapping to serve static files during development (only)
# from django.conf import settings
# from django.conf.urls.static import static

# urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

