## Django, MongoDB e PyMongo

**Por R N Launé Macêdo**

#djangomongodb #mongodbpymongo

### Apresentação

Aqui encontraremos tão somente  uns apontamentos sobre a utilização do **Django** utilizando o **MongoDB** como base de dados. Neste estudo, utilizamos uma instalação local do banco de dados MongoDB sem o uso dos acessos de segurança. De forma que, no meu caso, a string de conexão é bem simples `"mongodb://localhost:27017/"`. Será desnecessário esclarecer que uma conexão com itens de segurança, tanto `local` como `externa`, será bem mais complexa.

O meu ambiente de estudos é um *Notebook i5*, *Linux Mint 21.2*, *Cinnamon 5.8*. Além disso, excetuando o *Python 3*, *MongoDB 7.0* e *MongoDB Compass (opcional)*, instalei tudo em um ambiente virtual (venv).

Espero que estes apontamentos possam contribuir de alguma forma com os, como eu, curiosos.

Mais informações, consulte esta [fonte](https://www.digitalocean.com/community/tutorials/how-to-connect-a-django-app-to-mongodb-with-pymongo).

### Criar um ambiente Django

Criaremos um projeto (chamaremos **setup**) e um app (chamaremos **blogapp**)

````shell
$ mkdir e.djamongo
$ cd e.djamongo
$ python -m venv .e.djamongo
$ source .e.djamongo/bin/activate
$ pip install django
$ pip install pymongo
$ pip install dnspython
$ django-admin startproject setup .
$ python manage.py startapp contatos
$ mkdir contatos/templates
$ mkdir contatos/templates/contatos
````

#### Testando o ambiente

````shell
$ django-admin --version
$ python manage.py runserver
````

### Mão na Massa

#### Configurando o app

Arquivo: **setting.py**

````python
## Comentar a conexão Sqlite3
# DATABASES = {
#     'default': {
#         'ENGINE': 'django.db.backends.sqlite3',
#         'NAME': BASE_DIR / 'db.sqlite3',
#     }
# }
````

```python
## Adicionando o app
INSTALLED_APPS = [
    'conatos',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```



````python
## Configurando local
LANGUAGE_CODE = 'pt-br'

TIME_ZONE = 'America/Fortaleza'
````



#### Definindo roteamentos, views e templates

Arquivo: **urls.py**

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('',include('contatos.urls')),
    path('admin/', admin.site.urls),
]
```



Criar o arquivo `contatos/urls.py` e adicionar o código abaixo:

Arquivo: **contatos/urls.py**

```python
from django.urls import path

from . import views

urlpatterns = [path('',views.index,name='index'),]
```



Arquivo: **contatos/view.py**

```python
from django.shortcuts import render
import pymongo

client = pymongo.MongoClient("mongodb://localhost:27017/")

#Define Db Name
dbname = client['estudos']

#Define Collection
collContatos = dbname['contatos']

# Create your views here.
## Baseada em função (function-based)
from django.shortcuts import render
def index(request):
    conts = collContatos.find()
    return render(request, "contatos/lista_contatos.html", {"contatos": conts})
```



Criar o arquivo `lista_contatos.html` e adicionar o código abaixo:

Arquivo: **contatos/templates/contatos/lista_contatos.html**

```html
<!doctype html>
<html lang="pt-br">
  <head> Lista de contatos </head>
  <body>
    {% for contato in contatos %}
      <p>{{ contato }}</p>
    {% endfor %}
  </body>
</html>
```



### Testando

Rode o comando:

````shell
$ python manage.py runserver
````

A saída deve ser parecida com:

````
{'_id': ObjectId('654ce63b1ab6ff3e59a8871d'), 'nome': 'Yan', 'fone': ['1234-5678', '2234-5678', '2235-5678']}

{'_id': ObjectId('654ce63b1ab6ff3e59a8871e'), 'nome': 'Pedro', 'fone': ['9999-9999', '2244-5678']}

{'_id': ObjectId('654ce63b1ab6ff3e59a8871f'), 'nome': 'Ana', 'fone': '8765-4321'}

{'_id': ObjectId('654ce63b1ab6ff3e59a88720'), 'nome': 'Marina', 'fone': '8877-7788'}

{'_id': ObjectId('654ce63b1ab6ff3e59a88721'), 'nome': 'João', 'fone': ['9998-6667', '9898-6767']}

{'_id': ObjectId('654ce63b1ab6ff3e59a88722'), 'nome': 'Carol', 'fone': '2334-5678'}
````

