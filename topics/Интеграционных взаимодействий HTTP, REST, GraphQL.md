# Интеграционные взаимодействия: HTTP, REST, GraphQL

> **Теги**: #web #api #django #graphql #rest #http #backend

Сравнение и реализация интеграционных подходов: HTTP, REST и GraphQL, с примером GraphQL на Django.

---

## Сравнение подходов

| Критерий               | HTTP                          | REST                          | GraphQL                       |
|------------------------|-------------------------------|-------------------------------|-------------------------------|
| Тип                   | Протокол                      | Архитектурный стиль           | Язык запросов / API-подход    |
| Транспорт             | Сам по себе                   | Использует HTTP               | Использует HTTP               |
| Endpoint              | Любой                         | Множество (`/users`, `/posts`)| Один (`/graphql`)             |
| Гибкость              | Низкая                        | Средняя                       | Высокая                       |
| Over/Under-fetching   | Зависит от реализации         | Часто есть                    | Минимизировано                |
| Поддержка мутаций     | Да (POST, PUT, DELETE)        | Да                            | Да (через `mutation`)         |
| Поддержка подписок    | Нет (без WebSocket)           | Нет                           | Да (через WebSocket)          |

> 💡 **Связь между ними**:  
> REST и GraphQL строятся **поверх HTTP**. HTTP — транспорт, REST и GraphQL — способы организации API.

---

## Когда что использовать?

- **HTTP** — всегда, когда нужен веб-обмен.
- **REST** — для простых, стандартных API, публичных сервисов, микросервисов.
- **GraphQL** — когда важна гибкость, минимизация трафика, сложные клиентские запросы (например, мобильные приложения, дашборды).

См. также: [[API Design]]

---

## Пример: GraphQL на Django

Реализация GraphQL API в Django с использованием `graphene-django`.

### 🛠 Установка
### 🧱 Структура проекта

myproject/
├── myproject/
│   ├── settings.py
│   ├── urls.py
│   └── schema.py
├── users/
│   ├── models.py
│   └── ...

### 🔧 Настройка `settings.py`

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    # ...
    'users',
    'graphene_django',
]

GRAPHENE = {
    'SCHEMA': 'myproject.schema.schema',
}
### 📦 Модель

# users/models.py
from django.db import models

class User(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    age = models.IntegerField()


### 🧵 GraphQL Schema

python

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

⌄

⌄

⌄

⌄

⌄

⌄

⌄

⌄

⌄

# myproject/schema.py

# myproject/schema.py
import graphene
from graphene_django import DjangoObjectType
from users.models import User

class UserType(DjangoObjectType):
    class Meta:
        model = User
        fields = "__all__"

class Query(graphene.ObjectType):
    users = graphene.List(UserType)
    user = graphene.Field(UserType, id=graphene.Int())

    def resolve_users(self, info):
        return User.objects.all()

    def resolve_user(self, info, id):
        return User.objects.filter(pk=id).first()

class CreateUser(graphene.Mutation):
    class Arguments:
        name = graphene.String(required=True)
        email = graphene.String(required=True)
        age = graphene.Int(required=True)

    user = graphene.Field(UserType)

    def mutate(self, info, name, email, age):
        user = User(name=name, email=email, age=age)
        user.save()
        return CreateUser(user=user)

class Mutation(graphene.ObjectType):
    create_user = CreateUser.Field()

schema = graphene.Schema(query=Query, mutation=Mutation)

### 🌐 URL-маршруты

# myproject/urls.py
from django.urls import path
from graphene_django.views import GraphQLView
from django.views.decorators.csrf import csrf_exempt

urlpatterns = [
    path('admin/', admin.site.urls),
    path('graphql/', csrf_exempt(GraphQLView.as_view(graphiql=True))),
]

> ⚠️ `csrf_exempt` — только для разработки. В продакшене используйте JWT или другую аутентификацию.

---

## 🧪 Примеры запросов

### Запрос: получить всех пользователей

query {
  users {
    id
    name
    email
    age
  }
}

### Запрос: получить пользователя по ID

query {
  user(id: 1) {
    name
    email
    age
  }
}

### Мутация: создать пользователя

mutation {
  createUser(name: "Мария", email: "maria@example.com", age: 30) {
    user {
      id
      name
      email
      age
    }
  }
}

---

## 🔗 Полезные ссылки

- [Graphene-Django Docs](https://docs.graphene-python.org/projects/django/en/latest/)
- [GraphQL.org](https://graphql.org/)
- [[Django Best Practices]]
- [[API Security]]

---

