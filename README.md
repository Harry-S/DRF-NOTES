<!-- TOC -->

- [今日概要](#%E4%BB%8A%E6%97%A5%E6%A6%82%E8%A6%81)
- [知识点回顾](#%E7%9F%A5%E8%AF%86%E7%82%B9%E5%9B%9E%E9%A1%BE)
  - [开发模式](#%E5%BC%80%E5%8F%91%E6%A8%A1%E5%BC%8F)
  - [后端开发](#%E5%90%8E%E7%AB%AF%E5%BC%80%E5%8F%91)
  - [Django URL请求处理方式（FBV、CBV）](#django-url%E8%AF%B7%E6%B1%82%E5%A4%84%E7%90%86%E6%96%B9%E5%BC%8Ffbvcbv)
  - [列表生成式](#%E5%88%97%E8%A1%A8%E7%94%9F%E6%88%90%E5%BC%8F)
  - [面向对象](#%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1)
  - [面试题](#%E9%9D%A2%E8%AF%95%E9%A2%98)
- [接口开发](#%E6%8E%A5%E5%8F%A3%E5%BC%80%E5%8F%91)
  - [普通模式（非Restful）](#%E6%99%AE%E9%80%9A%E6%A8%A1%E5%BC%8F%E9%9D%9Erestful)
  - [Restful规范（建议使用）](#restful%E8%A7%84%E8%8C%83%E5%BB%BA%E8%AE%AE%E4%BD%BF%E7%94%A8)
    - [规范详解（http://www.cnblogs.com/wupeiqi/articles/7805382.html）](#%E8%A7%84%E8%8C%83%E8%AF%A6%E8%A7%A3httpwwwcnblogscomwupeiqiarticles7805382html)

<!-- /TOC -->

# 今日概要

- Restful规范（建议）
- Django Rest Framework框架

# 知识点回顾

## 开发模式

- 普通开发方式（前后端放在一起写）
- 前后端分离

## 后端开发

- 为前端提供URL（API/接口的开发）
- 备注：永远返回HttpResponse

## Django URL请求处理方式（FBV、CBV）

- **内部实现原理**
  - HTTP请求的本质是基于socket来做的

```python
# 知识点补充 - getattr
# getattr(object, name[,default])函数用于获取对象object的属性或者方法
# 语法：
    getattr(object, name[, default])
'''
参数
    1. object - 对象
    2. name - 字符串，对象属性
    3. default - 默认返回值，如果不提供该参数，在没有对应属性时，将触发AttributeError
'''

class A(obj):
    bar = 1

a = A()
print(getattr(a, 'bar'))        # 打印 “1”
print(getattr(a, 'attr', 2))    # 属性attr不存在，但设置了默认值，打印 “2”
```

- **FBV - Function Base View**

```python
# 路由
url(r'^xx/', views.users)

# 视图
def users(request):
    if request.method == "GET":
        pass
    elif request.method == "POST":
    return HttpResponse(json.dumps(...))
```

- **CBV - Class Base View**
  - 本质：基于反射实现根据请求方式不同，执行不同的方法
    - 语法：getattr(object, request.method)，去一些对象中匹配方法
    - 原理：url -> view方法 -> dispatch方法（反射执行其他:GET/POST/PUT/PATCH/DELETE/HEAD/TRACE/OPTIONS）
    - django内置模块：**django.views.generic.base**
  - 流程：
    1. `请求`
    2. `路由（as_view方法）`
    3. `view方法`
    4. `dispatch方法（里面做反射）`
  - 技巧：
    - 取消CSRF Toekn认证方式，装饰器要加到dispatch方法上，且需要method_decrator装饰

```python
# 路由
url(r'^xx/', views.StudentView.as_view())

# 视图
from django.views import View
class MyBasicView(object):
    def dispatch(self, request, *args, **kwargs):
        print('before')
        ret = super(MyBasicView, self).dispatch(request, *args, **kwargs)
        print('after')
        return ret

class StudentView(MyBasicView, View):
    # 方式3：注释dispatch方法
    def dispatch(self, request, *args, **kwargs):
        # 方式1
        '''
        func = getattr(self, request.method)
        ret = func(request, *args, **kwargs)
        return ret
        '''
        # 方式2
        '''
        ret = super(StudentsView, self).dispatch(request, *args, **kwargs)
        return ret
        '''
    def get(self, request, *args, **kwargs):
        pass
    def post(self, request, *args, **kwargs):
        pass
    def put(self, request, *args, **kwargs):
        pass
    def dete(self, request, *args, **kwargs):
        pass
```

## 列表生成式

```python
class Foo：
    pass

class Bar:
    pass

# 对象列表 - 方式1
v = [item() for item in [Foo, Bar]]

# 对象列表 - 方式2
v = []
for i in [Foo, Bar]:
    obj = i()
    v.append(obj)
```

## 面向对象

- 多态
- 继承: 多个类共用的功能，为了避免重复编写
- 封装
  - 对同一类方法封装到**类**中
  - 将数据封装到**对象**中

```python
class File
    # 文件增删改查方法
    def __init__(self, age, address):
        self.age = age
        self.address = address
    def get():
        pass
    def delete():
        pass
    def update():
        pass

class DB:
    # 数据库的方法

obj1 = File(17, 'SZ')
obj2 = File(18, 'GZ')    # 把数据18和'GZ'封装到一个对象里
```

```python
class Request(object):
    def __init__(self, obj):
        self.obj = obj

    @property   # 这里在执行user函数的时候，就不需要加()号
    def user(self):
        return self.obj.authicate()

class Auth(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age
    def authticate(self):
        return self.name
        # return True

class APIView(object):
    def dispatch(self):
        self.f2()
    def f2(self):
        a = Auth('alex', 18)
        b = Auth('andrew', 18)
        # req = Request(a)
        req = Request(b)
        print(req.obj)  # Auth对象
        print(req.user) # 打印 True

obj = APIView()
obj.dispatch()
```

## 面试题

- **django中间件函数有什么？（最多5个方法）**
  - `process_request`
  - `process_view`
  - `process_response`
  - `process_exception`
  - `process_render_template`
- **使用中间件做过什么**
  - 权限
  - 用户登录验证
  - Django的CSRF Token（如何实现？）
    - 基于中间件的**process_view方法**认证
      - 去请求体或cookies中获取token,然后根据token做校验
      - 在执行视图函数之前，判断是否带了随机字符串; 或者检查视图是否被@csrf_exempt装饰（代表免除csrf认证）。因为在**process_request方法**中，还没有拿到视图函数，所以如果需要使用csrf_excempt等的装饰器，就必须放在processs_view方法中
    - 装饰器给单独函数进行设置（认证或无需认证）
- **CSRF token的两种设置方式**
  - 中间件
  - 装饰器

```python
# 情况一：全局都用认证，某几个不用认证
# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',    # 全站使用csrf认证
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# 视图 - FBV
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def users(request):
    pass

# 视图 - CBV - 方式1
from django.utils.decorators import method_decorator
from django.views.decorators.csrf import csrf_exempt

class StudentView(MyBasicView, View):
    @method_decorator(csrf_exempt)  # 必须装饰在dispatch方法中，不可以装饰到其他单独的方法中
    def dispatch(self, request, *args, **kwargs):
        ret = super(StudentsView, self).dispatch(request, *args, **kwargs)
        return ret
    def get(self, request, *args, **kwargs):
        pass
    def post(self, request, *args, **kwargs):
        pass
    def put(self, request, *args, **kwargs):
        pass
    def dete(self, request, *args, **kwargs):
        pass

# 视图 - CBV - 方式2
from django.utils.decorators import method_decorator
from django.views.decorators.csrf import csrf_exempt

@method_decorator(csrf_exempt, name='dispatch')
class StudentView(MyBasicView, View):
    def dispatch(self, request, *args, **kwargs):
        ret = super(StudentsView, self).dispatch(request, *args, **kwargs)
        return ret
    def get(self, request, *args, **kwargs):
        pass
    def post(self, request, *args, **kwargs):
        pass
    def put(self, request, *args, **kwargs):
        pass
    def dete(self, request, *args, **kwargs):
        pass

```

```python
# 情况二：全局都不用认证，某几个用认证
# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# 视图 - FBV
from django.views.decorators.csrf import csrf_protect

@csrf_protect
def users(request):
    pass
```

# 接口开发

- 出发点：对某项资源做增删改查！后端开发接口给前端调用

## 普通模式（非Restful）

- 问题：
  - 对相同资源，不同的动作（增删改查），会有不同的URL
  - URL数量会越来越多（比如对10张表做增删改查，基本上就要有40个URLs）
  - 个人感觉：
    - 是对一个操作方式发出一个请求，然后再考虑资源。譬如：`我要做查询操作，针对订单资源`

```python
# 路由
url(r'^get_order/', views.get_order)
url(r'^add_order/', views.add_order)
url(r'^del_order/', views.del_order)
url(r'^update_order/', views.update_order)

# 视图
def get_order(request):
    return HttpResponse('')

def add_order(request):
    return HttpResponse('')

def del_order(request):
    return HttpResponse('')

def update_order(request):
    return HttpResponse('')
```

## Restful规范（建议使用）

- 对上述普通模式，约定的规范和建议
  - 对应request.method的不同，让接口做出不同的响应
  - 重要：对于一条数据（一个订单），对应一个URL
    - 怎么区分增删改查？根据methods的不同来区分！
  - 个人感觉：
    - 是对一种资源，发起一个请求操作。譬如：`我要针对订单资源，做查询操作`

```python
# 路由
url(r'^order/', views.order)

# FBV方式 - 视图 - 根据method不同做不同的操作
def order(request):
    if request.method == "GET":
        return HttpResponse('获取订单')
    elif request.method == "POST":
        return HttpResponse('新建订单')
    elif request.method == "PUT":
        return HttpResponse('更新订单')
    elif request.method == "DELETE":
        return HttpResponse('删除订单')
```

```python
# 路由
url(r'^order/', views.OrderView.as_view())

# CBV方式 - 视图
from django.views.generic import View
class OrderView(View):
    def get(self, request, *args, **kwargs):
        return HttpResponse('获取订单')
    def post(self, request, *args, **kwargs):
        return HttpResponse('新建订单')
    def put(self, request, *args,, **kwargs):
        return HttpResponse('更新订单')
    def delete(self, request, *args,, **kwargs):
        return HttpResponse('删除订单')
```

### 规范详解（http://www.cnblogs.com/wupeiqi/articles/7805382.html）

- API与用户的通信协议，总是使用HTTPS协议（推荐）
- 域名
  - `https://api.example.com`     子域名的方式（需要解决跨域问题）
  - `https://example.org/api/`    URL方式（同一个域名，只是URL不同）
- 版本
  - URL（建议）
    - `https://example.org/api/v1/`
    - `https://example.org/api/v2/`
    - `https://example.org/api/v3/`
  - 请求头
- 面向资源编程
  - 本质：把网络上任何东西都认为是**资源（均使用名词表示，也可复数）**，对这个资源可以做增删改查
  - https://example.org/api/v1/zoos
  - https://example.org/api/v1/animals
  - https://example.org/api/v1/employees
- Request Method
  - GET：从服务器去除资源（一项或多项）
  - POST：在服务器新建一个资源
  - PUT：在服务器更新资源（客户端提供改变后的完整资源）
  - PATCH：在服务器更新资源（客户端提供改变的属性）
  - DELETE：从服务器删除资源
