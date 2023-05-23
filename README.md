django-admin startproject agri_ecom
cd agri_ecom
python manage.py startapp store
from django.db import models
from django.contrib.auth.models import User


class Product(models.Model):
    name = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    price = models.DecimalField(max_digits=10, decimal_places=2)

    def __str__(self):
        return self.name


class Supplier(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    address = models.CharField(max_length=255)
    phone_number = models.CharField(max_length=20)
    products = models.ManyToManyField(Product, through='Supply')

    def __str__(self):
        return self.user.username


class Supply(models.Model):
    supplier = models.ForeignKey(Supplier, on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    quantity = models.PositiveIntegerField()

    def __str__(self):
        return f"{self.supplier.user.username} - {self.product.name}"


class Order(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    quantity = models.PositiveIntegerField()
    supplier = models.ForeignKey(Supplier, on_delete=models.CASCADE)
    date = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.user.username} - {self.product.name}"
# ...

INSTALLED_APPS = [
    # ...
    'store',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

# ...

# Add the following at the end of the file

# Authentication Configuration
LOGIN_REDIRECT_URL = '/'
LOGOUT_REDIRECT_URL = '/'

# ...
from django import forms
from django.contrib.auth.forms import UserCreationForm, AuthenticationForm
from .models import Order


class RegistrationForm(UserCreationForm):
    address = forms.CharField(max_length=255)
    phone_number = forms.CharField(max_length=20)

    class Meta:
        model = User
        fields = ['username', 'password1', 'password2', 'address', 'phone_number']


class LoginForm(AuthenticationForm):
    class Meta:
        model = User
        fields = ['username', 'password']


class OrderForm(forms.ModelForm):
    class Meta:
        model = Order
        fields = ['product', 'quantity']
        widgets = {
            'product': forms.Select(attrs={'class': 'form-control'}),
            'quantity': forms.NumberInput(attrs={'class': 'form-control'}),
        }
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
from .models import Product, Supplier, Order
from .forms import RegistrationForm, LoginForm, OrderForm


def index(request):
    products = Product.objects.all()
    context = {'products': products}
    return render(request, 'store/index.html', context)


def suppliers(request):
    suppliers = Supplier.objects.all()
    context = {'suppliers': suppliers}
    return render(request, 'store/suppliers.html', context)


def product_detail(request, product_id):
    product = Product.objects.get(id=product_id)
    supplies = product.supply_set.all()
    context = {'product': product, 'supplies': supplies}
    return render(request, 'store/product_detail.html', context)


@login_required
def order(request):
    if request.method == 'POST':
        form = OrderForm(request.POST)
        if form.is_valid():
            order = form.save(commit=False)
            order.user = request.user
            order.supplier = Supplier.objects.get(user=order.product.supplier.user)
            order.save()
            return redirect('orders')
    else:
        form = OrderForm()
    context = {'form': form}
    return render(request, 'store/order.html', context)


@login_required
def orders(request):
    user = request.user
    orders = Order.objects.filter(user=user)
    context = {'orders': orders}
    return render(request, 'store/orders.html', context)


def register(request):
    if request.method == 'POST':
        form = RegistrationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('login')
    else:
        form = RegistrationForm()
    context = {'form': form}
    return render(request, 'store/register.html', context)


def login(request):
    if request.method == 'POST':
        form = LoginForm(data=request.POST)
        if form.is_valid():
            user = form.get_user()
            auth_login(request, user)
            return redirect('index')
    else:
        form = LoginForm()
    context = {'form': form}
    return render(request, 'store/login.html', context)


def logout(request):
    auth_logout(request)
    return redirect('index')
