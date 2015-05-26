---
layout: post
title: "TDDing a Flask API Part II"
date: 2015-05-23T17:20:50-04:00
---


I want to move the view functions from the `__init__.py` file into their own `marginal_review/views.py` file.  

{% highlight bash %}
.
├── Procfile
├── README.md
├── marginal_review
│   ├── __init__.py
│   ├── config.py
│   ├── models.py
│   ├── views.py
├── requirements.txt
├── runserver.py
└── test
    ├── __init__.py
    ├── test_api.py
{% endhighlight %}

So I move the one route I've got into the new `views.py` file from `__init__.py`.

{% highlight python %}
from flask import jsonify
from marginal_review import app

@app.route('/')
def say_hello():
    return jsonify(success=True)
{% endhighlight %}

Notice that I have to import `app` from `__init__` just like with `models.py`.

The new `__init__.py` file looks like this:

{% highlight python %}
from flask import Flask

app = Flask(__name__)

from marginal_review.models import db
import marginal_review.views  
{% endhighlight %}

I run the tests and they pass (the tests are importing the `app` object and don't care where the views actually are, so we don't need to change them).



