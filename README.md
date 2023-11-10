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
```shell
$ pip install django
$ python -m django --version
```
## Criando um Projeto para Conter a Aplicação
Nomearemos o projeto como mysite
```shell
$ django-admin startproject mysite
```
## Criando a Aplicação Polls (Enquete)
O exemplo que constriremos consiste de uma aplicação para enquetes.
