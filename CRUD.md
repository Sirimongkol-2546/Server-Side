# C-Create
    obj1 = Product.objects.create(
        name="Philosopher's Stone (1997)", 
        description="By J. K. Rowling.",
        remaining_amount = 20,
        price = 790
    )
ดูว่าแต่ละตารางมีคอลัมอะไรบ้าง แล้วค่อยเขียน ถ้ามันเป็น ManyTOMany ต้องระวัง มันเพิ่มเลยไม่ได้ ต้องให้ .add()
>>> from blogs.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)

เราสามารถ add() ที่ละหลายๆตัวได้
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)

# R-Retrive aka. SELECT
>>> Entry.objects.filter(pub_date__year=2010)
    .filter() กรองหาข้อมูลที่ต้องการ

# U-Update
แก้ไข record ก็ใช้ save()  คือเขียนทับไปเลย
>>> b.name = "New name"
>>> b.save()

# D-Delete
ลบทีละตัว
>>> e.delete()
(1, {'blog.Entry': 1})

ลบทีละหลายตัว
>>> Entry.objects.filter(pub_date__year=2005).delete()
(5, {'blog.Entry': 5})