# Django Simple Serializer

---

Django Simple Serializer 是一个可以帮助开发者快速将 Django 数据或者 python data 序列化为 json|xml|raw 数据。

##为什么要用 Django Simple Serializer ?

对于序列化 Django 数据的解决方案已经有以下几种：  

###django.core.serializers
 Django内建序列化器, 它可以序列化Django model query set 但无法序列化单独的Django model数据。除此之外， 如果你的model里含有 DateTimeField , 这个序列化器同样无法使用(如果你想直接使用序列化数据)
###QuerySet.values()
 和上面一样, QuerySet.values() 同样没法工作如果你的model里有 DateTimeField 字段。
###django-rest-framework serializers
 django-rest-framework 是一个可以帮助你快速构建 REST API 的强力框架。 他拥有完善的序列化器，但在使用之前你不得不指定相应的 model serializer object。 
###django simple serializer
我们希望可以快速简单的序列化数据, 所以我设计了一种简单的方式可以不用任何额外的配置而将Django data 或者 python data 序列化为相应的数据，这就是为什么我写了 django simple serializer

----------

##运行需求


Django >= 1.4

##安装

Install using pip:

    pip install django-simple-serializer

##使用 django simple serializer 进行开发
###序列化Django data
假设我们有以下Django models：

    class Classification(models.Model):
        c_name = models.CharField(max_length=30, unique=True)
    
    class Article(models.Model):
        caption = models.CharField(max_length=50)
        classification = models.ForeignKey(Classification, related_name='cls_art')
        content = models.TextField()
        publish = models.BooleanField(default=False)

使用django simple serializer的简单例子：

    from dss.Serializer import serializer
    article_list = Article.objects.all()
    data = serializer(article_list)

data:

    [{'read_count': 0, 'create_time': 1432392456.0, 'modify_time': 1432392456.0, 'sub_caption': u'first', 'comment_count': 0, u'id': 31}, {'read_count': 0, 'create_time': 1432392499.0, 'modify_time': 1432392499.0, 'sub_caption': u'second', 'comment_count': 0, u'id': 32}]

默认情况下, 序列器会返回一个 list 或者 dict(对于单个model实例), 你可以设置参数 “output_type” 来决定序列器返回 json/xml/list.

----------

##API 手册

####dss.Serializer
提供序列器

*function* serializer(*data, datetime_format='timestamp', output_type='raw', include_attr=None, except_attr=None, foreign=False, many=False*)

####Parameters:

* data(_Required_|(QuerySet, Page, list, django model object))-待处理数据
* datetime_format(_Optional_|string)-如果包含 datetime 将 datetime 转换成相应格式.默认为 "timestamp"（时间戳）
* output_type(_Optional_|string)-serialize type. 默认“raw”原始数据，即返回list或dict
* include_attr(_Optional_|(list, tuple))-只序列化 include_attr 列表里的字段。默认为 None
* exclude_attr(_Optional_|(list, tuple))-不序列化 except_attr 列表里的字段。默认为 None
* foreign(_Optional_|bool)-是否序列化 ForeignKeyField 。include_attr 与 exclude_attr 对   ForeignKeyField 依旧有效。 默认为 False
* many(_Optional_|bool)-是否序列化 ManyToManyField 。include_attr 与 exclude_attr 对 ManyToManyField 依旧有效 默认为 False

####用法:  

**datetime_format:**  

|parameters|intro|
| --------------  | :---: |
|string|转换 datetime 为字符串。如： "2015-05-10 10:19:22"|
|timestamp|转换 datetime 为时间戳。如： "1432124420.0"|  

例子:

    from dss.Serializer import serializer
    article_list = Article.objects.all()
    data = serializer(article_list, datetime_format='string', output_type='json')

data:

    [
        {
            "read_count": 0,
            "sub_caption": "first",
            "publish": true,
            "content": "first article",
            "caption": "first",
            "comment_count": 0,
            "create_time": "2015-05-23 22:47:36",
            "modify_time": "2015-05-23 22:47:36",
            "id": 31
        },
        {
            "read_count": 0,
            "sub_caption": "second",
            "publish": false,
            "content": "second article",
            "caption": "second",
            "comment_count": 0,
            "create_time": "2015-05-23 22:48:19",
            "modify_time": "2015-05-23 22:48:19",
            "id": 32
        }
    ]

**output_type**  

|parameters|intro|
| --------------  | :---: |
|raw|转换数据为 dict 或者 list|
|dict|同 raw|  
|json|转换数据为 json|
|xml|转换数据为 xml|  

例子:

    from dss.Serializer import serializer
    article_list = Article.objects.all()[0]
    data = serializer(article_list, output_type='xml')

data:  

    <?xml version="1.0" encoding="utf-8"?>
    <root>
        <read_count>0</read_count>
        <sub_caption>first</sub_caption>
        <publish>True</publish>
        <content>first article</content>
        <caption>first</caption>
        <comment_count>0</comment_count>
        <create_time>1432392456.0</create_time>
        <modify_time>1432392456.0</modify_time>
        <id>31</id>
    </root>  

**include_attr**

例子:

    from dss.Serializer import serializer
    article_list = Article.objects.all()
    data = serializer(article_list, output_type='json', include_attr=('content', 'caption',))

data:  

    [
        {
            "content": "first article",
            "caption": "first"
        },
        {
            "content": "second article",
            "caption": "second"
        }
    ]

**exclude_attr**

例子:

    from dss.Serializer import serializer
    article_list = Article.objects.all()
    data = serializer(article_list, output_type='json', except_attr=('content',))

data:  

        [
            {
                "read_count": 0,
                "sub_caption": "first",
                "publish": true,
                "caption": "first",
                "comment_count": 0,
                "create_time": 1432392456,
                "modify_time": 1432392456,
                "id": 31
            },
            {
                "read_count": 0,
                "sub_caption": "second",
                "publish": false,
                "caption": "second",
                "comment_count": 0,
                "create_time": 1432392499,
                "modify_time": 1432392499,
                "id": 32
            }
        ]
        
**foreign**

例子:

    from dss.Serializer import serializer
    article_list = Article.objects.all()
    data = serializer(article_list, output_type='json', include_attr=('classification', 'caption', 'create_time', foreign=True)

data:  

        [
            {
                "caption": "first",
                "create_time": 1432392456,
                "classification": {
                    "create_time": 1429708506,
                    "c_name": "python",
                    "id": 1,
                    "modify_time": 1429708506
                }
            },
            {
                "caption": "second",
                "create_time": 1432392499,
                "classification": {
                    "create_time": 1430045890,
                    "c_name": "test",
                    "id": 5,
                    "modify_time": 1430045890
                }
            }
        ]

**many**

example:

    from dss.Serializer import serializer
    article_list = Article.objects.all()
    data = serializer(article_list, output_type='json', include_attr=('classification', 'caption', 'create_time', many=True)

测试数据无 ManyToManyField ，数据格式同上

####dss.Mixin
提供序列器 Mixin

*class JsonResponseMixin(object)*

####说明:

将普通class based view 转换为返回json数据的class based view，适用于DetailView等

####用法:  

例子:

    # view.py
    from dss.Mixin import JsonResponseMixin
    from django.views.generic import DetailView
    from model import Article
    
    class TestView(JsonResponseMixin, DetailView):
        model = Article
        datetime_type = 'string'
        pk_url_kwarg = 'id'
    
    
    # urls.py
    from view import TestView
    urlpatterns = patterns('',
        url(r'^test/(?P<id>(\d)+)/$', TestView.as_view()),
    )
        
访问：`localhost:8000/test/1/`

response:

    {
        "article": {
            "classification_id": 5, 
            "read_count": 0, 
            "sub_caption": "second", 
            "comments": [], 
            "content": "asdfasdfasdf", 
            "caption": "second", 
            "comment_count": 0, 
            "id": 32, 
            "publish": false
        }, 
        "object": {
            "classification_id": 5, 
            "read_count": 0, 
            "sub_caption": "second", 
            "comments": [], 
            "content": "asdfasdfasdf", 
            "caption": "second", 
            "comment_count": 0, 
            "id": 32, 
            "publish": false
        }, 
        "view": ""
    }


*class MultipleJsonResponseMixin(JsonResponseMixin):*

####说明: 

将列表类视图转换为返回json数据的类视图，适用于ListView等

####用法:  

例子:

    # view.py
    from dss.Mixin import MultipleJsonResponseMixin
    from django.views.generic import ListView
    from model import Article
    
    class TestView(MultipleJsonResponseMixin, ListView):
        model = Article
        query_set = Article.objects.all()
        paginate_by = 1
        datetime_type = 'string'
    
    
    # urls.py
    from view import TestView
    urlpatterns = patterns('',
        url(r'^test/$', TestView.as_view()),
    )
        
访问：`localhost:8000/test/`

response:

    {
        "paginator": "", 
        "article_list": [
            {
                "classification_id": 1, 
                "read_count": 2, 
                "sub_caption": "first", 
                "content": "first article", 
                "caption": "first", 
                "comment_count": 0, 
                "publish": false, 
                "id": 31
            }, 
            {
                "classification_id": 5, 
                "read_count": 0, 
                "sub_caption": "", 
                "content": "testseteset", 
                "caption": "hehe", 
                "comment_count": 0, 
                "publish": false, 
                "id": 33
            }, 
            {
                "classification_id": 5, 
                "read_count": 0, 
                "sub_caption": "second", 
                "content": "asdfasdfasdf", 
                "caption": "second", 
                "comment_count": 0, 
                "publish": false, 
                "id": 32
            }
        ], 
        "object_list": [
            {
                "classification_id": 1, 
                "read_count": 2, 
                "sub_caption": "first", 
                "content": "first article", 
                "caption": "first", 
                "comment_count": 0, 
                "publish": false, 
                "id": 31
            }, 
            {
                "classification_id": 5, 
                "read_count": 0, 
                "sub_caption": "", 
                "content": "testseteset", 
                "caption": "hehe", 
                "comment_count": 0, 
                "publish": false, 
                "id": 33
            }, 
            {
                "classification_id": 5, 
                "read_count": 0, 
                "sub_caption": "second", 
                "content": "asdfasdfasdf", 
                "caption": "second", 
                "comment_count": 0, 
                "publish": false, 
                "id": 32
            }
        ], 
        "page_obj": {
            "current": 1, 
            "next": 2, 
            "total": 3, 
            "page_range": [
                {
                    "page": 1
                }, 
                {
                    "page": 2
                }, 
                {
                    "page": 3
                }
            ], 
            "previous": null
        }, 
        "is_paginated": true, 
        "view": ""
    }

*class FormJsonResponseMixin(JsonResponseMixin):*

####说明:

将普通class based view 转换为返回json数据的class based view，适用于CreateView、UpdateView、FormView等

####用法:  

例子:

    # view.py
    from dss.Mixin import FormJsonResponseMixin
    from django.views.generic import UpdateView
    from model import Article
    
    class TestView(FormJsonResponseMixin, UpdateView):
        model = Article
        datetime_type = 'string'
        pk_url_kwarg = 'id'
    
    
    # urls.py
    from view import TestView
    urlpatterns = patterns('',
        url(r'^test/(?P<id>(\d)+)/$', TestView.as_view()),
    )
        
访问：`localhost:8000/test/1/`

response:

    {
        "article": {
            "classification_id": 5, 
            "read_count": 0, 
            "sub_caption": "second", 
            "content": "asdfasdfasdf", 
            "caption": "second", 
            "comment_count": 0, 
            "id": 32, 
            "publish": false
        }, 
        "form": [
            {
                "field": "caption"
            }, 
            {
                "field": "sub_caption"
            }, 
            {
                "field": "read_count"
            }, 
            {
                "field": "comment_count"
            }, 
            {
                "field": "classification"
            }, 
            {
                "field": "content"
            }, 
            {
                "field": "publish"
            }
        ], 
        "object": {
            "classification_id": 5, 
            "read_count": 0, 
            "sub_caption": "second", 
            "content": "asdfasdfasdf", 
            "caption": "second", 
            "comment_count": 0, 
            "id": 32, 
            "publish": false
        }, 
        "view": ""
    }


#License

Copyright © Tom Christie.

All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

Redistributions of source code must retain the above copyright notice, this
list of conditions and the following disclaimer.
Redistributions in binary form must reproduce the above copyright notice, this
list of conditions and the following disclaimer in the documentation and/or
other materials provided with the distribution.
THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.





