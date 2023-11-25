## Django com SqLite - Um Administrador de Tarefas

Por R N Launé Macêdo

Timon - MA, 25/nov/2023. 

#djangosqlite

Baseei os meus estudos nesta [TreinaWeb](https://www.youtube.com/watch?v=MsUL3Pgofl4&t=374s) e em outras fontes localmente citadas.

Meu ambiente de trabalho é Linux Mint 21.2, Cinnamon 5.8.

````shell
$ mkdir e.tarefas
$ cd e.tarefas
$ python3 -m venv .e.tarefas
$ source .e.tarefas/bin/activate
$ pip install django
$ django-admin startproject setup .
$ python manage.py tarefas
````

### Settings.py

Arquivo: `setup/settings.py`

````python
# Incluir o aplicativo na lista de apps
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'tarefas.apps.TarefasConfig',
]

# Alterar localidade
LANGUAGE_CODE = 'pt-br'

TIME_ZONE = 'America/Fortaleza'
````

### Testar o ambiente

Criar uma uma view para teste:

Arquivo: `tarefas/views.py`

````python
#from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.
def home(request):
    return HttpResponse("<h1>Olá Mundo!...</h1>")
````


Rotear a função views.home

Arquivo: `setup/urls.py`

````python
from django.contrib import admin
from django.urls import path

from tarefas.views import home

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', home),
]
````



No terminal, executar:

````shell
$ python manager.py runserver
````



Deverá ter como saída:

````shell
November 23, 2023 - 08:03:54
Django version 4.2.7, using settings 'setup.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
````



Basta abrir um `browser`, colar a `URL`  http://127.0.0.1:8000/ e a saída deverá ser: Olá Mundo!...

Funcionou!



### As templates

Agora vamos criar, coforme manda a *documentação*, detro do diretório `tarefas` a estrutura `templates/tarefas`:

Diretório: `tarefas`

````shell
$ mkdir templates
$ cd templates
$ mkdir tarefas
````

É dentro deste diretório que iremos criar os nossos templates (arquivos  `.html`), excetuando, caso haja, um arquivo de base comum, tipo `base.html`.

Vamos criar o primeiro template que chamaremos de `inicio.html` com o seguinte conteúdo:

Arquivo: `templates/tarefas/inicio.html`

````html
<h2>Página de Início</h2>
<p>{{ dados }}</p>
````

Em particular, esta template recebe um conteúdo na variável de template `{{dados}}` a ser enviado pela view por meio de um dicionário `{"dados":  <valor>}`, como veremos a seguir, então alterar a função `views.home`:

Arquivo: `tarefas/views.py`

````python
from django.shortcuts import render

# Create your views here.
def home(request):
    return render(request, 
                  "tarefas/inicio.html", 
                  {"dados": "Estamos testando o padrão Url-View-Tlemplate"}
                 )
````



Após atualizar a página no browser... Funcionou!

Devemos observar que, neste exemplo, temos:

1. Uma rota apontando para uma view;
2. Uma view que determina o que tem que ser feito quando executada, neste caso renderizar uma template ; e
3. Uma template que exibe no browser os dados possivelmente gerados na view.

### Modelagem de dados

Vamos cria a nossa estrutura de dados para o nosso aplicativo de **tarefas**, para isso vamos precisar editar o arquivo de modelos.

Arquivo: `tarefas/models.py`

````python
from django.db import models

# Create your models here.
class Tarefa(models.Model):
    titulo = models.CharField(max_length=100, null=False, blank=False)
    dt_criacao = models.DateTimeField(auto_now_add=True,verbose_name="Data criação")
    dt_previsao = models.DateField(null=False, blank=False, verbose_name="Data prevista")
    hr_previsao = models.TimeField(null=False, blank=False, verbose_name="Hora prevista")
    dt_execucao = models.DateField(null=True, blank=True, verbose_name="Data execução")
    hr_execucao = models.TimeField(null=True, blank=True, verbose_name="Hora execução")
````

Se rodamor o comando `migrate` (linha decomando: `python manage.py migrate`) todas as tabelas da estrutura do **Django** serão criadas, menos a que acabamos de definir (**Tarefa**) em `models.py`. Então antes precisamos criar um **ponto de migração** com o comando `makemigraions` da seguinte forma:

````shell
$ python manage.py makemigrations
````

Este comando deverá ter uma saída parecida com:

````shell
Migrations for 'tarefas':
  tarefas/migrations/0001_initial.py
    - Create model Tarefa
````



Agora vamos aplicar a migração, uma vez que a nossa tabela já foi incluida no ponto de migração:

````shell
$ python manage.py migrate
````



A saída deverá se parecer com:

````shell
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions, tarefas
Running migrations:
  Applying tarefas.0001_initial... OK
````



### Políticas de segurança

Para ativar alguns procedimentos de segurança, vamos instalar uma biblioteca chamada `python-decouple`:

````shell
$ pip install python-decouple
````

Na pasta onde está o arquivos `manage.py`, crie um novo arquivo chamado `.env` e insira as linhas:

Arquivo: `.env`

````env
SECRET_KEY=django-insecure-ei8(y(9_t@05j5_m2n*_-#61(5^9#6%flau39sj6+)l%)+f=i#
DEBUG=True
ALLOWED_HOSTS=*
````

>  Nota: 
>
>  Copie todo o conteúdo da linha em que aparece a chave `SECRET_KEY` e cole no arquivo `.env`. Retire os espaços antes e depois do sinal de igualdade (=) e também as aspas simples:
>
>  Exemplo: 
>
>  Mudar de: SECRET_KEY = 'django-insecure-ei8(y(9_t@05j5_m2n*_-#61(5^9#6%flau39sj6+)l%)+f=i#'
>
>  Para: SECRET_KEY=django-insecure-ei8(y(9_t@05j5_m2n*_-#61(5^9#6%flau39sj6+)l%)+f=i#



No aquivo `settings.py`, devemos importar de `decouple`, `Config` e `Csv`. Para as chaves `SECRET_KEY`,  `DEBUG` e `ALLOWED_HOSTS` deveremos ter:

Arquivo `settings.py`

````python
from decouple import config
...
SECRET_KEY = config("SECRET_KEY"))
DEBUG = config("DEBUG", cast=bool, default=False)
ALLOWED_HOSTS = config("ALLOWED_HOSTS", cast=Csv())
...
````

> Nota:
>
> Como `config` importa  somente string, em `DEBUG` tivemos que remoldar (`cast`) para booleano (bool), além disso definimos que, caso não esteja definido na `.env`, assuma por *default* falso (False).
>
> ALLOWED_HOSTS recebe uma lista ([]) assim, config deverá separar por vírgula os parâmetros de `host`.

### O Admin do Django

Caso queiramos usar o app administra do Django para manipular os dados do modelo **Tarefa** (tabela: tarefas.tarefa no banco), precisaremos, primeiro registrar um superusuário (comando: python manage.py createsuperuser) e depois registrar nosso modelo no arquivo admin.py.

Criando o superusuário:

````shell
$ python manage.py createsuperuser
Usuário (leave blank to use '<username>'): 
Endereço de email: <digitar algo do tipo: nada@nada.com>
Password: <digite uma senha>
Password (again): <confirme a senha>
Esta senha é muito curta. Ela precisa conter pelo menos 8 caracteres.
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.
````

Resgistrando o modelo.

Arquivo: `admin.py`

````python
from django.contrib import admin

from .models import Tarefa

# Register your models here.
admin.site.register(Tarefa)
````

Rode o servidor e visite o endereço: http://127.0.0.1:8000/admin/. Você será solicitado a se *logar*. Use o usuário e senha cadastrados acima e navegue à vontade!

Use a opção "+ Adicionar" na opção Tarefaa para adicionar novas tarefas...

### Avançando com o administrador de tarefas

Arquivo: `setup/urls.py`

````python
from django.contrib import admin
from django.urls import path

from tarefas.views import lita_tarefas

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', lita_tarefas),
]
````



Vamos criar a função `lista_tarefas`.

Arquivo: `tarefas/views.py`

````python
from django.shortcuts import render

from .models import Tarefa

# Create your views here.
def lita_tarefas(request):
    dados = Tarefa.objects.all()
    return render(request, "tarefas/lista_tarefas.html", {"lista": dados})
````

A função `lista_tarefas` gera, a partir dos dados importados do modelo Tarefa, uma consulta ao banco e trás todos os seus registros em `dados = Tarefa.objects.all()` e este resultado é enviada ao template `tarefas/lista_tarefas.html` como `"lista"`. Quando renderizada a template exibirá uma lista de registros.

Vamos cria à template:

Arquivo: `templates/tarefas/lista_tarefas.html`

````html
<h2>Lista de Tarefas</h2>
<ul>
{% for tarefa in lista %}
    <li> {{ tarefa.titulo }} </li>
{% endfor %}
</ul>
````

Funcionou!

### Utilizando o [Bootstrap](https://getbootstrap.com/docs/5.3/getting-started/introduction/)

Vamos utilizar o exemplo o item 2. (**Include Bootstrap’s CSS and JS**) desta página, cujo conteúdo é:

````html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Bootstrap demo</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
  </head>
  <body>
    <h1>Hello, world!</h1>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
  </body>
</html>
````



Faremos alguma alterações neste *script* conforme nosso gosto. Incluiremos uma *tag* `nav`no corpo do documento, além de outras alteração, como vemos abaixo:

Arquivo: `template/tarefas/lista_tarefas.html`

````html
<html lang="pt-br"> <!-- ALTERAMOS AQUI -->
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Administração de Tarefas</title>   <!-- ALTERAMOS AQUI -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
  </head>
  <body>
    <!--nav class="navbar bg-body-tertiary"-->
    <nav class="navbar bg-primary mb-4" data-bs-theme="dark">
        <div class="container-fluid">
          <a class="navbar-brand">
            Administração de Tarefas
          </a>
        </div>
      </nav>
      <main class="container">
        <h3 class="dark">Lista de tarefas</h3>
        {% if not lista %}
        <p>Não há tarefas cadastradas! 
            <a href="#" class="btn btn-success btn-sm">Cadastrar uma tarefa</a> </p>
        {% else %}

        <table class="table">
            <thead>
              <tr>
                <th scope="col">#</th>
                <th scope="col">Título</th>
                <th scope="col">Dt_Criação</th>
                <th scope="col">Dt_Prevista</th>
                <th scope="col">Hr_Prev</th>
                <th scope="col">Dt_Conclusão</th>
                <th scope="col">Hr_Concl</th>
                <th scope="col">Ação</th>
              </tr>
            </thead>
            <tbody>
            {% for tar in lista %}
              <tr>
                <th scope="row">{{tar.id}}</th>
                <td>{{tar.titulo}}</td>
                <td>{{tar.dt_criacao}}</td>
                <td>{{tar.dt_previsao}}</td>
                <td>{{tar.hr_previsao}}</td>
                <td>{{tar.dt_execucao|default:"-"}}</td>
                <td>{{tar.hr_execucao|default:"-"}}</td>
                <td><a href="#" class="btn btn-danger btn-sm">Excluir</a></td>
              </tr>
            {% endfor %}
            </tbody>
          </table>
          {% endif %}
      </main>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
  </body>
</html>
````

Funcionou e ficou bacana!

### Mudança de abordagem

Até o presente momento utilizamos abordagem de views baseadas em funções (FBV) iremos avançar para um modelo de views mais moderno baseadas em classe (CBV).

O Django disponibiliza no módulo `django.views.generic` um conjunto de views destinadas ao CRUD, dentre elas, ListView, UpdteView, CreateView e DeleteView. Daqui para frente iremos estudar seus usos.

#### ListView

Mudanças nas views e nos rotementos.

Arquivo: `tarefas/views.py`

```python
from django.views.generic import ListView

from .models import Tarefa

class lista_tarefas(ListView):
    model = Tarefa
    context_object_name = "lista"
    template_name = "tarefas/lista_tarefas.html"
```



Arquivo: `setup/urls.py`

```python
from django.contrib import admin
from django.urls import path

from tarefas.views import lista_tarefas

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', lista_tarefas.as_view()),
]
```

> Nota:
>
> Mudança sutil em `lista_tarefas` para **`lista_tarefas.as_view()`**.



Opcionalmente, é possível reescrever as classes, roteamentos e templates nas formas **implícitas** das seguintes formas:

Arquivo: `tarefas/views.py`

```python
from django.views.generic import ListView

from .models import Tarefa

class TarefaListView(ListView):
    model = Tarefa
```

O que está aqui implícito:   `template_name = "tarefas/tarefa_list.html"`;    `context_object_name = "tarefa_list"`.

Arquivo: `setup/urls.py`

```python
from django.contrib import admin
from django.urls import path

from tarefas.views import TarefaListView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', TarefaListView.as_view()),
]
```

Consequentemente a template deverá se chamar de `tarefa_list.html` e a variável de template deverá ser referida como `{{ tarefa_list }}`.

Particularmente, eu prefiro a primeira opção onde temos a liberdade de nomear os objetos a gosto.



#### CreateView

Vamos preparar a rotina de inserção de registro.

Arquivo: `tarefas/views.py`

```python
from django.views.generic import ListView, CreateView
from django.urls import reverse_lazy

from .models import Tarefa

class lista_tarefas(ListView):
    model = Tarefa
    context_object_name = "lista"
    template_name = "tarefas/lista_tarefas.html"

class nova_tarefa(CreateView):
    model = Tarefa
    fields = ["titulo", "dt_previsao", "hr_previsao"]
    template_name = "tarefas/nova_tarefa.html"
    success_url = reverse_lazy("lista_tarefas")
```



Arquivo: `setup/urls.py`

```python
from django.contrib import admin
from django.urls import path

from tarefas.views import lista_tarefas, nova_tarefa, exclui_tarefa

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', lista_tarefas.as_view(), name="lista_tarefas"),
    path('cadastro', nova_tarefa.as_view(), name="nova_tarefa"),
]
```



Antes de escrevermos a template de adição de nova tarefa, vamos otimizar o uso das templates e não ser repetitivos na escrita de `html`.

##### Otimização do uso das templates

O template base ficará no diretório `templates`  e conterá três blocos específicos: título da pagina; ação da página; e seu conteúdo.

Arquivo: `templates/base.html`

````html
<html lang="pt-br">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>
    
        {% block page_titulo %}{% endblock page_titulo %}
    
      </title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
  </head>
  <body>
    <!--nav class="navbar bg-body-tertiary"-->
    <nav class="navbar bg-primary mb-4" data-bs-theme="dark">
        <div class="container-fluid">
          <a class="navbar-brand">
            Administração de Tarefas
          </a>
        </div>
    </nav>
    <main class="container">
        <h3 class="dark">
        
            {% block page_acao %}{% endblock page_acao %}
        
        </h3>
        
        {% block conteudo %}{% endblock conteudo %}
      
    </main>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
  </body>
</html>
````



Dessa forma, a nova template que lista as tarefas terá o seguinte conteúdo:

Arquivo: `template/tarefas/lista_tarefas.html`

````html
{% extends "base.html" %}
{% block page_titulo %}Lista de Tarefa{% endblock page_titulo %}
{% block page_acao %}Lista de Tarefa{% endblock page_acao %}

{% block conteudo %}

        {% if not lista %}
        <p>Não há tarefas cadastradas! 
            <a href="#" class="btn btn-success btn-sm">Cadastrar uma tarefa</a> </p>
        {% else %}

        <table class="table">
            <thead>
              <tr>
                <th scope="col">#</th>
                <th scope="col">Título</th>
                <th scope="col">Agendamento</th>
                <th scope="col">Dt_Prevista</th>
                <th scope="col">Hr_Prev</th>
                <th scope="col">Dt_Conclusão</th>
                <th scope="col">Hr_Concl</th>
                <th scope="col">Ação</th>
              </tr>
            </thead>
            <tbody>
            {% for tar in lista %}
              <tr>
                <th scope="row">{{tar.id}}</th>
                <td><a href="{% url 'edita_tarefa' pk=tar.id %}">{{tar.titulo}}</a></td>
                <td>{{tar.dt_criacao|date:'d/M/Y h:i'}}</td>
                <td>{{tar.dt_previsao|date:'d/M/Y'}}</td>
                <td>{{tar.hr_previsao}}</td>
                <td>{{tar.dt_execucao|date:'d/M/Y'|default:"-"}}</td>
                <td>{{tar.hr_execucao|default:"-"}}</td>
                <td><a href="{% url 'exclui_tarefa' pk=tar.id %}" 
                       class="bt btn-danger btn-sm">Excluir</a></td>
              </tr>
            {% endfor %}
            </tbody>
        </table>
        {% endif %}
        <a href="{% url 'nova_tarefa' %}" class="btn btn-success btn-sm">Nova tarefa</a>
{% endblock conteudo %}
````



Observe as mudanças e tente entender sua função. A template que adiciona novas tarefas terá o seguinte conteúdo:

Arquivo: `template/tarefas/nova_tarefa.html`

````html
{% extends "base.html" %}
{% block page_titulo %}Adicionar Tarefa{% endblock page_titulo %}
{% block page_acao %}Nova Tarefa{% endblock page_acao %}

{% block conteudo %}

<form method="POST">
    {% csrf_token %}
    {{ form }}
    <button type="commit" class="btn btn-primary btn-sm">Salvar</button>     
</form>

{% endblock conteudo %}
````

Está funcionando perfeitamente, mas o `form` de entrada de dados precisa de uns aperfeiçoamento e preferimos construí-lo *na unha* ([fonte da pesquisa](https://getdatepicker.com/4/)). Tivemos que fazer alguns ajustes na template base. Ficaram assim:

Arquivo: `templates/base.html`

````html
<html lang="pt-br">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    
    <title>
    
        {% block page_titulo %}{% endblock page_titulo %}
    
    </title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">

    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
    <script type="text/javascript" src="//maxcdn.bootstrapcdn.com/bootstrap/3.3.1/js/bootstrap.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/moment.js/2.9.0/moment-with-locales.js"></script>
    <script src="//cdn.rawgit.com/Eonasdan/bootstrap-datetimepicker/e8bddc60e73c1ec2475f827be36e1957af72e2ea/src/js/bootstrap-datetimepicker.js"></script>
  </head>
  <body>
    <!--nav class="navbar bg-body-tertiary"-->
    <nav class="navbar bg-primary mb-4" data-bs-theme="dark">
        <div class="container-fluid">
          <a class="navbar-brand">
            Administração de Tarefas
          </a>
        </div>
      </nav>
      <main class="container">
        <h3 class="dark">
            
            {% block page_acao %}{% endblock page_acao %}
        
        </h3>

        {% block conteudo %}{% endblock conteudo %}
        
      </main>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
  </body>
</html>
````



Arquivo: `templates/tarefas/nova_tarefa.html`

````html
{% extends "base.html" %}
{% block page_titulo %}Adicionar Tarefa{% endblock page_titulo %}
{% block page_acao %}Nova Tarefa{% endblock page_acao %}

{% block conteudo %}
<form method="POST">
    {% csrf_token %}
    <p></p>

    <div class="container">
        <div class="row">
           <div class='col-sm-8'>
              <div class="form-group">
                <label>Título:</label>
                <div class='input-group'>
                    <input type="text" class="form-control" name="titulo" 
                           maxlength="100" required="" id="id_titulo">
                 </div>
              </div>
           </div>
        </div>
     </div>    
    
     <div class="container">
        <div class="row">
           <div class='col-sm-3'>
              <div class="form-group">
                <label>Data prevista:</label>
                <div class='input-group date' id='datetimepicker1'>
                    <input type="text" class="form-control" 
                           name="dt_previsao" required="" id="id_dt_previsao"/>
                    <span class="input-group-addon">
                    <span class="glyphicon glyphicon-calendar"></span>
                    </span>
                 </div>
              </div>
           </div>
           <script type="text/javascript">
              $(function () {
                  $('#datetimepicker1').datetimepicker({
                      format: 'DD/MM/YYYY', 
                  });
              });
           </script>
        </div>
     </div>    
    
    <div class="container">
        <div class="row">
           <div class='col-sm-3'>
              <div class="form-group">
                <label>Hora prevista:</label>
                <div class='input-group date' id='datetimepicker3'>
                    <input type="text" 
                           class="form-control" 
                           name="hr_previsao" required="" id="id_hr_previsao"/>
                    <span class="input-group-addon">
                    <span class="glyphicon glyphicon-time"></span>
                    </span>
                 </div>
              </div>
           </div>
           <script type="text/javascript">
              $(function () {
                  $('#datetimepicker3').datetimepicker({
                      format: 'LT', locale: 'pt-br'
                  });
              });
           </script>
        </div>
     </div>

     <div class="container">
        <div class="row">
            <div class='col-sm-1'>
                <div class="form-group">
                  <button type="commit" 
                          class="btn btn-primary btn-sm">Salvar</button>     
              </div>
             </div>
             <div class='col-sm-1'>
                <div class="form-group">
                  <a href="{% url 'lista_tarefas' %}" 
                     class="btn btn-primary btn-sm">Cancelar</a>     
              </div>
             </div>
            </div>
     </div>    
</form>
{% endblock conteudo %}

````



Continuando, agora com nova *cara*, vamos fazer a rotina de atualização de uma tarefa.

#### UpdateView

Arquivo: `tarefas/views.py`

````python
from django.views.generic import ListView, CreateView, UpdateView
from django.urls import reverse_lazy

from .models import Tarefa

class lista_tarefas(ListView):
    model = Tarefa
    context_object_name = "lista"
    template_name = "tarefas/lista_tarefas.html"

class nova_tarefa(CreateView):
    model = Tarefa
    fields = ["titulo", "dt_previsao", "hr_previsao"]
    template_name = "tarefas/nova_tarefa.html"
    success_url = reverse_lazy("lista_tarefas")

class edita_tarefa(UpdateView):
    model = Tarefa
    fields = ["titulo", "dt_previsao", "hr_previsao", "dt_execucao", "hr_execucao"]
    template_name = "tarefas/edita_tarefa.html"
    success_url = reverse_lazy("lista_tarefas")
````



Arquivo: `setup/urls.py`

````python
from django.contrib import admin
from django.urls import path

from tarefas.views import lista_tarefas, nova_tarefa, edita_tarefa, exclui_tarefa

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', lista_tarefas.as_view(), name="lista_tarefas"),
    path('cadastro', nova_tarefa.as_view(), name="nova_tarefa"),
    path('atualiza/<int:pk>', edita_tarefa.as_view(), name="edita_tarefa"),
]
````



Arquivo: `teplates/tarefas/edita_tarefa.html`

````html
{% extends "base.html" %}
{% block page_titulo %}Editar Tarefa{% endblock page_titulo %}
{% block page_acao %}Edição/atualização de Tarefa{% endblock page_acao %}

{% block conteudo %}
<form method="POST">
    {% csrf_token %}
    <p></p>
    <div class="container">
        <div class="row">
           <div class='col-sm-8'>
              <div class="form-group">
                <label>Título:</label>
                <div class='input-group'>
                    <input type="text" class="form-control" value="{{tarefa.titulo}}"
                    name="titulo" maxlength="100" required="" id="id_titulo">
                 </div>
              </div>
           </div>
        </div>
     </div>    
    
     <div class="container">
        <div class="row">
           <div class='col-sm-3'>
              <div class="form-group">
                <label>Data prevista:</label>
                {{tarefa.dt_previsao|date:'d/m/Y'}}
                <div class='input-group date' id='datetimepicker1'>
                    <input type="text"  class="form-control" 
                           value="{{tarefa.dt_previsao|date:'d/m/Y'}}"
                           name="dt_previsao" required="" id="id_dt_previsao"/>
                    <span class="input-group-addon">
                    <span class="glyphicon glyphicon-calendar"></span>
                    </span>
                 </div>
              </div>
           </div>
           <script type="text/javascript">
              $(function () {
                  $('#datetimepicker1').datetimepicker({
                      format: 'DD/MM/YYYY', 
                  });
              });
           </script>
        </div>
     </div>    
    
    <div class="container">
        <div class="row">
           <div class='col-sm-3'>
              <div class="form-group">
                <label>Hora prevista:</label>
                <div class='input-group date' id='datetimepicker2'>
                    <input type="text" class="form-control" value="{{tarefa.hr_previsao}}"
                           name="hr_previsao" required="" id="id_hr_previsao"/>
                    <span class="input-group-addon">
                    <span class="glyphicon glyphicon-time"></span>
                    </span>
                 </div>
              </div>
           </div>
           <script type="text/javascript">
              $(function () {
                  $('#datetimepicker2').datetimepicker({
                      format: 'LT', locale: 'pt-br'
                  });
              });
           </script>
        </div>
     </div>

     <div class="container">
        <div class="row">
           <div class='col-sm-3'>
              <div class="form-group">
                <label>Data execução:</label>
                {{tarefa.dt_previsao|date:'d/m/Y'}}
                <div class='input-group date' id='datetimepicker3'>
                    <input type="text"  class="form-control" 
                           value="{{tarefa.dt_execucao|date:'d/m/Y'}}"
                           name="dt_execucao" required="" id="id_dt_execucao"/>
                    <span class="input-group-addon">
                    <span class="glyphicon glyphicon-calendar"></span>
                    </span>
                 </div>
              </div>
           </div>
           <script type="text/javascript">
              $(function () {
                  $('#datetimepicker3').datetimepicker({
                      format: 'DD/MM/YYYY', 
                  });
              });
           </script>
        </div>
     </div>    
    
    <div class="container">
        <div class="row">
           <div class='col-sm-3'>
              <div class="form-group">
                <label>Hora execução:</label>
                <div class='input-group date' id='datetimepicker4'>
                    <input type="text" class="form-control" value="{{tarefa.hr_execucao}}"
                           name="hr_execucao" required="" id="id_hr_execucao"/>
                    <span class="input-group-addon">
                    <span class="glyphicon glyphicon-time"></span>
                    </span>
                 </div>
              </div>
           </div>
           <script type="text/javascript">
              $(function () {
                  $('#datetimepicker4').datetimepicker({
                      format: 'LT', locale: 'pt-br'
                  });
              });
           </script>
        </div>
     </div>

     <div class="container">
        <div class="row">
            <div class='col-sm-1'>
                <div class="form-group">
                  <button type="commit" class="btn btn-primary btn-sm">Salvar</button>     
              </div>
             </div>
             <div class='col-sm-1'>
                <div class="form-group">
                  <a href="{% url 'lista_tarefas' %}" 
                     class="btn btn-primary btn-sm">Cancelar</a>     
              </div>
             </div>
            </div>
     </div>    
</form>
{% endblock conteudo %}
````



Por último a exclusão de uma tarefa.

#### DeleteView

A última rotina, a de exclusão de uma tarefa  procede-se da seguinte forma:

Arquivo: `tarefas/views.py`

````python
from django.views.generic import ListView, CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy

from .models import Tarefa

class lista_tarefas(ListView):
    model = Tarefa
    context_object_name = "lista"
    template_name = "tarefas/lista_tarefas.html"

class nova_tarefa(CreateView):
    model = Tarefa
    fields = ["titulo", "dt_previsao", "hr_previsao"]
    template_name = "tarefas/nova_tarefa.html"
    success_url = reverse_lazy("lista_tarefas")

class edita_tarefa(UpdateView):
    model = Tarefa
    fields = ["titulo", "dt_previsao", "hr_previsao", "dt_execucao", "hr_execucao"]
    template_name = "tarefas/edita_tarefa.html"
    success_url = reverse_lazy("lista_tarefas")

class exclui_tarefa(DeleteView):
    model = Tarefa
    template_name = "tarefas/exclui_tarefa.html"
    success_url = reverse_lazy("lista_tarefas")

````



O roteamento para a classe `edita_tarefa`:

Arquivo: `setup/urls.py`

````python
from django.contrib import admin
from django.urls import path

from tarefas.views import lista_tarefas, nova_tarefa, edita_tarefa, exclui_tarefa

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', lista_tarefas.as_view(), name="lista_tarefas"),
    path('cadastro', nova_tarefa.as_view(), name="nova_tarefa"),
    path('atualiza/<int:pk>', edita_tarefa.as_view(), name="edita_tarefa"),
    path("excluir/<int:pk>", exclui_tarefa.as_view(), name="exclui_tarefa"),
]
````

Na template que lista as tarefas temos, para cada tarefa, um link do tipo:

Arquivo: `templates/tarefas/lista_tarefas.html (fragmento)`

````html
{% for tar in lista %}
    <tr>
      <th scope="row">{{tar.id}}</th>
      <td><a href="{% url 'edita_tarefa' pk=tar.id %}">{{tar.titulo}}</a></td>
      <td>{{tar.dt_criacao|date:'d/M/Y h:i'}}</td>
      <td>{{tar.dt_previsao|date:'d/M/Y'}}</td>
      <td>{{tar.hr_previsao}}</td>
      <td>{{tar.dt_execucao|date:'d/M/Y'|default:"-"}}</td>
      <td>{{tar.hr_execucao|default:"-"}}</td>
      <!-- Destaque para esta coluna em que temos o link -->  
      <td><a href="{% url 'exclui_tarefa' pk=tar.id %}" 
             class="bt btn-danger btn-sm">Excluir</a></td>
        
    </tr>
{% endfor %}
````

Ao clicar neste link temos um redirecionamento para o roteamento `'exclui_tarefa'` e para ele é passado o parâmetro `pk=tar.id` que é exatamento a *primary key* do registro a ser deletado. Lembrando que o roteamento `path("excluir/<int:pk>", exclui_tarefa.as_view(), name="exclui_tarefa")`, no arquivo `urls.py` está preparado para receber este parâmetro e enviar para a classe view  `exclui_tarefa` que, por sua vez chama a template de conformação:

Arquivo: `templates/tarefas/exclui_tarefa.html`

````html
{% extends "base.html" %}
{% block page_titulo %}Exclusão de Tarefa{% endblock page_titulo %}
{% block page_acao %}Excluir Tarefa{% endblock page_acao %}

{% block conteudo %}
<h3>Você está prestes a tarefa: <strong> {{ tarefa.titulo }} </strong><h3>
<form method="POST">
  {% csrf_token %}
  <div class="container">
    <div class="row">
       <div class='col-sm-8'>
          <div class="form-group">
            <label>Confirma a exclusão?</label>
            <button type="submit" class="btn btn-danger btn-sm">Excluir</button>
            <a href="{% url 'lista_tarefas' %}" 
               type="cancel" 
               class="btn btn-warning btn-sm">Cancelar</a>         
          </div>
       </div>
    </div>
 </div>    
</form>
{% endblock conteudo %}
````



E assim, o nosso **CRUD** está concluído.

Por questões de aprofundamento, e, quem sabe, *por questões de gosto* também, para o processo de exclusão de registro, adotamos pela exibição de um *form modal* que é exibido ao clicarmos no botão de exclusão de uma tarefa para confirmar ou não a exclusão.

Geralmente, coloca-se um form modal na template básica e, sendo assim, poderá ser utilizado a qualquer momento no aplicativo. Porém, dada a especificidade do form, decidimos colocar, tanto o form modal quanto o *script-JS*, na template que lista as tarefas.

Arquivo: `templates/tarefas/lista_tarefas.html`

````html
{% extends "base.html" %}
{% block page_titulo %}Lista de Tarefa{% endblock page_titulo %}
{% block page_acao %}Lista de Tarefa{% endblock page_acao %}

{% block conteudo %}
<!- INICIO DO FORM MODAL -->
    <div id="confirmationModal" class="modal">
      <div class="modal-content">
        <h3>Você está prestes a excluir a tarefa: 
          <strong><span id="mdTitulo"></span> </strong><h3>
          <div class="container">
            <div class="row">
              <div class='col-sm-8'>
                  <div class="form-group">
                    <form action="" id="mdForm" method="POST">
                      <label>Confirma a exclusão?</label>
                      {% csrf_token %}
                      <input type="submit" 
                            class="btn btn-danger btn-sm" 
                            id="confirmDelete" 
                            value=' Excluir '>

                      <a href="{% url 'lista_tarefas' %}" 
                        type="cancel" 
                        class="btn btn-warning btn-sm" 
                        id="cancelDelete">Cancelar</a>
                    </form>
                    </div>
              </div>
            </div>
        </div>    
      </div>    
    </div>
<!- FINAL DO FORM MODAL -->

    {% if not lista %}
    <p>Não há tarefas cadastradas! 
        <a href="#" class="btn btn-success btn-sm">Cadastrar uma tarefa</a> 
    </p>
    {% else %}

    <table class="table">
        <thead>
          <tr>
            <th scope="col">#</th>
            <th scope="col">Título</th>
            <th scope="col">Agendamento</th>
            <th scope="col">Dt_Prevista</th>
            <th scope="col">Hr_Prev</th>
            <th scope="col">Dt_Conclusão</th>
            <th scope="col">Hr_Concl</th>
            <th scope="col">Ação</th>
          </tr>
        </thead>
        <tbody>
          {% for tar in lista %}
            <tr>
              <th scope="row">{{tar.id}}</th>
              <td><a href="{% url 'edita_tarefa' pk=tar.id %}">{{tar.titulo}}</a></td>
              <td>{{tar.dt_criacao|date:'d/M/Y h:i'}}</td>
              <td>{{tar.dt_previsao|date:'d/M/Y'}}</td>
              <td>{{tar.hr_previsao}}</td>
              <td>{{tar.dt_execucao|date:'d/M/Y'|default:"-"}}</td>
              <td>{{tar.hr_execucao|default:"-"}}</td>
              <!-- COMENTAMOS ESTA LINHA DA ABORDAGEM ANTERIOR
              <td><a href="{% url 'exclui_tarefa' pk=tar.id %}" 
			         class="bt btn-danger btn-sm">Excluir</a></td>
			  -->
              <!-- ESTE LINK CHAMA EXIBE O FORM MODAL PASSANDO OS PARÂMETROS
				   id E titulo DA tarefa PARA A FUNÇÃO JS deleteItem() -->
              <td><a href="#" 
                     onclick="deleteItem({{tar.id}}, '{{tar.titulo}}')" 
                     class="btn btn-danger btn-sm">Excluir</a>
              </td>
            </tr>
          {% endfor %}
        </tbody>
    </table>
    {% endif %}
    <a href="{% url 'nova_tarefa' %}" class="btn btn-success btn-sm">Nova tarefa</a>

    <!-- INICIO DA SCRIPT JS -->
    <script>
      let modal = document.getElementById('confirmationModal');
      let confirmButton = document.getElementById('confirmDelete');
      let cancelButton = document.getElementById('cancelDelete');
      let spanTitulo = document.getElementById('mdTitulo');
      let formModal = document.getElementById('mdForm');

      function deleteItem(itemId, tarTitulo) {
        // Primeiro, exibir o modal de confirmação
        // formModal.action = `\{% url 'tarefa_delete' pk=${itemId} %\}`
        modal.style.display = "block";
        
        // Quando o usuário clica no botão Deletar
        confirmButton.onclick = function() {
          formModal.action = `deletar/${itemId}/`
          spanTitulo.textContent = tarTitulo;
            // Fechar o modal
          modal.style.display = "none";
        }
        
        // Quando o usuário clica no botão Cancelar
        cancelButton.onclick = function() {
          // Fechar o modal
          modal.style.display = "none";
        }
      }
    </script>
    <!-- FINAL DA SCRIPT JS -->

{% endblock conteudo %}
````



Arquivo: `tarefas/views.py`

````python
from django.views.generic import ListView, CreateView, UpdateView
from django.urls import reverse_lazy
from django.shortcuts import redirect


from .models import Tarefa

class lista_tarefas(ListView):
    model = Tarefa
    context_object_name = "lista"
    template_name = "tarefas/lista_tarefas.html"

class nova_tarefa(CreateView):
    model = Tarefa
    fields = ["titulo", "dt_previsao", "hr_previsao"]
    template_name = "tarefas/nova_tarefa.html"
    success_url = reverse_lazy("lista_tarefas")

class edita_tarefa(UpdateView):
    model = Tarefa
    fields = ["titulo", "dt_previsao", "hr_previsao", "dt_execucao", "hr_execucao"]
    template_name = "tarefas/edita_tarefa.html"
    success_url = reverse_lazy("lista_tarefas")

def tarefa_delete(request, id):
    record = Tarefa.objects.get(pk=id)
    record.delete()
    return redirect("lista_tarefas")
````



Tente reproduzir este passo a passo para ver se dá certo para você. Boa sorte em seus estudos.
