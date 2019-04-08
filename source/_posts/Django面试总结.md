---
title: Django面试总结
tags:
  - Django
  - 面试
---





自己整理的一些面试题，持续更新中。。。



<!--more-->



## django、flask、tornado框架的比较？

django：
优点：最全能的web开发框架，各种功能完备，可维护性高，开发速度快
缺点：django orm 跟数据库的交互，django同步特性导致吞吐量小的问题可以通过celery解决

tornado：
优点：天生异步，性能强悍
缺点：框架提供的功能比较少，需要自己实现，这样导致了开发速度偏慢

flask：
优点：自由、灵活，扩展性强，第三方库的选择面广
缺点：但是对程序员要求更高

------

## 什么是wsgi？

<https://www.jianshu.com/p/679dee0a4193>

WSGI，描述web server如何与web application通信的一种规范

WSGI协议主要包括server和application两部分：

WSGI server负责从客户端接收请求，将request转发给application，将application返回的response返回给客户端；
WSGI application接收由server转发的request，处理请求，并将处理结果返回给server。

application中可以包括多个栈式的中间件(middlewares)，这些中间件需要同时实现server与application，因此可以在WSGI服务器与WSGI应用之间起调节作用：对服务器来说，中间件扮演应用程序，对应用程序来说，中间件扮演服务器。

------

## django请求的生命周期？

Django的请求生命周期是指当用户在浏览器上输入url到用户看到网页的这个时间段内,Django后台所发生的事情

而Django的生命周期内到底发生了什么呢??

1. 当用户在浏览器中输入url时,浏览器会生成请求头和请求体发给服务端
   请求头和请求体中会包含浏览器的动作(action),这个动作通常为get或者post,体现在url之中.
2. url经过Django中的wsgi,再经过Django的中间件,最后url到过路由映射表,在路由中一条一条进行匹配,
   一旦其中一条匹配成功就执行对应的视图函数,后面的路由就不再继续匹配了.
3. 视图函数根据客户端的请求查询相应的数据.返回给Django,然后Django把客户端想要的数据做为一个字符串返回给客户端.
4. 客户端浏览器接收到返回的数据,经过渲染后显示给用户.

------

## 列举django的内置组件

- 认证组件
- 缓存
- 日志
- 邮件
- 分页
- 静态文件管理
- 资讯聚合
- 消息框架
- 数据验证

------

## 列举django中间件的5个方法？以及django中间件的应用场景？

1. 在django中，中间件其实就是一个类，在请求到来和结束后，django会根据自己的规则在合适的时机执行中间件中相应的方法

   在django项目的settings模块中，有一个 MIDDLEWARE_CLASSES 变量，其中每一个元素就是一个中间件

   默认的中间件有哪些

   中间件中可以定义五个方法，分别是：

```python
# 方法在请求到来的时候调用
process_request(self,request)
# 在本次将要执行的View函数被调用前调用本函数
process_view(self, request, callback, callback_args, callback_kwargs)
# 需使用render()方法才会执行process_template_response
process_template_response(self,request,response)
# View函数在抛出异常时该函数被调用，得到的exception参数是实际上抛出的异常实例。通过此方法可以进行很好的错误控制，提供友好的用户界面。
process_exception(self, request, exception)
# 在执行完View函数准备将响应发到客户端前被执行
process_response(self, request, response)
```

2. django中间件的使用场景

   如果你想修改请求，例如被传送到view中的HttpRequest对象

   或者你想修改view返回的HttpResponse对象，这些都可以通过中间件来实现

   可能你还想在view执行之前做一些操作，这种情况也可以用 middleware来实现。
   比如我们写一个判断浏览器来源，是pc还是手机，这里手机我们采用iphone，因为暂时没有其他设备。我们有不想把这个逻辑加到视图函数里，想作为一个通用服务，作为一个可插拔的组件被使用，最好的方法就是实现为中间件

   或者说做一个拦截器，发现一定的时间内某个ip对网页的访问次数过多，则将其加入黑名单

------

## 简述什么是FBV和CBV？

**FBV（function base views） 基于函数的视图**
**CBV（class base views） 基于类的视图**

使用fbv的模式,在url匹配成功之后,会直接执行对应的视图函数

使用cbv模式,在url匹配成功之后,会找到视图函数中对应的类,然后这个类回到请求头中找到对应的Request Method

用户发送url请求,Django会依次遍历路由映射表中的所有记录,一旦路由映射表其中的一条匹配成功了,就执行视图函数中对应的函数名,这是fbv的执行流程

当服务端使用cbv模式的时候,用户发给服务端的请求包含url和method,这两个信息都是字符串类型
服务端通过路由映射表匹配成功后会自动去找dispatch方法,然后Django会通过dispatch反射的方式找到类中对应的方法并执行
类中的方法执行完毕之后,会把客户端想要的数据返回给dispatch方法,由dispatch方法把数据返回经客户端

------

## django的request对象是在什么时候创建的？

当请求一个页面时,Django创建一个 HttpRequest 对象.该对象包含 request 的元数据. 然后 Django 调用相应的 view 函数(HttpRequest 对象自动传递给该view函数<作为第一个参数>), 每一个 view 负责返回一个 HttpResponse 对象.

requests的元数据包括path，get，put等方法，cookies，user等等

------

## 如何给CBV的程序添加装饰器？

CBV添加装饰器有两种方法

1. 在指定方法上添加装饰器
2. 在类上添加，但是要用name来指定方法

```PYTHON
## CBV中添加装饰器
def wrapper(func):
    def inner(*args,**kwargs):
        return func(*args,**kwargs)
    return inner
    
# 1. 指定方法上添加装饰器
class Foo(View):

    @method_decorator(wrapper)
    def get(self,request):
        pass
    def post(self,request):
        pass

2. 在类上添加

@method_decorator(wrapper,name='dispatch')
class Foo(View):
    def get(self,request):
        pass
def post(self,request):
        pass
```

总结

1. 添加装饰器前必须导入from django.utils.decorators import method_decorator
2. 添加装饰器的格式必须为@method_decorator()，括号里面为装饰器的函数名
3. 给类添加是必须声明name
4. 注意csrf-token装饰器的特殊性，它只能加在dispatch上面

------

## 列举django orm 中所有的方法（QuerySet对象的所有方法）

从数据库中查询出来的结果一般是一个集合，这个集合叫做 QuerySet

- filter 过滤
- exclude 排除
- annotate 聚合
- order_by 排序
- reverse 反向排序
- distinct 去除查询结果中重复的行
- values 迭代时返回字典而不是模型实例对象
- values_list 迭代时返回元组而不是字典
- dates 表示特定种类的所有可用日期
- datetimes 表示特定种类的所有可用日期
- none 不返回任何对象
- all 返回所有结果
- select_related 外键查询
- prefetch_related 在单个批处理中自动检索每个指定查找的相关对象
- defer 告诉django不要查询某些字段
- using 多个数据库时控制QuerySet在哪个数据库上求值

------

## only和defer的区别？

在复杂的情况下，表中可能有些字段内容非常多，取出来转化成 Python 对象会占用大量的资源。
这时候可以用 defer 来排除这些字段，比如我们在文章列表页，只需要文章的标题和作者，没有必要把文章的内容也获取出来（因为会转换成python对象，浪费内存）

和 defer 相反，only 用于取出需要的字段，假如我们只需要查出 作者的名称

------

## `select_related`和`prefetch_related`的区别？

<https://hk.saowen.com/a/bb124ab70580b722d7840c7c0377a326ac4ce022dc653f4cbc7e0ae7fc245232>

------

## filter和exclude的区别？

filter 设置要查询的字段
exclude 设置不要查询的字段

------

## 列举django orm中三种能写sql语句的方法

1. 使用extra：查询人民邮电出版社出版并且价格大于50元的书籍

```
Book.objects.filter(publisher__name='人民邮电出版社').extra(where=['price>50']) 
```

1. 使用raw

```
books=Book.objects.raw('select * from hello_book')  
for book in books:  
   print book  
```

1. 自定义sql

```
from django.db import connection  
  
cursor = connection.cursor()  
cursor.execute("insert into hello_author(name) VALUES ('郭敬明')")  
cursor.execute("update hello_author set name='韩寒' WHERE name='郭敬明'")  
cursor.execute("delete from hello_author where name='韩寒'")  
cursor.execute("select * from hello_author")  
cursor.fetchone()  
cursor.fetchall() 
```

------

## django orm 中如何设置读写分离？

<https://blog.csdn.net/Ayhan_huang/article/details/78784486>

<https://my.oschina.net/candiesyangyang/blog/203425>

------

## F和Q的作用?

F作用：操作数据表中的某列值，F()允许Django在未实际链接数据的情况下具有对数据库字段的值的引用，不用获取对象放在内存中再对字段进行操作，直接执行原生产sql语句操作

使用场景：对数据库中的所有的商品，在原价格的基础上涨价10元

```
from django.db.models import F
from app01.models import Book
Book.objects.update(price=F("price")+20)  # 对于book表中每本书的价格都在原价格的基础上增加20元
```

Q作用：对对象进行复杂查询，并支持&（and）,|（or），~（not）操作符

使用场景：filter查询条件只有一个，而使用Q可以设置多个查询条件

```
from django.db.models import Q
search_obj=Asset.objects.filter(Q(hostname__icontains=keyword)|Q(ip=keyword))
```

当同时使用filter的关键字查询和Q查询时，一定要把Q对象放在前面

```
Asset.objects.get(
Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),question__startswith='Who')
```

## values和values_list的区别

values 返回字典而不是模型查询对象

values-list 跟values一样，但是返回的是元组

<https://blog.csdn.net/weixin_40475396/article/details/79529256>

## 如何使用django orm批量创建数据？

如果使用django save()创建数据，则每次save的时候都会访问一次数据库

```
for i in resultlist:
    p = Account(name=i)
    p.save()
```

django1.4之后加入新特性可以批量创建对象，减少SQL查询次数

```
querysetlist=[]
for i in resultlist:
    querysetlist.append(Account(name=i))
Account.objects.bulk_create(querysetlist)
```

## django的Form和ModeForm的作用？

<http://www.cnblogs.com/caochao-/articles/8412830.html>

表单的作用是收集元素中的内容

Form 需要自己定义表单的字段

ModelForm 根据model来生成表单的字段

## django的Form组件中，如果字段中包含choices参数，请使用两种方式实现数据源实时更新

此问题只适用于From组件，ModelFrom组件不用考虑(自身已经解决)

示例：例如choice类型字段，添加了新的数据，而在页面中不能显示出来，只有再次刷新页面才能获取最新的数据，因为程序运行时静态字段只加载一次, choice的数据如果从数据库获取可能会造成数据无法实时更新

```python
# models.py
from django.db import models

class UserType(models.Model):
    title = models.CharField(max_length=32)
　　　
　　def __str__(self):
　　　　return self.title


class UserInfo(models.Model):
    name = models.CharField(max_length=32)
    email = models.CharField(max_length=32)
    ut = models.ForeignKey(to='UserType')
```

方法一：重写构造方法：

```python
# views.py      

from django.forms import Form
from django.forms import fields

class UserForm(Form):
    name = fields.CharField(label='用户名',max_length=32)
    email = fields.EmailField(label='邮箱')
    ut_id = fields.ChoiceField(
        # choices=[(1,'二笔用户'),(2,'闷骚')]
        choices=[]
    )

    def __init__(self,*args,**kwargs):
        super(UserForm,self).__init__(*args,**kwargs)
        
        # 每次实例化，重新去数据库获取数据并更新
        self.fields['ut_id'].choices = models.UserType.objects.all().values_list('id','title')

def user(request):
    if request.method == "GET":
        form = UserForm()
        return render(request,'user.html',{'form':form})
```

方法二：使用ModelChoiceField字段

```python
# views.py

from django.forms import Form
from django.forms import fields
from django.forms.models import ModelChoiceField
class UserForm(Form):
    name = fields.CharField(label='用户名',max_length=32)
    email = fields.EmailField(label='邮箱')
    ut_id = ModelChoiceField(queryset=models.UserType.objects.all())    
```

### django的Model中的ForeignKey字段中的on_delete参数有什么作用？

当一个被ForeignKey引用的对象删除后，django将会通过指定on_delete参数来仿真sql约束的行为

例如，如果你有一个可以为空的ForeignKey，在其引用的对象被删除的时你想把这个ForeignKey 设置为空：

```
user = models.ForeignKey(User, blank=True, null=True, on_delete=models.SET_NULL)
```

`on_delete`有`CASCADE`、`PROTECT`、`SET_NULL`、`SET_DEFAULT`、`SET()`五个可选择的值

1. CASCADE: 级联删除；默认值
2. PROTECT: 抛出ProtectedError 以阻止被引用对象的删除，它是django.db.IntegrityError 的一个子类
3. SET_NULL: 把ForeignKey 设置为null； null 参数为True 时才可以这样做
4. SET_DEFAULT: ForeignKey 值设置成它的默认值；此时必须设置ForeignKey 的default 参数
5. SET: 设置ForeignKey 为传递给SET() 的值，如果传递的是一个可调用对象，则为调用后的结果。在大部分情形下，[传递一个可调用对象用于避免models.py](http://xn--models-9m7igl44aw5bf6lb5io91a1s4fba7274gzkb249ap2b.py/) 在导入时执行查询

## django中csrf的实现机制？

<https://blog.csdn.net/u011715678/article/details/48752873>

<https://www.jianshu.com/p/991df812e2a5?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation>

## django如何实现websocket？

<http://www.ruanyifeng.com/blog/2017/05/websocket.html>

HTTP 协议有一个缺陷：通信只能由客户端发起

WebSocket 协议它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种

django实现websocket有多种方式：

1. 使用Channels实现websocket

<https://www.jianshu.com/p/3de90e457bb4>

1. 使用dwebsocket实现Websocket

<https://www.cnblogs.com/huguodong/p/6611602.html>

<http://gtcsq.readthedocs.io/en/latest/others/websocket.html>

## 基于django使用ajax发送post请求时，都可以使用哪种方法携带csrf token？

1、写在Ajax beforeSend

```
ajax（｛
　　xxx:xxx,
　　beforeSend:function(xhr, settings){
　　　　xhr.setRequestHeader("x-CSRFToken", "{{ csrf_token }}");
　　},
｝）
```

2、写到Ajax Data

```
ajax（｛
　　xxx:xxx,
　　data: $("#form").serialize(),
｝）
```

3、写到ajaxSetup

```
$("#add-business-form").submit(function () {
            $.ajaxSetup({
            data: {csrfmiddlewaretoken: '{{ csrf_token }}'}
            });
            $.ajax({
               xxx:xxxx,
            });
            return false;
        });
```

4、KindEditor携带CSRF

```
<script>
    var csrfitems = document.getElementsByName("csrfmiddlewaretoken");
    var csrftoken = "";
    if(csrfitems.length > 0)
    {
        csrftoken = csrfitems[0].value;
    }
    $(function () {
            initKindEditor();
        });
    function initKindEditor() {
        $.ajaxSetup({
        data: {csrfmiddlewaretoken: '{{ csrf_token }}'}
        });
        var kind = KindEditor.create('#content', {
            width: '100%',       // 文本框宽度(可以百分比或像素)
            height: '300px',     // 文本框高度(只能像素)
            minWidth: 200,       // 最小宽度（数字）
            minHeight: 400,      // 最小高度（数字）
            uploadJson: '{% url "upload_image" %}',
            extraFileUploadParams : {
                csrfmiddlewaretoken:csrftoken
            }
        });
    }
</script>
```

## django中如何实现orm表中添加数据时创建一条日志记录

1. 使用信号记录
2. 使用logger

<https://blog.csdn.net/apple9005/article/details/73608994>

## django缓存如何设置？

<https://www.cnblogs.com/linxiyue/p/7494540.html>

## django的缓存能使用redis吗？如果可以的话，如何配置？

<https://www.jianshu.com/p/04ef84c3fe3b>

## django路由系统中name的作用？

<https://www.cnblogs.com/no13bus/p/3767521.html>

<https://code.ziqiangxuetang.com/django/django-url-name.html>

## django的模板中filter和simple_tag的区别？

<https://blog.csdn.net/huanhuanq1209/article/details/77756446>

## django-debug-toolbar的作用？

<https://www.cnblogs.com/Lands-ljk/p/5506766.html>

## django中如何实现单元测试？

<https://www.jianshu.com/p/15af33d2c2c4>

## 解释orm中 db first 和 code first的含义

db first：现有数据库，[然后从数据库反向生成models.py](http://xn--models-ht8i645axqa6a510yi0i5neuviwr3c2cn.py/)

code first: [现有models.py](http://xn--models-283m094f.py/) 再进行数据库操作

## django中如何根据数据库表生成model中的类

<https://www.jianshu.com/p/037bd7e20a7a>

## 使用orm和原生sql的优缺点？

使用 ORM 最大的优点就是快速开发，让我们将更多的精力放在业务上而不是数据库上，下面是 ORM 的几个优点

1. 隐藏了数据访问细节，使通用数据库交互变得简单易行。同时 ORM 避免了不规范、冗余、风格不统一的 SQL 语句，可以避免很多人为的 bug，方便编码风格的统一和后期维护。
2. 将数据库表和对象模型关联，我们只需针对相关的对象模型进行编码，无须考虑对象模型和数据库表之间的转化，大大提高了程序的开发效率。
3. 方便数据库的迁移。当需要迁移到新的数据库时，不需要修改对象模型，只需要修改数据库的配置。

ORM 的最令人诟病的地方就是性能问题，不过现在已经提高了很多，下面是 ORM 的几个缺点

1. 性能问题
   1. 自动化进行数据库关系的映射需要消耗系统资源
   2. 程序员编码
   3. 在处理多表联查、where 条件复杂的查询时，ORM 可能会生成的效率低下的 SQL
   4. 通过 Lazy load 和 Cache 很大程度上改善了性能问题
2. SQL 调优，SQL 语句是由 ORM 框架自动生成，虽然减少了 SQL 语句错误的发生，但是也给 SQL 调优带来了困难。
3. 越是功能强大的 ORM 越消耗内存，因为一个 ORM Object 会带有很多成员变量和成员函数。
4. 对象和关系之间并不是完美映射
   一般来说 ORM 足以满足我们的需求，如果对性能要求特别高或者查询十分复杂，可以考虑使用原生 SQL 和 ORM 共用的方式

使用原生sql优点：

1. 进行复杂的查询时更加灵活
2. 可以根据需要编写特殊的sql语句

使用原生sql缺点：

1. 需要对输入进行严格的检测
2. 自己写的sql语句，很多时候使用的是字符串拼接，可能会有sql注入的漏洞
3. 不能使用django orm相关的一些特性

## 简述MVC和MTV

[http://www.dongwm.com/archives/浅谈MVC、MTV和MVVM/](http://www.dongwm.com/archives/%E6%B5%85%E8%B0%88MVC%E3%80%81MTV%E5%92%8CMVVM/)

## django的contenttype组件的作用？

<https://blog.csdn.net/Ayhan_huang/article/details/78626957>

<https://juejin.im/entry/581da04f128fe1005afdf618>

## 谈谈你对restfull 规范的认识？

<http://www.ruanyifeng.com/blog/2014/05/restful_api.html>

## 接口的幂等性是什么意思？

<https://www.jianshu.com/p/b09a2e9bcd29>

## 什么是RPC？

<https://www.jianshu.com/p/2accc2840a1b>

## Http和Https的区别？

<https://juejin.im/entry/58d7635e5c497d0057fae036>

## 为什么要使用django rest framework框架？

为什么要使用REST framework？

- 在线可视的API，对于赢得你的开发者们十分有用

验证策略涵盖了OAuth1a和OAuth2

同时支持ORM和非ORM数据源的序列化

可以配置各个环节，若无需更多强大的特性，使用一般基于方法（function-based）的视图（views）即可

大量的文档，强力的社区支持

大公司如同Mozilla和Eventbrite，也是忠实的使用者

## django rest framework框架中都有那些组件？

**1. 路由**

- 可以通过as_view传参数，根据请求方式不同执行相应的方法
- 可以在url中设置一个结尾，类似于： .json

**2. 视图**

- 帮助开发者提供了一些类，并在类中提供了多个方法以供我们使用

**3. 版本**

- 在url中设置version参数，用户请求时候传入参数。在request.version中获取版本，根据版本不同做不同处理

**4. 认证**
写一个类并注册到认证类，在类的的authticate方法中编写认证逻辑。

- 认证成功（user,auth）
- raise AuthticateFaild(….)
- None

**5. 权限**
写一个类并注册到权限类，在类的的has_permission方法中编写认证逻辑。

- True
- False

**6. 频率限制**
写一个类并注册到频率类，在类的的 allow_request/wait 方法中编写认证逻辑。

```
allow_request
	 True 
	 False  如果返回False，那么就要执行wait
```



**7. 解析器**

- 根据ContentType请求头，选择不同解析器对 请求体中的数据进行解析。
- POST /index/ http1.1.\r\nhost:11.11.11.11\r\nContent-Type:url-formendo…. \r\n\r\nuser=alex&age=123
- POST /index/ http1.1.\r\nhost:11.11.11.11\r\nContent-Type:application/json\r\n\r\n{….}

**8. 分页**

- 对从数据库中获取到的数据进行分页处理: SQL -> limit offset
  - 根据页码：<http://www.luffycity.com/api/v1/student/?page=1&size=10>
  - 根据索引：<http://www.luffycity.com/api/v1/student/?offset=60&limit=10>
  - 根据加密：<http://www.luffycity.com/api/v1/student/?page=erd8

## django models中null和blank得区别

`null` 是针对数据库而言，如果 `null=True`, 表示数据库的该字段可以为空

`blank` 是针对表单的，如果 `blank=True`，表示你的表单填写该字段的时候可以不填，比如 `admin` 界面下增加 `model` 一条记录的时候。直观的看到就是该字段不是粗体



### django rest framework框架中的视图都可以继承哪些类？

APIview、ViewSets、Generic views

### 简述 django rest framework框架的认证流程。

https://www.cnblogs.com/renpingsheng/p/9534984.html

### django rest framework如何实现的用户访问频率控制？