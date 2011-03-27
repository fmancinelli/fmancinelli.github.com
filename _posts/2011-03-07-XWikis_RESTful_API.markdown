---
layout: post
title: An overview of the XWiki's RESTful API
flattr: http://flattr.com/thing/156515/An-overview-of-the-XWikis-RESTful-API
---

[XWiki](http://www.xwiki.org) is a powerful platform for managing knowledge by means of wikis and for developing collaborative applications.

Starting from version 2.0, XWiki is equipped with a powerful API that exposes all its functionalities and that is accessible through the `HTTP` protocol. In this post I will describe the principles we followed during the design phase of this API and how to interact with it.

# What's a RESTful API by the way?

*RESTful API* is a widely (ab)used buzzword to indicate an API that is accessible by means of the `HTTP` protocol. Actually, in order to be RESTful, an API must follow some principles that are drawn from the [REST architectural style](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) proposed by Roy Fielding in his PhD dissertation.

These principles are the following:

* **Addressability** (each relevant resource should be exposed and addressable using a well defined identifier)
* Usage of a **Uniform interface** (every resource must be manipulable using a set of methods with a well defined semantics)
* **Stateless communication** (no conversational state: each request should carry all the information needed to its fulfillment)
* **Hypermedia As The Engine Of Application State** or **HATEOAS** (leverage hypermedia to navigate through application resources and states)

These principles drove the design of XWiki's RESTful API and we tried to stick to them as much as possible.

# The XWiki data model

In order to understand which resources are exposed and addressable by using the XWiki's RESTful API, it is worth to describe the XWiki data model. In fact there is almost a one-to-one correspondence between exposed resources and the entities of this model.

![XWiki model](/images/xwiki_model.png)

The central entity is the **page**. A page has an associated content and a set of metadata providing additionalinformation (e.g., its title, author, creation date, modification date, etc.). Page content is usually written using the XWiki markup and can contain scripts that may access the underlying platform for retrieving and displaying data stored in the system. 

Pages are grouped in **spaces** and a **wiki** is basically a container for a set of spaces.

Each page can have three kinds of entities associated with: **attachments**, **classes** and **objects**.

**Attachments** are binary blobs associated to a page. 

**Classes** are metadata that define a structure. A **class** provides the definition of several **properties**. Each **property** has a type and several attributes that are used to constrain the values that can be assigned to that property and the way it can be edited.

**Objects** represent structured data following the schema defined by a **class**. By using **objects** it is possible to organize and access data in a structured way.

**Classes** and **objects** are what makes XWiki so powerful. Data stored using objects can be edited, searched, manipulated and displayed in a well defined way; that's because of their well defined structure. Applications can leverage this mechanism in order to define, store and edit domain-specific data using the XWiki platform.

**Pages**, their associated **objects**, **classes** and **attachments** are all versioned. Each edit operation creates a new version of these entities, and previous versions are also available for retrieval or rollback.

# Resources and representations

The previously described data model is used to define what are the resources that are addressable and, thus, manipulable via the RESTful API. Each resource can be represented in different ways, depending on the capability of the client and its preferences.

The canonical representation is provided using an `XML` format whose schema if the defined in the [xwiki.rest.model.xsd](http://svn.xwiki.org/svnroot/xwiki/platform/core/trunk/xwiki-rest/xwiki-rest-model/src/main/resources/xwiki.rest.model.xsd) file, and is meant to be processed by automatic clients.

Canonical representations contain navigable hypermedia links that allow a client to discover resources incrementally. Each link is characterized by a well defined relation, the `rel` attribute of the `link` tag, that specify the semantics of the link. For example, in the case of a page, we will find in its representation the links to navigate to the *objects* associated to the page, to its *attachments*, to the history of its *versions* and so on. This mechanism is a realization of the **HATEOAS** principle.

The following snippet shows an excerpt of the `XML` representation of a page. Several links are present at the beginning, allowing the client to go back to the **space** the page is part of, to its previous versions, and so on.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<page xmlns="http://www.xwiki.org">
  <link href="http://localhost:8080/xwiki/rest/wikis/xwiki/spaces/Main"
        rel="http://www.xwiki.org/rel/space"/>
  <link href="http://localhost:8080/xwiki/rest/wikis/xwiki/spaces/Main/pages/WebHome/history"
        rel="http://www.xwiki.org/rel/history"/>
  <link href="http://localhost:8080/xwiki/rest/wikis/xwiki/spaces/Main/pages/WebHome/children"
        rel="http://www.xwiki.org/rel/children"/>
  <link href="http://localhost:8080/xwiki/rest/syntaxes"
        rel="http://www.xwiki.org/rel/syntaxes"/>
  <link href="http://localhost:8080/xwiki/rest/wikis/xwiki/spaces/Main/pages/WebHome"
        rel="self"/>
  <link href="http://localhost:8080/xwiki/rest/wikis/xwiki/classes/Main.WebHome"
        rel="http://www.xwiki.org/rel/class"/>
  <id>xwiki:Main.WebHome</id>
  <fullName>Main.WebHome</fullName>
  <wiki>xwiki</wiki>
  <space>Main</space>
  <name>WebHome</name>
  <title>Welcome to your wiki</title>
  ...
{% endhighlight %}

Some of the relations that we could find in the definition of a link are the following:

* `http://www.xwiki.org/rel/spaces`, link to the representation containing the list of spaces in a wiki.
* `http://www.xwiki.org/rel/pages`, link to the representation containing the list of pages in a space.
* `http://www.xwiki.org/rel/attachments`, link to the representation of the list of attachments associated to the current resource.
* `http://www.xwiki.org/rel/attachmentData`, link to the representation of the actual attachment data.
* `http://www.xwiki.org/rel/objects`, link to the representation of the list of objects associated to the current resource.

A complete *map* of the resources accessible via the RESTful API is available [here](http://platform.xwiki.org/xwiki/bin/download/Features/XWikiRESTfulAPI/XWikiHATEOAS.pdf). 

Nodes are *URI templates* specifying the address of a set of resources. A *URI template* is a parametric *URI* that specify the structure for addressing a collection of resources. For example `http://.../.../spaces/{space}` represents the structure for addressing **spaces** in a wiki where `{space}` is a variable that has to be replaced with the actual name of a **space**.

Edges are links between representations of different resources. For example, if a link exists from a resource `A` to a resource `B` then in the representation of the resource `A` there exist a link to a representation of a resource `B`. The label on the edge specify the relation associated to the link.

The graph shows how from the entry point a clinet can navigate all the resources exposed by the RESTful API. Which means that in order to use the RESTful API a client must know only the entry point or, put in another way, that the API is [hypertext driven](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven).

## Resource URI templates 

*URIs* for accessing XWiki's resources reflect the structure of the data model presented before and they were modeled in a way that a human user can easily figure out *URIs* their structure (or *serendipituosly* discover them):

* **Pages:** `/wikis/{wiki}/spaces/{space}/pages/{page}`
* **Translations:** `/wikis/{wiki}/spaces/{space}/pages/{page}/translations`
* **Objects:** `/wikis/{wiki}/spaces/{space}/pages/{page}/objects/{className}/{id}`
* **Object properties:** `.../pages/{page}/objects/{className}/{id}/properties/{property}`
* **Attachments:** `/wikis/{wiki}/spaces/{space}/pages/{page}/attachments/{attachment}`
* ...and so on.

Since almost everything in XWiki is versioned, most of the previous *URIs* can be used as a prefix for accessing the versioned resources:

* **Page at a given version:** `/wikis/{wiki}/spaces/{space}/pages/{page}/history/{version}`
* **Objects attached to a given version of a page:** 
`.../pages/{page}/history/{version}/objects`
* ...and so on. 

The entrypoint and base *URI* for accessing these resources is `http://host/xwiki/rest`.

Since resource representations contain links to other resources, the complete hierarchy can be discovered by simply navigating through resource representations, starting from representation returned when accessing the API entrypoint. 

The previously mentioned [map](http://platform.xwiki.org/xwiki/bin/download/Features/XWikiRESTfulAPI/XWikiHATEOAS.pdf), in fact, has been generated by simply visiting all the links returned by resource representations, starting from the entry point `http://host/xwiki/rest`.

# Interacting with the RESTful API

Since a RESTful API is by definition based on the `HTTP` protocol, every client that is able to *speak* `HTTP` can be used for interacting with the API. Even a browser!

You can, in fact, fire up your favourite browser and write in the address bar some of the *URIs* mentioned before... taking care of instantiating the variables inside `{}`)

By default you should see appear in your browser window the *XML* representation of the requested resources (more on resource representations later)

However the browser's address is useful only for `GET`ting representations. In order to fully interact with the RESTful API we need a client that is able to speak `HTTP` completely. 

I recommend [curl](http://curl.haxx.se/), a command line utility that is the swiss army knife for `HTTP`.

Resources can respond to different `HTTP` requests:

* **`GET`**: every resource is able to respond to `GET` requests. The result is the resource representation in some kind of format (depending on the client's request)
* **`PUT`**: some resources are able to respond to `PUT` requests. This is the case when a resource can be created or modified and the client is in charge of deciding its *URI*. This happens, for example, with *pages* and *attachments*.
* **`POST`**: some resources are able to respond to `POST` requests. This is the case when a resource can be created but the client is not able to determin its *URI*. This happens, for example, when dealing with *object*: the client cannot determine which `id` the object will have and, hence, its final *URI*.
* **`DELETE`**: some resource are able to respond to `DELETE` requests.

## Authentication

Of course resource manipulation is subject to authorization. The RESTful API provides two ways for authenticating the client:

* Using `HTTP` *basic authentication*
* Providing in the request a session started by using standard XWiki authentication mechanism (i.e., the login form). 

The second mechanism is very useful for calling the RESTful API from AJAX scripts.

If no credentials are sent, then requests are performed on behalf of the *guest* user.

## Representations

Each resource can be represented in different ways. As stated before, the canonical and most complete representation is the *XML* one. Representation are used for retrieving the state of a resource but also for updating it.

This means that if I want to modify a page, you will have to send the page representation in an acceptable format. Usually you will do this using *XML*, but other possibilities are available.

Representations can be selected using `HTTP` headers. `Accept` when requesting and `Content-type` when sending.

## Using the RESTful API

In this section I will present how to interact with the RESTful API using `curl`. You can refer to `curl` manual for a complete description of its features.

### GETting resources

In the following examples, the first line is the command line you have to type at the prompt `$` in order to execute the request.

{% highlight xml %}
$ curl http://localhost:8080/xwiki/rest
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<xwiki xmlns="http://www.xwiki.org">
  <link href="http://localhost:8080/xwiki/rest/wikis" rel="http://www.xwiki.org/rel/wikis"/>
  <link href="http://localhost:8080/xwiki/rest/syntaxes" rel="http://www.xwiki.org/rel/syntaxes"/>
  <version>3.0-SNAPSHOT.34655</version>
</xwiki>
{% endhighlight %}

This is the representation returned by the API entrypoint. You can notice the two links that point you towards the available syntaxes in this wiki, and to the list of the (sub)wikis.

{% highlight xml %}
$ curl -H "Accept: application/json" http://localhost:8080/xwiki/rest
{"version":"3.0-SNAPSHOT.34655"}
{% endhighlight %}

In this case we have requested the *same* resource but with a different format for its representation (i.e., JSON). This is done by sending the `Accept` header and by specifying the *media type* of the desired representation. You can notice that not all the information is present in the JSON information, namely links to other resources are missing.

It is preferable to always specify the desired format because the returned one by default could depend on the deployment. So it is not guaranteed that if no `Accept` header is specified the *XML* representation is returned. To be sure of this always specify the `-H "Content-type: application/xml"` switch.

{% highlight xml %}
$ curl -v http://localhost:8080/xwiki/rest/wikis/xwiki/spaces/Main/pages/Private
> GET /xwiki/rest/wikis/xwiki/spaces/Main/pages/Private HTTP/1.1
> User-Agent: curl/7.21.0 (i686-pc-linux-gnu) libcurl/7.21.0 OpenSSL/0.9.8o zlib/1.2.3.4 libidn/1.18
> Host: localhost:8080
> Accept: */*
> 
< HTTP/1.1 401 Unauthorized
< Content-Type: text/html; charset=ISO-8859-1
< Content-Length: 312
< 
<html>
<head>
   <title>Status page</title>
</head>
<body>
<h3>The request requires user authentication</h3>
<p>You can get technical details
<a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.2">here</a>.<br>
Please continue your visit at our <a href="/">home page</a>.
</p>
</body>
</html>
{% endhighlight %}

In this example we have requested a resource that needs authorization in order to be accessed. The result is an *HTML* page explaining the problem. However what is important to notice is status code returned by the request. The `-v` switch tells `curl` to print all the headers that are present in the request (prefixed by `>`) and in the reply (prefixed by `<`).

By looking at the status code of the reply, in this case `401`, we can understand the outcome of the request and taking appropriate action.

Status code are very important when interacting with the RESTful API. The [documentation](http://platform.xwiki.org/xwiki/bin/download/Features/XWikiRESTfulAPI/XWikiHATEOAS.pdf) describes the status code that can be returned when interacting with different resources.

{% highlight xml %}
$ curl -u Admin:admin http://localhost:8080/xwiki/rest/wikis/xwiki/spaces/Main/pages/Private
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<page xmlns="http://www.xwiki.org">
    <link href="http://localhost:8080/xwiki/rest/wikis/xwiki/spaces/Main" .../>
    <link href="http://localhost:8080/xwiki/rest/wikis/xwiki/spaces/Main/pages/WebHome" .../>
    ...
    <id>xwiki:Main.Private</id>
    <fullName>Main.Private</fullName>
    <wiki>xwiki</wiki>
    <space>Main</space>
    <name>Private</name>
    <title>Private</title>
    <parent>Main.WebHome</parent>
    <parentId>xwiki:Main.WebHome</parentId>
    ...
</page>
{% endhighlight %}

Using *basic authentication* with the `-u Admin:admin` switch we can access the page.

### PUTting and POSTing resources

`HTTP PUT` and `POST` requests are used to create and modify, depending if the client is able to control the *URI* name allocation for the resource or not.

Let's begin with modifying a page.

{% highlight xml %}
$ curl -u Admin:admin -X PUT -H "Content-type: text/plain" -H "Accept: application/xml" 
       -d "Hello world" http://localhost:8080/xwiki/rest/wikis/xwiki/spaces/Main/pages/WebHome
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<page xmlns="http://www.xwiki.org">
  ...
  <content>Hello world</content>
</page>  
{% endhighlight %}

In this example we used the `text/plain` content type in order to send the new representation for the page. The sent text, through the `-d` switch, will become the content of the page.

The response will contain the `XML` representation of the modified resource. We can see that the `<content>` tag contains the sent content.

Sometimes, modifying only the content is not enough...

{% highlight xml %}
$ curl -u Admin:admin -X PUT -H "Content-type: text/plain" -H "Accept: application/xml" 
       -d "@content.xml" http://localhost:8080/xwiki/rest/wikis/xwiki/spaces/Main/pages/WebHome
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<page xmlns="http://www.xwiki.org">
  ...
  <title>New title</title
  <content>New content</content>
</page>  

where content.xml contains

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<page xmlns="http://www.xwiki.org">
  <title>New title</title>
  <content>New content</content>
</page>
{% endhighlight %}

In this case we have sent an *XML* representation that contains also the new title for the page.

Another useful representation for modifying resources is the `x-www-form-urlencoded`.

{% highlight xml %}
$ curl -u Admin:admin -X PUT -H "Content-type: application/x-www-form-urlencoded" 
       --data-urlencode "title=Foo" 
       --data-urlencode "content=Bar" 
       http://localhost:8080/xwiki/rest/wikis/xwiki/spaces/Main/pages/WebHome
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<page xmlns="http://www.xwiki.org">
  <title>Foo</title>
  <content>Bar</content>
</page>       
{% endhighlight %}

In this case we have specified the new content by using key,value pairs as prescribed by the `x-www-form-urlencoded` format. This is actually the same format that is used when we submit forms using a browser. 

If you want to use the RESTful API inside from your web application you might want to use this format.

### DELETEing resources

Resource deletion is quite straightforward.

{% highlight xml %}
$ curl -v -u Admin:admin 
       -X DELETE http://localhost:8080/xwiki/rest/wikis/xwiki/spaces/Main/pages/WebHome
> DELETE /xwiki/rest/wikis/xwiki/spaces/Main/pages/WebHome HTTP/1.1
> Authorization: Basic QWRtaW46YWRtaW4=
> User-Agent: curl/7.21.0 (i686-pc-linux-gnu) libcurl/7.21.0 OpenSSL/0.9.8o zlib/1.2.3.4 libidn/1.18
> 
< HTTP/1.1 204 No Content
{% endhighlight %}

A `204` status code will tell us that the resource has been successfully deleted. Subsequent requests for retrieving the resource will produce `404 Not Found` status codes.

# Conclusion

This post was an introduction to XWiki's RESTful API. You can find more information on the [documentation](http://platform.xwiki.org/xwiki/bin/view/Features/XWikiRESTfulAPI) page.

In future posts I will show you how to extend the API in order to provide additional resources that might be useful for your use cases.
