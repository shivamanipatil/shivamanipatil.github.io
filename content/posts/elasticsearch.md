---
title: Integrating Elasticsearch 7 to Django project
description: Adding search ability to django using Elasticsearch
date: '2019-12-18T17:09:05.139Z'
categories: []
keywords: []
slug: /@shivamanipatil10/integrating-elasticsearch-7-to-django-project-c3812de78246
---

I had to migrate the Elasticsearch 2.x using django-haystack to latest Elasticsearch 7.5.0 for a project. Here the way I did it. You can also follow if you wish to use Elasticsearch for your django project. I will be using Django 2.1.5.

Haystack is great open-source package that provides modular search for Django.Unfortunately it doesn’t support recent versions of Elasticsearch. So for migration to latest version of Elasticsearch we need to remove django-haystack package. Which can done using :

`
pip unistall django-haystack
`

Packages we need are Elasticsearch-DSL and Django Elasticsearch DSL:

1)**Elasticsearch-DSL** : This is a high level library around official low-level client(elasticsearch-py).

2)**Django-Elasticsearch-DSL** : This allows easy integration of elasticsearch- dsl-py with django.

### Installing and running Elasticsearch server on machine/server :

For debian based machines like Ubuntu, Debian etc:

```
wget [https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.0-amd64.deb](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.0-amd64.deb)

wget [https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.0-amd64.deb.sha512](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.0-amd64.deb.sha512)

shasum -a 512 -c elasticsearch-7.5.0-amd64.deb.sha512sudo 

dpkg -i elasticsearch-7.5.0-amd64.deb
```

To start Elasticsearch automatically when system boots up run following:

`
sudo /bin/systemctl daemon-reload  
sudo /bin/systemctl enable elasticsearch.service
`

To start elasticsearch service on systemd

`
sudo systemctl start elasticsearch.service
`

To stop elasticsearch service on systemd

`
sudo systemctl stop elasticsearch.service
`

To get status of elasticsearch service on systemd

`
systemctl status elasticsearch.service
`

Also to check if Elasticsearch service is working you can look localhost:9200/ in your browser. For installation guide for other os refer [https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html).

### Installing Django-Elasticsearch-DSL in django:

First install Django-Elasticsearch-DSL for django.

`
pip install django-elasticsearch-dsl
`

Remember for Elasticsearch 7.0 and later, use the major version 7 (7.x.y) of this library. Doing _pip list_ shows.

![](/piplist.png)

Add this snippet to django setting:

```
ELASTICSEARCH_DSL={  
    'default': {  
        'hosts': 'localhost:9200'  
    },  
}
```

This tells Django where our Elasticsearch service is located.

### Creating documents.py:

I had to implement searching for App and Tag models for ‘apps’ app of a django project. So following code snippets will be related to it. But you can follow same procedure for your project.

To use App and Tag models with Elasticsearch we need to subclass _django_elasticsearch_dsl.Document_ and create class Index inside it to define indices and then register it the class using _registry.register_document_ decorator. This Document subclasses are defined in documents.py in ‘apps’ directory.

Create documents.py in your app directory and include this imports:

```
from django_elasticsearch_dsl import Document, fields  
from django_elasticsearch_dsl.registries import registry  
from .models import App, Tag
```

For App model we will be searching apps using name, title, description and abstract which are fields of App model. Add this to documents.py.

```
@registry.register_document 
class AppDocument(Document):  
    class Index:  
        name = 'apps'  
        settings = {'number_of_shards': 1,  
                    'number_of_replicas': 0}  
      
    description = fields.TextField(attr = 'get_description')  
    class Django:  
        model = App  
        fields = [  
            'name',  
            'title',  
            'website',  
            'abstract',  
        ]
```

name = ‘apps’ refers to name of index and fields refers to fields with which we will be searching and are present in your model. Also model = App which refers to the model for which we are building index.Shards and replicas can be set as per your needs.

Here description is _a markdownx_() field in App model which was giving error when included in fields section so to include description field for search I had to write a function(get_description) in App model class to return description field from App model. But this is just specific to mardownx() field.

```
def get_description(self):  
        return self.description
```

And for Tag model we will be searching tags using name and identity which can be done using the same procedure.

```
@registry.register_document
class TagDocument(Document):  
    class Index:  
        name = 'tags'  
        settings = {'number_of_shards': 1,  
                    'number_of_replicas': 0}

class Django:  
        model = Tag  
        fields = [  
            'name',  
            'identity',  
              
        ]
```

After creating documents.py we build the indices.

**To create Elasticsearch indices and mappings use following command:**

`
python manage.py search_index --rebuild
`

Once this is done existings objects will mapped as well as when new objects(here apps/tags) are added to the database they will be automatically mapped with Elasticsearch indices.

### Displaying search results

Now we need to display search results. Create a search app for your project. In this project it was already created. We need to modify views.py in search app for handling the search request and render it for our template.

First import AppDocument and TagDocument from documents.py in views.py of search app:

`
from apps.documents import AppDocument, TagDocument
`

Inside search(request) method inside views.py of search app:

```
query = request.GET.get('q', '')

sqs_app = AppDocument.search().query("multi_match", query = query  
     , fields = ["name", "title", "abstract", "description"])  
sqs_tag = TagDocument.search().query("match", name = query)

app = sqs_app.to_queryset()  
tag = sqs_tag.to_queryset()
```

Here query is actual query for searching. For App we do multi-match query as there are 4 fields to be searched and for tag we do just use name field query. sqs_app and sqs_tag are search querysets of results obtained. Existing html template expected django query set. So to return this query set as django query set I used to_queryset() method . Then return the queryset to your template.

Thank you for reading :)
