## Conecte o projeto Django ao MongoDB usando Djongo

Fonte: [Acervo Lima](https://acervolima.com/conecte-o-projeto-django-ao-mongodb-usando-djongo/)

Djongo é um transpiler de consulta SQL para mongodb. 
Usando djongo, podemos usar MongoDB como um banco de dados de 
back-end para nosso projeto Django. Nem mesmo precisamos mudar o Django ORM. 
A melhor parte é que podemos configurar o Django com MongoDB adicionando 
apenas uma linha de código. Não há necessidade de alterar serializadores, 
visualizações ou quaisquer outros módulos.

Documentos oficiais - https://pypi.org/project/djongo/ 

Trabalhando - 
Djongo traduz uma string de consulta SQL em um documento de consulta MongoDB. 
Portanto, não há necessidade de alterar os modelos, serializadores, 
visualizações ou quaisquer recursos do Django. 
Djongo suporta todas as bibliotecas contrib django, 
o que o torna um conector fácil de usar: 

. Django.contrib.admin
. Django.contrib.sessions
. Django.contrib.auth

Requisitos - 


1. Python 3.6 ou superior.

2. MongoDB 3.4 ou superior. (Se você estiver usando consultas aninhadas, o MongoDB 3.6 ou superior é necessário.)

Recursos :  

Reutilizar Modelos Django / ORM - 
Como os Modelos Django são compatíveis com Djongo, podemos reutilizá-los.
Verificações de integridade O 

Djongo permite verificações de integridade como valores ausentes antes de serem salvos no banco de dados.
Por exemplo: valores ausentes nunca são armazenados se definirmos null = False, blank = False em EmbeddedField

```python
class Students(models.Model):
    masks = models.EmbeddedField(Null=False, Blank=False)
```

Validadores 
Podemos aplicar verificações de validação como URLValidator, EmailValidator, RegexValidator etc. antes de cada campo ser salvo no banco de dados.


