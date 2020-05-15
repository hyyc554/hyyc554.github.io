---
title: django数据库查询优化
tags:
 - django
 - python
---
## 1. DBA 的建议
<!-- more -->
### 1.1 表字段设计

- 避免出现 null 值，null 值难以查询优化且占用额外的索引空间
- 尽量使用 INT 而非 BIGINT，尽可能准确描述字段
- 使用枚举或整数，替代字符串类型
- 使用 TIMESTAMP 替代 DATETIME
- 单表字段不要超过 20
- 使用整型存储 IP

### 1.2 索引

- 在 Where 和 Order By 操作上建立索引
- 值分布稀少的字段不适合建立索引
- 字符串最好不要作为主键
- 在应用层保证 UNIQUE 特性

### 1.3 SQL 查询

- 不要做列运算，可能导致表扫描
- 避免 %xxx 式查询
- 减少 JOIN 操作
- 使用 LIMIT 拿取分页数据，而不要拿全部

## 2. Django Model 建议

ORM 与 DB 的对应关系：

| ORM  |   DB   |
| :--: | :----: |
|  类  | 数据表 |
| 对象 | 数据行 |
| 属性 |  字段  |

- 字段索引

使用 db_index=True 添加索引

```python
title = models.CharField(max_length=255, db_index=True)
```

- 联合索引

利用组合在一起的字段名字建立索引

```python
class Meta:
    index_together = ['field_name_1', 'field_name_2']
```

- 联合唯一索引

组合在一起的字段名称唯一，可以是多个元组，也可以是单个元组。

```python
class Meta:
    # 多元组
    unique_together = (('field_name_1', 'field_name_2'),)
class Meta:
    # 单元组
    unique_together = ('field_name_1', 'field_name_1')
```

## 3. 查询建议

善用`select_related `与`prefetch_related`，如果没有时间，请直接看这个例子：

``````python
class ModelA(models.Model):
    pass

class ModelB(models.Model):
    a = ForeignKey(ModelA)

ModelB.objects.select_related('a').all() # Forward ForeignKey relationship
ModelA.objects.prefetch_related('modelb_set').all() # Reverse ForeignKey relationship
``````

下面是解释

### 3.1 select_related 解决外键关系 N + 1 查询

`select_related` 通过多表 join 关联查询，一次性获取所有数据，减少查询次数。这样讲，可能还是不够明白，看看下面的例子：

```python
class Country(models.Model):
    name = models.CharField(max_length=32)

    def __unicode__(self):
        return self.name

class House(models.Model):
    country = models.ForeignKey(Country, related_name='houses')
```

如果需要查询某个 country 的房屋信息，然后序列化处理。通常情况，可能会这样写：

```python
houses = House.objects.filter(country=country)
for item in houses:
    # 会产生新的数据库查询操作
    country_name = item.country.name
    ...
```

由于 Django 的 Lazy 特性，在执行 filter 操作时，并不会将 country 的 name 字段取出，而是在使用时，实时查询。这样会产生大量的数据库查询操作。

使用 `select_related` 可以避免这种情况，一次性将外键值取出。

```python
houses = House.objects.filter(country=country).select_related('country')
for item in houses:
    # 不会产生新的数据库查询操作
    country_name = item.country.name
    ...
```

### 3.2 prefetch_related 解决多对多关系 N + 1 查询

`prefetch_related` 主要针对一对多、多对多关系进行优化。看一个例子：

```python
class Tag(models.Model):
    name = models.CharField(max_length=32)

class Article(models.Model):
    title = models.CharField(max_length=32)
    tags = models.ManyToManyField(
        to="Tag",
        through='Article2Tag',
        through_fields=('article', 'tag'),
    )
```

如果需要查询指定 Article 的 Tag 信息，然后序列化处理。通常情况，可能会这样写：

```python
articles = Article.objects.filter(id__in=(1,2))
for item in articles:
    # 会产生新的数据库查询操作
    item.tags.all()
```

同样，上面的查询会产生 N + 1 问题，导致大量 IO 消耗。如果使用 `prefetch_related`，可以避免在循环中持续进行数据库查询操作。

```python
articles = Article.objects.prefetch_related("tags").filter(id__in=(1,2))
for item in articles:
    # 不会产生新的数据库查询操作
    item.tags.all()
```

### 3.3 仅查询需要的数据

默认情况下， Django 查询时会提取 ORM 中的全部字段。但是在使用场景中，我们仅关注某些字段。为了节省查询多余字段的时间，可以使用 Django 提供的这两个函数：

- defer()，指定哪些字段不要立即加载

```python
Entry.objects.defer('headline', 'body')
```

- only()，指定立即加载哪些字段，其他忽略

```python
Entry.objects.only("body", "rating").only("headline")
```

defer 和 only 的使用很灵活，可以链式延时加载，也可以链式逐步加载，还可以混合使用。

## 参考资料：

> [Django 性能之数据库查询优化](https://www.chenshaowen.com/blog/database-query-optimization-of-django-performance.html)
>
> [What's the difference between select_related and prefetch_related in Django ORM?](https://stackoverflow.com/questions/31237042/whats-the-difference-between-select-related-and-prefetch-related-in-django-orm)
>
> [Optimizing slow Django REST Framework performance](https://ses4j.github.io/2015/11/23/optimizing-slow-django-rest-framework-performance/)