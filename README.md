# Django
Neste estudo, criaremos um diretório `est_django`, nele criaremos um ambiente virtuaal chamado `.django` e neste ambiente instalaremos o Django.

## Criando/Ativando um Ambiente Virtual
```shell
$ mkdir est_django
$ cd est_django
$ pytho3 -m venv .django
$ source .django/bin/activate
```
## Instalando o Django
Diretório: **`est_django`**
```shell
$ pip install django
$ python -m django --version
```
## Criando um Projeto para Conter a Aplicação
Nomearemos o projeto como mysite.
Diretório: **`est_django`**
```shell
$ django-admin startproject mysite
```
O `startproject` criará a estrutura similar a:
```shell
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

## Servidor Web para Desenvolvimento
Neste ponto já podemos rodar um servidor web de desenvolvimento. Para tanto vamos digitar o segunte comando:
```shell
$ python manage.py runserver
```
Saida:
```shell
Django version 4.2, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```
Abra um browser e digite `http://127.0.0.0:8000` para ver o resultado: Um fogrete pronto para ser lançado!
## Criando a Aplicação Polls (Enquete)
O exemplo que construiremos consiste de uma aplicação para enquetes. Para criar a aplicação, usaremos o comando do código abaixo.

Diretório: **`mysite`**
```shell
$ python manage.py startapp polls
```
O `startapp` criará uma estrutura parecida com esta:
```shell
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```
## Escrevendo a Primeira View
Arquivo: **`polls/views.py`**
```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Olá, mundo. Você está no index de enquetes.")
```
## Mapeando Uma Nova View
Para ser reconhecida, toda `View` precisa ser mapeada na aplicação. 
Para tanto criaremos o arquivo de rotas `urls.py` no diretório `polls`.

Arquivo: **`polls/urls.py`**
```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```
## Atualizando o Arquivo de Rotas da Root
O próximo passo será apontar no arquivo de rotas, também chamado de `urls.py`, na `root` com a função `include()` que faz parte do módulo `django.urls`.

Arquivo: **`polls/urls.py`**
```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls")),
    path("admin/", admin.site.urls),
]
```
Agora que incluímos uma `view` ao índice do URLconf. Vamos ver se está funcionando com o seguinte comando:

```shell
$ python manage.py runserver
```
Vamos para http://localhost:8000/polls/ no navegador e deveremos ver o texto “Olá, mundo. Você está no índice de enquetes.”, que você definiu na visualização do índice.
```
## Configurando A Base de Dados
Agora vamos abrir o arquivo `mysite/settings.py`. Trata-se de um módulo Python que representa as diversas configurações do Django, inclusive a de bando de dados. Usaremos como exemplo o banco de dados default **SQLite**.

Manteremos a chave `DATABASE` como está:

Arquivo: **`mysite/settings.py`**
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
...
```
Alteraremos as chaves `LANGUAGE_CODE` e `TIME_ZONE` para **'pt-br'** e **'America/Fortaleza'**, respectivamente. E teremos

Arquivo: **`mysite/settings.py`**
```python
LANGUAGE_CODE = 'pt-br'

TIME_ZONE = 'America/Fortaleza'
...
```

