# CRUD Simples de Tarefas com Python - Django e Insomnia
CRUD (Create, Read, Update e Delete) simples de tarefas em Python Django, com a comunicação da API através de arquivos JSON.
- Crie uma pasta para o projeto e dentro dela crie um ambiente virtual e o ative, para mais informações no link.
- Com o ambiente virtual ativado vamos instalar a biblioteca do Django.
     
		 pip install djangorestframework

- Para este exemplo será feito um projeto do zero, conforme [referência](https://docs.djangoproject.com/en/3.2/intro/tutorial01/): 

		django-admin startproject projeto_crud_simples

- Dentro da pasta do projeto teste se esta tudo funcionando.
		 
		python3 manage.py runserver

- Abra no seu navegador ou no Insomnia, por padrão o Django estará ativo na porta 8000, você deverá ver algo parecido com isto:
![image](https://user-images.githubusercontent.com/73707400/136260688-51f9ec0a-b8ef-4175-b021-36d2c01e5002.png)

- Criaremos um app chamada tarefas, para realizar as ações do CRUD.

		python manage.py startapp tarefas

- Registre o novo aplicativo ao seu projeto

```diff
INSTALLED_APPS = [
		'django.contrib.admin',
		'django.contrib.auth',
		'django.contrib.contenttypes',
		'django.contrib.sessions',
		'django.contrib.messages',
		'django.contrib.staticfiles',
		'tarefas',
]
```

- Crie o [model](https://docs.djangoproject.com/en/3.2/topics/db/models/) 'Tarefa', com um campo de texto e status.

```diff
from django.db import models

class Tarefa(models.Model):
	nome = models.CharField(max_length=40)
	status = models.BooleanField(default=False)
```

- Agora iremos criar o arquivo que cria a base de dados, com todas as tabelas padrões do django e o novo model Tarefa.

		python manage.py makemigrations

Isto criará um arquivo na pasta migrations no seu app

- Para executar o arquivo criado acima execute

		python manage.py migrate

- Para a [serialização](https://www.django-rest-framework.org/api-guide/serializers/) crie um arquivo serializers.py no seu aplicativo, como vamos permitir que a API interaja com todos os campos do nosso model podemos colocar __all__ em fields.

```diff
from rest_framework.serializers import ModelSerializer
from tarefas.models import Tarefa

class TarefaSerializacao(ModelSerializer):
	class Meta:
		model = Tarefa
		fields = '__all__'
```

- Para as ações de CRUD vamos inserir uma configuração de paginação para nossa API, optaremos por 15 items por página, em settings.py na pasta do projeto adicione ao seu código:

```diff
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 15
}
```

- As duas primeiras ações que vamos inserir na API serão a adição de tarefas e retornar a tarefa em função de sua id, usando as views com classes, no arquivo views.py no aplicativo 
		
```diff
from .models import Tarefa
from rest_framework.response import Response
from rest_framework.views import APIView
from rest_framework.pagination import PageNumberPagination
from .serializers import TarefaSerializacao

# Classe da API para mostrar, editar e deletar uma única tarefa
class TarefaAPIView(APIView):

# Função Mostrar
    def get(self, request, id, format=None):

# O programa tenta encontrar uma tarefa no banco com a id solicitada
	try:
	    item = Tarefa.objects.get(pk=id)

# Serialização dos dados com o model Tarefa
	    serializer = TarefaSerializacao(item)

# Retorna os valores da tarefa
	    return Response(serializer.data)

# Caso não haja uma tarefa com a id fornecida volta a Response status 404, "Not found"
	except Tarefa.DoesNotExist:
	    return Response(status=404)

# Classe da API para mostrar as Tarefas em lista e adicionar novas
class TarefaAPIListView(APIView):

# Função nova tarefa 
    def post(self, request, format=None):

# Serialização dos dados fornecidos
	serializer = TarefaSerializacao(data=request.data)

# Validação dos dados conforme definido no Model
	if serializer.is_valid():

# Caso os dados estiverem corretos ele salva o objeto na Base de Dados
	    serializer.save()

# Retorna a resposta para o Client com o novo objeto inserido
	    response = Response(serializer.data, status=201)
	    return response

# Caso os dados não estejam válidos a API retorna os erros para o Client 
	return Response(serializer.errors, status=400)

```

- Crie o arquivo urls.py no aplicativo 'tarefas', para registrar todas as [rotas](https://docs.djangoproject.com/en/3.2/topics/http/urls/) do seu app		

```diff
from django.urls import path
from tarefas import views

urlpatterns = [
  path('tarefa/<int:id>/', views.TarefaAPIView.as_view(), name='tarefa'),
  path('tarefa/', views.TarefaAPIListView.as_view(),name='tarefas'),
]
```

- No arquivo urls.py agora na pasta do seu projeto inclua todos os urls do app tarefas

```diff
from django.urls import path, include

urlpatterns = [
	path('', include('tarefas.urls'))
]
```
- Agora teste a adição de tarefas no Insomnia, envie em formato Json para a rota "tarefa/" uma requisição POST, com nome da tarefa:
![image](https://user-images.githubusercontent.com/73707400/136432072-e3f6f112-a65c-4826-8a95-d4eaa0cf4a1d.png)
Você deve receber uma resposta com a nova tarefa criado e status 201

- Vocẽ pode testar a rota de mostrar requisitando um GET na rota tarefa/'id da tarefa'

- Neste ponto seu projeto deve estar parecido com este:

		projeto_crud_simples/
			manage.py
			projeto_crud_simples/
				__init__.py
				settings.py
				urls.py
				asgi.py
				wsgi.py
			 tarefas/
				__init__.py
				admin.py
				apps.py
				migrations/
					__init__.py
					0001_initial.py
				models.py
				tests.py
				views.py
				serializers.py
				urls.py             
 
- Voltando para as views, vamos criar as rotas de editar, deletar tarefa individual e mostrar em lista, as views ficariam parecidas com isto:

```diff
from django.shortcuts import render
from .models import Tarefa
from rest_framework.response import Response
from rest_framework.views import APIView
from rest_framework.pagination import PageNumberPagination
from .serializers import TarefaSerializacao

# Classe da API para mostrar, editar e deletar uma única tarefa
class TarefaAPIView(APIView):

# Função Mostrar
    def get(self, request, id, format=None):

# O programa tenta encontrar uma tarefa no banco com a id solicitada
	try:
	    item = Tarefa.objects.get(pk=id)

# Serialização dos dados com o model Tarefa
	    serializer = TarefaSerializacao(item)

# Retorna os valores da tarefa
	    return Response(serializer.data)

# Caso não haja uma tarefa com a id fornecida volta a Response status 404, "Not found"
	except Tarefa.DoesNotExist:
	    return Response(status=404)

# Função editar
    def put(self, request, id, format=None):

# Procura se há alguma Tarefa com a id fornecida
	try:
	    item = Tarefa.objects.get(pk=id)

# Caso não encontre volta status 404 Not found 
	except Tarefa.DoesNotExist:
	    return Response(status=404)

# Serialização dos dados fornecidos    
	serializer = TarefaSerializacao(item, data=request.data)

# Validação dos dados conforme definido no Model
	if serializer.is_valid():

# Caso os dados estiverem corretos ele salva o objeto na Base de Dados
	    serializer.save()

# Retorna a resposta para o Client com o novo objeto inserido
	    return Response(serializer.data)

# Caso os dados não estejam válidos a API retorna os erros para o Client 
	return Response(serializer.errors, status=400)

# Função deletar
    def delete(self, request, id, format=None):

# Procura se há alguma Tarefa com a id fornecida
	try:
	    item = Tarefa.objects.get(pk=id)

# Caso não encontre volta status 404 Not found     
	except Tarefa.DoesNotExist:
	    return Response(status=404)

# Deleta o item
	item.delete()

# Retorna status 204
	return Response(status=204)

# Classe da API para mostrar as Tarefas em lista e adicionar novas
class TarefaAPIListView(APIView):

# Função mostrar lista de tarefas
    def get(self, request, format=None):

# Busca todas as tarefas e as ordena por id, pk - primary key
	items = Tarefa.objects.order_by('pk')

# Cria o objeto de paginação
	paginator = PageNumberPagination()

# Aplicação das tarefas encontradas no objeto de paginação
	result_page = paginator.paginate_queryset(items, request)

# Serialização dos dados
	serializer = TarefaSerializacao(result_page, many=True)

# Retorno dos dados
	return paginator.get_paginated_response(serializer.data)

# Função nova tarefa 
    def post(self, request, format=None):

# Serialização dos dados fornecidos
	serializer = TarefaSerializacao(data=request.data)

# Validação dos dados conforme definido no Model
	if serializer.is_valid():

# Caso os dados estiverem corretos ele salva o objeto na Base de Dados
	    serializer.save()

# Retorna a resposta para o Client com o novo objeto inserido
	    response = Response(serializer.data, status=201)
	    return response

# Caso os dados não estejam válidos a API retorna os erros para o Client 
	return Response(serializer.errors, status=400)
```			
			
- Pronto agora você pode testar todas as operações CRUD no Insomnia			
