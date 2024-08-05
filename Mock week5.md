# **หมายเหตุ: รบกวนรี database ถ้าอยากได้ข้อมูลตรง**


```python
import os
os.environ['DJANGO_ALLOW_ASYNC_UNSAFE'] = "true"


from shop.models import *
from django.db.models import F, Q, Count, Value, Avg, Max, Min, Sum
from django.db.models.functions import Length, Upper, Concat
from django.db.models.lookups import GreaterThan
import json
```

### 1. annotate()
1.1 ให้ค้นหาข้อมูล `Product` โดยให้เพิ่ม field ราคารวมทั้งหมดของสินค้า โดยกำหนดให้ชื่อ field ว่า "total_price" โดยใช้แสดงข้อมูล 5 ตัวแรกเรียงตาม "total_price" จากมากไปน้อย

**หมายเหตุ: จะต้องใช้ annotate() ให้เอา `Product.price` * `Product.remaining_amount`**

ตัวอย่าง Output

```
ID: 61, PRICE: 320000.00, AMOUNT: 10, TOTAL PRICE: 3200000.00
ID: 65, PRICE: 1200000.00, AMOUNT: 2, TOTAL PRICE: 2400000.00
ID: 62, PRICE: 70000.00, AMOUNT: 15, TOTAL PRICE: 1050000.00
ID: 63, PRICE: 59000.00, AMOUNT: 12, TOTAL PRICE: 708000.00
ID: 14, PRICE: 18900.00, AMOUNT: 30, TOTAL PRICE: 567000.00
```


```python
q1 = Product.objects.annotate(total_price=F("price")*F("remaining_amount")).order_by("-total_price")[:5]
for i in q1:
    print(f"ID: {i.id}, PRICE: {i.price}, AMOUNT: {i.remaining_amount}, TOTAL PRICE: {i.total_price}")
```

    ID: 61, PRICE: 320000.00, AMOUNT: 10, TOTAL PRICE: 3200000.00
    ID: 65, PRICE: 1200000.00, AMOUNT: 2, TOTAL PRICE: 2400000.00
    ID: 62, PRICE: 70000.00, AMOUNT: 15, TOTAL PRICE: 1050000.00
    ID: 63, PRICE: 59000.00, AMOUNT: 12, TOTAL PRICE: 708000.00
    ID: 14, PRICE: 18900.00, AMOUNT: 30, TOTAL PRICE: 567000.00
    

1.2 ต่อเนื่องจากข้อ 1.1 ให้ filter เฉพาะข้อมูล Product ที่มี "total_price" มากกว่า 1,000,000 และ "remaining_amount" น้อยกว่า 10 ชิ้น

ตัวอย่าง Output

```
ID: 65, PRICE: 1200000.00, AMOUNT: 2, TOTAL PRICE: 2400000.00
```


```python
q2 = Product.objects.annotate(total_price=F("price")*F("remaining_amount")).filter(total_price__gt=1000000, remaining_amount__lt=10).order_by("-total_price")[:5]
for i in q2:
    print(f"ID: {i.id}, PRICE: {i.price}, AMOUNT: {i.remaining_amount}, TOTAL PRICE: {i.total_price}")
```

    ID: 65, PRICE: 1200000.00, AMOUNT: 2, TOTAL PRICE: 2400000.00
    

1.3 ให้นักศึกษาเรียงลำดับข้อมูลลูกค้า (`Customer`) แสดงเพียงแค่ field `id`, `email` และ `full_name` โดยเรียงลำดับข้อมูลตาม (`id`) จาก `น้อยไปมาก` โดยแสดง 10 คนแรก 

**Hint:** Field `full_name` นั้นจะต้องถูก annotate ขึ้นมาโดยการนำ `first_name` มาต่อกับ `last_name` โดยใช้ `Concat(*expressions, **extra)` 

```python
>>> Product.objects.filter(description__icontains="advance").values()
<QuerySet [{'id': 1, 'name': 'Smartphone', 'description': 'A sleek and powerful smartphone with advanced features.', 'remaining_amount': 24, 'price': Decimal('5900.00')}, {'id': 7, 'name': 'Digital Camera', 'description': 'High-resolution digital camera with advanced photography features.', 'remaining_amount': 4, 'price': Decimal('32000.00')}]>
```

**Hint:** แปลง object เป็น dict ใช้ `values()`

**Hint:** อยาก print dictionary สวยๆ ใช้ `json.dumps`

```python
print(json.dumps(dictionary, indent=4, sort_keys=False))
```

ตัวอย่าง Output 

```
[
    {
        "id": 1,
        "email": "panita.hong@gmail.com",
        "full_name": "Panita Hongsakulpan"
    },
    {
        "id": 2,
        "email": "pakin.jan@gmail.com",
        "full_name": "Pakin Janpen"
    },
    {
        "id": 3,
        "email": "jenjira.su@gmail.com",
        "full_name": "Jenjira Sukanansarn"
    },
    {
        "id": 4,
        "email": "dejwit.tt@gmail.com",
        "full_name": "Dejwit Tangjareonsakul"
    },
    {
        "id": 5,
        "email": "pong.23@gmail.com",
        "full_name": "Pong Sawadiwong"
    },
    {
        "id": 6,
        "email": "thiti.za@gmail.com",
        "full_name": "Thitirat Sukkesorn"
    },
    {
        "id": 7,
        "email": "prontipa.za@gmail.com",
        "full_name": "Porntipa Pasakul"
    },
    {
        "id": 8,
        "email": "warit.za@gmail.com",
        "full_name": "Warit Pititat"
    },
    {
        "id": 9,
        "email": "sira.za@gmail.com",
        "full_name": "Sira Pititat"
    },
    {
        "id": 10,
        "email": "wanaporn.over@gmail.com",
        "full_name": "Wanaporn Klabpetch"
    }
]
```


```python
q3 = Customer.objects.annotate(full_name=Concat(F("first_name"), Value(" "), F("last_name"))).order_by("id")[:10]
print(json.dumps(list(q3.values("id", "email", "full_name")), indent=4, sort_keys=False))
```

    [
        {
            "id": 1,
            "email": "panita.hong@gmail.com",
            "full_name": "Panita Hongsakulpan"
        },
        {
            "id": 2,
            "email": "pakin.jan@gmail.com",
            "full_name": "Pakin Janpen"
        },
        {
            "id": 3,
            "email": "jenjira.su@gmail.com",
            "full_name": "Jenjira Sukanansarn"
        },
        {
            "id": 4,
            "email": "dejwit.tt@gmail.com",
            "full_name": "Dejwit Tangjareonsakul"
        },
        {
            "id": 5,
            "email": "pong.23@gmail.com",
            "full_name": "Pong Sawadiwong"
        },
        {
            "id": 6,
            "email": "thiti.za@gmail.com",
            "full_name": "Thitirat Sukkesorn"
        },
        {
            "id": 7,
            "email": "prontipa.za@gmail.com",
            "full_name": "Porntipa Pasakul"
        },
        {
            "id": 8,
            "email": "warit.za@gmail.com",
            "full_name": "Warit Pititat"
        },
        {
            "id": 9,
            "email": "sira.za@gmail.com",
            "full_name": "Sira Pititat"
        },
        {
            "id": 10,
            "email": "wanaporn.over@gmail.com",
            "full_name": "Wanaporn Klabpetch"
        }
    ]
    

### 2. aggregation
2.1 ให้นักศึกษาหาค่าเฉลี่ยของราคาสินค้า (`Product.price`) ที่มีจำนวนคงเหลือ (`Product.remaining_amount`) ตั้งแต่ 100 ชิ้นขึ้นไป 

ตัวอย่าง Output 

``` PYTHON
Average Price: 664.4545454545454545
```



```python
q4 = Product.objects.filter(remaining_amount__gt=100).aggregate(avg=Avg("price"))
print(f"Average Price: {q4['avg']}")
```

    Average Price: 664.4545454545454545
    

2.2 ให้นักศึกษาหาราคาของสินค้า (`Product.price`) ที่มากที่สุด และ ราคาของสินค้าที่น้อยที่สุด ของสินค้าที่หมด (`Product.remaining_amount`) 

ตัวอย่าง Output 

``` PYTHON
Max Price: 990.00
Min Price: 129.00
```


```python
q5 = Product.objects.filter(remaining_amount=0).aggregate(max_price=Max("price"), min_price=Min("price"))
print(f"Max Price: {q5["max_price"]}")
print(f"Max Price: {q5["min_price"]}")
```

    Max Price: 990.00
    Max Price: 129.00
    

2.3 จงหาผลรวมราคา (`CartItem.product.price`) ที่อยู่ในตระกร้าสินค้าของวันที่ 1 (ดูจาก `Cart.create_date`)

**หมายเหตุ: ผลรวมราคา คือ  sum ของ `CartItem.product.price` * `CartItem.amount`**

ตัวอย่าง Output 

``` PYTHON
Sum Price: 830237.00
```


```python
q6 = CartItem.objects.filter(cart__create_date__day=1).aggregate(sum_price=Sum(F("product__price")*F("amount")))
print(f"Sum Price: {q7['sum_price']}")


print("------------------------------")

sum_price = CartItem.objects.filter(cart__create_date__day=1).annotate(total_price=F('product__price') * F('amount')).aggregate(sum=Sum('total_price'))

print(f"Sum Price: {sum_price['sum']}")
```

    Sum Price: 830237.00
    ------------------------------
    Sum Price: 830237.00
    

2.4 นับจำนวนสินค้าที่อยู่ประเภท Clothing and Apparel, Furniture และ ราคาของสินค้าอยู่ในช่วง 1,000.00 - 10,000.00

ตัวอย่าง Output 

``` PYTHON
PRODUCT CATEGORY NAME: Clothing and Apparel PRODUCT COUNT: 1
PRODUCT CATEGORY NAME: Furniture PRODUCT COUNT: 5
```


```python
q7 = Product.objects.filter(categories__name__in=["Clothing and Apparel","Furniture"], price__range=(1000, 10000)).values("categories__name").annotate(count=Count("id"))
for i in q7:
    print(f"PRODUCT CATEGORY NAME: {i['categories__name']} PRODUCT COUNT: {i['count']}")
```

    PRODUCT CATEGORY NAME: Clothing and Apparel PRODUCT COUNT: 1
    PRODUCT CATEGORY NAME: Furniture PRODUCT COUNT: 5
    

### 3. many-to-many
3.1 ให้ค้นหาข้อมูลสินค้า (Product) ที่อยู่ในประเภทสินค้า "Information Technology" 10 รายการแรก (เรียงลำดับด้วย `Product.id`) และแสดงชื่อประเภทสินค้า (ProductCategory)

ตัวอย่าง Output บางส่วน

``` PYTHON
Product ID: 1, Product Name: Smartphone, Categories Name: Information Technology, Electronics, Price: 5900.00
Product ID: 2, Product Name: Laptop, Categories Name: Information Technology, Electronics, Price: 25999.00
Product ID: 3, Product Name: Smart TV, Categories Name: Information Technology, Electronics, Price: 8900.00
Product ID: 4, Product Name: Bluetooth Earphones, Categories Name: Information Technology, Electronics, Price: 350.00
Product ID: 5, Product Name: Tablet, Categories Name: Information Technology, Electronics, Price: 12900.00
```


```python

```

3.2 ให้ทำตามขั้นตอนดังนี้ 

    1. เปลี่ยนชื่อประเภทสินค้า `Home Appliances` เป็น `Home Decor` 
    2. เปลี่ยนประเภทสินค้า `Furniture` ให้เป็น `Home Decor` แทน
    3. ค้นหาว่าสินค้าที่มีประเภทสินค้าเป็น `Home Decor` ทั้งหมดมีจำนวนเท่าไหร่


```python

```


```python

```


```python

```
