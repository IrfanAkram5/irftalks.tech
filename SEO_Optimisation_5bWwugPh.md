# Search Engine Optimisation

#### Intro

[*Skip the intro and just jump to the django sitemaps section!*](#sitemaps)

This doc goes over items/considerations for Search Engine Optimisation (SEO) so that your website can be found by users.  Initially, I will concentrate on optimising for Google as that is by far the most popular search engine.  Here is the [link to Google's developer guide to SEO](https://developers.google.com/search/docs/beginner/get-started).

The good news is that at its core, implementing basic SEO appears to boil down to these three things for most websites.

1.  Provide a short relevant title in the title table.  Don't repeat the same title for every page.  If you have a blog site for example, make sure each unique article also has a unique title tab relevant to the blog entry.  See an example of a relevant tab title below.  

    ![provide short relevant titles](../Astatic/mdp/OTHR/example_title_tab_5bWwugPh.png) 

    This is done with the `<title>` tag in the  header section of your html.

2. Provide a summary that will be displayed in the google results.  If you do not, google just takes the first paragraph at the top of your page and uses that instead.  If it does not mention the relevant key words, you are handicapping yourself.  You provide the summary via a description meta tag in the header section. e.g.

    ```html
    <head>
        <!-- Some standard meta tags -->
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <!-- add in the below meta tag replacing the content with your own summary -->
        <meta name="description" content="Learning Python, Django as a complete beginner is difficult enough.  Throw in all the other technologies you will need or come across such as Postgresql( database ), Ubuntu Linux, Javascript, CSS, markdown, etc. plus a whole bunch of libraries to bring things together, it is downright overwhelming." >

        <title>Tab title goes here</title>
    </head>
    ```  

    If the page shows up in the results, it will be used to provide a quick synopsis. e.g. From google search results for this website:

    ![google search result](../Astatic/mdp/OTHR/google_search_result_5bWwugPh.jpg)

    The first result in the above image shows google using the meta description for it's result.  The 2nd result was how the first result was appearing.  The reason for the result being different is because (and I'm not 100% sure on this ) I have not specified which address is canonical and therefore, the lower result is not being processed by google.

3.  Provide a sitemap.  This is simply a webpage/fie which lists all the unique pages that you want google to index.

There are of course other items such as marking canonical pages so if you have multiple address all pointing to the same content, you mark one of them as canonical and therefore google knows not to index the others.  There is also robots.txt if you want to tell google not to index the site or certain pages and a few other meta tags.  I would advise you to read google's own notes on this by using the previously provided links.  But, in the main, the above 3 items are the core items to get right initially.

In case you are wondering, providing a keywords meta tag has been a no no for years.  Even if you provide it, google ignores it.  This is not to say keywords are not important - they absolutely are.  But just stuffing keywords into a description regardless of relevance will probably lead to a lower ranking rather than a higher one.

### Adding structured data

Implementing the above three items is straightforward.  The next level to enable google/others to understand the content of your pages is through structured data.  Apart from skimming over the basics, I have not implemented structured data in these pages yet.  Hopefully, a page on it shall appear sometime in the future.  

If you wish to investigate structured data further, you could start with the [google developers link on the topic](https://developers.google.com/search/docs/advanced/structured-data/intro-structured-data). If you want to see the available schemas for describing page content, visit [schema.org](https://schema.org/).   


# Sitemaps

A sitemap is basically a file that lists all your webpages.  It is used by webcrawlers to index your site.

> There are only two hard things in Computer Science: cache invalidation and naming things.
> -- Phil Karlton

The truth of this old joke is shown if full force with django documentation on the sitemap framework where the term sitemap or sitemaps is used so often for different objects that it become very confusing as to exactly what is being referred to.

If your site is small with a few static pages, you can provide google with a [hand crafted text file](https://developers.google.com/search/docs/advanced/sitemaps/build-sitemap#manualsitemap) listing all your pages to be crawled.  However, this blog site is primarily about python and django,  so let go ahead and become confused.

## Sitemap - simple use case

For django projects with a single app proceed as follows.

### Modify your django settings file


```python
# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'django.contrib.sitemaps', # Addition for sitemaps
    'yourapp',   # your app
]

```  


<h3>In the app's subfolder.....</h3>  

> If you only have a single app, it will not matter whether you put the url code in the app's url or the project's url.  In this example, we are using the app's url. 

<break>  

### Modify urls.py

You should re-confirm with the [django docs](https://docs.djangoproject.com/en/4.0/ref/contrib/sitemaps/) as things can change with new releases.

Add the following lines to your urls.py module.  First time round, you will get coding errors because you have not coded the `sitemaps` module yet in the line `from .sitemaps import StaticViewSitemap`.  But once you do, the errors will resolve themselves.

```python
# Add in sitemap function
from django.contrib.sitemaps.views import sitemap

# You will be coding the sitemaps module so first time round you will get error messages
from .sitemaps import StaticViewSitemap # This is a class you define in sitemaps 

# this dictionary is referenced from the path.    
sitemaps = {'static': StaticViewSitemap,}

# Add the following path to the url patterns.......
urlpatterns = [
    path('sitemap.xml', sitemap, {'sitemaps': sitemaps}, name='django.contrib.sitemaps.views.sitemap'),
    # your other paths
    path("", index, name="index"),    
     # etc
]

```


### Create the sitemaps.py file

In your app's directory add a python file called `sitemaps.py`

> Again, for a single app, it will not matter where you put this.  But if you have multiple apps, the overall project folder may be a better location.

```python
from django.contrib import sitemaps
from django.urls import reverse

class StaticViewSitemap(sitemaps.Sitemap):

# See the django docs for attribute values
# https://docs.djangoproject.com/en/4.0/ref/contrib/sitemaps/#sitemap-class-reference
    changefreq = "never"
    priority = 0.5

# have to define items.  Check the spellings - it is items and not item!
    def items(self):
        return ["index", "attributions", ]

# Have to define location as well.  If not, it will default to absolute url call
    def location(self, item):
        return reverse(item)
```  

<br>

> In the `return` list of the items' function, provide the url names,e.g. if you have `path("music/genre/jazz/", jazzview, name="jazzmusic")` - provide "jazzmusic" and not the actual path.


## Sitemap Dynamic links

If your website only has a handful of pages with fixed addresses, the previous section will suffice.  However, if your website consists of many pages and the web address has a dynamic aspect based on primary key, slug field, etc, in a django model( i.e. database table), you want a programmatic way of feeding in all the links. 

Using this site as an example, at the bottom of every card is a link to "More Python/Ubuntu/Django/etc articles" which, when clicked, will list all the articles under that category.

The path for that link is `path("articles/<category>/", article.list_of_articles, name="listArticles")`  The `<category>` signifies that bit of the path is an argument.

To do this, we need to write another class in sitemaps.py.  But first we update urls.py with the class name we will use.

### Update urls.py

```python

# import the new in the class name
from .sitemaps import ArticlesListsByCatSitemap, StaticViewSitemap  # you create this module

# add another entry in the sitemaps dictionary
sitemaps = {'static': StaticViewSitemap,
            'listsByCat1': ArticlesListsByCatSitemap,}


```

Now add the new class in sitemaps.py

```python
class ArticlesListsByCatSitemap(sitemaps.Sitemap):

    """ Returns urls for article lists for categories which are displayed"""

    changefreq = "weekly"
    priority = 0.6

# Return a queryset  
    def items(self):
        return FP_HtmlCards.objects.filter(display_card=True)
        
# Where the field is a foreign key, remember to add _id to the field name to get
# just field value.  Otherwise you get full string representation with description
    def location(self, item):
        return reverse("listArticles", args=[item.cat1_id])

```

As you can see, the operation is very similar the static sites. The differences are;

1.  In items, we are returning a queryset of a model.  Here we have added filter so we only add to the sitemap pages pages that are actually displayed.

2. In location, along with the path name, we provide the argument for `<category>`.

It is not necessary to write the `location` function here.  We can leave it out here and write effectively the same function in the model object itself.  We must call the function `get_absolute_url()" though.  I prefer to overwrite the location in sitemaps.py.  See the django docs if you wish to choose that path.

### Potential refactor of ArticlesListsByCatSitemap

Given we now have a sense of how Django's sitemap framework works, if as in  ArticlesListsByCatSitemap we are only really interested in one field, we can trim the footprint of the queryset. As a rule of thumb, passing around only the data you need will help both memory usage and performance.

```python

class ArticlesListsByCatSitemap(sitemaps.Sitemap):

    """ Returns urls for articles lists for categories which are displayed"""

    changefreq = "weekly"
    priority = 0.6

    def items(self):
    # we are only interested in the cat1 field, so let just get that instead of
    # the entire record.  values_list returns a queryset of tuples
        return FP_HtmlCards.objects.filter(display_card=True).values_list("cat1_id")
        

    def location(self, item):
        # unpack the tuple - there is only one item per tuple
        # You need the comma otherwise you are just copying the tuple!
        category, = item
        # pass it to the reverse function as normal
        return reverse("listArticles", args=[category])

```

We can refactor this further.  Where you are only interested in one value, we can use `flat = True` in the query. 

```python

class ArticlesListsByCatSitemap(sitemaps.Sitemap):

    """ Returns urls for articles lists for categories which are displayed"""

    changefreq = "weekly"
    priority = 0.6

    def items(self):
        # add flat=True, return object becomes a list ...
        return FP_HtmlCards.objects.filter(display_card=True).values_list("cat1_id", flat=True)
        
    # No longer need to unpack the tuples
    def location(self, item):
        return reverse("listArticles", args=[item])

```

## View the result

If you updated the files at app level, with django, chances are you you will need to type in something like `127.0.0.1:8000/yourAppName/sitemap.xml`.

If you implemented it at project level, you will just go back one. i.e. `127.0.0.1:8000/sitemap.xml`.

Here is a snippet from my site;

```xml

<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:xhtml="http://www.w3.org/1999/xhtml">
    <url>
        <loc>https://www.irftalks.tech/mdp/</loc>
        <changefreq>yearly</changefreq>
        <priority>0.5</priority>
    </url>
    <url>
        <loc>https://www.irftalks.tech/mdp/allAttributions/</loc>
        <changefreq>yearly</changefreq>
        <priority>0.5</priority>
    </url>
    <url>
        <loc>https://www.irftalks.tech/mdp/articles/DJGO/</loc>
        <changefreq>weekly</changefreq>
        <priority>0.6</priority>
    </url>
</urlset>
```


## More than 50 000 entries

If your site has more than 50000 addresses, firstly congratulations!  Secondly, you will need to split the sitemap.xml file as 50000 is the maximum number of entries.  Refer back to the django documentation on this.

## Give Google the location of the sitemap

The final step is to update Google's search console for your site.  On the left hand side, there is a "Sitemaps" category where you can add the address for your sitemap. Obviously, you will need to have a google account and register your site with google.

![google search console](../Astatic/mdp/OTHR/google_search_console_sitemap_5bWwugPh.jpg)



<br><br><br>


##### Appendix1

Apart from google, bing, and other search engines which provide good documentation, I need to research how to make social media sites such as Facebook aware of the website too.  The markdown version of this doc is on github.  If you have insights which you can share on this aspect, please consider cloning and updating the doc.

##### Appendix2 Project structure

In the example above I updated the app level urls.py and put the sitemaps.py within the app folder.  However, long term, it probably makes more sense to keep things at project level for the sitemap and therefore I shifted the code to project level folder.

```bash
#I have removed a lot of directories and files for brevity.  Your django 
#project structure should look something like the below with a top 
#level Django folder.


djProjFolder  <-- outer django folder name
.
|-- 2.11
|-- README.md
|-- db.sqlite3
|-- manage.py
|-- djProjFolder    <--- Inner project folder name - default is same
|   |-- asgi.py
|   |-- settings
|   |   |-- base_settings.py
|   |   |-- dev_settings.py
|   |   |-- prod_settings.py
|   |   `-- test_settings.py
|   |-- sitemaps.py       <-- I have put my sitemap code outside app folder
|   |-- urls.py           <-- I also updated the project level urls.py folder
|   `-- wsgi.py
|-- markd.sock
`-- yourApp                  <-- Your django app folder
    |-- admin.py
    |-- apps.py
    |-- management
    |   `-- commands
    |-- models
    |-- standalone_scripts
    |   |-- gen_unique_ids.py
    |-- templates
    |   |-- 404.html
    |   `-- mdp
    |       |-- article.html
    |-- tests.py
    |-- urls.py
    `-- views.py

```
