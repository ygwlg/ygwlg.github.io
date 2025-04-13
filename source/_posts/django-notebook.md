---
title: Django笔记
date: 2025-02-17 14:47:35
tags: 
   - Django
   - Python

categories:
   - Django
---

## Django 的MVT模式

传统的MVC模式包括

1. **Model(模型)** 数据模型， 对应于数据库的数据关系。

2. **View(视图)** 与用户交互，把Controller层的数据传给用户，把用户的数据传给Controller层

3. **Controller(控制器)** 处理从View中获取的用户数据的逻辑，对Model层中的数据进行查找/修改

Django的MVT模型

1. **Model(模型)** 数据模型，同上

2. **View(视图)** 和上述的Controller层类似；从Template接收用户需求，查询/修改数据库，再将响应数据传输给Template层

3. **Template(模板)** 从 View层接收数据传给用户，把用户数据传给View

## **Model** 模型层

### 模型的字段选项

+ CharField 必传参数 max_length, DataField 的max_length非必传

+ ForeignKey 必传参数 on_delete

  1. CASCADE: 级联删除，关联对象被删除时，删除所有关联到该对象的对象
  2. PROTECT: 删除保护，关联对象被删除时，触发异常
  3. SET_NULL: 被删除后自动设置为NULL
  4. SET_NULL: ...，被设置为默认值
  5. SET(): ...， 设置为传进去的值
  6. DO_NOTING: ...，啥也不干 

+ Field.null 数据库相关逻辑，允许在基于字符串的字段上使用null（否贼为空字符串），例如CharField和DataField

+ Field.blank 校验逻辑，允许字段为空

+ Field.choices 校验逻辑，为传入值限制选项

+ Field.db_column 数据库列名

+ Field.db_comment 数据库列注释

+ Field.db_index 为该字段创建数据库索引

+ Field.db_tablespace 如果字段有索引，需要指定索引使用的数据库表空间的名称，默认是 DEFAULT_INDEX_TABLESPACE 

+ Field.default 字段默认值，如果是可变对象需要包裹成函数，否则默认值为该对象的引用

+ Field.primary_key 表的主键

+ Field.unique 字段必须在整个表中保持唯一，并自动创建索引

+ Field.unique_for_data/month/year 不允许有相同的日期/月/年，不在数据库层面约束，只在save时验证

### 字段类型

+ AutoField 自增的 IntegerField

+ BigAutoField 自增的BigIntegerField，会自动创建作为主键

+ BinaryField

+ BoolField

+ CharField 

+ DataField 日期

+ DateTimeField

+ DecimalField 固定精度的实数

+ DurationField 

+ EmailField

+ FileField

+ FilePathField

+ FloatField

+ GeneratedField 根据模型中的其他字段始终计算的字段

+ GenericIPAddressField IPv4或IPv6地址

+ ImageField 

+ JsonField Python格式的json本地数据，例如列表、字典等

+ TextField 大文本字段

+ TimeField datatime.time实例

+ URLField Url 的 CharField，由URLValidator验证

+ UUIDField 唯一标识符字段，使用Python的UUID类

+ ManyToManyField 多对多关系

+ OneToOneField 单对单

#### 延迟关系

延迟关系允许通过模型名称引用模型

+ 递归

```python
class Manufacturer(models.Model):
    name = models.TextField()
    suppliers = models.ManyToManyField("self", symmetrical=False)
```

+ 相对

```python
class Car(models.Model):
    manufacturer = models.ForeignKey(
        "Manufacturer",
        on_delete=models.CASCADE,
    )


class Manufacturer(models.Model):
    name = models.TextField()
    suppliers = models.ManyToManyField("self", symmetrical=False)
```

+ 绝对

```python
class Car(models.Model):
    manufacturer = models.ForeignKey(
        "thirdpartyapp.Manufacturer",
        on_delete=models.CASCADE,
    )
```

### 模型Meta选项

```python
class FunModel(models.Model):
name = models.CharField(max_length=100)
    class Meta:
        abstract = True  # 抽象模型
        app_label  # 属于哪个程序
        base_manager_name  # 管理器类名
        db_table  # 数据库表名
        db_tablespace  # 表空间名
        default_manager_name  # 默认管理器名
        default_related_name  # 默认关系名
        managed  # migrate时会创建对应库表，成为迁移的一部分
        get_latest_by  # 
        order_with_respect_to  # 按照外键排序
        ordering  # 获取对象的排序顺序
        
```

### 执行查询

创建对象

```python
b = Blog(name="Beatles Blog", tagline="All the latest Beatles news")
b.save()
```

修改对象

```python
b.name = "new name"
b.save()
```

对象检索

```python
# 检索全部对象
Entry.objects.all()

# 过滤，返回新的QuerySet
Entry.objects.filter()
Entry.objects.filter()

# 链式过滤
Entry.objects.filter().exclude()

# 获取单个对象
Entry.objects.get()

# get方法会抛出两个异常DoesNotExist 和 MultipleObjectsReturned

```

```python
# 用查询表达式列表对QuerySet每个对象进行注解
q = Blog.objects.annotate(Count("entry"))

# 保存 

```



## Django的ORM

orm：Object Relation Mapping (对象映射关系)，作为一种数据库访问技术，允许开发者使用python对象操作数据库，而不是直接编写SQL查询语句。这样做的优点:

1. 防止SQL注入

2. 不修改源代码的基础上切换数据库

3. 简化数据库操作

4. 自动生成数据库表

5. 便捷的查询API

6. 事务支持

7. 可测试性

### 具体的查询表达式和SQL语句的关系

假设现在有两个表Author 和 Book，分别有name字段代表作者名和书籍名，两者的关系是多对多：即一个作者可能会有几本书，一本书可能会有对个共同作者。

在django的ORM中如下创建

```python
class Author(models.Model):
    name = models.CharField(max_length=20)


class Book(models.Model):
    name = models.CharField(max_length=20)
    authors = models.ManyToManyField("Author")
```

多对多的关系一般用一张额外的表来存储关系数据

SQL语句如下创建

```SQL
CREATE TABLE books_book (
    name VARCHAR(20)
                        );

CREATE TABLE books_author(
    name VARCHAR(20)
);
CREATE TABLE books_book_author(
    id INT CONSTRAINT PK_books_book_author PRIMARY KEY ,
    book_id INT,
    author_id INT
);
```

创建一行数据

```python
Author(name='a1').save()
```

```SQL
INSERT INTO books_author (name) VALUES ("a1");
```

删除一行数据

```python
Author.objects.get(name='book1').delete()
```

```python
DELETE FROM books_author WHERE name = "a1";
```

单表查询

给这两个表添加一些字段，用于实践单表查询

```python
class Author(models.Model):
    name = models.CharField(max_length=20)
    birthday = models.DateField()


class Book(models.Model):
    name = models.CharField(max_length=20)
    pub_date = models.DateField()
    authors = models.ManyToManyField("Author")
```

```SQL
ALTER TABLE books_author ADD COLUMN birthday DATE;

ALTER TABLE books_book ADD COLUMN pub_date DATE;
```

1. 查找字段等于某个值的数据

```python
Book.objects.all().filter(name="a1")
```

```SQL
Select * FROM books_book WHERE name = "a1";
```

2. 按照某个字段的条件筛选

```python
Book.objects.filter(pub_date__gt=datetime.date(2005, 1, 3))
```

```SQL
SELECT * FROM books_author WHERE birthday >= '1970-01-01';
SELECT * FROM books_author WHERE DATE(birthday) <= CURRENT_DATE;
```

3. 连表查询某个字段的值

再创建一个表BookDetail，作为Book的详情页

```python
class Author(models.Model):
    name = models.CharField(max_length=20)
    birthday = models.DateField()


class Book(models.Model):
    name = models.CharField(max_length=20)
    pub_date = models.DateField()
    authors = models.ManyToManyField("Author")


class BookDetail(models.Model):
    detail = models.CharField(max_length=100)
    book = models.ForeignKey('Book', on_delete=models.CASCADE)

```

运行SQL语句的时候还踩坑了：sqlite不支持通过alter table添加外键。。。

```SQL
ALTER TABLE books_bookdetail ADD  CONSTRAINT fk_book_id FOREIGN KEY (book_id) REFERENCES books_book(id);
```

```python
BookDetail.objects.filter(book__pub_date__gt=datetime.date(2005, 1, 3))
```

```SQL
SELECT * FROM books_book Inner Join books_bookdetail ON books_book.id = books_bookdetail.book_id WHERE books_book.pub_date > '2005-1-3';
```

反过来呢，也就是查询有detail的book表的值，如下的查询方式将会在第一次迭代的时候查询detail表，在执行d.book.name时又会访问数据库。

```python
details = BookDetail.objects.filter(book__pub_date__gt=datetime.date(2005, 1, 3)).all()
for d in details:
    print(d.book.name)
```

这个时候需要用到Django为关系创建的反向关联字段

```python
Book.objects.annotate(num_detail=Count('bookdetail')).filter(num_detail__gte=1, pub_date__gt=datetime.date(1970, 1, 1))
```

## Django对事务的支持

### 1. 全局配置

```python
ATOMIC_REQUESTS = True
```

### 2. 装饰器

```python

# 非事务视图
@transaction.non_atomic_requests
def func():
    pass

# 事务视图
@transaction.atomic
def func():
    pass

```
### 3. 上下文管理器

```python
def func():
    with transaction.atomic():
        pass
```

官方文档建议不要在atomic内捕获异常，否则会影响事务的回滚。如果恰好在事务回滚的时候查询数据库，Django将会触发TransactionManagementError 异常

捕获到数据库回滚异常时，需要在except块中手动还原程序状态，不然可能会导致逻辑错误，例如

```python
from django.db import DatabaseError, transaction

obj = MyModel(active=False)
obj.active = True
try:
    with transaction.atomic():
        obj.save()
except DatabaseError:
    obj.active = False

if obj.active:
    ...
```
事务锁

[悲观锁与乐观锁](https://cloud.tencent.com/developer/article/1552304?policyId=1003)
悲观锁假设数据在并发访问时会发生冲突，因此每次访问数据时都会先加锁，实现方式一般为数据库的行级锁，表级锁等。适用场景：高并发，数据竞争激烈的场景。缺点：可能导致系统吞吐量变低；可能会导致死锁。

```SQL
begin;
SELECT num FROM goods WHERE id=1 FOR UPDATE;
UPDATE goods SET num = num - 1 WHERE id=1;
commit;
```



```python
cursor.execute("SELECT * FROM table WHERE ... FOR UPDATE")

# 相当于

MyModel.objects.select_for_update()
```


乐观锁假设数据的变动不会太频繁，因此运行多个事务同事修改数据。
通畅通过在表中加入一个版本或时间戳来实现。当事务从数据库中读取数据时，会同时读取到数据的版本v1，当事务对数据变动完毕想要更新回表中时，会与最新的版本v2对比，如果v1=v2，那么说明数据在变动期间，没有其余事务对数据进行修改，此时允许事务对表中的数据进行修改并将version加一写入。如果v1不等于v2，说明期间数据被改动了，此时不允许更新到表中，一般处理方法是通知用户让其重新操作。这一过程是代码控制的，而非数据库。

```SQL
SELECT num,version FROM goods WHERE id=1;
UPDATE goods SET num=num-1,version=version+1 WHERE id=1;
```

事务的三种不正常现象

1. 脏读（读未提交） 数据A正在访问并修改数据，但在修改尚未提交到数据库时，事务B访问到了未提交的数据并使用了该脏数据。

2. 不可重复读（多次读取数据不一致） 事务A多次读取同一个数据，读取的结果不一样，因为在此期间该数据被别的事务所修改。这里读取到的是别的事务已经提交的数据。

3. 幻读（前后多次读取，数据不一致） 事务A按照某个条件先后两次查询数据库，但是查询结果的条数不同。

事务的隔离等级

1. 读未提交 允许读取到尚未提交的数据

2. 读提交 事务要等到另一个事务提交才能够读取数据，如果有事务对数据进行更新的时候，读操作需要等待该更新提交后才能读取。可以解决脏读问题。

3. 重复读 开始读取数据后不允许修改 可以解决不可重复读问题(数据库的默认隔离级别)

4. 串行化 事务按照顺序执行 隔离级别地下，一般不使用

设置数据库的事务隔离级别

1. 全局设置

```SQL
set global transaction isolation level read uncommitted; 
```

2. 单独设置当前连接

```SQL
set transaction isolation level read committed;
```

[MVCC](https://cloud.tencent.com/developer/article/2378614)

多版本并发控制，目的是提高数据库的并发性能，用更好的方式处理读写冲突，这里的多版本指的是数据库中同时存在一条记录的多个版本，而非整个数据库的多个版本

当前读是悲观锁的实现:

1. SELECT ... LOCK IN SHARE MODE

2. SELECT ... FOR UPDATE

3. UPDATE

4. DELETE

5. INSERT

MYSQL用MVCC来实现快照读，即事务开始时会创建一个一致性视图，该视图反映了事务开始时刻数据库的快照。并记录数据库的快照版本。

**事务的隔离级别是串行化时，快照读取会退化为当前读**

MVCC的实现原理

1. Innodb数据库中的隐藏字段：DB_TRX_ID 记录创建这条数据上次修改它的事务ID,DB_ROLL_PTR 回滚指针记录这条记录的上一个版本

2. 回滚日志
    + insert的undo log，事务提交之后就会被删除，因为新插入的数据没有历史版本

    + update、delete的undo log，只有在快照读和事务回滚不涉及该日志时，对应的日志才会被purge线程统一删除

3. 一致性视图

事务进行快照读操作时候生成的读视图（Read View），在该事务执行快照读的那一刻，会生成数据库系统当前的一个快照，记录并维护系统当前活跃事务的ID；当数据要被修改时，遍历undo log链表，对比DB_TRX_ID，直到当前事务能看到的版本。


Django的查询

```python
# exact查询
BookInfo.objects.get(id=1)

# 模糊查询
BookInfo.objects.filter(btitle__contains='传')
BookInfo.objects.filter(btitle__endswith='部')

# 空查询
select * from booktest_bookinfo where btitle is not null;
BookInfo.objects.filter(btitle__isnull=False)

# 范围查询
select * from booktest_bookinfo where btitle is not null;
BookInfo.objects.filter(btitle__isnull=False)
```

F对象，用于类的属性比较

```python
from django.db.models import F

# 查询阅读量大于评论量的图书
BookInfo.objects.filter(bread__gt=F('bcomment'))
BookInfo.objects.filter(bread__gt=F('bcomment')*2)
```

Q对象，用于组合查询逻辑

```python
BookInfo.objects.filter(id__gt=3, bread__gt=30)
BookInfo.objects.filter(Q(id__gt=3)&Q(bread__gt=30))
BookInfo.objects.filter(Q(id__gt=3)|Q(bread__gt=30))
BookInfo.objects.filter(~Q(id=3))
```

聚合函数 summ, count, avg, max, min

```python
from django.db.models import Sum,Count,Max,Min,Avg

BookInfo.objects.all().aggreate(Count('id'))
BookInfo.objects.aggregate(Sum('bread'))
```