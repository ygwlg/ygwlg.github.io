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
