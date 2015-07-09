微应用Micro Applications
===============================
使用Phalcon框架开发者可以创建微框架应用。 这样开发者只需要书写极少的代码即可创建一个PHP应用。 微应用适用于书写小的应用， API或原型等

With Phalcon you can create "Micro-Framework like" applications. By doing this, you only need to write a minimal amount of
code to create a PHP application. Micro applications are suitable to implement small applications, APIs and
prototypes in a practical way.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro;

    $app = new Micro();

    $app->get('/say/welcome/{name}', function ($name) {
        echo "<h1>Welcome $name!</h1>";
    });

    $app->handle();

创建微应用Creating a Micro Application
----------------------------------------------
Phalcon中 使用 :doc:`Phalcon\\Mvc\\Micro <../api/Phalcon_Mvc_Micro>`来实现微应用。

:doc:`Phalcon\\Mvc\\Micro <../api/Phalcon_Mvc_Micro>` is the class responsible for implementing a micro application.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro;

    $app = new Micro();

定义路由Defining routes
--------------------------
实例化后， 开发者需要添加一些路由规则。 Phalcon内部使用 :doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>`来管理路由。 路由必须以 / 开头。 定义路由时通常会书写http方法约束， 这样路由规则只适用于那些和规则及htttp方法相匹配的路由。 下面的方法展示了如何定义了HTTP get方法路由：

After instantiating the object, you will need to add some routes. :doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` manages routing internally.
Routes must always start with /. A HTTP method constraint is optionally required when defining routes, so as to instruct
the router to match only if the request also matches the HTTP methods. The following example shows how to define
a route for the method GET:

.. code-block:: php

    <?php

    $app->get('/say/hello/{name}', function ($name) {
        echo "<h1>Hello! $name</h1>";
    });

get 方法指定了要匹配的请求方法。 路由规则 /say/hello/{name} 中含有一个参数 {$name}, 此参数会直接传递给路由的处理器（此处为匿名函数）。 路由规则匹配时处理器即会执行。 处理器是PHP中任何可以被调用的项。 下面的示例中展示了如何定义不同种类的处理器：	
	
The "get" method indicates that the associated HTTP method is GET. The route /say/hello/{name} also has a parameter {$name} that is passed
directly to the route handler (the anonymous function). Handlers are executed when a route is matched. A handler could be
any callable item in the PHP userland. The following example shows how to define different types of handlers:

.. code-block:: php

    <?php

    // With a function
    function say_hello($name) {
        echo "<h1>Hello! $name</h1>";
    }

    $app->get('/say/hello/{name}', "say_hello");

    // With a static method
    $app->get('/say/hello/{name}', "SomeClass::someSayMethod");

    // With a method in an object
    $myController = new MyController();
    $app->get('/say/hello/{name}', array($myController, "someAction"));

    //Anonymous function
    $app->get('/say/hello/{name}', function ($name) {
        echo "<h1>Hello! $name</h1>";
    });

:doc:`Phalcon\\Mvc\\Micro <../api/Phalcon_Mvc_Micro>` 提供了一系列的用于定义http方法的限定方法：	
	
:doc:`Phalcon\\Mvc\\Micro <../api/Phalcon_Mvc_Micro>` provides a set of methods to define the HTTP method (or methods)
which the route is constrained for:

.. code-block:: php

    <?php

    //Matches if the HTTP method is GET
    $app->get('/api/products', "get_products");

    //Matches if the HTTP method is POST
    $app->post('/api/products/add', "add_product");

    //Matches if the HTTP method is PUT
    $app->put('/api/products/update/{id}', "update_product");

    //Matches if the HTTP method is DELETE
    $app->delete('/api/products/remove/{id}', "delete_product");

    //Matches if the HTTP method is OPTIONS
    $app->options('/api/products/info/{id}', "info_product");

    //Matches if the HTTP method is PATCH
    $app->patch('/api/products/update/{id}', "info_product");

    //Matches if the HTTP method is GET or POST
    $app->map('/repos/store/refs',"action_product")->via(array('GET', 'POST'));


路由参数Routes with Parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如上面的例子中展示的那样在路由中定义参数是非常容易的。 参数名需要放在花括号内。 参数格式亦可使用正则表达式以确保数据一致性。 例子如下：

Defining parameters in routes is very easy as demonstrated above. The name of the parameter has to be enclosed in brackets. Parameter
formatting is also available using regular expressions to ensure consistency of data. This is demonstrated in the example below:

.. code-block:: php

    <?php

    //This route have two parameters and each of them have a format
    $app->get('/posts/{year:[0-9]+}/{title:[a-zA-Z\-]+}', function ($year, $title) {
        echo "<h1>Title: $title</h1>";
        echo "<h2>Year: $year</h2>";
    });

起始路由Starting Route
^^^^^^^^^^^^^^^^^^^^^^^^^
通常情况下， 应用一般由 / 路径开始访问， 当然此访问多为 GET方法。 这种情况代码如下：

Normally, the starting route in an application is the route /, and it will more frequent to be accessed by the method GET.
This scenario is coded as follows:

.. code-block:: php

    <?php

    //This is the start route
    $app->get('/', function () {
        echo "<h1>Welcome!</h1>";
    });

重写规则Rewrite Rules
^^^^^^^^^^^^^^^^^^^^^^^^
下面的规则用来实现apache重写：

The following rules can be used together with Apache to rewrite the URis:

.. code-block:: apacheconf

    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php?_url=/$1 [QSA,L]
    </IfModule>

处理响应Working with Responses
-----------------------------------
开发者可以在路由处理器中设置任务种类的响应：直接输出， 使用模板引擎， 包含视图， 返回json数据等。

You are free to produce any kind of response in a handler: directly make an output, use a template engine, include a view,
return a json, etc.:

.. code-block:: php

    <?php

    //Direct output
    $app->get('/say/hello', function () {
        echo "<h1>Hello! $name</h1>";
    });

    //Requiring another file
    $app->get('/show/results', function () {
        require 'views/results.php';
    });

    //Returning a JSON
    $app->get('/get/some-json', function () {
        echo json_encode(array("some", "important", "data"));
    });

另外开发者还可以使用 :doc:`"response" <response>` ， 这样开发者可以更好的处理结果：	
	
In addition to that, you have access to the service :doc:`"response" <response>`, with which you can manipulate better the
response:

.. code-block:: php

    <?php

    $app->get('/show/data', function () use ($app) {

        //Set the Content-Type header
        $app->response->setContentType('text/plain')->sendHeaders();

        //Print a file
        readfile("data.txt");

    });

或回复response对象：	
	
Or create a response object and return it from the handler:

.. code-block:: php

    <?php

    $app->get('/show/data', function () {

        //Create a response
        $response = new Phalcon\Http\Response();

        //Set the Content-Type header
        $response->setContentType('text/plain');

        //Pass the content of a file
        $response->setContent(file_get_contents("data.txt"));

        //Return the response
        return $response;
    });

重定向Making redirections
-------------------------------
重定向用来在当前的处理中跳转到其它的处理流：

Redirections could be performed to forward the execution flow to another route:

.. code-block:: php

    <?php

    //This route makes a redirection to another route
    $app->post('/old/welcome', function () use ($app) {
        $app->response->redirect("new/welcome")->sendHeaders();
    });

    $app->post('/new/welcome', function () use ($app) {
        echo 'This is the new Welcome';
    });

根据路由生成 URLGenerating URLs for Routes
---------------------------------------------
Phalcon中使用 :doc:`Phalcon\\Mvc\\Url <url>` 来生成其它的基于路由的URL。 开发者可以为路由设置名字， 通过这种方式 “url” 服务可以产生相关的路由：

:doc:`Phalcon\\Mvc\\Url <url>` can be used to produce URLs based on the defined routes. You need to set up a name for the route;
by this way the "url" service can produce the corresponding URL:

.. code-block:: php

    <?php

    //Set a route with the name "show-post"
    $app->get('/blog/{year}/{title}', function ($year, $title) use ($app) {

        //.. show the post here

    })->setName('show-post');

    //produce an URL somewhere
    $app->get('/', function() use ($app) {

        echo '<a href="', $app->url->get(array(
            'for'   => 'show-post',
            'title' => 'php-is-a-great-framework',
            'year'  => 2012
        )), '">Show the post</a>';

    });


与依赖注入的交互Interacting with the Dependency Injector
----------------------------------------------------------
微应用中， :doc:`Phalcon\\DI\\FactoryDefault <di>` 是隐含生成的， 不过开发者可以明确的生成此类的实例以用来管理相关的服务：

In the micro application, a :doc:`Phalcon\\DI\\FactoryDefault <di>` services container is created implicitly; additionally you
can create outside the application a container to manipulate its services:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro;
    use Phalcon\DI\FactoryDefault;
    use Phalcon\Config\Adapter\Ini as IniConfig;

    $di = new FactoryDefault();

    $di->set('config', function() {
        return new IniConfig("config.ini");
    });

    $app = new Micro();

    $app->setDI($di);

    $app->get('/', function () use ($app) {
        //Read a setting from the config
        echo $app->config->app_name;
    });

    $app->post('/contact', function () use ($app) {
        $app->flash->success('Yes!, the contact was made!');
    });

服务容器中可以使用数据类的语法来设置或取服务实例：	
	
The array-syntax is allowed to easily set/get services in the internal services container:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro;
    use Phalcon\Db\Adapter\Pdo\Mysql as MysqlAdapter;

    $app = new Micro();

    //Setup the database service
    $app['db'] = function() {
        return new MysqlAdapter(array(
            "host"     => "localhost",
            "username" => "root",
            "password" => "secret",
            "dbname"   => "test_db"
        ));
    };

    $app->get('/blog', function () use ($app) {
        $news = $app['db']->query('SELECT * FROM news');
        foreach ($news as $new) {
            echo $new->title;
        }
    });

处理Not-FoundNot-Found Handler
--------------------------------
当用户访问未定义的路由时， 微应用会试着执行 “Not-Found”处理器。 示例如下：

When an user tries to access a route that is not defined, the micro application will try to execute the "Not-Found" handler.
An example of that behavior is below:

.. code-block:: php

    <?php

    $app->notFound(function () use ($app) {
        $app->response->setStatusCode(404, "Not Found")->sendHeaders();
        echo 'This is crazy, but this page was not found!';
    });

微应用中的模型Models in Micro Applications
-----------------------------------------------
Phalcon中开发者可以直接使用 :doc:`Models <models>` ， 开发者只需要一个类自动加载器来加载模型：

:doc:`Models <models>` can be used transparently in Micro Applications, only is required an autoloader to load models:

.. code-block:: php

    <?php

    $loader = new \Phalcon\Loader();

    $loader->registerDirs(array(
        __DIR__ . '/models/'
    ))->register();

    $app = new \Phalcon\Mvc\Micro();

    $app->get('/products/find', function(){

        foreach (Products::find() as $product) {
            echo $product->name, '<br>';
        }

    });

    $app->handle();

微应用中的事件Micro Application Events
------------------------------------------
当有事件发生时 :doc:`Phalcon\\Mvc\\Micro <../api/Phalcon_Mvc_Micro>` 会发送事件到 EventsManager 。 这里使用 “micro” 来绑定处理事件。 支持如下事件：

:doc:`Phalcon\\Mvc\\Micro <../api/Phalcon_Mvc_Micro>` is able to send events to the :doc:`EventsManager <events>` (if it is present).
Events are triggered using the type "micro". The following events are supported:

+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| Event Name          | Triggered                                                                                                                  | Can stop operation?  |
+=====================+============================================================================================================================+======================+
| beforeHandleRoute   | The main method is just called, at this point the application doesn't know if there is some matched route                  | Yes                  |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| beforeExecuteRoute  | A route has been matched and it contains a valid handler, at this point the handler has not been executed                  | Yes                  |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| afterExecuteRoute   | Triggered after running the handler                                                                                        | No                   |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| beforeNotFound      | Triggered when any of the defined routes match the requested URI                                                           | Yes                  |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| afterHandleRoute    | Triggered after completing the whole process in a successful way                                                           | Yes                  |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+

下面的例子中， 我们阐述了如何使用事件来控制应用的安全性:

In the following example, we explain how to control the application security using events:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro,
        Phalcon\Events\Manager as EventsManager;

    //Create a events manager
    $eventManager = new EventsManager();

    //Listen all the application events
    $eventManager->attach('micro', function($event, $app) {

        if ($event->getType() == 'beforeExecuteRoute') {
            if ($app->session->get('auth') == false) {

                $app->flashSession->error("The user isn't authenticated");
                $app->response->redirect("/")->sendHeaders();

                //Return (false) stop the operation
                return false;
            }
        }

    });

    $app = new Micro();

    //Bind the events manager to the app
    $app->setEventsManager($eventManager);

中间件事件Middleware events
--------------------------------
此外， 应用事件亦可使用 ‘before’, ‘after’, ‘finish’等来绑定：

In addition to the events manager, events can be added using the methods 'before', 'after' and 'finish':

.. code-block:: php

    <?php

    $app = new Phalcon\Mvc\Micro();

    //Executed before every route is executed
    //Return false cancels the route execution
    $app->before(function() use ($app) {
        if ($app['session']->get('auth') == false) {
            return false;
        }
        return true;
    });

    $app->map('/api/robots', function(){
        return array(
            'status' => 'OK'
        );
    });

    $app->after(function() use ($app) {
        //This is executed after the route was executed
        echo json_encode($app->getReturnedValue());
    });

    $app->finish(function() use ($app) {
        //This is executed when the request has been served
    });

开发者可以对同一事件注册多个处理器:	
	
You can call the methods several times to add more events of the same type:

.. code-block:: php

    <?php

    $app->finish(function() use ($app) {
        //First 'finish' middleware
    });

    $app->finish(function() use ($app) {
        //Second 'finish' middleware
    });

把这些代码放在另外的文件中以达到重用的目的:	
	
Code for middlewares can be reused using separate classes:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro\MiddlewareInterface;

    /**
     * CacheMiddleware
     *
     * Caches pages to reduce processing
     */
    class CacheMiddleware implements MiddlewareInterface
    {
        public function call($application)
        {

            $cache  = $application['cache'];
            $router = $application['router'];

            $key    = preg_replace('/^[a-zA-Z0-9]/', '', $router->getRewriteUri());

            //Check if the request is cached
            if ($cache->exists($key)) {
                echo $cache->get($key);
                return false;
            }

            return true;
        }
    }

添加实例到应用:	
	
Then add the instance to the application:

.. code-block:: php

    <?php

    $app->before(new CacheMiddleware());

支持如下的中间件事件	
	
The following middleware events are available:

+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| Event Name          | Triggered                                                                                                                  | Can stop operation?  |
+=====================+============================================================================================================================+======================+
| before              | Before executing the handler. It can be used to control the access to the application                                      | Yes                  |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| after               | Executed after the handler is executed. It can be used to prepare the response                                             | No                   |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| finish              | Executed after sending the response. It can be used to perform clean-up                                                    | No                   |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+

使用控制器处理Using Controllers as Handlers
---------------------------------------------
中型的应用可以使用 Micro\\MVC 来组织控制器中的处理器。 开发者也可以使用 :doc:`Phalcon\\Mvc\\Micro\\Collection <../api/Phalcon_Mvc_Micro_Collection>`来对控制器中的处理器进行归组：

Medium applications using the Micro\\MVC approach may require organize handlers in controllers.
You can use :doc:`Phalcon\\Mvc\\Micro\\Collection <../api/Phalcon_Mvc_Micro_Collection>` to group handlers that belongs to controllers:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro\Collection as MicroCollection;

    $posts = new MicroCollection();

    //Set the main handler. ie. a controller instance
    $posts->setHandler(new PostsController());

    //Set a common prefix for all routes
    $posts->setPrefix('/posts');

    //Use the method 'index' in PostsController
    $posts->get('/', 'index');

    //Use the method 'show' in PostsController
    $posts->get('/show/{slug}', 'show');

    $app->mount($posts);

PostsController形如下：	
	
The controller 'PostsController' might look like this:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function index()
        {
            //...
        }

        public function show($slug)
        {
            //...
        }
    }

上面的例子中，我们直接对控制器进行了实例化， 使用集合时Phalcon会提供了迟加载的能力， 这样程序只有在匹配路由时才加载控制器：	
	
In the above example the controller is directly instantiated, Collection also have the ability to lazy-load controllers, this option
provide better performance loading controllers only if the related routes are matched:

.. code-block:: php

    <?php

    $posts->setHandler('PostsController', true);
    $posts->setHandler('Blog\Controllers\PostsController', true);

返回响应Returning Responses
------------------------------
处理器可能会返回原生的  :doc:`Phalcon\\Http\\Response <response>` 实例或实现了相关接口的组件。 当返回Response对象时， 应用会自动的把处理结果返回到客户端。

Handlers may return raw responses using :doc:`Phalcon\\Http\\Response <response>` or a component that implements the relevant interface.
When responses are returned by handlers they are automatically sent by the application.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro;
    use Phalcon\Http\Response;

    $app = new Micro();

    //Return a response
    $app->get('/welcome/index', function() {

        $response = new Response();

        $response->setStatusCode(401, "Unauthorized");

        $response->setContent("Access is not authorized");

        return $response;
    });

渲染视图Rendering Views
---------------------------
:doc:`Phalcon\\Mvc\\View\\Simple <views>`可用来渲染视图， 示例如下：

:doc:`Phalcon\\Mvc\\View\\Simple <views>` can be used to render views, the following example shows how to do that:

.. code-block:: php

    <?php

    $app = new Phalcon\Mvc\Micro();

    $app['view'] = function() {
        $view = new \Phalcon\Mvc\View\Simple();
        $view->setViewsDir('app/views/');
        return $view;
    };

    //Return a rendered view
    $app->get('/products/show', function() use ($app) {

        // Render app/views/products/show.phtml passing some variables
        echo $app['view']->render('products/show', array(
            'id'   => 100,
            'name' => 'Artichoke'
        ));

    });

错误处理Error Handling
--------------------------
在微应用中异常被抛出的时候可以产生一个合适的返回：

A proper response can be generated if an exception is raised in a micro handler:

.. code-block:: php

    <?php

    $app = new Phalcon\Mvc\Micro();

    $app->get('/', function() {
        throw new \Exception("An error");
    });

    $app->error(function($exception) {
        echo "An error has occurred";
    });

如果返回false则异常处理终止。	
	
If the handler returns "false" the exception is stopped.

相关资源Related Sources
----------------------------
* :doc:`Creating a Simple REST API <tutorial-rest>` is a tutorial that explains how to create a micro application to implement a RESTful web service.
* `Stickers Store <http://store.phalconphp.com>`_ is a very simple micro-application making use of the micro-mvc approach [`Github <https://github.com/phalcon/store>`_].
