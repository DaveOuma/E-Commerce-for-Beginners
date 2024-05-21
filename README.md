# E-Commerce-for-Beginners
Creating a simple E-commerce website that sells clothes is a great project for beginners using Python, Django, and JavaScript:  Project Overview.....The website includes:  Home, Product Listing, Product Detail, Shopping Cart, User Authentication (Sign Up, Login, Logout), Checkout,  Order Confirmation, Admin Interface for managing products pages.

# Simple E-commerce Website

This project is designed to help beginners learn how to create a simple e-commerce website using Python, Django, and JavaScript. The website will sell clothes and includes basic features such as user authentication, product listing, product details, shopping cart, and a checkout process.

## Project Structure
ecommerce/
manage.py
ecommerce/
init.py
settings.py
urls.py
wsgi.py
products/
init.py
admin.py
apps.py
models.py
views.py
urls.py
templates/
products/
product_list.html
product_detail.html
cart/
init.py
admin.py
apps.py
models.py
views.py
urls.py
templates/
cart/
cart_detail.html
users/
init.py
admin.py
apps.py
models.py
views.py
urls.py
templates/
users/
login.html
signup.html
static/
css/
js/
templates/
base.html

## Step-by-Step Guide

### 1. Set Up Your Django Project

First, create a new Django project and applications for products, cart, and users.

```bash
django-admin startproject ecommerce
cd ecommerce
python manage.py startapp products
python manage.py startapp cart
python manage.py startapp users

### 2. Configure Settings
Update settings.py to include your apps and set up static and media files.
INSTALLED_APPS = [
    ...
    'products',
    'cart',
    'users',
    'django.contrib.staticfiles',
]

STATIC_URL = '/static/'
MEDIA_URL = '/media/'

STATICFILES_DIRS = [BASE_DIR / "static"]

### 3. Create Models
Define the models for your products and cart in products/models.py and cart/models.py.

products/models.py

from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=200)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    image = models.ImageField(upload_to='products/')

    def __str__(self):
        return self.name

cart/models.py

from django.db import models
from products.models import Product

class Cart(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return str(self.id)

class CartItem(models.Model):
    cart = models.ForeignKey(Cart, on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    quantity = models.PositiveIntegerField(default=1)

    def __str__(self):
        return f'{self.quantity} of {self.product.name}'

### 4. User Authentication
Use Django's built-in user authentication. Create views for login and signup in users/views.py.

users/views.py

from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.forms import UserCreationForm, AuthenticationForm

def signup_view(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('login')
    else:
        form = UserCreationForm()
    return render(request, 'users/signup.html', {'form': form})

def login_view(request):
    if request.method == 'POST':
        form = AuthenticationForm(data=request.POST)
        if form.is_valid():
            user = form.get_user()
            login(request, user)
            return redirect('product_list')
    else:
        form = AuthenticationForm()
    return render(request, 'users/login.html', {'form': form})

def logout_view(request):
    logout(request)
    return redirect('login')

### 5. Create Views and Templates
Create views for listing products, product details, and managing the cart.

products/views.py

from django.shortcuts import render, get_object_or_404
from .models import Product

def product_list(request):
    products = Product.objects.all()
    return render(request, 'products/product_list.html', {'products': products})

def product_detail(request, pk):
    product = get_object_or_404(Product, pk=pk)
    return render(request, 'products/product_detail.html', {'product': product})

cart/views.py

from django.shortcuts import render, redirect
from .models import Cart, CartItem
from products.models import Product

def cart_detail(request):
    cart, created = Cart.objects.get_or_create(pk=request.session.get('cart_id'))
    if created:
        request.session['cart_id'] = cart.id
    return render(request, 'cart/cart_detail.html', {'cart': cart})

def add_to_cart(request, product_id):
    product = get_object_or_404(Product, id=product_id)
    cart, created = Cart.objects.get_or_create(pk=request.session.get('cart_id'))
    if created:
        request.session['cart_id'] = cart.id
    cart_item, created = CartItem.objects.get_or_create(cart=cart, product=product)
    if not created:
        cart_item.quantity += 1
        cart_item.save()
    return redirect('cart_detail')


products/templates/products/product_list.html

{% extends "base.html" %}

{% block content %}
<h1>Products</h1>
<ul>
    {% for product in products %}
    <li>
        <a href="{% url 'product_detail' product.pk %}">{{ product.name }}</a>
        - {{ product.price }}
    </li>
    {% endfor %}
</ul>
{% endblock %}

products/templates/products/product_detail.html

{% extends "base.html" %}

{% block content %}
<h1>{{ product.name }}</h1>
<p>{{ product.description }}</p>
<p>{{ product.price }}</p>
<img src="{{ product.image.url }}" alt="{{ product.name }}">
<form action="{% url 'add_to_cart' product.id %}" method="post">
    {% csrf_token %}
    <button type="submit">Add to Cart</button>
</form>
{% endblock %}


cart/templates/cart/cart_detail.html

{% extends "base.html" %}

{% block content %}
<h1>Shopping Cart</h1>
<ul>
    {% for item in cart.cartitem_set.all %}
    <li>{{ item.product.name }} - {{ item.quantity }}</li>
    {% endfor %}
</ul>
{% endblock %}


### 6. URLs Configuration
Set up the URLs in each app and the main urls.py.

ecommerce/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('products/', include('products.urls')),
    path('cart/', include('cart.urls')),
    path('users/', include('users.urls')),
]

products/urls.py

from django.urls import path
from .views import product_list, product_detail

urlpatterns = [
    path('', product_list, name='product_list'),
    path('<int:pk>/', product_detail, name='product_detail'),
]

cart/urls.py

from django.urls import path
from .views import cart_detail, add_to_cart

urlpatterns = [
    path('', cart_detail, name='cart_detail'),
    path('add/<int:product_id>/', add_to_cart, name='add_to_cart'),
]


users/urls.py

from django.urls import path
from .views import signup_view, login_view, logout_view

urlpatterns = [
    path('signup/', signup_view, name='signup'),
    path('login/', login_view, name='login'),
    path('logout/', logout_view, name='logout'),
]


### 7. Static Files and Templates
Place your CSS and JS files in the static directory and reference them in your templates. Create a base.html template that includes the common structure (header, footer, etc.).

templates/base.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>E-commerce Website</title>
    <link rel="stylesheet" href="{% static 'css/styles.css' %}">
</head>
<body>
    <header>
        <nav>
            <a href="{% url 'product_list' %}">Home</a>
            <a href="{% url 'cart_detail' %}">Cart</a>
            {% if user.is_authenticated %}
                <a href="{% url 'logout' %}">Logout</a>
            {% else %}
                <a href="{% url 'login' %}">Login</a>
                <a href="{% url 'signup' %}">Sign Up</a>
            {% endif %}
        </nav>
    </header>
    <main>
        {% block content %}
        {% endblock %}
    </main>
    <script src="{% static 'js/scripts.js' %}"></script>
</body>
</html>

<h1>Conclusion</h1>
This project structure provides a solid foundation for a beginner-friendly e-commerce website using Django and JavaScript. You can expand the project by adding more features such as product categories, user profiles, order history, and real payment integration. This will help beginners get hands-on experience with web development concepts and best practices.



