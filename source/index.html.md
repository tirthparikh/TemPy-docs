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

It's possible to add elements inside TemPy objects:

* single objects: `Div()(Span())`
* lists: `Div()(['something', Span(), 1])`
* generators: `Div()(Span() for _ in range(5))`
* single objects with a name (so they can be accessed by name later): `Div()(some_child=Span())`
* or using the TemPy objects's API: `Div().append((Span())` *see below for a complete API listing*

<aside class="warning">Attention: named insertion is safe only using Python >= 3.6</aside>

HTML tags have attributes, and so TemPy tags have too. It's possible to define tag attributes in different ways:

* during the element instantiation: `Div(some_attribute='some_value')`
* usind the `attr` API: `Div().attr(some_attribute='some_value')`


The resulting tree is the DOM, and can be rendered by calling the `.render()` method.

Calling `render` on some TemPy object will return the html representation of the tree starting from the current element including all the childs.
`tempy_object.render()` will:
* render `tempy_object` own tag, with foud tag attributes
* loop over `tempy_object` childs to retrieve the tag inner content, for every child:
  * a valid `TempyREPR` is searched inside the child class definition, if found it's used.
  * a `render` method will be searched and called if present into the child object.
  * if the object is a subclass of `Escaped`, the `Escaped`'s content is returned
  * if no other contition is met, `str()` will be called on the child
* evry content found will be joined using ''.join()

### Tempy tags classes
```python
from tempy.tags import Div, Br

Div.render()  # Subclass of tempy.elements.Tag
>>> <div></div>

Br.render()  # Subclass of tempy.elements.VoidTag
>>> <br/>
```
```python
from tempy.elements import Tag

# Making a tag that repeats itself, whi? Because!
class Double(Tag):
    __tag = 'custom'
    def render(self, *args, **kwargs):
        return super().render() * 2

Double()('content').render()
>>> <custom>content</custom><custom>content</custom>
```

TemPy provides a class for every HTML5 element defined in the [W3C reference](https://www.w3.org/wiki/HTML/Elements), those classes can be imported from the `tempy.tags` submodule.
Each Tempy Tag class is either a `tempy.elements.Tag` with a start and an end tag, that can contain something, or a `tempy.elements.VoidTag` that can not contain thigs and is composed of a single html tag mark.

A few TemPy tags have custom behaviour:

* the `tempy.tags.Comment` tag needs as first argment the comment string
* the `tempy.tags.Doctype` tag needs as first parameter a doctype code to choose between:
    * html
    * html_strict
    * html_transitional
    * html_frameset
    * xhtml_strict
    * xhtml_transitional
    * xhtml_frameset
    * xhtml_1_1_dtd
    * xhtml_basic_1_1
* the `tempy.tags.Html` tag accepts the keyword argument "doctype", so it adds a Doctype tag before himself (default is "HTML" doctype)
* a `tempy.tags.A` tag with nothing inside it will render himself with the href string iside it

It's possible to define custom tag subclassing either `tempy.elements.Tag` or `tempy.elements.VoidTag` providing a custom `__tag` attribute, a custom `__template` and/or a custom `render` method.

### Make Tempy tags with the T object
```python
from tempy import T
from tempy.elements import Tag

my_tag = T.Custom
my_tag
>>> <class tempy.t.Custom>
issubclass(my_tag, Tag)
>>> True
my_tag().render()
>>> <custom></custom>

my_tag = T['custom with spaces']
my_tag().render()
>>> <custom with spaces></custom with spaces>

# Same for void tags using the Void specialized class factory
my_void_tag = T.Void.CustomVoid
my_void_tag().render()
>>> <customvoid/>
```

Another way to make custom tag is to use the `T` object. The T object is a multi-feature object that work as a class factory for custom tags.

Accessing an attribute/key of the T object will produce a TemPy Tag (of VoidTag) subclass named after the given attribute/key (in lowercase).

Classes made with `T` are subclasses of `tempy.tempy.DOMElement` and behave like any other TemPy Tag, they inherits the api and the features of TemPy objects.


`T` can also produce TemPy tags from html strings on the fly. Using the `from_string` method it converts html strings into a list of TemPy trees:

<code id='lefty-code'>from tempy import T
html_string = '&lt;div&gt;I come from a &lt;i&gt;weird&lt;/i&gt; webservice or from an old file, &lt;b&gt;beware!&lt;/b&gt;&lt;/div&gt;'
parsed = T.from_string(html_string)
div = parsed[0]
div 
&gt;&gt;&gt; &lt;tempy.tags.Div 111803135243328262154888799873263607712. 4 childs.&gt;
div[0]
&gt;&gt;&gt; 'I come from a '
div[1]
&gt;&gt;&gt; &lt;tempy.tags.I 42050800055174656216927079271212875762. Son of Div. 1 childs.&gt;
div[1][0]
&gt;&gt;&gt; 'weird'
</code>


## Blocks and Content

```python
# --- file: base_elements.py
from tempy.tags import Div, Img, Ul, Li, A
from somewhere import links, foot_imgs
# define some common blocks
header = Div(klass='header')(title=Div()('My website'), logo=Img(src='img.png'))
menu = Div(klass='menu')(Ul()((Li()(A(href=link)) for link in links))
footer = Div(klass='coolFooterClass')(Img(src=img) for img in foot_imgs)
```

```python
# --- file: pages.py
from tempy.tags import Html, Head, Body
from base_elements import header, menu, footer

# import the common blocks and use them inside your page
home_page = Html()(Head(), body=Body()(header, menu, content='Hello world.', footer=footer))
content_page = Html()(Head(), body=Body()(header, menu, Content('header'), Content('content'), footer=footer))
```

```python
# --- file: my_controller.py
from tempy.elements import Content
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

```python
from tempy import Escaped

_ = Div()(Escaped("""<p>
here is some html I had in my closet
</p>
"""))
```

TemPy lets you build blocks and put them together using the manipulation api, each TemPy object can be used later inside another TemPy object.

Static parts of pages can be created once an then used in different pages. Blocks can be created and imported as normal Python objects.

<aside class="warning">Depending on the web framework you use, TemPy instances can be shared between http requests. Keep in mind that if you modify a TemPy instance used in various pages, this will be modified in every page and in every subsequent request.</aside>

To make a Block a dynamic, so it can contain different contents each request/use, we can use TemPy's `Content` class.

Those elements are just containers with no html representation, at render time his childs will be rendered inside the `Content`'s father. `Content` can have a fixed content so it can be used as 'html invisible box' (this fixed content can, however, be dynamic), or it can just have a name.

Every TemPy objects can contain extra data that will not be rendered, you can manage this extra data with the `TemPyClass.data()` api as if it's a common dictionary. At render time TemPy will search into the extra data of the `Content` container, and recursively into his parents, looking for a key matching the `Content`'s name. If it's found then it's value is used in rendering.

<code id='lefty-code'>container = Div()(
    Content(content='This is a fixed content')
)
container.render()
&gt;&gt;&gt; &lt;div&gt;This is a fixed content&lt;/div&gt;
</code>

<code id='lefty-code'>container = Div()(
    Content('a_content_name')
)
container.data({'a_content_name': 'This is dynamic content'})
container.render()
&gt;&gt;&gt; &lt;div&gt;This is dynamic content&lt;/div&gt;
</code>

<code id='lefty-code'>root_container = Span()
container = Div()(
    Content('a_content_name')
)
root_container.append(container)
root_container.inject({'a_content_name': 'This is dynamic content'})
root_container.render()
&gt;&gt;&gt; &lt;span&gt;&lt;div&gt;This is dynamic content&lt;/div&gt;&lt;/span&gt;
</code>

Another special TemPy class is the `Escaped` class. With this class you can add content that will not be html escaped, it's useful to inject plain html blocks (in string format) into a TemPy tree.

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

class HomePage(BasePage):

    def init(self):
        self.body.container.content(
            Div()(
                Br(),
                'This is my home page content', Br(),
                H3()('Hame page important content'),
                'Look, I\'m a string!', Br(),
                H3()('H3 is big, really big'),
                H1()('Today\'s content:'),
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

TemPy is designed to provide Object Oriented Templating. You can subclass TemPy classes and define their inner tree structure, and also add custom methods.

For example we can define a basic `tempy.tags.Html` subclass where we define the basic shared page structure (i.e: header, footer, menu and container div structure) and then use this custom page implementation as a base class for several differtent pages of our site.

All the work is made defining a custom `init` method. This method will be called when creating new instances of our class, the concept is similar to Python's `__init__` magic method. TemPy executes each base class `init` method in reverse mro, so your subclass can access all the elements defined in his parent classes. It' like the first thing every `init` does is calling `super().init`.

## TemPy repr's

```python
class MyClass:
    def __init__(self):
        self.foo = 'foo'
        self.bar = 'bar'
        self.link = 'www.foobar.com'
    
    class Div(TempyREPR):
        def repr(self):
            # When a MyClass is placed inside a <div> tag, we add 2 other divs inside with some of our MyClass instance values
            self(
                Div()(self.foo),
                Div()(self.bar)
            )

    class HtmlREPR(TempyREPR):
        def repr(self):
            # We can use HtmlREPR as a fallback for html representation
            self(
                Div()(self.foo),
                Div()(self.bar)
            )
    
    class A(TempyREPR):
        def repr(self):
            # note: This TempyREPR will be used when placing an instance of MyClass inside a Tempy A instance
            # self here will be the instance of MyClass but using the TemPy api we can modify the parent <a> tag
            self.parent.attrs['href'] = self.link
            self('Link to ', self.bar)

    class Td(TempyREPR):
        def repr(self):
            # we define a custom representation when a MyClass is placed inside a <td>
            self(self.bar.upper())
    
    class HomePage(TempyREPR):
        def repr(self):
            # HomePage is supposed to be the name of the TemPy root used to rendere the home page
            # when a MyClass is placed anywhere inside the home page this repr will be used
            # note: here self is the object's parent, not the root
            self('Hello World, this is bar: ', self.bar)


my_instance = MyClass()

Div()(my_instance).render()  # my_instance is rendered using Div(TempyREPR) nested class

A()(my_instance).render()  # my_instance is rendered using A(TempyREPR) nested class

my_list = [MyClass() for _ in range 10]
Table()(Tr()(Td()(instance)) for instance in my_class_list).render()  # Td(TempyREPR) is used.

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
The `TempyREPR.repr` method accepts `self` as the only argument, with a little magic this `self` is both your object and the tree element, so the TemPy API is avaiable and your object attributes are accessible using `self`.

You can define several `TempyREPR` nested classes, TemPy will search for a `TempyREPR` subclass following this priority order:

1. a `TempyREPR` subclass with the same name of his TemPy container
2. a `TempyREPR` subclass with the same name of his TemPy container's root
3. a `TempyREPR` subclass named `HtmlREPR`
4. the first `TempyREPR` found.
5. if none of the previous if found, the object will be rendered calling his `__str__` method

You can use this order to set different renderings for different situation/pages. Here is an example of how `TempyREPR` would work with SQLAlchemy models:

<code id='lefty-code'>from sqlalchemy import Column, DateTime, String, Integer, ForeignKey, func
from sqlalchemy.orm import relationship, backref
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()
</code>
<code id='lefty-code'>
class Department(Base):
    __tablename__ = 'department'
    id = Column(Integer, primary_key=True)
    name = Column(String)
</code>
<code id='lefty-code'>    class HtmlREPR(TempyREPR):
        def repr(self):
            self.attr(klass='department')
            self(self.name.title())
</code>
<code id='lefty-code'>
class Employee(Base):
    __tablename__ = 'employee'
    id = Column(Integer, primary_key=True)
    name = Column(String)
    hired_on = Column(DateTime, default=func.now())
    department_id = Column(Integer, ForeignKey('department.id'))
    department = relationship(
        Department,
        backref=backref('employees',
                         uselist=True,
                         cascade='delete,all'))
</code>
<code id='lefty-code'>    class Table(TempyREPR):
        def repr(self):
            self(
                Tr()(
                    Td()(self.id),
                    Td()(self.name),
                    Td()(self.hired_on.strftime('%Y-%m-%d'))
                )
            )
</code>
<code id='lefty-code'>    class EmployeePage(TempyREPR):
        def repr(self):
            self(
                Div(klass='employee')(
                    Div()('Name: ', self.name),
                    Div()('Department: ', self.department),
                )
            )
</code>

Adding one or more `TempyREPR` into your models will provide the ability to just put instances of those models inside the DOM directly, and they will be rendered according to they're location.

So making a table of employees will be very easy:
<code id='lefty-code'>from tempy.tags import Table
employees_table = Table()(Employee.query.all())</code>

And making a employee page will be easy as well:
<code id='lefty-code'>from tempy.widgest import TempyPage
page = TempyPage().body(Employee.query.first())
</code>


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
page = Html()

# single insertion
page(Head())
>>> <html><head></head></html>

# list insertion
page([Div() for _ in range(2)])
>>> <html><head></head><div></div><div></div></html>

# insertion from generator
page(Span() for _ in range(2))
>>> <html><head></head><div></div><div></div><span></span><span></span></html>

# named insertion
page(some_name='some content')
>>> <html><head></head><div></div><div></div><span></span><span></span>some content</html>
page.some_content = 'some other content'
>>> <html><head></head><div></div><div></div><span></span><span></span>some other content</html>

# mixing up
Html()(
    Head()(
        Title()('Page title')
    ),
    body=Body()(
        'Hello world!',
        'Here some multiplications! ',
        interesting_data=[Div()(n * n2 for n2 in range(10)) for n in range(10)]
    )
)
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

Add TemPy elements end content to a TemPy object by calling this objects like a function. Every call will place the elements at the end of the container's children list.

It's possible to feed a TemPy object with a single element, a list of element, or a generator of elements. Insertion can also be done giving names to a particular child, so it can bew accessed later by name like an attribute of his container element.

<aside class="warning">
Correct ordering with named Tag insertion is ensured with Python >= 3.6 (because kwargs are ordered)
</aside>

Insertion can be mixed, but named insertion can not be before an unnamed insertion.

Another way to manipulate the DOM is to use one of the jQuery-like api.

api | action 
-------------- | --------------
div1.after(div2) | div1 will be the sibling next to div2 inside div2's container | div1
div1.before(div2) | div1 will be the sibling before div2 inside div2's container | div1
div1.prepend(div2) | div2 will be the first child of div1 | div1
div1.prepend_to(div2) | div1 will be the first child of div2 | div1
div1.append(div2) | div2 will be the last child of div1 | div1
div1.append_to(div2) | div1 will be the list child of div2 | div1
div1.wrap(div2) | div1 will be the only child of div2
div1.wrap_inner(div2) | div1 will be added between div2 and his previous parent
div1.replace_with(div2) | elements will be swapped
div1.remove(div2) | div2 will be removed from div1
div1.move_childs(div2) | all the childs of div1 will be moved to div2
div1.move(div2) | div1 will be detached from his father and moved inside div2
div1.pop() | pop by index from div1 childs
div1.empty() | deletes all div1 childs.

## Traversing the DOM
```python
container = Div()(Span() for _ in range(2))
for span in container:
    span('some text')
>>> <div><span>some text</span><span>some text</span></div>
```

```python
container = Div()(Span() for _ in range(2))
container[1]('some text')
>>> <div><span></span><span>some text</span></div>
```

```python
container_div = Div()
container_div(content_div=Div())

container_div.content_div('Some content')
>>> <div><div>Some content</div></div>
```

Every TemPy Tag is iterable and accessible like a Python list, iteration over a TemPy object is an iteration over his childs.

It's also possible to access some child by name, as if they're attributes of the parent.

A jQuery-ish is given to access TemPy instance's childs:
<code id='lefty-code'>container_div.children()
container_div.first()
container_div.last()
container_div.next()
container_div.next_all()
container_div.prev()
container_div.prev_all()
container_div.parent()
container_div.slice(from_index, to_index)
</code>

# API Reference

Several api's are provided to modify you're existing DOM elements, docs ASAP.

For now open your consolle and `help(tempy.Tag)`.

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
