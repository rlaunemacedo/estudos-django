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
```shell
$ django-admin startproject meusite
$ cd meusite
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
### Criar o arquivo `urls.py` no diretório `catalog`
Arquivo: `catalog/urls.py`
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
Arquivo: `catalog/models.py`
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
Arquivo: `catalog/admin.py`
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




