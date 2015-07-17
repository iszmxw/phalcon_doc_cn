生成 URL 和 路径Generating URLs and Paths
===============================================
:doc:`Phalcon\\Mvc\\Url <../api/Phalcon_Mvc_Url>`是phalcon应用中生成urls的组件。可以根据路由生成独立的urls.

:doc:`Phalcon\\Mvc\\Url <../api/Phalcon_Mvc_Url>` is the component responsible of generate urls in a Phalcon application. It's
capable of produce independent urls based on routes.

设置站点基地址 Setting a base URI
------------------------------------
根据应用安装的根目录，会有个基uri或没有。

Depending of which directory of your document root your application is installed, it may have a base uri or not.

例如，如果文档根目录为/var/www/htdocs，应用安装在/var/www/htdocs/invo 。那基url就是/invo/。如果使用虚拟目录或者是安装在根目录则基url为/。执行如下代码去检查Phalcon的基url。

For example, if your document root is /var/www/htdocs and your application is installed in /var/www/htdocs/invo then your
baseUri will be /invo/. If you are using a VirtualHost or your application is installed on the document root, then your baseUri is /.
Execute the following code to know the base uri detected by Phalcon:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Url;

    $url = new Url();
    echo $url->getBaseUri();

默认情况下Phalcon自动检测到基url。为了增加应用搞的性能可以手动设置：	
	
By default, Phalcon automatically may detect your baseUri, but if you want to increase the performance of your application
is recommended setting up it manually:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Url;

    $url = new Url();

    //Setting a relative base URI
    $url->setBaseUri('/invo/');

    //Setting a full domain as base URI
    $url->setBaseUri('//my.domain.com/');

    //Setting a full domain as base URI
    $url->setBaseUri('http://my.domain.com/my-app/');

通常，这个组件在依赖注入容器中完成注册。如下所示：	
	
Usually, this component must be registered in the Dependency Injector container, so you can set up it there:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Url;

    $di->set('url', function(){
        $url = new Url();
        $url->setBaseUri('/invo/');
        return $url;
    });

生成 URIGenerating URIs
-----------------------------
如果使用:doc:`Router <routing>`的默认行为，我们的应用将匹配/:controller/:action/:params模式。使用get方法将非常简单的实现这种模式的链接:

If you are using the :doc:`Router <routing>` with its default behavior. Your application is able to match routes based on the
following pattern: /:controller/:action/:params. Accordingly it is easy to create routes that satisfy that pattern (or any other
pattern defined in the router) passing a string to the method "get":

.. code-block:: php

    <?php echo $url->get("products/save") ?>

注意并不需要必须添加基url的前缀。如果有一个命名的路由，可以动态的去变更，如下所示：	
	
Note that isn't necessary to prepend the base uri. If you have named routes you can easily change it creating it dynamically.
For Example if you have the following route:

.. code-block:: php

    <?php

    $route->add('/blog/{year}/{month}/{title}', array(
        'controller' => 'posts',
        'action'     => 'show'
    ))->setName('show-post');

可以用如下方式生成url:	
	
A URL can be generated in the following way:

.. code-block:: php

    <?php

    //This produces: /blog/2012/01/some-blog-post
    $url->get(array(
        'for'   => 'show-post',
        'year'  => 2012,
        'month' => '01',
        'title' => 'some-blog-post'
    ));

非伪静态生成URL Producing URLs without Mod-Rewrite
-----------------------------------------------------
使用这个组件还可以生成没有重写的url地址：

You can use this component also to create urls without mod-rewrite:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Url;

    $url = new Url();

    //Pass the URI in $_GET["_url"]
    $url->setBaseUri('/invo/index.php?_url=/');

    //This produce: /invo/index.php?_url=/products/save
    echo $url->get("products/save");

同样可以使用$_SERVER["REQUEST_URI"]：	
	
You can also use $_SERVER["REQUEST_URI"]:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Url;

    $url = new Url();

    //Pass the URI in $_GET["_url"]
    $url->setBaseUri('/invo/index.php?_url=/');

    //Pass the URI using $_SERVER["REQUEST_URI"]
    $url->setBaseUri('/invo/index.php/');

这样需要在路由中手动处理请求的URI：	
	
In this case, it's necessary to manually handle the required URI in the Router:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    $router = new Router();

    // ... define routes

    $uri = str_replace($_SERVER["SCRIPT_NAME"], '', $_SERVER["REQUEST_URI"]);
    $router->handle($uri);

生成路由如下所示:	
	
The produced routes would look like:

.. code-block:: php

    <?php

    //This produce: /invo/index.php/products/save
    echo $url->get("products/save");

Volt 中生成 URL Producing URLs from Volt
-----------------------------------------------
在模板引擎volt中url是可以用来生成url的：

The function "url" is available in volt to generate URLs using this component:

.. code-block:: html+jinja

    <a href="{{ url("posts/edit/1002") }}">Edit</a>

生成静态url：	
	
Generate static routes:

.. code-block:: html+jinja

    <link rel="stylesheet" href="{{ static_url("css/style.css") }}" type="text/css" />

静态 URI 与 动态 URI Static vs. Dynamic Uris
----------------------------------------------
这个组件可以让我们在应用中为静态资源设置一个不同的基url地址：

This component allow you to set up a different base uri for static resources in the application:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Url;

    $url = new Url();

    //Dynamic URIs are
    $url->setBaseUri('/');

    //Static resources go through a CDN
    $url->setStaticBaseUri('http://static.mywebsite.com/');

使用:doc:`Phalcon\\Tag <tags>`需要提供动态和静态的urls。	
	
:doc:`Phalcon\\Tag <tags>` will request both dynamical and static URIs using this component.

自定义 URL 生成器 Implementing your own Url Generator
-----------------------------------------------------------
如果要自定义phalcon中url生成方法:doc:`Phalcon\\Mvc\\UrlInterface <../api/Phalcon_Mvc_UrlInterface>`这个必须要被集成实现。

The :doc:`Phalcon\\Mvc\\UrlInterface <../api/Phalcon_Mvc_UrlInterface>` interface must be implemented to create your own URL
generator replacing the one provided by Phalcon.
