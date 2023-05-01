# -a-Farm-to-market-e-commerce-platform
from django.db import models
from django.contrib.auth.models import User

class Product(models.Model):
    name = models.CharField(max_length=255)
    description = models.TextField()
    quantity = models.IntegerField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    farmer = models.ForeignKey(User, on_delete=models.CASCADE)

class Order(models.Model):
    buyer = models.ForeignKey(User, on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    quantity = models.IntegerField()
    total_price = models.DecimalField(max_digits=10, decimal_places=2)
    date_ordered = models.DateTimeField(auto_now_add=True)

class Review(models.Model):
    buyer = models.ForeignKey(User, on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    rating = models.IntegerField()
    comment = models.TextField()
    date_added = models.DateTimeField(auto_now_add=True)

class Delivery(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE)
    delivery_address = models.TextField()
    delivery_date = models.DateTimeField()

class Payment(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE)
    payment_date = models.DateTimeField()
    payment_amount = models.DecimalField(max_digits=10, decimal_places=2)
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
from django.contrib import messages
from .models import Product, Order, Review, Delivery, Payment
from .forms import ProductForm, OrderForm, ReviewForm, DeliveryForm, PaymentForm

@login_required
def product_list(request):
    products = Product.objects.all()
    context = {'products': products}
    return render(request, 'product_list.html', context)

@login_required
def product_detail(request, product_id):
    product = Product.objects.get(id=product_id)
    reviews = Review.objects.filter(product=product)
    context = {'product': product, 'reviews': reviews}
    return render(request, 'product_detail.html', context)

@login_required
def product_create(request):
    if request.method == 'POST':
        form = ProductForm(request.POST)
        if form.is_valid():
            form.save()
            messages.success(request, 'Product created successfully')
            return redirect('product_list')
    else:
        form = ProductForm()
    context = {'form': form}
    return render(request, 'product_create.html', context)

@login_required
def product_update(request, product_id):
    product = Product.objects.get(id=product_id)
    if request.method == 'POST':
        form = ProductForm(request.POST, instance=product)
        if form.is_valid():
            form.save()
            messages.success(request, 'Product updated successfully')
            return redirect('product_list')
    else:
        form = ProductForm(instance=product)
    context = {'form': form, 'product': product}
    return render(request, 'product_update.html', context)

login_required
def order_create(request, product_id):
    product = Product.objects.get(id=product_id)
    if request.method == 'POST':
        form = OrderForm(request.POST)
        if form.is_valid():
            order = form.save(commit=False)
            order.buyer = request.user
            order.product = product
            order.total_price = order.quantity * product.price
            order.save()
            messages.success(request, 'Order created successfully')
            return redirect('order_list')
    else:
        form = OrderForm()
    context = {'form': form, 'product': product}
    return render(request, 'order_create.html', context)



