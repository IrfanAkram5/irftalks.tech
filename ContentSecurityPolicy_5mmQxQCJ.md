# Securing Your Website

## Introduction

This article concentrates on the "Content Security Policy" response header.  There are other headers as well dealing with security but given that security is a very broad area, one thing at a time!

## Overview and better reading material
Because modern websites use and pull in resources from servers other then your own, having a CSP header is a good way to whitelist from where and what type of resource is allowed along with, if relevant, what it is allowed to do.  A good starting point for CSP ( and many other topics ) tends to be Mozilla.  I would start of by reading [Mozilla's CSP page](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP).  If you need a broader security overview, I would also read[Mozilla's Security Guidelines](https://infosec.mozilla.org/guidelines/web_security.html) which handily also provides a cheat sheet listing different guidelines, their impact in terms of benefit and also a difficulty rate in terms of implementation.

If you really want to take it to the next level, you can also read [W3C's CSP Level3 - technical draft ](https://www.w3.org/TR/CSP3/).  

The main purpose of have a CSP header is to reduce the risk of injection attacks from malicious websites.

### Complexity

Although one can understand and implement the basics of CSP relatively quickly, things do get complicated quickly when mixing external libraries with own scripts.  If someone has experience of how to implement a CSP whereby you need to allow `unsafe-inline` or `unsafe-eval` due to an external library but not have it conflict with your own nonce protected scripts, please do help update this article with an example policy.  

## Where to implement a CSP

Choice is a great thing.  But choice can also be a pain in the proverbial if it leads to confusion and decision paralysis.  One of the annoying things about web development is sometimes the same item can be implemented in multiple location.  This is true with CSP.  You can implement it;

* Within the web server
* Within a html page using the `<meta>` tag
* Within a web framework
* Within a framework package

## Django-CSP

I chose to implement CSP using the Django-CSP package.  With implementation at web server level, I think there could be a temptation to given the widest possible privileges just to make sure all your pages render correctly and not have to think about it which, in effect. renders CSP near useless.  

### Installation and Usage

Installation is straightforward - just install it via `pip install django-csp`.  There does not appear to ba a conda recipe for it.  The link to the documentation is [here](https://django-csp-test.readthedocs.io/en/latest/index.html).

Usage is straight forward - simply include the policies you want in your `settings.py` file.

```python
# This is settings.py file in django

CSP_DEFAULT_SRC = ["'self'",]
CSP_SCRIPT_SRC = ["'self'",] 
CSP_FONT_SRC = ['https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/webfonts/']

# etc.
```

Then via the decorators, you can update/replace specific policies.  This allows for a flexible implementation, whereby you can set a baseline global policy in the `settings.py` file but then at view level, you can update it to the needs of your page.  There are four decorators ( csp_exempt, csp_update, csp_replace, csp ).  Jump to the docs above for full details.


e.g.  Update to allow bokeh scripts in this particular view - the values will be *appended* to the current list

```python
@csp_update(
    SCRIPT_SRC=[
        "https://cdn.bokeh.org/bokeh/release/bokeh-2.4.3.min.js",
        "https://cdn.bokeh.org/bokeh/release/bokeh-widgets-2.4.3.min.js",
    ])

def myview(request):
    pass

```

### Gotchas

As you can see with the `CSP_DEFAULT_SRC = ["'self'",]` example above, the `self` is first enclosed with single quotes and then double.  This is a must if using reserved values such as `self`, `unsafe-inline`, etc.  Basically, if the value is single quoted in the MDN docs, you should double quote it in django.


### Example Nonce Value

In your settings you will need to add `CSP_INCLUDE_NONCE_IN = ['script-src',]`

And then in your html, just add nonce value after inline script tag.

```html
\\ Add the nonce value
<script nonce="{{request.csp_nonce}}">

    document.addEventListener('DOMContentLoaded', () => {

        // Get all "navbar-burger" elements
        const $navbarBurgers = Array.prototype.slice.call(document.querySelectorAll('.navbar-burger'), 0);

        // Add a click event on each of them
        // removed code for brevity - just more javascript
        
    });
</script>

***However***, if you use a library at the same time, that needs the `unsafe-inline` to function, then you cannot have an inline script with a nonce value at the same time. 

```

## Example nginx

As previously mentioned, you can add a CSP policy at the webserver level.  Below is example for Nginx.

```bash

# In prod I had to have everything on one line like so for the setting to work.......
add_header Content-Security-Policy   "script-src 'self' https://cdn.bokeh.org/bokeh/release/bokeh-2.4.3.min.js 'unsafe-inline' 'unsafe-eval' https://cdn.bokeh.org/bokeh/release/bokeh-widgets-2.4.3.min.js  https://cdn.bokeh.org/bokeh/release/bokeh-tables-2.4.3.min.js https://cdn.bokeh.org/bokeh/release/bokeh-gl-2.4.3.min.js https://cdn.bokeh.org/bokeh/release/bokeh-mathjax-2.4.3.min.js; default-src 'self'; frame-ancestors 'self';  img-src 'self' https://img.icons8.com/color/48/000000/linux--v2.png data:; style-src 'self' https://cdn.jsdelivr.net/npm/bulma@0.9.3/css/bulma.min.css 'unsafe-inline';" always;

# .....whilst in dev it accepted the same thing separated over multiple lines
 add_header Content-Security-Policy   "script-src 'self' https://cdn.bokeh.org/bokeh/release/bokeh-2.4.3.min.js 'unsafe-inline' 'unsafe-eval' https://cdn.bokeh.org/bokeh/release/bokeh-widgets-2.4.3.min.js  https://cdn.bokeh.org/bokeh/release/bokeh-tables-2.4.3.min.js https://cdn.bokeh.org/bokeh/release/bokeh-gl-2.4.3.min.js https://cdn.bokeh.org/bokeh/release/bokeh-mathjax-2.4.3.min.js;
                                           default-src 'self';
                                           frame-ancestors 'self';
                                           img-src 'self' https://img.icons8.com/color/48/000000/linux--v2.png data:;
                                           style-src 'self' https://cdn.jsdelivr.net/npm/bulma@0.9.3/css/bulma.min.css 'unsafe-inline'";

```

## Example html

If you want to set the CSP in a  html page directly, use the `<meta>` tag in the header section.

```html

<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'self'; img-src https://*; child-src 'none';" />


```


## Resources

[Test your CSP implementation](https://csp-evaluator.withgoogle.com/)

[Django-CSP](https://github.com/mozilla/django-csp) - is maintained and created by Mozilla for the django framework.  
In my opinion it make it more flexible to implement and maintain as you can target policies for specific pages.  You can install it install via `pip`.

[Blog - Some guidance/examples on using Django-csp](https://www.laac.dev/blog/content-security-policy-using-django/)  
[Digital Ocean Community Tutorial on Django-CSP](https://www.digitalocean.com/community/tutorials/how-to-secure-your-django-application-with-a-content-security-policy)

</br>  
  
Diagram showing CSP mappings from wikipedia.  The webserver in the diagram could be your html page or web framework instead.  
![Mapping between HTML5 and JS to CSP](../../Astatic/mdp/django/images/5mmQxQCJ_CSP3_diagram.png)  
[*By Kravietz - Own work, CC BY-SA 4.0, https://commons.wikimedia.org/w/index.php?curid=44990361*](https://commons.wikimedia.org/w/index.php?curid=44990361)


Ref: 5mmQxQCJ