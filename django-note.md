## filters  

[Django Template Filters](https://docs.djangoproject.com/en/2.2/ref/templates/builtins/)

介绍几个重要的

### save

```python
context = {
        'Hello': "你好",
        "lala": "啦啦啦",
        'my_list': [1, 2, 3, 5],
        'my_html': '<h1>Hello World</h1>'
    }
```

假设有一段html源代码被传入， 如果直接用{{ my_html }}调用是不行的，没有被渲染出来

需要

```django
{{ my_html|safe}}
```

需要添加save filter才能诶渲染

## app中的template

在app中建立template可以使得这个app成为一个插件，在哪里都可以用

### 在app中建立template文件夹

这里我们的app是product， 所以在product文件夹下建立template文件夹

### 在建立的template文件夹中建立一个名称为app名字的文件夹

这里我们建立的是/products/template/products

> ---products
>
> -------template
>
> ------------products
>
> -----------------product_detail.html
>
> -----

### 在设置中给'APP_DIRS'赋值为True

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
		,
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

APP_DIRS决定了Djang是否会去寻找app中的template

**如果说根目录下的template和app中的template有相同的路径，那么Django会先去找根目录下的template中的模板**

比如 

app下的
 ![1](/home/u/Documents/blog/1.png)

根目录下
 ![2](/home/u/Documents/blog/2.png)

这样的情况下，在view.py中设置render模板的路径为'/products/product_detail.html'Django会先选择/mysite/template/products/product_detail.html, 而不是/mysite/products/product_detail.html

## forms

我们需要用户上传数据，这就需要forms

### 在app根目录下创建forms.py

```python
from django import forms

from .models import Product

class ProductForm(forms.ModelForm):
    class Meta:
        model = Product
        fields = [
            'title',
            'description',
            'price',
        ]

```

注意这边的files如果有一个是没有设置defaul、不能是空的，那么用户提交的数据将会不完整， 无法进入数据库，导致错误发生

 ### 为表单提交创建新的view

```python
def product_create_view(request):
    form = ProductForm(request.POST or None)
    if form.is_valid():
        form.save()

        # rerender it, to clear the text
        form = ProductForm()
    context = {
        'form': form
    }
    return render(request, "products/product_create.html", context)


```

其中

```python
form.save()
form.ProductForm()
```

是保存了数据之后刷新表单， 将输入框中的数据清空

### 创建html模板文件

在app目录下的/template/products中创建product_create.html



```django
{% extends 'base.html' %}
{% block content %}
    <form method="POST">
    {% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Save">
    </form>
{% endblock %}
```

### 添加到路由中

```python
path('create/', product_create_view, name="product_create"),
```

## Django POST 和 GET

<https://docs.djangoproject.com/en/2.2/ref/request-response/>

在view.py中定义的函数，第一个参数是request

当有post和get请求时， 返回的request中有GET和POST两种属性

其实他们两个都是QueryDict对象，可以通过request.POST.get() 和request.GET.get()方法获取参数

