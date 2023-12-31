# MINHA RECEITA DJANGO
**R N Launé Macêdo**

**Timon-MA, 11../11/2023**

Para a construção desta ingênua **receita**, usei como base a excelente fonte [**mdn web docs**](https://developer.mozilla.org/pt-BR/docs/Learn/Server-side/Django). Fique claro, portanto que, por se tratar de uma receita, omitimos as valiosas observações e detalhamentos encontrados na supracitada fonte. Espero sinceramente que este trabalho seja útil aos iniciantes, como eu, ao estudo do framework **Django**.

Meu ambiente de trabalho é o Linux Mint 21.2, Cinnamon (5.8.4) e Django 4.2
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

Nomeei o projeto de `setup` por conveniencia e coloquei um `.` para que não seja criado um diretório com o mesmo nome setup dentro de setup (`setup/setup`).

```shell
$ django-admin startproject setup .
$ cd setup
```
## Criando um Aplicativo
```shell
$ python3 manage.py startapp catalog
$ cd catalog
```
## Configurações
### Registrando o app
Arquivo: `meusite/settings.py`
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
Arquivo: `meusite/settings.py`
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```
### Outras configurações
Arquivo: `meusite/settings.py`
```python
LANGUAGE_CODE = 'pt-br'
TIME_ZONE = 'America/Fortaleza'
```
## Definindo as Rotas
### Conectando os mapeadores de URL
Arquivo: `meusite/urls.py`
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('catalog/', include('catalog.urls')),
]
```
### Mudando a raiz do site de 127.0.0.1:8000 para 127.0.0.1:8000/catalog/
Arquivo: `meusite/urls.py`
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
Arquivo: `meusite/urls.py`
```python
from django.contrib import admin
from django.urls import path, include
from django.views.generic import RedirectView
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('catalog/', include('catalog.urls')),
    path('', RedirectView.as_view(url='/catalog/')),
]

## Use static() to add url mapping to serve static files during development (only)
urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```
### Criar o arquivo `urls.py` no diretório `catalog`
A partir do terminal de comandos, no diretório `catalog/`, usaremos `touch` para criar o arquivo urls.py, como segue:

**Diretório:** `catalog/`

```shell
$ touch urls.py
```
O arquivo deverá ter o seguinte conteúdo por enquanto, mais tarde o atualizaremos:

**Arquivo:** `catalog/urls.py`
```python
from django.urls import path
from catalog import views

urlpatterns = [

]

```
## Testando o Framework do Site
### Migrando o banco de dados
```shell
$ python3 manage.py makemigrations
$ python3 manage.py migrate
```
### Testando o website
```shell
$ python3 manage.py runserver
```
Apesar da mensagem de erro, **está funcionando!**.

## Modelagem dos Dados
### Definindo as tabelas
**Arquivo:** `catalog/models.py`
```python
from django.db import models
from django.urls import reverse     # Used to generate URLs by reversing the URL patterns
import uuid # Required for unique book instances

## Create your models here.
class Genre(models.Model):
    """Model representing a book genre."""
    name = models.CharField(max_length=200, help_text='Enter a book genre (e.g. Science Fiction)')

    def __str__(self):
        """String for representing the Model object."""
        return self.name

class Book(models.Model):
    """Model representing a book (but not a specific copy of a book)."""
    title = models.CharField(max_length=200)

    # Foreign Key used because book can only have one author, but authors can have multiple books
    # Author as a string rather than object because it hasn't been declared yet in the file
    author = models.ForeignKey('Author', on_delete=models.SET_NULL, null=True)

    summary = models.TextField(max_length=1000, help_text='Enter a brief description of the book')
    isbn = models.CharField('ISBN', max_length=13, help_text='13 Character <a href="https://www.isbn-international.org/content/what-isbn">ISBN number</a>')

    # ManyToManyField used because genre can contain many books. Books can cover many genres.
    # Genre class has already been defined so we can specify the object above.
    genre = models.ManyToManyField(Genre, help_text='Select a genre for this book')

    def __str__(self):
        """String for representing the Model object."""
        return self.title

    def get_absolute_url(self):
        """Returns the url to access a detail record for this book."""
        return reverse('book-detail', args=[str(self.id)])

class BookInstance(models.Model):
    """Model representing a specific copy of a book (i.e. that can be borrowed from the library)."""
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, help_text='Unique ID for this particular book across whole library')
    book = models.ForeignKey('Book', on_delete=models.SET_NULL, null=True)
    imprint = models.CharField(max_length=200)
    due_back = models.DateField(null=True, blank=True)

    LOAN_STATUS = (
        ('m', 'Maintenance'),
        ('o', 'On loan'),
        ('a', 'Available'),
        ('r', 'Reserved'),
    )

    status = models.CharField(
        max_length=1,
        choices=LOAN_STATUS,
        blank=True,
        default='m',
        help_text='Book availability',
    )

    class Meta:
        ordering = ['due_back']

    def __str__(self):
        """String for representing the Model object."""
        return f'{self.id} ({self.book.title})'

class Author(models.Model):
    """Model representing an author."""
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    date_of_birth = models.DateField(null=True, blank=True)
    date_of_death = models.DateField('Died', null=True, blank=True)

    class Meta:
        ordering = ['last_name', 'first_name']

    def get_absolute_url(self):
        """Returns the url to access a particular author instance."""
        return reverse('author-detail', args=[str(self.id)])

    def __str__(self):
        """String for representing the Model object."""
        return f'{self.last_name}, {self.first_name}'
```
### Migrando o banco de dados
```shell
$ python3 manage.py makemigrations
$ python3 manage.py migrate
```
## Registrando os Modelos
**Arquivo:** `catalog/admin.py`

```python
from django.contrib import admin
from catalog.models import Author, Genre, Book, BookInstance

## Register your models here.
admin.site.register(Book)
admin.site.register(Author)
admin.site.register(Genre)
admin.site.register(BookInstance)
```

## Administração do Website
### Criando um super usuário
```shell
$ python3 manage.py createsuperuser
```
### Vamos testar
```shell
$ python3 manage.py runserver
```
### Fazer login
Para fazer login no site, abra o `URL` `/admin` (e.i. http://127.0.0.1:8000/admin)

## Mapeando URLs
O objetivo é ter os seguintes links:

    catalog/ — A página inicial (index).
    catalog/books/ — Uma lista de todos os livros.
    catalog/authors/ — Uma lista de todos os autores.
    catalog/book/<id> — A exibição de detalhes de um livro específico, com uma chave primária de campo <id> (o padrão). Por exemplo, o URL do terceiro livro adicionado à lista será /catalog/book/3.
    catalog/author/<id> — A exibição de detalhes para o autor específico com um campo de chave primária de <id>. Por exemplo, o URL do 11º autor adicionado à lista será /catalog/author/11.

### Criando o ambiente das templates
Criar o diretório `catalog/templates` e dentro dele o arquivo `base_generic.html`. e cole as linhas:

**Diretório:** `catalog/`
```shell
$ mkdir templates
```
#### Configurando a chave TEMPLATES em `settings.py`
**Arquivo:** `settings.py`

```python
## Não esquecer de importar os
import os

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            os.path.join(BASE_DIR, 'templates'),
        ],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
### Criando um arquivo html base
Chamaremos de `base_generic.html` o script htlm carregado por todas as páginas.

**Diretório:** `catalog/templates`
```shell
$ touch base_generic.html
```
Que terá o seguinte conteúdo:

**Arquivo:** `catalog/templates/base_generic.html`
```html
<!doctype html>
<html lang="en">
  <head>
    {% block title %}<title>Local Library</title>{% endblock %}
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link
      rel="stylesheet"
      href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css"
      integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO"
      crossorigin="anonymous" />
    <!-- Add additional CSS in static file -->
    {% load static %}
    <link rel="stylesheet" href="{% static 'css/styles.css' %}" />
  </head>
  <body>
    <div class="container-fluid">
      <div class="row">
        <div class="col-sm-2">
          {% block sidebar %}
            <ul class="sidebar-nav">
              <li><a href="{% url 'index' %}">Home</a></li>
              <li><a href="">Todos os livros</a></li>
              <li><a href="">Todos os autores</a></li>
            </ul>
          {% endblock %}
        </div>
        <div class="col-sm-10">{% block content %}{% endblock %}</div>
      </div>
    </div>
  </body>
</html>
```

#### Criando um arquivo de estilos (css)
Como o `base_generic.html` faz alusão a `style.css`, devemos criá-lo dentro do diretório `catalog/static/css` que, por sua vez deve ser ciado antes. Uma vez criados, colemos as linhas:

**Diretório:** `catalog/`
```shell
$ mkdir static
$ mkdir static/css
$ cd static/css
$ touch style.css
```
**Arquivo:** `catalog/static/css/style.css`
```css
.sidebar-nav {
    margin-top: 20px;
    padding: 0;
    list-style: none;
  }
```
## A Página de Início
### Criando o template `index.html`
**Diretório:** `catalog/templates/`
```shell
$ touch index.html
```
**Arquivo:** `catalog/templates/index.html`
```html
{% extends "base_generic.html" %}

{% block content %}
  <h1>Local Library Home</h1>
  <p>
    Welcome to LocalLibrary, a website developed by
    <em>Mozilla Developer Network</em>!
  </p>
  <h2>Dynamic content</h2>
  <p>The library has the following record counts:</p>
  <ul>
    <li><strong>Books:</strong> {{ num_books }}</li>
    <li><strong>Copies:</strong> {{ num_instances }}</li>
    <li><strong>Copies available:</strong> {{ num_instances_available }}</li>
    <li><strong>Authors:</strong> {{ num_authors }}</li>
  </ul>
{% endblock %}
```
### Adicionando a função `views.index` que carrega o `index.html`
**Arquivo:** `catalog/views.py`
```python
from django.shortcuts import render
from catalog.models import Book, Author, BookInstance, Genre

# Create your views here.
def index(request):
    """View function for home page of site."""

    # Generate counts of some of the main objects
    num_books = Book.objects.all().count()
    num_instances = BookInstance.objects.all().count()

    # Available books (status = 'a')
    num_instances_available = BookInstance.objects.filter(status__exact='a').count()

    # The 'all()' is implied by default.
    num_authors = Author.objects.count()

    context = {
        'num_books': num_books,
        'num_instances': num_instances,
        'num_instances_available': num_instances_available,
        'num_authors': num_authors,
    }

    # Render the HTML template index.html with the data in the context variable
    return render(request, 'index.html', context=context)
```

### Roteando `views.index`
Como dissemos anteriormente, vamos atualizar o arquivo `catalog/urls.py`.

**Arquivo:** `catalog/urls.py`
```python
from django.urls import path
from catalog import views

urlpatterns = [
    path('', views.index, name='index'),
]
```
**Vamos ver como está?**
```shell
$ python3 manage.py runserver
```
E abra http://127.0.0.1:8000/ no seu navegador.

> Nota: Os links *Todos os livros* e *Todos os autores* **ainda não funcionarão** porque os caminhos, visualizações e modelos para essas páginas não estão definidos. Acabamos de inserir espaços reservados para esses links no template base_generic.html.

## A Página que Lista *Todos os Livros*

### Criando o template `book_list.html`

**Diretório:** `catalog/templates/`
```shell
$ mkdir catalog
$ cd catalog
$ touch book_list.html
```
Contendo:

**Arquivo:** `catalog/templates/catalog/book_list.html`
```html
{% extends "base_generic.html" %}

{% block content %}
  <h1>Book List</h1>
  {% if book_list %}
  <ul>
    {% for book in book_list %}
      <li>
        <a href="{{ book.get_absolute_url }}">{{ book.title }}</a> ({{book.author}})
      </li>
    {% endfor %}
  </ul>
  {% else %}
    <p>There are no books in the library.</p>
  {% endif %}
{% endblock %}

```

### Adicionando a função `views.BookListView` que carrega `book_list.html`

**Arquivo:** `catalog/views.py`
```python
## adicionar no início
from django.views import generic

## adicionar ao final
class BookListView(generic.ListView):   
    model = Book
```
### Roteando a função `views.BookListView`

Adicionando o item `path('books/', views.BookListView.as_view(), name='books'),` à lista `urlpatterns`, teremos:

**Arquivo:** `catalog/urls.py`
```python
from django.urls import path
from catalog import views

urlpatterns = [
    path('', views.index, name='index'),
    path('books/', views.BookListView.as_view(), name='books'),
]
```

**Atualizando o link `Todos os livros` no template base**

Abra o template base (/locallibrary/catalog/templates/base_generic.html) e insira {% url 'books' %} no link da URL para All books,como mostrado abaixo. Isso habilitará o link em todas as páginas (podemos colocá-lo em prática agora que criamos o mapeador de URL "books").

**Arquivo:** `templates/base_generic.html`
```html
<li><a href="{% url 'index' %}">Início</a></li>
<li><a href="{% url 'books' %}">Todos os livros</a></li>
<li><a href="">Todos os autores</a></l
```
### Criando o template ´book_detail.html´

**Diretório:** `catalog/templates/catalog/`
```shell
$ touch book_detail.html
```
Contendo:

**Arquivo:** `catalog/templates/catalog/book_detail.html`
```html
{% extends "base_generic.html" %}

{% block content %}
  <h1>Title: {{ book.title }}</h1>

  <p><strong>Author:</strong> <a href="">{{ book.author }}</a></p> <!-- author detail link not yet defined -->
  <p><strong>Summary:</strong> {{ book.summary }}</p>
  <p><strong>ISBN:</strong> {{ book.isbn }}</p>
  <p><strong>Language:</strong> {{ book.language }}</p>
  <p><strong>Genre:</strong> {% for genre in book.genre.all %} {{ genre }}{% if not forloop.last %}, {% endif %}{% endfor %}</p>

  <div style="margin-left:20px;margin-top:20px">
    <h4>Copies</h4>

    {% for copy in book.bookinstance_set.all %}
      <hr>
      <p class="{% if copy.status == 'a' %}text-success{% elif copy.status == 'm' %}text-danger{% else %}text-warning{% endif %}">{{ copy.get_status_display }}</p>
      {% if copy.status != 'a' %}
        <p><strong>Due to be returned:</strong> {{copy.due_back}}</p>
      {% endif %}
      <p><strong>Imprint:</strong> {{copy.imprint}}</p>
      <p class="text-muted"><strong>Id:</strong> {{copy.id}}</p>
    {% endfor %}
  </div>
{% endblock %}
```

### Adicionando a função `views.BookDetailView` que carrega `book_detail.html` 

**Arquivo:** `catalog/views.py`
```python
## adicionar no início
from django.views import generic

## adicionar ao final
class BookListView(generic.ListView):   
    model = Book

class BookDetailView(generic.DetailView):    
    model = Book
```
### Roteando a função `views.BookDetailView`

Adicionando o item `path('book/<int:pk>', views.BookDetailView.as_view(), name='book-detail'),` à lista `urlpatterns`, teremos:

**Arquivo:** `catalog/urls.py`
```python
from django.urls import path
from catalog import views

urlpatterns = [
    path('', views.index, name='index'),
    path('books/', views.BookListView.as_view(), name='books'),
    path('book/<int:pk>', views.BookDetailView.as_view(), name='book-detail'),
]
```
**Vamos ver como está?**
```shell
$ python3 manage.py runserver
```
E abra http://127.0.0.1:8000/ no seu navegador.
