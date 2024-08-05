![erd](images/ERD.png)

## Instruction

1. ใช้ project myshop เดิมจากของ WEEK 4 มาทำต่อได้เลยครับ (ใครที่หา project ของ WEEK 4 ไม่เจอแล้ว ให้กลับไปทำตามขั้ยตอนใน week4_exercises.ipynb)
2. ใช้ database `shop` เดิมได้เลยเช่นกันครับ

**หมายเหตุ: ถ้านักศึกษาใช้ database เดิมจาก WEEK4 ซึ่งนักศึกษาเคยเพิ่มหรือแก้ไขข้อมูลบางส่วนแล้ว ดังนั้น output อาจจะไม่ตรงกับในตัวอย่างทั้งหมดไม่ต้องตกใจนะครับ**


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

### 1. annotate(), F()

1.1 ให้นักศึกษาค้นหาข้อมูล `Payment` โดยให้เพิ่ม field ราคาที่ลบกับส่วนลดแล้ว โดยกำหนดให้ชื่อ field ว่า "after_discount_price" โดยใช้แสดงข้อมูล 10 ตัวแรกเรียงตาม "after_discount_price" จากมากไปน้อย (0.25 คะแนน)

**หมายเหตุ: จะต้องใช้ annotate() นะครับ ให้เอา `Payment.price` - `Payment.discount`**

ตัวอย่าง Output

```
ID: 92, PRICE: 1200500.00, DISCOUNT 29433.25, AFTER_DISCOUNT 1171066.75
ID: 82, PRICE: 1200280.00, DISCOUNT 46229.40, AFTER_DISCOUNT 1154050.60
ID: 137, PRICE: 1200690.00, DISCOUNT 71407.25, AFTER_DISCOUNT 1129282.75
ID: 105, PRICE: 1200390.00, DISCOUNT 105019.11, AFTER_DISCOUNT 1095370.89
ID: 45, PRICE: 1218900.00, DISCOUNT 126859.95, AFTER_DISCOUNT 1092040.05
ID: 7, PRICE: 1201200.00, DISCOUNT 113446.20, AFTER_DISCOUNT 1087753.80
ID: 18, PRICE: 1202190.00, DISCOUNT 121922.64, AFTER_DISCOUNT 1080267.36
ID: 77, PRICE: 379000.00, DISCOUNT 19397.00, AFTER_DISCOUNT 359603.00
ID: 127, PRICE: 320450.00, DISCOUNT 14578.90, AFTER_DISCOUNT 305871.10
ID: 125, PRICE: 320399.00, DISCOUNT 17939.55, AFTER_DISCOUNT 302459.45
```


```python
q1 = Payment.objects.annotate(after_discount_price=F("price")-F("discount")).order_by("-after_discount_price")[:10]
for i in q1:
    print(f"ID: {i.id}, PRICE: {i.price}, DISCOUNT {i.discount}, AFTER_DISCOUNT {i.after_discount_price}")
```

    ID: 92, PRICE: 1200500.00, DISCOUNT 29433.25, AFTER_DISCOUNT 1171066.75
    ID: 82, PRICE: 1200280.00, DISCOUNT 46229.40, AFTER_DISCOUNT 1154050.60
    ID: 137, PRICE: 1200690.00, DISCOUNT 71407.25, AFTER_DISCOUNT 1129282.75
    ID: 105, PRICE: 1200390.00, DISCOUNT 105019.11, AFTER_DISCOUNT 1095370.89
    ID: 45, PRICE: 1218900.00, DISCOUNT 126859.95, AFTER_DISCOUNT 1092040.05
    ID: 7, PRICE: 1201200.00, DISCOUNT 113446.20, AFTER_DISCOUNT 1087753.80
    ID: 18, PRICE: 1202190.00, DISCOUNT 121922.64, AFTER_DISCOUNT 1080267.36
    ID: 77, PRICE: 379000.00, DISCOUNT 19397.00, AFTER_DISCOUNT 359603.00
    ID: 127, PRICE: 320450.00, DISCOUNT 14578.90, AFTER_DISCOUNT 305871.10
    ID: 125, PRICE: 320399.00, DISCOUNT 17939.55, AFTER_DISCOUNT 302459.45
    

1.2 ต่อเนื่องจากข้อ 1.1 ให้ filter เฉพาะข้อมูล `Payment` ที่มี "after_discount_price" มากกว่า 500,000 (0.25 คะแนน)

ตัวอย่าง Output

```
ID: 92, PRICE: 1200500.00, DISCOUNT 29433.25, AFTER_DISCOUNT 1171066.75
ID: 82, PRICE: 1200280.00, DISCOUNT 46229.40, AFTER_DISCOUNT 1154050.60
ID: 137, PRICE: 1200690.00, DISCOUNT 71407.25, AFTER_DISCOUNT 1129282.75
ID: 105, PRICE: 1200390.00, DISCOUNT 105019.11, AFTER_DISCOUNT 1095370.89
ID: 45, PRICE: 1218900.00, DISCOUNT 126859.95, AFTER_DISCOUNT 1092040.05
ID: 7, PRICE: 1201200.00, DISCOUNT 113446.20, AFTER_DISCOUNT 1087753.80
ID: 18, PRICE: 1202190.00, DISCOUNT 121922.64, AFTER_DISCOUNT 1080267.36
```


```python
q2 = Payment.objects.annotate(after_discount_price=F("price")-F("discount")).filter(after_discount_price__gt=500000).order_by("-after_discount_price")[:10]
for i in q2:
    print(f"ID: {i.id}, PRICE: {i.price}, DISCOUNT {i.discount}, AFTER_DISCOUNT {i.after_discount_price}")
    
print("--------------------")
p = Payment.objects.annotate(after_discount_price= F('price') - F('discount') , filter=Q(after_discount_price__gt = 500000)).order_by('-after_discount_price')[:10]
for i in p:
    print(f"ID: {i.id}, PRICE: {i.price}, DISCOUNT {i.discount}, AFTER_DISCOUNT {i.after_discount_price}")
# p.values()
```

    ID: 92, PRICE: 1200500.00, DISCOUNT 29433.25, AFTER_DISCOUNT 1171066.75
    ID: 82, PRICE: 1200280.00, DISCOUNT 46229.40, AFTER_DISCOUNT 1154050.60
    ID: 137, PRICE: 1200690.00, DISCOUNT 71407.25, AFTER_DISCOUNT 1129282.75
    ID: 105, PRICE: 1200390.00, DISCOUNT 105019.11, AFTER_DISCOUNT 1095370.89
    ID: 45, PRICE: 1218900.00, DISCOUNT 126859.95, AFTER_DISCOUNT 1092040.05
    ID: 7, PRICE: 1201200.00, DISCOUNT 113446.20, AFTER_DISCOUNT 1087753.80
    ID: 18, PRICE: 1202190.00, DISCOUNT 121922.64, AFTER_DISCOUNT 1080267.36
    --------------------
    ID: 92, PRICE: 1200500.00, DISCOUNT 29433.25, AFTER_DISCOUNT 1171066.75
    ID: 82, PRICE: 1200280.00, DISCOUNT 46229.40, AFTER_DISCOUNT 1154050.60
    ID: 137, PRICE: 1200690.00, DISCOUNT 71407.25, AFTER_DISCOUNT 1129282.75
    ID: 105, PRICE: 1200390.00, DISCOUNT 105019.11, AFTER_DISCOUNT 1095370.89
    ID: 45, PRICE: 1218900.00, DISCOUNT 126859.95, AFTER_DISCOUNT 1092040.05
    ID: 7, PRICE: 1201200.00, DISCOUNT 113446.20, AFTER_DISCOUNT 1087753.80
    ID: 18, PRICE: 1202190.00, DISCOUNT 121922.64, AFTER_DISCOUNT 1080267.36
    ID: 77, PRICE: 379000.00, DISCOUNT 19397.00, AFTER_DISCOUNT 359603.00
    ID: 127, PRICE: 320450.00, DISCOUNT 14578.90, AFTER_DISCOUNT 305871.10
    ID: 125, PRICE: 320399.00, DISCOUNT 17939.55, AFTER_DISCOUNT 302459.45
    

1.3 ให้นักศึกษาเรียงลำดับข้อมูลลูกค้า (`Customer`) โดยเรียงลำดับตามลำดับตัวอักษร `น้อยไปมาก` จากชื่อเต็มของลูกค้า (`full_name`) โดยแสดง 5 คนแรก (0.5 คะแนน)

**Hint:** Field `full_name` นั้นจะต้องถูก annotate ขึ้นมาโดยการนำ `first_name` มาต่อกับ `last_name` โดยใช้ `Concat(*expressions, **extra)` 

**Hint:** แปลง object เป็น dict ใช้ `values()` [doc](https://docs.djangoproject.com/en/5.0/ref/models/querysets/#values)

```python
>>> Blog.objects.filter(name__startswith="Beatles").values()
<QuerySet [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]>
```

**Hint:** อยาก print dictionary สวยๆ ใช้ `json.dumps`

```python
print(json.dumps(dictionary, indent=4, sort_keys=False))
```

[doc](https://docs.djangoproject.com/en/5.0/ref/models/database-functions/#concat)

ตัวอย่าง Output 

```python
{
    "id": 17,
    "email": "anantaya.deena@gmail.com",
    "address": {
        "district": "Yan Nawa",
        "location": "60 Thanon Chan Road",
        "province": "Bangkok",
        "postal_code": 10120
    },
    "full_name": "Anantaya Tontong"
}
{
    "id": 25,
    "email": "bancha.zeyou@gmail.com",
    "address": {
        "district": "Watthana",
        "location": "6 Thong Lo Road",
        "province": "Bangkok",
        "postal_code": 10110
    },
    "full_name": "Bancha Kittisompong"
}
{
    "id": 19,
    "email": "chayapol.231@gmail.com",
    "address": {
        "district": "Hang Chat",
        "location": "160 Lampang Road",
        "province": "Lampang",
        "postal_code": 52190
    },
    "full_name": "Chayapol Komprach"
}
{
    "id": 4,
    "email": "dejwit.tt@gmail.com",
    "address": {
        "district": "Chiang Khan",
        "location": "150 Loei Road",
        "province": "Loei",
        "postal_code": 42110
    },
    "full_name": "Dejwit Tangjareonsakul"
}
{
    "id": 11,
    "email": "jack.maa@gmail.com",
    "address": {
        "district": "Bang Khen",
        "location": "88 Phahonyothin Road",
        "province": "Bangkok",
        "postal_code": 10220
    },
    "full_name": "Jack Maa"
}

```


```python
q3 = Customer.objects.annotate(full_name=Concat(F("first_name"), Value(" "), F("last_name"))).order_by('full_name')[:5]
print(json.dumps(list(q3.values()), indent=4, sort_keys=False))
```

    [
        {
            "id": 17,
            "first_name": "Anantaya",
            "last_name": "Tontong",
            "email": "anantaya.deena@gmail.com",
            "address": {
                "district": "Yan Nawa",
                "location": "60 Thanon Chan Road",
                "province": "Bangkok",
                "postal_code": 10120
            },
            "full_name": "Anantaya Tontong"
        },
        {
            "id": 25,
            "first_name": "Bancha",
            "last_name": "Kittisompong",
            "email": "bancha.zeyou@gmail.com",
            "address": {
                "district": "Watthana",
                "location": "6 Thong Lo Road",
                "province": "Bangkok",
                "postal_code": 10110
            },
            "full_name": "Bancha Kittisompong"
        },
        {
            "id": 19,
            "first_name": "Chayapol",
            "last_name": "Komprach",
            "email": "chayapol.231@gmail.com",
            "address": {
                "district": "Hang Chat",
                "location": "160 Lampang Road",
                "province": "Lampang",
                "postal_code": 52190
            },
            "full_name": "Chayapol Komprach"
        },
        {
            "id": 4,
            "first_name": "Dejwit",
            "last_name": "Tangjareonsakul",
            "email": "dejwit.tt@gmail.com",
            "address": {
                "district": "Chiang Khan",
                "location": "150 Loei Road",
                "province": "Loei",
                "postal_code": 42110
            },
            "full_name": "Dejwit Tangjareonsakul"
        },
        {
            "id": 11,
            "first_name": "Jack",
            "last_name": "Maa",
            "email": "jack.maa@gmail.com",
            "address": {
                "district": "Bang Khen",
                "location": "88 Phahonyothin Road",
                "province": "Bangkok",
                "postal_code": 10220
            },
            "full_name": "Jack Maa"
        }
    ]
    

### 3. aggregation - count(), sum(), AVG()


3.1 ให้นักศึกษาหาค่าเฉลี่ยของราคาสินค้า (`Product.price`) ที่มีจำนวนคงเหลือ (`Product.remaining_amount`) มากกว่า 0 (0.25 คะแนน)

``` PYTHON
{'avg': Decimal('29308.000000000000')}
```



```python
q4 = Product.objects.filter(remaining_amount__gt=0).aggregate(avg=Avg("price"))
print(q4["avg"])
```

    29308.000000000000
    

3.2 จงหาผลรวมราคา (`CartItem.product.price`) ที่อยู่ในตระกร้าสินค้าของเดือน `พฤษภาคม` (ดูจาก `Cart.create_date`) (0.5 คะแนน)

```PYTHON
{'sum': Decimal('9912555.00')}

```


```python
q5 = CartItem.objects.filter(cart__create_date__month=5).aggregate(sum=Sum("product__price"))
print(q5["sum"])
```

    9912555.00
    

3.3 ให้นักศึกษานับจำนวนสินค้าที่อยู่ประเภท `Electronics`,  `Jewelry` และ ราคาของสินค้าอยู่ในช่วง 8,000.00 - 50,000.00 (0.25 คะแนน)

```
PRODUCT CATEGORY NAME: Electronics, PRODUCT COUNT: 6
PRODUCT CATEGORY NAME: Jewelry, PRODUCT COUNT: 1
```


```python
q6 = Product.objects.filter(categories__name__in=("Electronics", "Jewelry"), price__range=(8000, 50000)).values("categories__name").annotate(number=Count("id"))
for i in q6:
    print(f"PRODUCT CATEGORY NAME: {i['categories__name']}, PRODUCT COUNT: {i['number']}")
    
print("--------------------")
pro = Product.objects.filter(Q(categories__name="Electronics") | Q(categories__name="Jewelry"), price__range=(8000, 50000)).values('categories__name').annotate(count=Count('id'))
for i in pro:
    print(f"PRODUCT CATEGORY NAME: {i['categories__name']}, PRODUCT COUNT: {i['count']}")
    
print("--------------------")
procat = ProductCategory.objects.filter(Q(name="Electronics") | Q(name="Jewelry"), product__price__range=(8000, 50000)).values("name").annotate(count=Count("id"))
for i in procat:
    print(f"PRODUCT CATEGORY NAME: {i['name']}, PRODUCT COUNT: {i['count']}")
```

    PRODUCT CATEGORY NAME: Electronics, PRODUCT COUNT: 6
    PRODUCT CATEGORY NAME: Jewelry, PRODUCT COUNT: 1
    --------------------
    PRODUCT CATEGORY NAME: Electronics, PRODUCT COUNT: 6
    PRODUCT CATEGORY NAME: Jewelry, PRODUCT COUNT: 1
    --------------------
    PRODUCT CATEGORY NAME: Electronics, PRODUCT COUNT: 6
    PRODUCT CATEGORY NAME: Jewelry, PRODUCT COUNT: 1
    

### 4. one-to-one & one-to-many

4.1 ให้นักศึกษาทำการ INSERT ข้อมูลใบสั่งซื้อ (`Order`) และการชำระเงิน (`Payment`) ของลูกค้าชื่อ `Manit Senapan` ตามรายการดังนี้ให้สมบูรณ์ (0.5 คะแนน)

**Hint:** ใน model `Payment` เรามีการเก็บค่า `price` เป็น Decimal ทำให้ค่าของราคาเป็น Decimal เช่นกัน)
[Decimal](https://docs.python.org/3/library/decimal.html)

- ออกใบสั่งซื้อวันที่ 5 สิงหาคม 2024
- ชำระเงินวันที่ 6 สิงหาคม 2024
- หมายเหตุ: `ฉันรวย อยากใช้เงินเยอะๆ`
    
    โดย Manit สั่งสิ้นค้าดั่งนี้

        - Diamond Stud Earrings จำนวน 1 ชิ้น

        - Sofa จำนวน 2 ชิ้น

        - Rose Gold Hoop Earrings จำนวน 1 ชิ้น
    
- โดยที่ Manit ได้รับส่วนลดชิ้นละ 10 % ของสินค้า และมีการระบุหมายเหตุตอนชำระเงินว่า `ลูกค้า VIP ของเรา`
    
- พร้อมชำระเงินโดยให้ 50 % ของราคาทั้งหมดชำระด้วยการแสกน QR code และที่เหลือชำระผ่านบัตรเคดิต

- จากนั้นให้ระบบแสดงผลการสร้างใบสั่งซื้อ และการชำระเงินของ Manit ให้ถูกต้อง


ตัวอย่าง output ที่ต้องการ
```PYTHON
{
    'order_id': 186,
    'order_date': '2024-08-05',
    'order_remark': 'ฉันรวย อยากใช้เงินเยอะๆ',
    'products': [
        {
            'product': 'Diamond Stud Earrings',
            'amount': 1,
            'price': 320000.0,
            'discount': 32000.0
        },
        {
            'product': 'Sofa', 
            'amount': 2, 
            'price': 14000.0, 
            'discount': 1400.0
        },
        {
            'product': 'Rose Gold Hoop Earrings',
            'amount': 1,
            'price': 1200000.0,
            'discount': 120000.0
        }
    ],
    'payment_date': '2024-08-06',
    'payment_remark': 'ลูกค้า VIP ของเรา',
    'payment_method': [
        {
            'method': 'QR', 
            'price': 767000.0
        },
        {
            'method': 'CREDIT', 
            'price': 767000.0
        }
    ]
}

```


```python
# code here - INSERT
```


```python
# code here - แสดงผล
```

### 5. many-to-many

5.1 ให้นักศึกษาค้นหาข้อมูลสินค้า (`Product`) ที่อยู่ในประเภทสินค้า "Information Technology" 10 รายการแรก (เรียงลำดับด้วย `Product.id`) และแสดงชื่อประเภทสินค้า (`ProductCategory`) (0.25 คะแนน)

ตัวอย่าง Output บางส่วน

```
PRODUCT ID: 1, PRODUCT NAME: Smartphone, PRODUCT CATEGORY: Information technology, Electronics
PRODUCT ID: 2, PRODUCT NAME: Laptop, PRODUCT CATEGORY: Information technology, Electronics
PRODUCT ID: 3, PRODUCT NAME: Smart TV, PRODUCT CATEGORY: Information technology, Electronics
```


```python
q7 = Product.objects.filter(categories__name="Information Technology").order_by("id")
for i in q7:
    cat_name = " ,".join(cat.name for cat in i.categories.all())
    print(f"PRODUCT ID: {i.id}, PRODUCT NAME: {i.name}, PRODUCT CATEGORY: {cat_name}")
```

    PRODUCT ID: 1, PRODUCT NAME: Smartphone, PRODUCT CATEGORY: Information Technology ,Electronics
    PRODUCT ID: 2, PRODUCT NAME: Laptop, PRODUCT CATEGORY: Information Technology ,Electronics
    PRODUCT ID: 3, PRODUCT NAME: Smart TV, PRODUCT CATEGORY: Information Technology ,Electronics
    PRODUCT ID: 4, PRODUCT NAME: Bluetooth Earphones, PRODUCT CATEGORY: Information Technology ,Electronics
    PRODUCT ID: 5, PRODUCT NAME: Tablet, PRODUCT CATEGORY: Information Technology ,Electronics
    PRODUCT ID: 6, PRODUCT NAME: Gaming Console, PRODUCT CATEGORY: Information Technology ,Electronics
    PRODUCT ID: 7, PRODUCT NAME: Digital Camera, PRODUCT CATEGORY: Information Technology ,Electronics
    PRODUCT ID: 8, PRODUCT NAME: Wireless Router, PRODUCT CATEGORY: Information Technology ,Electronics
    PRODUCT ID: 9, PRODUCT NAME: Portable Power Bank, PRODUCT CATEGORY: Information Technology ,Electronics
    PRODUCT ID: 10, PRODUCT NAME: Smartwatch, PRODUCT CATEGORY: Information Technology ,Electronics
    

5.2 ให้นักศึกษาทำตามขั้นตอนดังนี้ (0.25 คะแนน)

**หมายเหตุ: ถ้าใช้ DB จาก WEEK4 `Books and Media` อาจจะถูกเปลี่ยนเป็น `Books` แล้ว**

    1. เปลี่ยนชื่อประเภทสินค้า `Books and Media` เป็น `Books and Toys` 
    2. ลบประเภท `Toys and Games` ออกโดยให้ใช้เป็น `Books and Toys` แทน
    3. ค้นหาว่าสินค้าที่มีประเภทสินค้าเป็น `Books and Toys` ทั้งหมดมีจำนวนเท่าไหร่


```python
cat = ProductCategory.objects.get(name="Books and Media")
cat.name = "Books and Toys"
cat.save()
```


```python
protoy = Product.objects.filter(categories__name="Toys and Games")
cattoy = ProductCategory.objects.get(name="Toys and Games")

for i in protoy:
    i.categories.add(cat)
    i.categories.remove(cattoy)
```


```python
count = ProductCategory.objects.filter(name="Books and Toys").count()
count
```




    1




```python

```
