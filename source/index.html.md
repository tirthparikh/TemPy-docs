---
title: TemPy - docs

language_tabs:
  - python: Python

toc_footers:
  - <h2><a href='https://tempy-dev.slack.com/messages'>Get in touch using SLACK!</a></h2>

search: true
---

# Overview

```python
from tempy.tags import Html, Head, Body, Meta, Link, Div, P, A
my_text_list = ['This is foo', 'This is Bar', 'Have you met my friend Baz?']
another_list = ['Lorem ipsum ', 'dolor sit amet, ', 'consectetur adipiscing elit']

page = Html()(  # add tags inside the one you created calling the parent
    Head()(  # add multiple tags in one call
        Meta(charset='utf-8'),  # add tag attributes using kwargs in tag initialization
        Link(href="my.css", typ="text/css", rel="stylesheet")
    ),
    body=Body()(  # give them a name so you can navigate the DOM with those names
        Div(klass='linkBox')(
            A(href='www.foo.com')
        ),
        (P()(text) for text in my_text_list),  # tag insertion accepts generators
        another_list  # add text from a list, str.join is used in rendering
    )
)
```

```
page.render()
>>> <html>
>>>     <head>
>>>         <meta charset="utf-8"/>
>>>         <link href="my.css" type="text/css" rel="stylesheet"/>
>>>     </head>
>>>     <body>
>>>         <div class="linkBox">
>>>             <a href="www.foo.com"></a>
>>>             <a href="www.bar.com"></a>
>>>             <a href="www.baz.com"></a>
>>>             <a href="www.python.org">This is a link to Python</a>
>>>         </div>
>>>         <p>This is foo</p>
>>>         <p>This is Bar</p>
>>>         <p>Have you met my friend Baz?</p>
>>>         Lorem ipsum dolor sit amet, consectetur adipiscing elit
>>>     </body>
>>> </html>
```

Build HTML without writing a single tag.

HTML is like SQL: we all use it, we know it works, we all recognize it's important, but our biggest dream is to never write a single line of it again. For SQL we have ORM's, but we're not there yet for HTML.

Templating systems are cool (Python syntax in html code) but not cool enough (you still have to write html somehow)..

..so the idea of TemPy.

TemPy let the developer build the DOM using only Python objects and classes. It provides a simple but complete API to dynamically create, navigate, modify and manage "HTML" templates and objects in a pure Python.

Navigating the DOM and manipulating tags is possible in a Python or jQuery-similar sintax. Then later your controllers can serve the page by just calling the `render()` method on the root element.

TemPy is designed to offer Object-Oriented Templating, giving the developer the ability to use and manage html templates following the OOP paradigms. Sublassing, overriding and all the other OOP techiques will make HTML templating more flexible and mantainable.

**Speed**

One of the main slow-speed factors when developing webapps are the template engines. TemPy have a different approach to the HTML generation resulting in a big speed gain.

No parsing and a simple structure makes TemPy fast. TemPy simply adds html tags around your data, and the actual html string exists only at render time.

# Installation

```shell
pip3 install tem-py
```

```shell
git clone https://github.com/Hrabal/TemPy.git
cd TemPy
python3 setup.py install
```

<aside class="notice">TemPy does not support Python 2.x. TemPy requires Python >= 3.3 to work.</aside>

TemPy is avaiable on PyPi, so you can pip him. [PyPi.org](https://pypi.org/project/tem-py/) page.

If you want to customize TemPy you can clone the main [GitHub repo](https://github.com/Hrabal/TemPy).

# How to use TemPy

## Basic Usage

```python
from tempy.tags import * 

# Create some empty TemPy objects/tags
div = Div()
span = Span()

# Create non empty TemPy objects/tags
container = Div()(
    'content: ', Div()('this is the content')
)

# Build the TemPy tree calling your objects
div(span)
span(A(href='www.bar.com')('this is the link text'))
container(test=div) 

# Or use the API
container.append(A(href='www.baz.com')('another useful link'))

# If you gave a name to a TemPy object call him by name
link = Link().append_to(container.test)

# Add attributes and content to already placed tags
link.attr(href='www.python.org')('This is a link to Python')

# Render your TemPy tree
container.render()
>>><div>content: 
>>>    <div>this is the content</div>
>>>    <div>
>>>        <span>
>>>            <a href="www.bar.com">this is the link text</a>
>>>        </span>
>>>        <link href="www.python.org"/>
>>>    </div>
>>>    <a href="www.baz.com">another useful link</a>
>>></div>
```

TemPy offers clean syntax for building pages in pure python. Every TemPy object is a container of other objects, when rendered TemPy objects will produce an html tag containing the `str` representation of all his children.

TemPy objects can be arranged togheter dynamically to build the DOM tree. Every TemPy instance is a node of the DOM, and can be father or child of other TemPy onbjects.

Every HTML tag have his corresponding TemPy class, to create a tag just instantiate the TemPy class: `Div()` will produce an object that can contain other objects (TemPy objects or not) and can be rendered into and HTML string.

TemPy tags can have attributes, that will be rendered inside the tag, it's possible to define attributes when instantiating the object (`Div(attribute='value')`) or later using the api (`Div().attr(attribute='value')`).

Once a TemPy tag or widget is instantiated you can add tags and content by calling the instance as if it's a function: `div(Span())`. Element creation and insertion can be performed in a single instruction: `Div()(Span())`.

It's possible to add multiple elements inside another and every TemPy object accepts different kind of element insertion:

* single insertion: `Div()(Span())`
* list insertion: `Div()(['something', Span(), 1])`
* insetion from a generator: `Div()(Span() for _ in range(5))`
* named insertion: `Div()(some_child=Span())`
* using the TemPy objects's API: `Div().append((Span())` *see below for a complete API listing*

<aside class="warning">Attention: named insertion is safe only when using Python >= 3.6</aside>

HTML tags have attributes, and so TemPy tags have too. It's possible to define tag attributes in different ways:

* during the element instantiation: `Div(some_attribute='some_value')`
* usind the `attr` API: `Div().attr(some_attribute='some_value')`


The resulting tree (in fact, the DOM), can be rendered by calling the `.render()` method.

Calling `render` on some TemPy object will return the html representation of the tree starting from the current element including all the childs.
`render` will be called on every TemPy child, and `str()` will be called on every non-TemPy child.

## Blocks and Content

```python
# --- file: base_elements.py
from somewhere import links, foot_imgs
# define some common blocks
header = Div(klass='header')(title=Div()('My website'), logo=Img(src='img.png'))
menu = Div(klass='menu')(Li()(A(href=link)) for link in links)
footer = Div(klass='coolFooterClass')(Img(src=img) for img in foot_imgs)
```

```python
# --- file: pages.py
from base_elements import header, menu, footer

# import the common blocks and use them inside your page
home_page = Html()(Head(), body=Body()(header, menu, content='Hello world.', footer=footer))
content_page = Html()(Head(), body=Body()(header, menu, Content('header'), Content('content'), footer=footer))
```

```python
# --- file: my_controller.py
from tempy import Content
from tempy.tags import Div
from pages import home_page, content_page

@controller_framework_decorator
def my_home_controller(url='/'):
    return home_page.render()

@controller_framework_decorator
def my_content_controller(url='/content'):
    header = Div()('This is my header!')
    content = "Hi, I'm a content"
    return content_page.render(header=header, content=content)
```

TemPy lets you build blocks and put them together using the manipulation api, each TemPy object can be used later inside another TemPy object.

Static parts of pages can be created once an then used in different pages. Blocks can be created and imported as normal Python objects.

<aside class="warning">Depending on the web framework you use, TemPy instances can be shared between http requests. Keep in mind that if you modify a TemPy class instance used in various pages, this will remain modified in every page and in every subsequent request.</aside>

To make a Block a dynamic, so it can contain different contents each request/use, we can use TemPy's `Content` class. Those elements are just containers with no html representation, at render time his childs will be rendered inside the `Content`'s father.

`Content` can have a fixed content so it can be used as 'html invisible box' (this fixed content can, however, be dynamic), or it can just have a name.

Every TemPy objects can contain extra data that will not be rendered, you can manage this extra data with the `TemPyClass.data()` api as if it's a common dictionary. At render time TemPy will search into the extra data of the `Content` container, and recursively into his parents, looking for a key matching the `Content`'s name. If it's found then it's value is used in rendering.

## OOT - Object Oriented Templating

```python
from tempy.widgets import TempyPage

class BasePage(TempyPage):
    def js(self):
        return [
            Script(src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"),
        ]

    def css(self):
        return [
            Link(href=url_for('static', filename='style.css'),
                 rel="stylesheet",
                 typ="text/css"),
            Link(href='https://fonts.googleapis.com/css?family=Quicksand:300',
                 rel="stylesheet"),
            Link(href=url_for('static',
                              filename='/resources/font-awesome-4.7.0/css/font-awesome.min.css'),
                 rel="stylesheet"),
        ]

    # Define the init method as a constructor of your block structure
    def init(self):
        self.head(self.css(), self.js())
        self.body(
            container=Div(id='container')(
                title=Div(id='title')(
                    Div(id='page_title')(A(href='/')('MySite')),
                    menu=self.make_menu('MAIN')
                ),
                content=Div(id='content')(Hr())
            )
        )
    
    # Your subclass can have his own methods like any other class
    def make_menu(self, typ):
        return Div(klass='menu')(
                            Nav()(
                                Ul()(
                                    Li()(
                                        A(href=item[1])(item[0]))
                                    for item in self.get_menu(typ)
                                )
                            ),
                        )

    def get_menu(self, typ):
        return [(mi.name, mi.link)
                for mi in Menu.query.filter_by(active=True, menu=typ
                                               ).order_by(Menu.order).all()]

```

```python
class HomePage(BasePage):

    def init(self):
        self.body.container.content(
            Div()(
                Br(),
                'This is my home page content', Br(),
                H3()('Hame page important content'),
                'Look, I\'m a string!', Br(),
                H3()('H3 is big, really big'),
                H1()('Today's content:'),
                self.get_dynamic_content()
            )
        )

    def get_dynamic_content(self):
        # Here using SQLAlchemy:
        current_content = Content.query.outerjoin(Content.comments).order_by(Content.date.desc(), Content.id.desc()).limit(1).first()
        if not current_content:
            return 'No content today!'
        return Div()(Span()(current_content.title),
                     Span()(current_content.text)),
                     Div()(comment for comment in current_content.comments))
```

```python
class CustomTag(tempy.Tag):
    __tag = 'my_own_tag'

CustomTag().render()
>>> <my_own_tag></my_own_tag>

class CustomVoidTag(CustomTag, VoidTag): pass
CustomTag().render()
>>> </my_own_tag>
```

TemPy is designed to provide Object Oriented Templating. You can subclass TemPy classes and define their inner tree structure, and also add custom methods.



TemPy executes each base class `init` method in reverse mro, so your subclass can access all the elements defined in his parent classes.

To create a custom TemPy tag, subclass the base `tempy.Tag` class and provide a custom `__tag` class variable

<pre>


</pre>

## TemPy repr's

```python
class MyClass:
    def __init__(self):
        self.foo = 'foo'
        self.bar = 'bar'

    class HtmlREPR(TempyREPR):
        def repr(self):
            self(
                Div()(self.foo),
                Div()(self.bar)
            )
```

```python
class MyClass:
    def __init__(self):
        self.foo = 'foo'
        self.bar = 'bar'
        self.link = 'www.foobar.com'
    
    # If an instance on MyClass is found inside a div
    class Div(TempyREPR):
        def repr(self):
            self(
                Div()(self.foo),
                Div()(self.bar)
            )
    
    # If an instance on MyClass is found inside a link
    class A(TempyREPR):
        def repr(self):
            self.parent.attrs['href'] = self.link
            self('Link to ', self.bar)

    # If an instance on MyClass is found inside a table cell
    class Td(TempyREPR):
        def repr(self):
            self(self.bar.upper())
    
    # If an instance on MyClass is found when rendering the a TempyPage called 'HomePage'
    class HomePage(TempyREPR):
        def repr(self):  # note: here self is the object's parent, not the root
            self('Hello World, this is bar: ', self.bar)


my_instance = MyClass()

Div()(my_instance).render()  # my_instance is rendered using Div(TempyREPR) nested class

A()(my_instance).render()  # my_instance is rendered using A(TempyREPR) nested class

Table()(Tr()(Td()(my_instance))).render()  # my_instance is rendered using Td(TempyREPR) nested class

class HomePage(Html): pass

HomePage()(
    Head()(),
    Body()(
        Div()(
            H1()(my_instance)
        )
    )
).render()  # my_instance is rendered using HomePage(TempyREPR) nested class
```

Another way to use TemPy is to define a nested `TempyREPR` subclass inside your own classes.

You can think the `TempyREPR` nested class as a `__repr__` magic method equivalent: TemPy uses the `TempyREPR` nested class to represent objects just like Python uses the `__repr__` method.

When an object is placed inside a tree TemPy searches for a `TempyREPR` class inside this object, if it's found, the `repr` method of this class is used as a template.
The `TempyREPR.repr` method accepts `self` as the only argument, with a little magic this `self` is both your object and the tree element, so all the common TemPy API is usable and your object attributes are accessible using `self`.

You can define several `TempyREPR` nested classes, when dealing with non-TemPy object TemPy will search for a `TempyREPR` subclass following this priority:

1. a `TempyREPR` subclass with the same name of his TemPy container
2. a `TempyREPR` subclass with the same name of his TemPy container's root
3. a `TempyREPR` subclass named `HtmlREPR`
4. the first `TempyREPR` found.
5. if none of the previous if found, the object will be rendered calling his `__str__` method

You can use this order to set different renderings for different situation/pages.

# TemPy ojects

## Tag Creation 

```python
page = Html()
div = Div()

new_page = page.clone()

new_page_2 = page + div
new_page_3 = new_page_2 - div
list_of_5_divs = div * 5
```

Create DOM elements by instantiating tag classes. Those elemets are nodes in the DOM tree and can be attached, detached, moved and composed togheter dynamically.

There are other 2 ways to create TemPy objects:

* using the `clone()` API
* adding subtracting or multiplying TemPy objects


## Tag Attributes 

```python
div = Div(id='my_html_id', klass='someHtmlClass') # 'klass' because 'class' is a Python's buildin keyword
>>> <div id="my_dom_id" class="someHtmlClass"></div>

a = A(klass='someHtmlClass')('text of this link')
a.attr(id='another_dom_id')
a.attr({'href': 'www.thisisalink.com'})
>>> <a id="another_dom_id" class="someHtmlClass" href="www.thisisalink.com">text of this link</a>
```

```python
div2.css(width='100px', float='left')
div2.css({'height': '100em'})
div2.css('background-color', 'blue')
>>> <div id="another_dom_id" class="someHtmlClass comeOtherClass" style="width: 100px; float: left; height: 100em; background-color: blue"></div>
```

HTML tag attributes can be managed in two ways: you can add attributes to every element at definition time (TemPy object instantiation) `Div(my_html_attribute='my_html_attribute_value')` or later using the API.

TemPy supports normal and boolean (attributes with no explicit value) HTML attributes. To define a boolean attribute just assign `bool`, `True` or `False` to the attribute in TemPy: `Div(normal_attribute='foo', boolean_attribute=bool)` (a boolean attribute with `False` value will not be rendered).

TemPy supports multiple, space separated, attributes like the HTML `class` attribute. In TemPy these are attributes that should contain iterables.

Few axceptions are needed, for some common HTML attributes names (i.e: 'class', 'type', etc..) are Python native keyword and can not be used as argument names, those are mapped to custom TemPy names:

* `klass` -> 'class'
* `typ` -> 'type'
* `_for` -> 'for'

Another kind of managed attributes are the mapping kind, like the `style` tag attribute. Style can be managed using passing a dictionary to the `style` attribute directly (`Div(style={'background-color': 'blue'})`), or can be edited in the jQuery fashion with the `.css()` method.

The `css` method mimic the jQuery's brother behavior:

* called with no arguments returns the style dictionary: `Div(style={'color': 'white'}).css() -returns-> {'color': 'white'}`
* called with a dict as the only unnamed argument will `dict.update` the element's style with the given imput: `Div().css({'color': 'white'})`
* called with kwargs will `dict.update` the element's style with the given kwargs: `Div().css(color='white')`


## Building the DOM

```python
page(Head())
>>> <html><head></head></html>
```

```python
body = Body()
page.append(body)
>>> <html><head></head><body></body></html>
div = Div().append_to(body)
>>> <html><head></head><body><div></div></body></html>
div.append('This is some content', Br(), 'And some Other')
>>> <html><head></head><body><div>This is some content<br>And some Other</div></body></html>
```

```python
head.remove()
>>> <html><body><div></div></body></html>
body.empty()
>>> <html><body></body></html>
page.pop()
>>> <html></html>
```


Add elements or content by calling them like a function...

> page(Head())

<aside class="warning">
Correct ordering with named Tag insertion is ensured with Python >= 3.6 (because kwargs are ordered)
</aside>

...or use one of the jQuery-like apis:

...same for removing:

## Traversing the DOM
```python
container_div = Div()
container_div(content_div=Div())

container_div.content_div('Some content')
>>> <div><div>Some content</div></div>
```

```python
container_div.children()
container_div.first()
container_div.last()
container_div.next()
container_div.prev()
container_div.prev_all()
container_div.parent()
container_div.slice()
```

Every TemPy Tag content is iterable and accessible like a Python list:

...or access elements inside a container as if it they were attributes:

..or if you feel jQuery-ish you can use:


# API Reference

```python
div1 = Div()
div2 = Div()
div1.after(div2)
div1.before(div2)
div1.prepend(div2)
div1.prepend_to(div2)
div1.append(div2)
div1.append_to(div2)
div1.wrap(div2)
div1.wrap_inner(div2)
div1.replace_with(div2)
div1.remove(div2)
div1.move_childs(div2)
div1.move(div2)
div1.pop(div2)
div1.empty(div2)
div1.children(div2)
div1.contents(div2)
div1.first(div2)
div1.last(div2)
div1.next(div2)
div1.next_all(div2)
div1.prev(div2)
div1.prev_all(div2)
div1.siblings(div2)
div1.slice(div2)
```

Several api's are provided to modify you're existing DOM elements:

* after() - 
* before() - 
* prepend() - 
* prepend_to() - 
* append() - 
* append_to() - 
* wrap() - 
* wrap_inner() - 
* replace_with() - 
* remove() - 
* move_childs() - 
* move() - 
* pop() - 
* empty() - 
* children() - 
* contents() - 
* first() - 
* last() - 
* next() - 
* next_all() - 
* prev() - 
* prev_all() - 
* siblings() - 
* slice() - 

# Performance
Performance varies considerably based on the complexity of the rendered content, the amount of dynamic content on the page, the size of the produced output and many other factors.

TemPy does not parse strings, does not use regex and does not load .html files, resulting in great speed compared to the traditional frameworks such as Jinja2 and Mako.

Here are a few benchmarks of TemPy in action, used in a Flask app, rendering a template (see code [here](benchmarks))

Used HW: 2010 IMac, CPU:2,8 GHz Intel Core i7 RAM:16 GB 1067 MHz DDR3 Osx: 10.12.6.

Benchmark made using [WRK](https://github.com/wg/wrk)

![TemPy Web Rendering](bench.jpg)

Running 20s test @ http://127.0.0.1:8888/tempy + http://127.0.0.1:8888/j2 - 10 threads and 200 connections


Tempy | Avg | Stdev | Max | +/- Stdev
----- | --- | ----- | --- | ---------
Latency | 109.55ms | 52.04ms | 515.33ms | 93.09%
Req/Sec | 118.27 | 37.36 | 240.00 | 73.77%

16111 requests in 20.09s, 96.23MB read
Requests/sec: 801.91
Transfer/sec: 4.79MB

Jinja2 | Avg | Stdev | Max | +/- Stdev
----- | --- | ----- | --- | ---------
Latency | 216.04ms | 16.05ms | 267.06ms | 91.16%
Req/Sec | 59.29 | 20.53 | 151.00 | 71.23%

11841 requests in 20.08s, 72.80MB read
Requests/sec:    589.70
Transfer/sec:      3.63MB

Performance difference is even higher in Jinja2 plain (no Flask) rendering:
![TemPy No-Web Rednering](bench_plain.jpg)

# Credits

## Made and mantained by Federico Cerchiari

Get in touch on GitHub: [Hrabal](https://github.com/Hrabal)

Get in touch using Slack: [tempy-dev.slack.com](https://tempy-dev.slack.com/messages)

## Contribute
Any contribution is welcome. Please refer to the [contributing page](https://github.com/Hrabal/TemPy/blob/master/CONTRIBUTING.md) on the master GitHub repo.

# Project Info

## Compatibility

**Python >= 3 is a must**, there is no intention to backport this project to Python 2.7.

**Python >= 3.3 is needed**, for TemPy uses the delegation to subgenerator (the `yield from` statement) proposed in [PEP 380](https://www.python.org/dev/peps/pep-0380/).

This form of yielding is used for speed and can be easily removed if you plan to use TemPy in Python 3.0 to 3.2.x , you'll just need to substitute the `yield from` with a loop on the inner generator yielding single values.


**Python >= 3.6 is preferred**, some useful features depends on the preserved order of kwargs proposed in [PEP 468](https://www.python.org/dev/peps/pep-0468/).

The main feature is the possibility to have named child tags in the correct order. Naming child tags is possible in Python < 3.6, but the tag's order will probably not be correct.

<aside class="success"><b>It it very reccomended to use TemPy with Python >= 3.6.</b></aside>

## Status

Current [PyPi](https://pypi.org/project/tem-py/) version

[![PyPI version](https://badge.fury.io/py/tem-py.svg)](https://badge.fury.io/py/tem-py)

[Coveralls](https://coveralls.io/github/Hrabal/TemPy?branch=master) CI Test coverage

[![Coverage Status](https://coveralls.io/repos/github/Hrabal/TemPy/badge.svg?branch=master)](https://coveralls.io/github/Hrabal/TemPy?branch=master)

[Travis](https://travis-ci.org/Hrabal/TemPy) CI Build status

[![Build Status](https://travis-ci.org/Hrabal/TemPy.svg?branch=master)](https://travis-ci.org/Hrabal/TemPy)

# Apache 2.0
See [LICENSE](https://github.com/Hrabal/TemPy/blob/master/LICENSE) for details.
