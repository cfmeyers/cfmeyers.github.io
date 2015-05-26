---
layout: post
title: "TDDing a Flask API Part I"
date: 2015-05-20T21:56:43-04:00
---

#Concept:  

I'm a big fan of the blog [Marginal Revolution](http://marginalrevolution.com/), written by two economists at George Mason.  One in particular, [Tyler Cowen](http://en.wikipedia.org/wiki/Tyler_Cowen), reads a ridiculous number of books each week.   

I built [MR-Reviews-API](https://github.com/cfmeyers/mr-reviews-api), a JSON API for reviews of books and other media on Marginal Revolution, with Flask, SQLAlchemy, Postgres, and Heroku.  

For example, here is a subset of what's returned from [https://marginal-review-api.herokuapp.com/api/v1/reviews?author=Borges](https://marginal-review-api.herokuapp.com/api/v1/reviews?author=Borges)

{% highlight javascript %}
results: [
    { author: "Tyler Cowen",
    date: "2008-05-02",
    genres: "Authors Essays Criticism & Theory United States Caribbean & Latin American Literary",
    item_asin: "0140290117",
    item_author: "Jorge Luis Borges",
    item_image_url: "http://ecx.images-amazon.com/images/I/51uh24-JIqL._SL160_.jpg",
    item_title: "Borges: Selected Non-Fictions",
    post_url: "http://marginalrevolution.com/marginalrevolution/2008/05/markets-in-ever.html" },

    { author: "Tyler Cowen",
    date: "2004-09-15",
    genres: "Short Stories Literary Reference",
    item_asin: "0802130305",
    item_author: "Jorge Luis Borges",
    item_image_url: "http://ecx.images-amazon.com/images/I/51IAM0xHjXL._SL160_.jpg",
    item_title: "Ficciones",
    post_url: "http://marginalrevolution.com/marginalrevolution/2004/09/the_politics_of.html" }
    ]
}
{% endhighlight %}

The `item_image_url` points to the image on Amazon.  The `item_author` is the book author, and the `author` is the blog-post-review author.  `post_url` is the link to the blog post with the review.  The contents of the database were put together by indexing the [Marginal Revolution](http://marginalrevolution.com/) blog with BeautifulSoup and querying the Amazon API using the extremely easy-to-use [Python Amazon Simple Product API](https://github.com/yoavaviram/python-amazon-simple-product-api).  At a later date I plan to subscribe to the RSS feed and parse it on the fly on my server.

This post is a step-by-step of how I TDD'd the Flask app that serves up the API.

#Initial Setup

Initialize a repo with an `app.py` file and `test` directory (with its `__init__.py` and `test_api.py`):

{% highlight bash %}
.
├── app.py
└── test
    ├── __init__.py
    └── test_api.py
{% endhighlight %}

Next create a virtualenv (using virtualenvwrapper) called 'marginal-review':

{% highlight bash %}
mkvirtualenv marginal-review
{% endhighlight %}

`pip install` my testing libraries (nose is a Python test runner, rednose is an addon for nose that gives you colorized output to the terminal):

{% highlight bash %}
pip install nose rednose 
{% endhighlight %}

##Wiring up the tests

Our initial `test_api.py` is 

{% highlight python %}
import unittest

class TestCase(unittest.TestCase):

    def setUp(self):
        self.app = app.test_client()
        
    def tearDown(self):
        pass

    def test_wired_up(self):
        with self.app as c:
            response = c.get('/')
            data = json.loads(response.data)
            self.assertSequenceEqual( {'success':'hello world'}, data )
{% endhighlight %}

###Initial Errors:

1.)  `NameError: global name 'app' is not defined`, fixed with a quick import in the test file, `from app import app` 

2.)  `ImportError: cannot import name app`, fixed with `app.py`: 

        {% highlight python %}
        from flask import Flask, jsonify

        app = Flask(__name__)
        {% endhighlight %}

3.)  `ImportError: No module named flask`, fixed with `pip install flask`

4.)  `ValueError: No JSON object could be decoded` is our first real error, prompting us to actually set up the 'hello world' route we're testing.

        {% highlight python %}
        from flask import Flask, jsonify

        app = Flask(__name__)

        @app.route('/')
        def hello_world():
            return jsonify({'success':'hello world'})
        {% endhighlight %}

Now our "wired up" test passes.  This is a good point to stop and commit our changes.

##Initial Deploy to Heroku

This section assumes you have Heroku toolbelt installed.

-  Create the app on Heroku with a name:  `heroku create marginal-review-api`

-  Install Gunicorn using `pip install gunicorn`

-  Create a requirements.txt file for Heroku to use to download dependencies (if you've been using virtualenv, this is as easy as `pip freeze > requirements.txt`).


We'll need to create a file called "Procfile" in the root directory that will tell Heroku how to start up the app.

{% highlight bash %}
web: gunicorn app:app --log-file=-
{% endhighlight %}

Notice the structure of that line:  `app:app`;  the first `app` refers to the module (e.g. `app.py`) where the Flask "`app`" gets created. (explained [here](http://stackoverflow.com/questions/10670958/procfile-gunicorn-custom-module-name)).

You can test this part out on your dev machine by running `foreman start`; this should launch a server running at port 5000, just as if you'd run `python runserver.py`.  (You need to have the foreman gem installed for this to work.)

Now commit all that to master, run `git push heroku master`.

Test it out with `heroku open`.  This'll open up your default web browser to the page your app is deployed at on Heroku, and let you test it out.  We haven't hooked up the database yet, but you should still be able to visit `/` and see the correct JSON response.


##Set up the database

I'm not going to be TDDing the SQLAlchemy models and database.  If this were a Rails app and I were using ActiveRecord and RSpec, I'd gradually add fields and methods for my models, running migrations along the way.  SQLAlchemy doesn't have built in migrations, and given the scale of the project adding migrations would be overkill.  


{% highlight bash %}
pip install flask-sqlalchemy  
{% endhighlight %}

The only model-specific test I'm going to write is for a `to_dict` method on the `Review` model:

{% highlight python %}

    def test_review_to_dict_method(self):
        review_info = {'item_author':'Jorge Luis Borges', 'item_title':'Labrynthes'}
        review1 = Review(**review_info)
        self.assertEqual(review_info['item_author'], review1.to_dict()['item_author'])
        self.assertEqual(review_info['item_title'], review1.to_dict()['item_title'])

{% endhighlight %}

These pass with the following model classes, defined in the `app.py`:

{% highlight python %}
from flask import Flask, jsonify
from flask.ext.sqlalchemy import SQLAlchemy
from sqlalchemy.orm import relationship

app = Flask(__name__)
db = SQLAlchemy(app)
app.config.from_object('config')

####################################################################
association_table = db.Table('association', db.Model.metadata,
    db.Column('review_id', db.Integer, db.ForeignKey('review.id')), #left
    db.Column('genre_id', db.Integer, db.ForeignKey('genre.id')) #right
)
####################################################################


class Review(db.Model):
    """
    author, item_title, item_author, item_asin, item_image_url, post_url, date fields
    also genres, a many-to-many association table with Genre
    """

    __tablename__ = 'review'
    id = db.Column(db.Integer, primary_key=True)
    author = db.Column(db.String(250))
    item_title = db.Column(db.String(250))
    item_author = db.Column(db.String(250))
    item_asin = db.Column(db.String(50))
    item_image_url = db.Column(db.String(250))
    post_url = db.Column(db.String(250))
    date = db.Column(db.Date)
    genres = relationship('Genre', secondary=association_table, backref='reviews')

    def to_dict(self):
        return {'id': self.id, 'author': self.author, 
                'item_title': self.item_title, 'item_author': self.item_author,
                'item_asin': self.item_asin, 'item_image_url': self.item_image_url,
                'post_url': self.post_url, 'date': self.date, 'genres': self.genres }


####################################################################
class Genre(db.Model):

    __tablename__ = 'genre'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(120))

    def __init__(self, name):
        self.name = name


####################################################################
{% endhighlight %}

Next I create a development Postgresql database and test Postgresql database:

{% highlight bash %}
createdb marginal-review
{% endhighlight %}

Now the development database should be located at `'postgresql://localhost/marginal-review'` and the test database should be at `'postgresql://localhost/marginal-review-test'`.

Our app needs to know where the postgres database is.  It's a little more complicated than just hard-coding it into the `app.py` file: we're really dealing with three different postgres databases (local dev, local testing, and production heroku), not just one.  To that end we can set up a `config.py` file like so:

{% highlight python %}
import os

if os.environ.get('DATABASE_URL') is None:
    SQLALCHEMY_DATABASE_URI = 'postgresql://localhost/marginal-review'
else:
    SQLALCHEMY_DATABASE_URI = os.environ['DATABASE_URL']
{% endhighlight %}

In `app.py` we include the line:

{% highlight python %}
app.config.from_object('config')
{% endhighlight %}

where `'config'` refers to the file, `config.py`.  Now we can refer to any variable we define in the `config.py` file using `app.config['VARIABLE_NAME']` (right now we've just got the `SQLALCHEMY_DATABASE_URI` variable defined).

##Tests that touch the postgres database

Before we write our first tests that touch the database, we need to make some changes to the `setUp` and `tearDown` methods in our `TestCase` class.

{% highlight python %}
    def setUp(self):
        app.config['TESTING'] = True
        app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://localhost/marginal-review-test'
        self.app = app.test_client()
        db.create_all()
{% endhighlight %}

Here's we're setting the database URI to our testing database (which we have yet to create) and using the `db.create_all()` method to turn our SQLAlchemy models into Postgres tables.

{% highlight python %}
    def tearDown(self):
        db.session.remove()
        db.drop_all()
{% endhighlight %}

With `tearDown` we're ensuring that all the changes to the test database are destroyed after each test.

The first test that interacts with the database:

{% highlight python %}
    def test_reviews_are_persisted(self):
        review_info = {'item_author':'Jorge Luis Borges', 'item_title':'Labrynthes'}
        review1 = Review(**review_info)
        db.session.add(review1)
        db.session.commit()
        self.assertEqual(review_info['item_author'], db.session.query(Review).first().item_author)
{% endhighlight %}

1.)  `NameError: global name 'db' is not defined`, fixed with `from app import app, db, Review` in our test file

2.)  `ImportError: No module named psycopg2`, fixed with `pip install psycopg2`

3.)  `OperationalError: (psycopg2.OperationalError) FATAL:  database "marginal-review-test" does not exist`, fixed by creating a testing database on the command line with  `createdb marginal-review-test`

At this point the tests should pass (another good time to commit changes).


##Set up the database on Heroku

The initial Flask app we pushed to Heroku doesn't have a database yet.  Unlike with Rails, we have to manually create a database for Heroku.  We can do that on the command line with `heroku addons:create heroku-postgresql:dev`.  After running this, Heroku returns the message:

{% highlight bash %}
Creating cooling-slyly-7309... done
Adding cooling-slyly-7309 to marginal-review-api... done
Setting DATABASE_URL and restarting marginal-review-api... done, v6
Database has been created and is available
 ! This database is empty. If upgrading, you can transfer
 ! data from another database with pgbackups:restore
Use `heroku addons:docs heroku-postgresql` to view documentation.
{% endhighlight %}

Notice it doesn't assign a color name to the new database.  Most of the tutorials online assume that it does, and recommend that for the next step you run `heroku pg:promote HEROKU_POSTGRESQL_COLOR_URL`.  Since Heroku didn't assign me a color, I just ran `heroku pg:promote DATABASE` and got the following message:

{% highlight bash %}
Ensuring an alternate alias for existing DATABASE... done, HEROKU_POSTGRESQL_GREEN
Promoting cooling-slyly-7309 to DATABASE_URL on marginal-review-api... done
{% endhighlight %}

So I guess my Heroku color is green.

In order for Heroku to load the postgres database with the correct tables (that I defined from SQLAlchemy models), we need to add another line to `Procfile`:

{% highlight bash %}
web: gunicorn app:app --log-file=-
init: python db_create.py
{% endhighlight %}

This line sets up a little task called `init` that rens the `db_create.py` script, which is:

{% highlight python %}
from app import db
db.create_all()
{% endhighlight %}

Be sure to update your requirements.txt to include `psycopg2`.  After we commit our changes and push them to Heroku, we use `heroku run init` to initialize the database.

The rest of the tests and app are relatively prosaic; you can see the code at [this github repo](https://github.com/cfmeyers/mr-reviews-api).  The API itself is deployed to Heroku [here](https://marginal-review-api.herokuapp.com/api/v1/reviews).

##Helpful Links

-  [setting-up-unit-tests-with-flask](http://kronosapiens.github.io/blog/2014/07/29/setting-up-unit-tests-with-flask/)

-  [Heroku's Getting-Started-With-Flask Article](https://devcenter.heroku.com/articles/getting-started-with-python-o)

-  [Yuval Adam's Article on Flask, Postgresql, and Heroku](http://blog.y3xz.com/blog/2012/08/16/flask-and-postgresql-on-heroku)
