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

		INSTALLED_APPS = [
				'django.contrib.admin',
				'django.contrib.auth',
				'django.contrib.contenttypes',
				'django.contrib.sessions',
				'django.contrib.messages',
				'django.contrib.staticfiles',
				'tarefas',
		]

- Crie o [model](https://docs.djangoproject.com/en/3.2/topics/db/models/) 'Tarefa', com um campo de texto e status.

		from django.db import models

		class Tarefa(models.Model):
			nome = models.CharField(max_length=40)
			status = models.BooleanField(default=False)

- Agora iremos criar o arquivo que cria a base de dados, com todas as tabelas padrões do django e o novo model Tarefa.

		python manage.py makemigrations

Isto criará um arquivo na pasta migrations no seu app

- Para executar o arquivo criado acima execute

		python manage.py migrate

- Para a [serialização](https://www.django-rest-framework.org/api-guide/serializers/) crie um arquivo serializers.py no seu aplicativo, como vamos permitir que a API interaja por completo com nosso model podemos colocar __all__ em fields.

		from rest_framework.serializers import ModelSerializer
		from tarefas.models import Tarefa

		class TarefaSerializacao(ModelSerializer):
			class Meta:
				model = Tarefa
				fields = '__all__'
- Para as ações de CRUD vamos inserir uma configuração de paginação para nossa API, optaremos por 15 items por página, em settings.py na pasta do projeto adicione ao seu código:

		REST_FRAMEWORK = {
		    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
		    'PAGE_SIZE': 15
		}

- A primeira ação que vamos inserir na API será a adição de tarefas, usando as views com classes, no arquivos view.py no aplicativo 

		from .models import Tarefa
		from rest_framework.response import Response
		from rest_framework.views import APIView
		from .serializers import TarefaSerializacao
		from rest_framework.pagination import PageNumberPagination
		
		class TarefaAPIListView(APIView):
		    def post(self, request, format=None):
			serializer = TarefaSerializacao(data=request.data)
			if serializer.is_valid():
			    serializer.save()
			    response = Response(serializer.data, status=201)
			    return response
			return Response(serializer.errors, status=400)








- Crie o arquivo urls.py no aplicativo 'tarefas', para registrar todas as [rotas](https://docs.djangoproject.com/en/3.2/topics/http/urls/) do seu app		

		from django.conf.urls import url
		from tarefas import views

		urlpatterns = [
			url(r'^tarefa/(?P<id>[0-9]+)/$', views.TarefaAPIView.as_view()),
			url(r'^tarefa/', views.TarefaAPIListView.as_view()),
		]


- No arquivo urls.py agora na pasta do seu projeto inclua todos os urls do app tarefas

		from django.urls import path, include

		urlpatterns = [
		        path('', include('tarefas.urls'))
		]

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
 
