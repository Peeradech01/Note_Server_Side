# WEEK 4 Exercises - Making Queries

![ERD-E-COMMERCE](https://github.com/it-web-pro/django-week4/blob/main/images/WEEK3-ERD(e-commerce).png?raw=true)

## Instruction

1. สร้าง `virtual environment`
2. ติดตั้ง `django` และ `psycopg2` libraries
3. สร้างโปรเจคใหม่ใหม่ชื่อ`myshop`
4. จากนั้นให้ทำการ startapp ใหม่ชื่อ `shop`
5. สร้าง database ชื่อ `shop` ใน Postgres DB
6. ทำการเพิ่ม code ด้านล่างนี้ในไฟล์ `shop/models.py`
7. เพิ่ม **'shop'** ใน `settings.py`
8. ทำการ `makemigrations` และ `migrate`

```python
from django.db import models

# Create your models here.


class Customer(models.Model):
    first_name = models.CharField(max_length=150)
    last_name = models.CharField(max_length=200)
    email = models.CharField(max_length=150)
    address = models.JSONField(null=True)

class ProductCategory(models.Model):
    name = models.CharField(max_length=150)

class Product(models.Model):
    name = models.CharField(max_length=150)
    description = models.TextField(null=True, blank=True)
    remaining_amount = models.PositiveIntegerField(default=0)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    categories = models.ManyToManyField(ProductCategory)

class Cart(models.Model):
    customer = models.ForeignKey(Customer, on_delete=models.CASCADE)
    create_date = models.DateTimeField()
    expired_in = models.PositiveIntegerField(default=60)
    
class CartItem(models.Model):
    cart = models.ForeignKey(Cart, on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    amount = models.PositiveIntegerField(default=1)
    
class Order(models.Model):
    customer = models.ForeignKey(Customer, on_delete=models.CASCADE)
    order_date = models.DateField()
    remark = models.TextField(null=True, blank=True)

class OrderItem(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    amount = models.PositiveIntegerField(default=1)
    
class Payment(models.Model):
    order = models.OneToOneField(Order, on_delete=models.PROTECT)
    payment_date = models.DateField()
    remark = models.TextField(null=True, blank=True)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    discount = models.DecimalField(max_digits=10, decimal_places=2, default=0)

class PaymentItem(models.Model):
    payment = models.ForeignKey(Payment, on_delete=models.CASCADE)
    order_item = models.OneToOneField(OrderItem, on_delete=models.CASCADE)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    discount = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    
class PaymentMethod(models.Model):
    class MethodChoices(models.Choices):
        QR = "QR"
        CREDIT = "CREDIT"
    
    payment = models.ForeignKey(Payment, on_delete=models.CASCADE)
    method = models.CharField(max_length=15, choices=MethodChoices.choices)
    price = models.DecimalField(max_digits=10, decimal_places=2)
```

**จากนั้นให้ทำการ migrate และ run คำสั่งในไฟล์ `shop.sql` ใน PgAdmin4**


```python
import os
os.environ['DJANGO_ALLOW_ASYNC_UNSAFE'] = "true"

# import modules
from shop.models import *
from django.db.models import F, Q, Count, Value, Avg, Max, Min, Sum
from django.db.models.functions import Length, Upper, Concat
from django.db.models.lookups import GreaterThan
import json

```

### 1. ให้นักศึกษา Query ค้นหาข้อมูลมาแสดงให้ถูกต้องตามโจทย์

1.1 query หาข้อมูล `Order` ทั้งหมดที่เกิดขึ้นในเดือน `พฤษภาคม` มาแสดงผล 10 รายการแรก และแสดงผลดังตัวอย่าง (0.5 คะแนน)

```txt
ORDER ID:22, DATE: 2024-05-01, PRICE: 4890.00
ORDER ID:23, DATE: 2024-05-01, PRICE: 2540.00
ORDER ID:24, DATE: 2024-05-01, PRICE: 1720.00
ORDER ID:25, DATE: 2024-05-02, PRICE: 322499.00
ORDER ID:26, DATE: 2024-05-02, PRICE: 3399.00
ORDER ID:27, DATE: 2024-05-02, PRICE: 1190.00
ORDER ID:28, DATE: 2024-05-03, PRICE: 9499.00
ORDER ID:29, DATE: 2024-05-03, PRICE: 700.00
ORDER ID:30, DATE: 2024-05-03, PRICE: 1690.00
ORDER ID:31, DATE: 2024-05-04, PRICE: 3280.00
```


```python
q1 = Order.objects.filter(order_date__month=5)[:10]
for i in q1:
    print(f"ORDER ID:{i.id}, DATE: {i.order_date}, PRICE: {i.payment.price}")
```

    ORDER ID:22, DATE: 2024-05-01, PRICE: 4890.00
    ORDER ID:23, DATE: 2024-05-01, PRICE: 2540.00
    ORDER ID:24, DATE: 2024-05-01, PRICE: 1720.00
    ORDER ID:25, DATE: 2024-05-02, PRICE: 322499.00
    ORDER ID:26, DATE: 2024-05-02, PRICE: 3399.00
    ORDER ID:27, DATE: 2024-05-02, PRICE: 1190.00
    ORDER ID:28, DATE: 2024-05-03, PRICE: 9499.00
    ORDER ID:29, DATE: 2024-05-03, PRICE: 700.00
    ORDER ID:30, DATE: 2024-05-03, PRICE: 1690.00
    ORDER ID:31, DATE: 2024-05-04, PRICE: 3280.00
    

1.2 query หาข้อมูล `Product` ที่มีคำลงท้ายว่า `features.` ในรายละเอียดสินค้า และแสดงผลดังตัวอย่าง (0.5 คะแนน)

```txt
PRODUCT ID: 1, DESCRIPTION: A sleek and powerful smartphone with advanced features.
PRODUCT ID: 7, DESCRIPTION: High-resolution digital camera with advanced photography features.
PRODUCT ID: 10, DESCRIPTION: A stylish smartwatch with health monitoring and notification features.
PRODUCT ID: 14, DESCRIPTION: Split air conditioner with remote control and energy-saving features.
PRODUCT ID: 45, DESCRIPTION: Customizable racing track set with loop and jump features.
```


```python
q2 = Product.objects.filter(description__endswith="features.")
for i in q2:
    print(f"PRODUCT ID: {i.id}, DESCRIPTION: {i.description}")
```

    PRODUCT ID: 1, DESCRIPTION: A sleek and powerful smartphone with advanced features.
    PRODUCT ID: 7, DESCRIPTION: High-resolution digital camera with advanced photography features.
    PRODUCT ID: 10, DESCRIPTION: A stylish smartwatch with health monitoring and notification features.
    PRODUCT ID: 14, DESCRIPTION: Split air conditioner with remote control and energy-saving features.
    PRODUCT ID: 45, DESCRIPTION: Customizable racing track set with loop and jump features.
    

1.3 query หาข้อมูล `Product` ที่มีราคาสินค้าตั้งแต่ `5000.00` ขึ้นไป และอยู่ในหมวดหมู่ `Information Technology` และแสดงผลดังตัวอย่าง (0.5 คะแนน)

```txt
 # ตัวอย่างบางส่วน 
PRODUCT ID: 1, NAME: Smartphone, PRICE: 5900.00
```


```python
q3 = Product.objects.filter(price__gt=5000, categories__name="Information Technology")

for i in q3:
    print(f"PRODUCT ID: {i.id}, NAME: {i.name}, PRICE: {i.price}")
```

    PRODUCT ID: 1, NAME: Smartphone, PRICE: 5900.00
    PRODUCT ID: 2, NAME: Laptop, PRICE: 25999.00
    PRODUCT ID: 3, NAME: Smart TV, PRICE: 8900.00
    PRODUCT ID: 5, NAME: Tablet, PRICE: 12900.00
    PRODUCT ID: 7, NAME: Digital Camera, PRICE: 32000.00
    PRODUCT ID: 69, NAME: Notebook HP Pavilion Silver, PRICE: 20000.00
    

1.4 query หาข้อมูล `Product` ที่มีราคาสินค้าน้อยกว่า `200.00` และมากกว่า `100.00` และแสดงผลดังตัวอย่าง (0.5 คะแนน)

```txt
PRODUCT ID: 28, NAME: Women's Sweater, PRICE: 190.00
PRODUCT ID: 66, NAME: Salvage the Bones, PRICE: 129.00
```


```python
q4 = Product.objects.filter(price__lt=200, price__gt=100)
for i in q4:
    print(f"PRODUCT ID: {i.id}, NAME: {i.name}, PRICE: {i.price}")
```

    PRODUCT ID: 28, NAME: Women's Sweater, PRICE: 190.00
    

### 2. เพิ่ม ลบ แก้ไข สินค้า

2.1 ให้เพิ่มสินค้าใหม่จำนวน 3 รายการ (0.5 คะแนน)

```txt
สินค้าที่ 1
ชื่อ: Philosopher's Stone (1997)
หมวดหมู่สินค้า: Books and Media
จำนวนคงเหลือ: 20
รายละเอียดซ: By J. K. Rowling.
ราคา: 790

สินค้าที่ 2
ชื่อ: Me Before You
หมวดหมู่สินค้า: Books and Media
จำนวนคงเหลือ: 40
รายละเอียดซ: A romance novel written by Jojo
ราคา: 390

สินค้าที่ 3
ชื่อ: Notebook HP Pavilion Silver
หมวดหมู่สินค้า: Information Technology และ Electronics
จำนวนคงเหลือ: 10
รายละเอียดซ: Display Screen. 16.0
ราคา: 20000
```


```python
cat1 = ProductCategory.objects.get(name="Books and Media")
cat2 = ProductCategory.objects.get(name="Information Technology")
cat3 = ProductCategory.objects.get(name="Electronics")

pro1 = Product.objects.create(
    name="Philosopher's Stone (1997)", 
    remaining_amount=20, 
    description="By J. K. Rowling.", 
    price=790
)
pro2 = Product.objects.create(
    name="Me Before You", 
    remaining_amount=40, 
    description="A romance novel written by Jojo", 
    price=390
)
pro3 = Product.objects.create(
    name="Notebook HP Pavilion Silver", 
    remaining_amount=10, 
    description="Display Screen. 16.0", 
    price=20000
)

pro1.categories.add(cat1)
pro2.categories.add(cat1)
pro3.categories.add(cat2, cat3)

```

2.2 แก้ไขชื่อสินค้า จาก `Philosopher's Stone (1997)` เป็น `Half-Blood Prince (2005)` (0.5 คะแนน)


```python
getp = Product.objects.get(name="Philosopher's Stone (1997)")
getp.name = "Half-Blood Prince (2005)"
getp.save() #อย่าลืม save()
```

2.3 แก้ไขชื่อหมวดหมู่สินค้า จาก `Books and Media` เป็น `Books` (0.5 คะแนน)


```python
getcat = ProductCategory.objects.get(name="Books and Media")
getcat.name = "Books"
getcat.save() #อย่าลืม save()
```

2.4 ลบสินค้าทุกตัวที่อยู่ในหมวดหมู่ `Books` (0.5 คะแนน)


```python
delpro = Product.objects.filter(categories__name="Books")
delpro.delete()
```




    (4, {'shop.Product_categories': 2, 'shop.Product': 2})


