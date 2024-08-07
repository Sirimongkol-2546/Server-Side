## SETTING ##
# Install virtualenv
> pip install virtualenv

# Create a virtual environment
> py -m venv myvenv

# Activate virtual environment
> myvenv\Scripts\activate.bat

# Install Django และ psycopg2
> pip install django  psycopg2

# Create project "myblogs"
> django-admin startproject myblogs

# Create the "blogs" app อย่าลืม cd เข้า project ก่อน
> python manage.py startapp blogs

# Start server
> python manage.py runserver

# Database setting
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "blogs",
        "USER": "db_username",
        "PASSWORD": "password",
        "HOST": "localhost",
        "PORT": "5432",
    }
}

# Add app blogs to INSTALLED_APPS
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    # Add your apps here
    "blogs",
]

# python manage.py makemigrations เพื่อให้ Django ทำการสร้างไฟล์ python manage.py migration ขึ้นมา

# notebook
pip install django-extensions ipython jupyter notebook
pip install ipython==8.25.0 jupyter_server==2.14.1 jupyterlab==4.2.2 jupyterlab_server==2.27.2
pip install notebook==6.5.7
python manage.py shell_plus --notebook

# เพิ่ม django-extensions ใน INSTALLED_APPS ในไฟล์ settings.py
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",

    "django_extensions",
    "blogs",
]

# --------------------------------------------------------------------------------

# ----WEEK 04----#

# Making Query
# Creating objects
***import ตารางที่เราจะใช้มาก่อน         
>>> from blogs.models import Blog
>>> b = Blog(name="Beatles Blog", tagline="All the latest Beatles news.")
>>> b.save()

# Saving changes to objects
>>> b.name = "New name"
>>> b.save()

# Saving ForeignKey and ManyToManyField fields
>>> from blogs.models import Blog, Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog # Update FK blog ของ entry (ID = 1) ไปที่ cheese_blog (name = "Cheddar Talk")
>>> entry.save()

แต่การ update ManyToManyField จะทำงานแตกต่างไปนิดหน่อย เราจะต้องใช้ add() ดังตัวอย่าง
สมมติเราต้องการ add instance joe เป็นหนึ่งใน authors ของ instance entry (ID = 1)
>>> from blogs.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)

เราสามารถ add() ที่ละหลายๆ instances เข้าไปก็ได้
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)

# ---------------------------------------------------------------------

# Retrieving objects
>>> Entry.objects.all() # SELECT * FROM entry;
# with filters
>>> Entry.objects.filter(pub_date__year=2010)

# Chaining filters
เราสามารถ chain method filter() และ exclude() ได้
NOTE: exclude() คือการใส่เงื่อนไขที่จะกรองข้อมูลออก ดังในตัวอย่างด้านล่างคือการกรองข้อมูล blog entries หลังจากวันปัจจุบันออก
>>> Entry.objects.filter(headline__startswith="What").
    exclude(pub_date__gte=datetime.date.today()).
    filter(pub_date__gte=datetime.date(2005, 1, 30))

# Retrieving a single object with get()
>>> one_entry = Entry.objects.get(pk=1)
>>> one_entry = Entry.objects.filter(pk=1).first()
>>> one_entry = Entry.objects.filter(pk=1)[0]
>>> # ทั้ง 3 บรรทัดนี้ให้ผลเหมือนกัน

# Limiting QuerySets
>>> Entry.objects.all()[:5] # LIMIT 5
>>> Entry.objects.all()[5:10] # OFFSET 5 LIMIT 5

# ---------------------------------------------------------------------

# Comparing objects
>>> some_entry == other_entry
>>> some_entry.id == other_entry.id

# ---------------------------------------------------------------------

# Deleting objects
ลบทีละตัว
>>> e.delete()
(1, {'blog.Entry': 1})

ลบทีละหลายตัว
>>> Entry.objects.filter(pub_date__year=2005).delete()
(5, {'blog.Entry': 5})

# ---------------------------------------------------------------------

# Field lookup
- LIKE : contains
ex.   Entry.objects.filter(headline__contains='Lennon')
- LIKE : icontains but ไม่สนใหญ่เล็ก
- startswith, endswith, in ex. Entry.objects.filter(headline__in=('a', 'b', 'c'))
- gt > , gte >= , lt < , lte <= ex. Product.objects.filter(price__lt = 200)
- BETWEEN AND - range ex. Entry.objects.filter(pub_date__range=(start_date, end_date))
- date, year, month, day, week, week_day
    - Entry.objects.filter(pub_date__year=2005)
    - Entry.objects.filter(pub_date__year__gte=2005)

# Lookups that span relationships
เป็นการ query ข้อมูลไปกี่ต่อก็ได้
Blog.objects.filter(entry__authors__name="Lennon")
Blog.objects.filter(entry__authors__name__isnull=True)

# Filters can reference fields on the model
ต้องการเปรียบเทียบค่าของ field ใน model กับ field อื่นใน model เดียวกัน ใช้ F expressions ได้ F() ***อย่าลืม import!!!
รวมถึงพวก + - * หาร ด้วย ใช้ตัวนี้หมด
>>> from django.db.models import F
>>> Entry.objects.filter(number_of_comments__gt=F("number_of_pingbacks"))
>>> Entry.objects.filter(authors__name=F("blog__name")) # span relationships
การคำนวณ + - * /
>>> Entry.objects.filter(number_of_comments__gt=F("number_of_pingbacks") * 2)
>>> Entry.objects.filter(rating__lt=F("number_of_comments") + F("number_of_pingbacks"))

# Complex lookups with Q objects
    -- Entry.objects.filter(headline__contains='Lennon', pub_date__year=2005)
    SELECT * FROM entry WHERE headline LIKE '%Lennon%' AND pub_date BETWEEN '2005-01-01' AND '2005-12-31';
ใช้ , ขั้นระหว่าง filter condition จะเป็นการ AND กัน

ในกรณีที่เราต้องการทำการ query ที่ซับซ้อน อาจจะต้องการใช้ OR | หรือ NOT ร่วมด้วย เราจะต้องใช้ Q objects
# OR
    Entry.objects.filter(Q(headline__startswith="Who") | Q(headline__startswith="What"))
    SELECT ... WHERE headline LIKE 'Who%' OR headline LIKE 'What%'
# NOT
    Entry.objects.filter(Q(headline__startswith="Who") | ~Q(pub_date__year=2005))
    SELECT ... WHERE headline LIKE 'Who%' OR pub_date NOT BETWEEN '2005-01-01' AND '2005-12-31'; 

# --------------------------------------------------------------------------------

# ----WEEK 05----#
# Making Queries (Advanced)

# *** Query Expressions

# aggregate
SUM(), MIN(), MAX(), COUNT(), AVG() ---> มันจะออกมาเป็นเลขตัวเดียว
from django.db.models import Avg, Max, Sum, Min, Count

- Count
# Total number of books with publisher=Penguin Books
>>> Book.objects.filter(publisher__name="Penguin Books").count()
20

- Avg
>>> Book.objects.aggregate(Avg("price", default=0))
{'price__avg': Decimal('9.7018644067796610')}

- Max
>>> Book.objects.aggregate(Max("price", default=0))
{'price__max': Decimal('14.99')}

# annotate
>>> pubs = Publisher.objects.annotate(num_books=Count("book"))
>>> pubs
<QuerySet [<Publisher: BaloneyPress>, <Publisher: SalamiPress>, ...]>
>>> pubs[0].num_books
20

- = มากไปน้อย
# The top 5 publishers, in order by number of books.
>>> pubs = Publisher.objects.annotate(num_books=Count("book")).order_by("-num_books")[:5]
>>> pubs[0].num_books
39

# Each publisher, with a separate count of books with a rating above and below 4
>>> from django.db.models import Q
>>> above = Publisher.objects.annotate(above_4=Count("book", filter=Q(book__rating__gt=4)))
>>> below = Publisher.objects.annotate(below_4=Count("book", filter=Q(book__rating__lte=4)))
>>> above[0].above_4
16
>>> below[0].below_4
4

# import
>>> from company.models import Company
>>> from django.db.models import Count, F, Value
>>> from django.db.models.functions import Length, Upper
>>> from django.db.models.lookups import GreaterThan
>>> from django.db.models.functions import Concat
>>> from django.db.models import CharField, Value as V

# Concat
obj3 = Customer.objects.annotate(full_name = Concat("first_name", V(" "), "last_name"))
for i in obj3:
    print(f"Full name: {i.full_name}")
>>> Full name: Panita Hongsakulpan
>>> Full name: Pakin Janpen

# Json Dump
อยาก print dictionary สวยๆ ใช้ json.dumps
print(json.dumps(dictionary, indent=4, sort_keys=False))
---> เปลี่ยน dic เป็น i

Values ----> เเปลงเป็น dict
ถ้าใช้กรณีทำเป็น group by คือ values จะให้เราเลือกว่าอยากให้ field ไหนเป็นตัว group
group by คือการใช้ values() คู่กับ annotate(ข้างในเป็น aggregate) 
เช่นอยากได้ รายได้เเต่ละบริษัท เราใช้ values(ชื่อบริษัท).annotate(sum = Sum(เงิน))
.values(ใส่ตัวที่จะเลือก)

# Becareful 
***ต้อง filter ก่อน aggregate !!!

# --------------------------------------------------------------------------------

# Many-to-one relationships
ManytoMany จะมีตารางกลาง
