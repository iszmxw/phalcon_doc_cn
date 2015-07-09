返回响应Returning Responses
===============================
HTTP周期的一部分是返回响应给客户端。 :doc:`Phalcon\\Http\\Response <../api/Phalcon_Http_Response>`是phalcon为此设计的组件。HTTP响应包含响应头和响应体。下面代码演示了基本使用：

Part of the HTTP cycle is returning responses to clients. :doc:`Phalcon\\Http\\Response <../api/Phalcon_Http_Response>` is the Phalcon
component designed to achieve this task. HTTP responses are usually composed by headers and body. The following is an example of basic usage:

.. code-block:: php

    <?php

    use Phalcon\Http\Response;

    //Getting a response instance
    $response = new Response();

    //Set status code
    $response->setStatusCode(404, "Not Found");

    //Set the content of the response
    $response->setContent("Sorry, the page doesn't exist");

    //Send response to the client
    $response->send();

如果使用MVC全栈框架没有必要手动处理返回。然而如果想要从控制器直接返回数据就需要使用了。如下所示：	
	
If you are using the full MVC stack there is no need to create responses manually. However, if you need to return a response
directly from a controller's action follow this example:

.. code-block:: php

    <?php

    use Phalcon\Http\Response;
    use Phalcon\Mvc\Controller;

    class FeedController extends Controller
    {

        public function getAction()
        {
            // Getting a response instance
            $response = new Response();

            $feed     = //.. load here the feed

            //Set the content of the response
            $response->setContent($feed->asString());

            //Return the response
            return $response;
        }

    }

使用头部信息Working with Headers
---------------------------------------
头信息在HTTP响应中也非常重要。它包含了返回HTTP状态，类型和其他信息。

Headers are an important part of the HTTP response. It contains useful information about the response state like the HTTP status,
type of response and much more.

可以如下设置返回头信息：

You can set headers in the following way:

.. code-block:: php

    <?php

    //Setting a header by it's name
    $response->setHeader("Content-Type", "application/pdf");
    $response->setHeader("Content-Disposition", 'attachment; filename="downloaded.pdf"');

    //Setting a raw header
    $response->setRawHeader("HTTP/1.1 200 OK");

:doc:`Phalcon\\Http\\Response\\Headers <../api/Phalcon_Http_Response_Headers>`管理头信息。这个类在发送头信息到客户端之前获得头信息。	
	
A :doc:`Phalcon\\Http\\Response\\Headers <../api/Phalcon_Http_Response_Headers>` bag internally manages headers. This class
retrieves the headers before sending it to client:

.. code-block:: php

    <?php

    //Get the headers bag
    $headers = $response->getHeaders();

    //Get a header by its name
    $contentType = $response->getHeaders()->get("Content-Type");

重定向Making Redirections
-------------------------------
使用:doc:`Phalcon\\Http\\Response <../api/Phalcon_Http_Response>`可以执行HTTP重定向：

With :doc:`Phalcon\\Http\\Response <../api/Phalcon_Http_Response>` you can also execute HTTP redirections:

.. code-block:: php

    <?php

    //Redirect to the default URI
    $response->redirect();

    //Redirect to the local base URI
    $response->redirect("posts/index");

    //Redirect to an external URL
    $response->redirect("http://en.wikipedia.org", true);

    //Redirect specifyng the HTTP status code
    $response->redirect("http://www.example.com/new-location", true, 301);

所有的内部链接由url服务生成。(默认是 :doc:`Phalcon\\Mvc\\Url <url>`)。下面例子演示了如何在应用已定义好的路由中执行重定向操作：	
	
All internal URIs are generated using the 'url' service (by default :doc:`Phalcon\\Mvc\\Url <url>`). This example demonstrates
how you can redirect using a route you have defined in your application:

.. code-block:: php

    <?php

    //Redirect based on a named route
    return $response->redirect(array(
        "for"        => "index-lang",
        "lang"       => "jp",
        "controller" => "index"
    ));

需要注意的是并没有禁用掉试图组件。如果有和当前操作关联的视图也将会执行。可以使用$this->view->disable()从控制器中关闭视图的执行；	
	
Note that a redirection doesn't disable the view component, so if there is a view associated with the current action it
will be executed anyway. You can disable the view from a controller by executing $this->view->disable();

HTTP 缓存HTTP Cache
--------------------
增加应用性能减少服务器压力的一种简单的办法就是使用HTTP缓存。现代新的浏览器都支持HTTP缓存。使用HTTP缓存也就是为什么现在很多网站速度非常快的原因。

One of the easiest ways to improve the performance in your applications and reduce the server traffic is using HTTP Cache.
Most modern browsers support HTTP caching. HTTP Cache is one of the reasons many websites are currently fast.

在应用程序发送第一个页面给客户端时，可以使用以下头值更改HTTP缓存:

HTTP Cache can be altered in the following header values sent by the application when serving a page for the first time:

* *Expires:* 使用这个值应用程序可以设置一个未来或过去的日期去告诉浏览器页面到期时间.
* *Cache-Control:* 这个头允许指定一个页面应该考虑多长时间内在浏览器中认为是新页面.
* *Last-Modified:* 这头告诉浏览器页页面上一次的更新时间避免页面重新加载
* *ETag:* etag是一个包括当前页面的修改时间戳的惟一的标识符

* *Expires:* With this header the application can set a date in the future or the past telling the browser when the page must expire.
* *Cache-Control:* This header allows to specify how much time a page should be considered fresh in the browser.
* *Last-Modified:* This header tells the browser which was the last time the site was updated avoiding page re-loads
* *ETag:* An etag is a unique identifier that must be created including the modification timestamp of the current page

设置过期时间Setting an Expiration Time
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
到期日期是最简单和最有效的方法之一来缓存页面在客户端(浏览器)。　　从当前日期到我们添加的截止日期为止所有的页面信息会被储存在浏览器缓存。在这个日期没有过期前不会从服务器请求新的内容:

The expiration date is one of the easiest and most effective ways to cache a page in the client (browser).
Starting from the current date we add the amount of time the page will be stored
in the browser cache. Until this date expires no new content will be requested from the server:

.. code-block:: php

    <?php

    $expireDate = new DateTime();
    $expireDate->modify('+2 months');

    $response->setExpires($expireDate);

响应组件将会在Expires header中显示相应的GMT时区的日期。如果我们设置过期时间为一个过去的时间，则浏览器每次都会从服务器请求新的数据；	
	
The Response component automatically shows the date in GMT timezone as expected in an Expires header.

If we set this value to a date in the past the browser will always refresh the requested page:

.. code-block:: php

    <?php

    $expireDate = new DateTime();
    $expireDate->modify('-10 minutes');

    $response->setExpires($expireDate);

浏览器缓存依赖于客户端设备的时钟时间。如果客户端时间设置为过去的时间，那每次将会从服务器请求新的数据，这种浏览器缓存机制作用有限。	
	
Browsers rely on the client's clock to assess if this date has passed or not. The client clock can be modified to
make pages expire and this may represent a limitation for this cache mechanism.

缓存控制Cache-Control
^^^^^^^^^^^^^^^^^^^^^^^^
这个头提供了一种更安全的方法来缓存页面。我们只需要为浏览器指定要缓存页面数据多长时间就可以了。

This header provides a safer way to cache the pages served. We simply must specify a time in seconds telling the browser
how long it must keep the page in its cache:

.. code-block:: php

    <?php

    //Starting from now, cache the page for one day
    $response->setHeader('Cache-Control', 'max-age=86400');

相反可以设置如下避免页面缓存	
	
The opposite effect (avoid page caching) is achieved in this way:

.. code-block:: php

    <?php

    //Never cache the served page
    $response->setHeader('Cache-Control', 'private, max-age=0, must-revalidate');

E-Tag
^^^^^
"entity-tag" 或者是 "E-tag" 是一个特定唯一的标识符去告诉浏览器在请求之间页面是否有更新。在已经缓存的页面发生变化的时候必须要更改这个标识符去修改内容：

An "entity-tag" or "E-tag" is a unique identifier that helps the browser realize if the page has changed or not between two requests.
The identifier must be calculated taking into account that this must change if the previously served content has changed:

.. code-block:: php

    <?php

    //Calculate the E-Tag based on the modification time of the latest news
    $recentDate = News::maximum(array('column' => 'created_at'));
    $eTag       = md5($recentDate);

    //Send an E-Tag header
    $response->setHeader('E-Tag', $eTag);

