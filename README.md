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
Agora que incluímos uma `view` ao índice do URLconf, vamos ver se está funcionando com o seguinte comando:

```shell
$ python manage.py runserver
```
Vamos para http://localhost:8000/polls/ no navegador e deveremos ver o texto “Olá, mundo. Você está no índice de enquetes.”, que você definiu na visualização do índice.

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
# Internationalization
# https://docs.djangoproject.com/en/4.2/topics/i18n/

LANGUAGE_CODE = 'pt-br'

TIME_ZONE = 'America/Fortaleza'
...
```
Após fazer estas alterações no arquivo `mysite/settings.py`, salve-o e rode este comando:

```shell
$ python manage.py migrate
```
O comando `migrate` analisa o arquivo de configuração `mysite/settings.py` até a chave `INSTALLED_APPS` e cria as tabelas necessárias no banco de dados escolhido.

## Definindo a Modelagem dos Dados
Agora nós vamos definir os modelos para a nossa enquete. Essencialmente o layoute do banco de dados e os metadados.

O nosso enquete possuirá dois modelos: `Question` e `Choice`. Uma `Pergunta` tem uma pergunta e uma data de publicação. Uma `Escolha` possui dois campos: o texto da escolha e uma contagem de votos. Cada `Escolha` está associada a uma `Pergunta`.

Esses conceitos são representados por classes Python. Edite o arquivo `polls/models.py` para que fique assim:

[`polls/models.py`](https://docs.djangoproject.com/en/4.2/intro/tutorial02/#id2)

```python
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```
Aqui cada modelo é representado por uma classe que por sua vez é uma subclasse de `django.db.models.Model`. Uma vez informado, o Django criará as tabelas e seus campos definidos dentra das classes.

## Ativando os Modelos
Para que o Django crie as tabelas e API's necessŕias para acessar os dados, precisamos informar no arquivo `mysite/settings.py`, chave `INSTALLED_APPS`, a classe `polls.apps.PollsConfig` da seguinte forma:

Arquivo: **`mysite/settings.py`**
```python
INSTALLED_APPS = [
    "polls.apps.PollsConfig",
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]
```
Agora que Django está pronto para incluir o app Polls, vamos rodar o seguinte comando:
```shell
$ python manage.py makemigrations polls
```
O comando acima deve exibir umamensagem similara a:
```shell
Saída:
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
```
Para além disso, ao executar este comando, um novo arquivo deverá ser criado em `polls/migrations/0001_initial.py` que pode ser visto e alterado conforme você queira.

Agora precisamos executar as migrações e gerenciamento do esquema do banco de dados. Faremos isso de forma automatica com o comando abaixo:
```shell
$ python manage.py sqlmigrate polls 0001
```
Teremos como resposta algo como:
```shell
Saída:
BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" (
    "id" bigint NOT NULL PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
    "question_text" varchar(200) NOT NULL,
    "pub_date" timestamp with time zone NOT NULL
);
--
-- Create model Choice
--
CREATE TABLE "polls_choice" (
    "id" bigint NOT NULL PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL,
    "question_id" bigint NOT NULL
);
ALTER TABLE "polls_choice"
  ADD CONSTRAINT "polls_choice_question_id_c5b4b260_fk_polls_question_id"
    FOREIGN KEY ("question_id")
    REFERENCES "polls_question" ("id")
    DEFERRABLE INITIALLY DEFERRED;
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");

COMMIT;
```
### Problemas ao Alterar um Modelo
Solução:
1. Delete all the migrations in your app and in django_migrations all the fields with django_migrations.app = your-app-name. How to do this depends on which DB you are using Example for MySQL: `delete from django_migrations where app = "your-app-name"`;

2. Create an initial migration with the same schema as the existing table, with these steps:

3. Modify your models.py to match with the current table in your database (Here, use the command `python manage.py inspectdb`. This command is like reverse migration. It generates the corresponding models.py for the current database tables schema.)

4. Delete all files in "migrations"

5. Run `python manage.py makemigrations polls`

6. If you already have an existing database then run `python manage.py migrate --fake-initial` and then follow the step below.

7. Modify your models.py to match the new schema (e.i. the schema that you need now)

8. Make new migration by running `python manage.py makemigrations polls`

9. Run `python manage.py migrate polls`

