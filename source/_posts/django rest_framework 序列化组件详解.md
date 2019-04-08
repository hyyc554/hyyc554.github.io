---
title: django rest_framework 序列化组件详解
tag: 
 - Django
---


#### 为什么要用序列化组件

当我们做前后端分离的项目,我们前后端交互一般都选择JSON数据格式，JSON是一个轻量级的数据交互格式。

那么我们给前端数据的时候都要转成json格式，那就需要对我们从数据库拿到的数据进行序列化。

接下来我们看下django序列化和rest_framework序列化的对比
<!-- more -->

## Django的序列化方法

```python
class BooksView(View):
    def get(self, request):
        book_list = Book.objects.values("id", "title", "chapter", "pub_time", "publisher")
        book_list = list(book_list)
        # 如果我们需要取外键关联的字段信息 需要循环获取外键 再去数据库查然后拼接成我们想要的
        ret = []
        for book in book_list:
            pub_dict = {}
            pub_obj = Publish.objects.filter(pk=book["publisher"]).first()
            pub_dict["id"] = pub_obj.pk
            pub_dict["title"] = pub_obj.title
            book["publisher"] = pub_dict
            ret.append(book)
        ret = json.dumps(book_list, ensure_ascii=False, cls=MyJson)
        return HttpResponse(ret)


# json.JSONEncoder.default()
# 解决json不能序列化时间字段的问题
class MyJson(json.JSONEncoder):
    def default(self, field):
        if isinstance(field, datetime.datetime):
            return field.strftime('%Y-%m-%d %H:%M:%S')
        elif isinstance(field, datetime.date):
            return field.strftime('%Y-%m-%d')
        else:
            return json.JSONEncoder.default(self, field)
```

```python
from django.core import serializers


# 能够得到我们要的效果 结构有点复杂
class BooksView(View):
    def get(self, request):
        book_list = Book.objects.all()
        ret = serializers.serialize("json", book_list)
        return HttpResponse(ret)
```



##  DRF序列化的方法

首先，我们要用DRF的序列化，就要遵循人家框架的一些标准，

- Django我们CBV继承类是View，现在DRF我们要用APIView

- Django中返回的时候我们用HTTPResponse，JsonResponse，render ，DRF我们用Response

为什么这么用~我们之后会详细讲我们继续来看序列化

### 序列化

serializer:

```python
class BookSerializer(serializers.Serializer):
    id = serializers.IntegerField()
    title = serializers.CharField(max_length=32)
    CHOICES = ((1, "Linux"), (2, "Django"), (3, "Python"))
    chapter = serializers.ChoiceField(choices=CHOICES, source="get_chapter_display")
    pub_time = serializers.DateField()
```

view:

```python
from rest_framework.views import APIView
from rest_framework.response import Response

class BookView(APIView):
    def get(self, request):
        book_list = Book.objects.all()
        ret = BookSerializer(book_list, many=True)
        return Response(ret.data)
```



### 外键关系的序列化

```python
from rest_framework import serializers
from .models import Book


class PublisherSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(max_length=32)


class UserSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    name = serializers.CharField(max_length=32)
    age = serializers.IntegerField()


class BookSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(max_length=32)
    CHOICES = ((1, "Linux"), (2, "Django"), (3, "Python"))
    chapter = serializers.ChoiceField(choices=CHOICES, source="get_chapter_display", read_only=True)
    pub_time = serializers.DateField()

    publisher = PublisherSerializer(read_only=True)
    user = UserSerializer(many=True, read_only=True)
```



### 反序列化

当前端给我们发post的请求的时候,我们要进行一些校验然后保存到数据库

这些校验以及保存工作，DRF的Serializer也给我们提供了一些方法了

首先~我们要写反序列化用的一些字段~有些字段要跟序列化区分开

Serializer提供了.is_valid()  和.save()方法

serializers:

```python
# serializers.py 文件
class BookSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(max_length=32)
    CHOICES = ((1, "Linux"), (2, "Django"), (3, "Python"))
    chapter = serializers.ChoiceField(choices=CHOICES, source="get_chapter_display", read_only=True)
    w_chapter = serializers.IntegerField(write_only=True)
    pub_time = serializers.DateField()

    publisher = PublisherSerializer(read_only=True)
    user = UserSerializer(many=True, read_only=True)

    users = serializers.ListField(write_only=True)
    publisher_id = serializers.IntegerField(write_only=True)

    def create(self, validated_data):
        book = Book.objects.create(title=validated_data["title"], chapter=validated_data["w_chapter"], pub_time=validated_data["pub_time"],                                  publisher_id=validated_data["publisher_id"])
        book.user.add(*validated_data["users"])
        return book
```

view:

```python
class BookView(APIView):
    def get(self, request):
        book_list = Book.objects.all()
        ret = BookSerializer(book_list, many=True)
        return Response(ret.data)

    def post(self, request):
        # book_obj = request.data
        print(request.data)
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            print(12341253)
            serializer.save()
            return Response(serializer.validated_data)
        else:
            return Response(serializer.errors)
```

当前端给我们发送patch请求的时候，前端传给我们用户要更新的数据，我们要对数据进行部分验证~~

Serializer:

```python
class BookSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(max_length=32)
    CHOICES = ((1, "Linux"), (2, "Django"), (3, "Python"))
    chapter = serializers.ChoiceField(choices=CHOICES, source="get_chapter_display", read_only=True)
    w_chapter = serializers.IntegerField(write_only=True)
    pub_time = serializers.DateField()

    publisher = PublisherSerializer(read_only=True)
    user = UserSerializer(many=True, read_only=True)

    users = serializers.ListField(write_only=True)
    publisher_id = serializers.IntegerField(write_only=True)

    def create(self, validated_data):
        book = Book.objects.create(title=validated_data["title"], chapter=validated_data["w_chapter"], pub_time=validated_data["pub_time"],
                                   publisher_id=validated_data["publisher_id"])
        book.user.add(*validated_data["users"])
        return book

    def update(self, instance, validated_data):
        instance.title = validated_data.get("title", instance.title)
        instance.chapter = validated_data.get("w_chapter", instance.chapter)
        instance.pub_time = validated_data.get("pub_time", instance.pub_time)
        instance.publisher_id = validated_data.get("publisher_id", instance.publisher_id)
        if validated_data.get("users"):
            instance.user.set(validated_data.get("users"))
        instance.save()
        return instance
```

view:

```python
class BookView(APIView):
     def patch(self, request):
        print(request.data)
        book_id = request.data["id"]
        book_info = request.data["book_info"]
        book_obj = Book.objects.filter(pk=book_id).first()
        serializer = BookSerializer(book_obj, data=book_info, partial=True)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        else:
            return Response(serializer.errors)
```

### 验证

如果我们需要对一些字段进行自定义的验证~DRF也给我们提供了钩子方法~~

Serializer:

```python
class BookSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(max_length=32)
    # 省略了一些字段 跟上面代码里一样的
    # 。。。。。
     def validate_title(self, value):
        if "python" not in value.lower():
            raise serializers.ValidationError("标题必须含有Python")
        return value
```

view:

```python
class BookSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(max_length=32)
    CHOICES = ((1, "Linux"), (2, "Django"), (3, "Python"))
    chapter = serializers.ChoiceField(choices=CHOICES, source="get_chapter_display", read_only=True)
    w_chapter = serializers.IntegerField(write_only=True)
    pub_time = serializers.DateField()
    date_added = serializers.DateField(write_only=True)
    # 新增了一个上架时间字段  
    # 省略一些字段。。都是在原基础代码上增加的
    # 。。。。。。

    # 对多个字段进行验证 要求上架日期不能早于出版日期 上架日期要大
    def validate(self, attrs):
        if attrs["pub_time"] > attrs["date_added"]:
            raise serializers.ValidationError("上架日期不能早于出版日期")
        return attrs
```



```python
def my_validate(value):
    if "敏感词汇" in value.lower:
        raise serializers.ValidationError("包含敏感词汇，请重新提交")
    return value


class BookSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(max_length=32, validators=[my_validate])
    # 。。。。。。
    
```



## ModelSerializer

现在我们已经清楚了Serializer的用法，会发现我们所有的序列化跟我们的模型都紧密相关~

那么，DRF也给我们提供了跟模型紧密相关的序列化器~~ModelSerializer~~

- 它会根据模型自动生成一组字段

- 它简单的默认实现了.update()以及.create()方法

### 定义一个ModelSerializer序列化器

```python
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = "__all__"
        # fields = ["id", "title", "pub_time"]
        # exclude = ["user"]
        # 分别是所有字段 包含某些字段 排除某些字段
```

### 外键关系的序列化

注意：当序列化类MATE中定义了depth时，这个序列化类中引用字段（外键）则自动变为只读

```python
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = "__all__"
        # fields = ["id", "title", "pub_time"]
        # exclude = ["user"]
        # 分别是所有字段 包含某些字段 排除某些字段
        depth = 1
# depth 代表找嵌套关系的第几层
```

### 自定义字段

我们可以声明一些字段来覆盖默认字段，来进行自定制~

比如我们的选择字段，默认显示的是选择的key，我们要给用户展示的是value。

```python
class BookSerializer(serializers.ModelSerializer):
    chapter = serializers.CharField(source="get_chapter_display", read_only=True)
    
    class Meta:
        model = Book
        fields = "__all__"
        # fields = ["id", "title", "pub_time"]
        # exclude = ["user"]
        # 分别是所有字段 包含某些字段 排除某些字段
        depth = 1
```

### Meta中其它关键字参数

```python
class BookSerializer(serializers.ModelSerializer):
    chapter = serializers.CharField(source="get_chapter_display", read_only=True)

    class Meta:
        model = Book
        fields = "__all__"
        # fields = ["id", "title", "pub_time"]
        # exclude = ["user"]
        # 分别是所有字段 包含某些字段 排除某些字段
        depth = 1
        read_only_fields = ["id"]
        extra_kwargs = {"title": {"validators": [my_validate,]}}
```

### post以及patch请求

由于depth会让我们外键变成只读，所以我们再定义一个序列化的类，其实只要去掉depth就可以了~~

```python
class BookSerializer(serializers.ModelSerializer):
    chapter = serializers.CharField(source="get_chapter_display", read_only=True)

    class Meta:
        model = Book
        fields = "__all__"
        # fields = ["id", "title", "pub_time"]
        # exclude = ["user"]
        # 分别是所有字段 包含某些字段 排除某些字段
        read_only_fields = ["id"]
        extra_kwargs = {"title": {"validators": [my_validate,]}}
```

### SerializerMethodField

外键关联的对象有很多字段我们是用不到的~都传给前端会有数据冗余~就需要我们自己去定制序列化外键对象的哪些字段~~

```python
class BookSerializer(serializers.ModelSerializer):
    chapter = serializers.CharField(source="get_chapter_display", read_only=True)
    user = serializers.SerializerMethodField()
    publisher = serializers.SerializerMethodField()

    def get_user(self, obj):
        # obj是当前序列化的book对象
        users_query_set = obj.user.all()
        return [{"id": user_obj.pk, "name": user_obj.name} for user_obj in users_query_set]

    def get_publisher(self, obj):
        publisher_obj = obj.publisher
        return {"id": publisher_obj.pk, "title": publisher_obj.title}

    class Meta:
        model = Book
        fields = "__all__"
        # fields = ["id", "title", "pub_time"]
        # exclude = ["user"]
        # 分别是所有字段 包含某些字段 排除某些字段
        read_only_fields = ["id"]
        extra_kwargs = {"title": {"validators": [my_validate,]}}
```

### 用ModelSerializer改进上面Serializer的完整版

```python
class BookSerializer(serializers.ModelSerializer):
    dis_chapter = serializers.SerializerMethodField(read_only=True)
    users = serializers.SerializerMethodField(read_only=True)
    publishers = serializers.SerializerMethodField(read_only=True)

    def get_users(self, obj):
        # obj是当前序列化的book对象
        users_query_set = obj.user.all()
        return [{"id": user_obj.pk, "name": user_obj.name} for user_obj in users_query_set]

    def get_publishers(self, obj):
        publisher_obj = obj.publisher
        return {"id": publisher_obj.pk, "title": publisher_obj.title}

    def get_dis_chapter(self, obj):
        return obj.get_chapter_display()

    class Meta:
        model = Book
        # fields = "__all__"
        # 字段是有序的
        fields = ["id", "title","dis_chapter", "pub_time", "publishers", "users","chapter", "user", "publisher"]
        # exclude = ["user"]
        # 分别是所有字段 包含某些字段 排除某些字段
        read_only_fields = ["id", "dis_chapter", "users", "publishers"]
        extra_kwargs = {"title": {"validators": [my_validate,]}, "user": {"write_only": True}, "publisher": {"write_only": True},
                        "chapter": {"write_only": True}}
```



 