# Building REST APIs with Django REST Framework

---

## Introduction

A REST API is just a set of URLs that let a frontend (or Postman, or curl) talk to your database using HTTP methods:

| Method | What it does      | Example              |
|--------|--------------------|-----------------------|
| GET    | Fetch data          | `/api/books/`         |
| POST   | Create new data     | `/api/books/`         |
| PUT/PATCH | Update data      | `/api/books/1/`        |
| DELETE | Remove data         | `/api/books/1/`        |

**Django REST Framework (DRF)** just gives us ready-made tools so we don't build all this from scratch. We'll build a simple **Book API**:

- **Part 1** — CRUD (add, view, update, delete books) — open to everyone for now
- **Part 2** — Login system so only authenticated users can add/update/delete

---
---

# PART 1 — CRUD API

---

## Step 1: Project Setup

```bash
mkdir django_rest_demo
cd django_rest_demo

python -m venv venv
venv\Scripts\activate        # Windows
source venv/bin/activate     # Mac/Linux

pip install django djangorestframework

django-admin startproject core .
python manage.py startapp books

python manage.py migrate
python manage.py createsuperuser
```

**What's happening here:**
- We made a folder, turned on a virtual environment (keeps our packages separate from other projects), and installed Django + DRF.
- `startproject core` creates the main project. `startapp books` creates our app where the Book logic will live.
- `migrate` sets up the default database tables. `createsuperuser` gives us an admin login.

---

## Step 2: Turn On DRF

Open **`core/settings.py`** and add `rest_framework` and `books` to `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'rest_framework',
    'books',
]
```

That's it for now — just telling Django "these two apps exist, load them." We'll add security settings here in Part 2.

---

## Step 3: Create the Book Model

The model is just our database table, written as a Python class.

```python
# books/models.py
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.CharField(max_length=100)
    isbn = models.CharField(max_length=13, unique=True)
    price = models.DecimalField(max_digits=8, decimal_places=2)
    published_date = models.DateField()
    in_stock = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

```bash
python manage.py makemigrations books
python manage.py migrate
```

**What's happening here:**
- Every field here becomes a column in our `Book` table.
- `unique=True` on `isbn` just stops two books from having the same ISBN.
- `auto_now_add=True` auto-fills the date when a book is created — we never set this ourselves.
- `makemigrations` writes out the "instructions" for the new table, and `migrate` actually creates it in the database.

---

## Step 4: Create the Serializer

Think of the serializer as a translator: it converts a `Book` object into JSON (to send to the frontend) and converts incoming JSON back into a `Book` object (to save it).

```python
# books/serializers.py
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'isbn', 'price', 'published_date', 'in_stock', 'created_at']
        read_only_fields = ['id', 'created_at']
```

**What's happening here:**
- `ModelSerializer` auto-builds the fields for us based on the `Book` model, so we don't type them out manually.
- `fields` lists what's allowed to go in/out of the API.
- `read_only_fields` means these show up in the response, but the user can't set or change them themselves (Django sets them automatically).

---

## Step 5: Build the CRUD Views

This is where the actual GET/POST/PUT/DELETE logic happens. We'll use **function-based views** with the `@api_view` decorator — one simple function per action, easy to read top to bottom.

```python
# books/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from django.shortcuts import get_object_or_404
from .models import Book
from .serializers import BookSerializer


@api_view(['GET'])
def book_list(request):
    books = Book.objects.all()
    serializer = BookSerializer(books, many=True)
    return Response(serializer.data)


@api_view(['POST'])
def book_create(request):
    serializer = BookSerializer(data=request.data)
    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data, status=status.HTTP_201_CREATED)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(['GET'])
def book_view(request, pk):
    book = get_object_or_404(Book, pk=pk)
    serializer = BookSerializer(book)
    return Response(serializer.data)


@api_view(['PUT'])
def book_update(request, pk):
    book = get_object_or_404(Book, pk=pk)
    serializer = BookSerializer(book, data=request.data)
    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(['PATCH'])
def book_partial_update(request, pk):
    book = get_object_or_404(Book, pk=pk)
    serializer = BookSerializer(book, data=request.data, partial=True)
    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(['DELETE'])
def book_delete(request, pk):
    book = get_object_or_404(Book, pk=pk)
    book.delete()
    return Response(status=status.HTTP_204_NO_CONTENT)
```

```python
# books/urls.py
from django.urls import path
from .views import (
    book_list, book_create,
    book_view, book_update, book_partial_update, book_delete
)

urlpatterns = [
    path('books/', book_list, name='book-list'),
    path('books/create/', book_create, name='book-create'),
    path('books/<int:pk>/', book_view, name='book-view'),
    path('books/<int:pk>/update/', book_update, name='book-update'),
    path('books/<int:pk>/partial-update/', book_partial_update, name='book-partial-update'),
    path('books/<int:pk>/delete/', book_delete, name='book-delete'),
]
```

```python
# core/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('books.urls')),
]
```

**What's happening here:**
- Every action now has its **own function and its own URL** — one job per function, nothing shared. `book_list` only lists, `book_create` only creates, and so on.
- `@api_view(['GET'])` (or `['POST']`, `['PUT']`, etc.) tells DRF exactly which method this function accepts — anything else gets auto-rejected with an error.
- `get_object_or_404` fetches the book by its `pk` (ID from the URL), and automatically returns a proper 404 error if that book doesn't exist — no manual checking needed.
- `partial=True` in `book_partial_update` means "only validate the fields that were actually sent" — that's what makes PATCH different from PUT.
- `include('books.urls')` just plugs our app's URLs into the main project under `/api/`.

---

## Step 6: Test the CRUD API

```bash
# Get all books
curl http://127.0.0.1:8000/api/books/

# Create a book
curl -X POST http://127.0.0.1:8000/api/books/create/ \
  -H "Content-Type: application/json" \
  -d '{"title": "Clean Code", "author": "Robert C. Martin", "isbn": "9780132350884", "price": "35.00", "published_date": "2008-08-01"}'

# Get one book
curl http://127.0.0.1:8000/api/books/1/

# Update a book fully
curl -X PUT http://127.0.0.1:8000/api/books/1/update/ \
  -H "Content-Type: application/json" \
  -d '{"title": "Clean Code (2nd Ed)", "author": "Robert C. Martin", "isbn": "9780132350884", "price": "40.00", "published_date": "2008-08-01", "in_stock": true}'

# Update just one field
curl -X PATCH http://127.0.0.1:8000/api/books/1/partial-update/ \
  -H "Content-Type: application/json" \
  -d '{"price": "38.00"}'

# Delete a book
curl -X DELETE http://127.0.0.1:8000/api/books/1/delete/
```

**What's happening here:**
- Simple rule: **PUT** = send everything again (full replace). **PATCH** = send only what you want to change.
- Right now, all of this works for anyone — there's no login required yet. That's the problem Part 2 solves.

---
---

# PART 2 — AUTHENTICATION API

Right now anyone can create, edit, or delete books — not good. Let's add a login system so only logged-in users can make changes, while anyone can still just view books.

---

## Step 1: Turn On Token Authentication

Update **`core/settings.py`**:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'rest_framework',
    'rest_framework.authtoken',   # new
    'books',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
}
```

```bash
python manage.py migrate
```

**What's happening here:**
- Think of a "token" like a temporary ID card. Once a user logs in, they get one, and they show it on every request to prove who they are.
- `IsAuthenticatedOrReadOnly` = "anyone can look, only logged-in users can touch."
- `migrate` just creates the table needed to store these tokens.

---

## Step 2: Add a Login Endpoint

Update **`core/urls.py`**:

```python
from django.contrib import admin
from django.urls import path, include
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('books.urls')),
    path('api/token-auth/', obtain_auth_token),
]
```

Now log in and grab a token:

```bash
curl -X POST http://127.0.0.1:8000/api/token-auth/ \
  -H "Content-Type: application/json" \
  -d '{"username": "faraz", "password": "yourpassword"}'
```

You'll get back something like:

```json
{ "token": "9944b09199c62bcf9418ad846dd0e4bbdfc6ee4" }
```

Now use that token on any write request:

```bash
curl -X POST http://127.0.0.1:8000/api/books/create/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4" \
  -d '{"title": "The Pragmatic Programmer", "author": "Andrew Hunt", "isbn": "9780135957059", "price": "42.00", "published_date": "1999-10-20"}'
```

**What's happening here:**
- `obtain_auth_token` is a ready-made view — you send it username + password, it gives you a token back. No custom code needed.
- That `Authorization: Token ...` header is how every future request proves "yes it's really me, logged in."
- Try skipping the header now — you'll get a 401/403 error. That's the security actually working.

---

## Step 3: Let Users Sign Up (Register)

Right now, only existing users (created via `createsuperuser`) can log in. Let's let new users register through the API.

```python
# books/serializers.py — add this
from django.contrib.auth.models import User

class RegisterSerializer(serializers.ModelSerializer):
    password = serializers.CharField(write_only=True)

    class Meta:
        model = User
        fields = ['username', 'email', 'password']

    def create(self, validated_data):
        return User.objects.create_user(
            username=validated_data['username'],
            email=validated_data.get('email', ''),
            password=validated_data['password']
        )
```

```python
# books/views.py — add this
from rest_framework import generics, permissions
from rest_framework.authtoken.models import Token
from rest_framework.response import Response
from .serializers import RegisterSerializer

class RegisterView(generics.CreateAPIView):
    serializer_class = RegisterSerializer
    permission_classes = [permissions.AllowAny]

    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = serializer.save()
        token, _ = Token.objects.get_or_create(user=user)
        return Response({'username': user.username, 'token': token.key}, status=201)
```

```python
# books/urls.py — add this route
path('register/', RegisterView.as_view(), name='register'),
```

**What's happening here:**
- `write_only=True` on password just means: accept it as input, but never send it back in a response.
- `create_user()` is Django's safe way of saving a password — it automatically hashes it, so it's never stored as plain text.
- `AllowAny` makes sense here — a brand-new user obviously isn't "logged in" yet, so registration has to be open.
- After saving the new user, we immediately hand them a token too, so they don't need to log in separately right after signing up.

---

## Step 4: Test the Full Flow

```bash
# Register a new user
curl -X POST http://127.0.0.1:8000/api/register/ \
  -H "Content-Type: application/json" \
  -d '{"username": "student1", "email": "student1@example.com", "password": "StrongPass123"}'

# Try creating a book WITHOUT a token — should fail
curl -X POST http://127.0.0.1:8000/api/books/create/ \
  -H "Content-Type: application/json" \
  -d '{"title": "Test Book", "author": "Someone", "isbn": "1112223334445", "price": "10.00", "published_date": "2024-01-01"}'

# Now create it WITH the token you got from registering — should work
curl -X POST http://127.0.0.1:8000/api/books/create/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Token abc123..." \
  -d '{"title": "Test Book", "author": "Someone", "isbn": "1112223334445", "price": "10.00", "published_date": "2024-01-01"}'
```

**What's happening here:**
- This proves the whole thing works end-to-end: register → get token → use token → protected action succeeds.
- Without the token, it should fail. That failure is a good sign — it means your security is doing its job.

---
