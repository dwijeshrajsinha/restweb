# Ex.07 Restaurant Website
## Date:

## AIM:
To develop a static Restaurant website to display the food items and services provided by them.

## DESIGN STEPS:

### Step 1:
Requirement collection.

### Step 2:
Creating the layout using HTML and CSS.

### Step 3:
Updating the sample content.

### Step 4:
Choose the appropriate style and color scheme.

### Step 5:
Validate the layout in various browsers.

### Step 6:
Validate the HTML code.

### Step 7:
Publish the website in the given URL.

## PROGRAM:
~~~
base.html
<!DOCTYPE html>
<html>
<head>
<title>Authentic Flavours</title>
<style>
body{margin:0;font-family:Arial;background:#f4f4f4}
header{background:#7a1c1c;color:white;padding:15px;text-align:center}
.container{padding:30px}
.center-box{
    width:350px;margin:80px auto;background:white;
    padding:25px;border-radius:8px;
    box-shadow:0 5px 20px rgba(0,0,0,.2)
}
.grid{
    display:grid;
    grid-template-columns:repeat(auto-fill,minmax(220px,1fr));
    gap:20px
}
.card{
    background:white;border-radius:8px;
    box-shadow:0 3px 10px rgba(0,0,0,.1);
    overflow:hidden
}
.card img{
    width:100%;height:140px;object-fit:cover
}
.card div{padding:10px}
button{
    background:#7a1c1c;color:white;
    border:none;padding:8px;width:100%
}
input{width:100%;padding:8px;margin-bottom:10px}
.cart{margin-top:30px;background:white;padding:15px;border-radius:6px}
</style>
</head>
<body>

<header><h2>Authentic Flavours</h2></header>
<div class="container">{% block content %}{% endblock %}</div>

</body>
</html>

home.html

{% extends "base.html" %}
{% load static %}
{% block content %}

<div class="grid">
{% for food in foods %}
<div class="card">
    <img src="{% static food.img %}">
    <div>
        <h4>{{ food.name }}</h4>
        <p>₹{{ food.price }}</p>
        <a href="/add/{{ food.id }}/">
            <button>Add to Cart</button>
        </a>
    </div>
</div>
{% endfor %}
</div>

<div class="cart">
<h3>Cart</h3>
{% if cart %}
    <ul>
    {% for i in cart %}
        <li>{{ i.name }} - ₹{{ i.price }}</li>
    {% endfor %}
    </ul>
    <a href="/order/"><button>Order Now</button></a>
{% else %}
    <p>Cart is empty</p>
{% endif %}
</div>

{% endblock %}

login.html

{% extends "base.html" %}
{% block content %}
<div class="center-box">
<h3>Login</h3>
<form method="post">{% csrf_token %}
<input name="username" placeholder="Username">
<input name="password" type="password" placeholder="Password">
<button>Login</button>
</form>
<p style="color:red">{{ error }}</p>
<p>New user? <a href="/signup/">Signup</a></p>
</div>
{% endblock %}

signup.html

{% extends "base.html" %}
{% block content %}
<div class="center-box">
<h3>Signup</h3>
<form method="post">{% csrf_token %}
<input name="username" placeholder="Username">
<input name="password" type="password" placeholder="Password">
<input name="phone" placeholder="Phone">
<button>Create Account</button>
</form>
</div>
{% endblock %}

bill.html

{% extends "base.html" %}
{% block content %}
<h2>Order Bill</h2>
<table border="1" cellpadding="8">
<tr><th>Food</th><th>Price</th></tr>
{% for i in cart %}
<tr><td>{{ i.name }}</td><td>₹{{ i.price }}</td></tr>
{% endfor %}
</table>
<h3>Total: ₹{{ total }}</h3>
<a href="/home/"><button>Back</button></a>
{% endblock %}

views.py

# restaurant_app/views.py

from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.models import User
from django.contrib.auth.decorators import login_required
from .models import UserDetails, OrderDetails


FOODS = [
    {"id":1,"name":"Idli","price":40,"img":"images/idly.jpg"},
    {"id":2,"name":"Dosa","price":60,"img":"images/dosa.jpg"},
    {"id":3,"name":"Pongal","price":50,"img":"images/pongal.jpg"},
    {"id":4,"name":"Vada","price":30,"img":"images/vada.jpg"},
    {"id":5,"name":"Poori","price":45,"img":"images/poori.jpg"},
    {"id":6,"name":"Parotta","price":50,"img":"images/parota.jpg"},
    {"id":7,"name":"Pizza","price":150,"img":"images/pizza.jpg"},
    {"id":8,"name":"Burger","price":120,"img":"images/burger.png"},
]


def signup_view(request):
    if request.method == "POST":
        user = User.objects.create_user(
            username=request.POST["username"],
            password=request.POST["password"]
        )
        UserDetails.objects.create(user=user, phone=request.POST["phone"])
        login(request, user)   # 🔥 auto-login
        return redirect("home")
    return render(request, "signup.html")

def login_view(request):
    if request.method == "POST":
        user = authenticate(
            username=request.POST["username"],
            password=request.POST["password"]
        )
        if user:
            login(request, user)
            return redirect("home")
        return render(request, "login.html", {"error":"Invalid credentials"})
    return render(request, "login.html")

@login_required
def home(request):
    cart = request.session.get("cart", [])
    return render(request, "home.html", {"foods": FOODS, "cart": cart})

@login_required
def add_to_cart(request, food_id):
    cart = request.session.get("cart", [])
    for food in FOODS:
        if food["id"] == food_id:
            cart.append(food)
            break
    request.session["cart"] = cart
    return redirect("home")

@login_required
def place_order(request):
    cart = request.session.get("cart", [])
    total = 0
    for item in cart:
        OrderDetails.objects.create(
            user=request.user,
            food_name=item["name"],
            price=item["price"]
        )
        total += item["price"]
    request.session["cart"] = []
    return render(request, "bill.html", {"cart": cart, "total": total})

def logout_view(request):
    logout(request)
    return redirect("login")
~~~

## OUTPUT:
<img width="1146" height="559" alt="image" src="https://github.com/user-attachments/assets/9d231a5d-e8a6-4ed1-bae8-ec73f694cdf3" />
<img width="1155" height="402" alt="image" src="https://github.com/user-attachments/assets/e8bc8ba7-5adb-43ca-b6c4-1d945fb02881" />
<img width="1144" height="366" alt="image" src="https://github.com/user-attachments/assets/2a3f1612-64c3-453f-a680-c7239b347a38" />
<img width="1151" height="551" alt="image" src="https://github.com/user-attachments/assets/f1c3a850-134d-4bed-802b-228894f5d02c" />
<img width="1142" height="559" alt="image" src="https://github.com/user-attachments/assets/745ee164-96c3-475c-a561-cd96010496e3" />
<img width="1141" height="540" alt="image" src="https://github.com/user-attachments/assets/f63499f1-d5aa-4ab0-b25c-466005ee3a50" />


## RESULT:
The program for designing software company website using HTML and CSS is completed successfully.
