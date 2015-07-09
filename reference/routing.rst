路由Routing
===================
路由器组件用来定义处理接收到的请求的路由，指向相应的控制器或者处理程序。路由器只是简单解析一个URI获取这些信息。 路由器有两种模式：MVC模式以及匹配模式。第一种模式主要适合MVC应用。

The router component allows defining routes that are mapped to controllers or handlers that should receive
the request. A router simply parses a URI to determine this information. The router has two modes: MVC
mode and match-only mode. The first mode is ideal for working with MVC applications.

定义路由Defining Routes
---------------------------
:doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` 提供高级路由支持。在MVC模式下，你可以定义路由并映射向需要的控制器/动作。 一个路由定义方法如下所示：

:doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` provides advanced routing capabilities. In MVC mode,
you can define routes and map them to controllers/actions that you require. A route is defined as follows:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    // Create the router
    $router = new Router();

    //Define a route
    $router->add(
        "/admin/users/my-profile",
        array(
            "controller" => "users",
            "action"     => "profile",
        )
    );

    //Another route
    $router->add(
        "/admin/users/change-password",
        array(
            "controller" => "users",
            "action"     => "changePassword",
        )
    );

    $router->handle();

add()方法接受一个url作为第一个参数，实际执行控制器和方法数组作为第二个参数。路由并不执行任何操作，它只是包含了为组件(例如 :doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>`)提供controller/action的信息集合。	
	
The method add() receives as first parameter a pattern and optionally a set of paths as second parameter.
In this case, if the URI is exactly: /admin/users/my-profile, then the "users" controller with its action "profile"
will be executed. Currently, the router does not execute the controller and action, it only collects this
information to inform the correct component (ie. :doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>`)
that this is controller/action it should to execute.

一个应用可能会有很多url路径，一个个去定义非常麻烦。这种情况下我们可以定义一个更加灵活的路由：

An application can have many paths, define routes one by one can be a cumbersome task. In these cases we can
create more flexible routes:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    // Create the router
    $router = new Router();

    //Define a route
    $router->add(
        "/admin/:controller/a/:action/:params",
        array(
            "controller" => 1,
            "action"     => 2,
            "params"     => 3,
        )
    );

上面的例子中，使用通配符可以匹配更多的URIs。例如访问(/admin/users/a/delete/dave/301)将会被解析为如下：	
	
In the example above, using wildcards we make a route valid for many URIs. For example, by accessing the
following URL (/admin/users/a/delete/dave/301) then:

+------------+---------------+
| Controller | users         |
+------------+---------------+
| Action     | delete        |
+------------+---------------+
| Parameter  | dave          |
+------------+---------------+
| Parameter  | 301           |
+------------+---------------+

add()方法可以接受预定义占位符和正则表达式修饰符。所有的路由路径必须由(/)开始。正则表达式语法和`PCRE regular expressions`_是一致的。　没有必要添加正则表达式分隔符。所有路由匹配模式是不区分大小写的。

The method add() receives a pattern that optionally could have predefined placeholders and regular expression
modifiers. All the routing patterns must start with a slash character (/). The regular expression syntax used
is the same as the `PCRE regular expressions`_. Note that, it is not necessary to add regular expression
delimiters. All routes patterns are case-insensitive.

第二次参数定义了将绑定的参数如何匹配到controller/action/parameters。由括号分隔开的占位符或子模式(圆括号)是匹配部分。在上面的例子中, 第一子模式匹配(:controller)是路由的控制器,第二个动作一次类推。

The second parameter defines how the matched parts should bind to the controller/action/parameters. Matching
parts are placeholders or subpatterns delimited by parentheses (round brackets). In the example given above, the
first subpattern matched (:controller) is the controller part of the route, the second the action and so on.

这些占位符帮助开发人员编写更可读的和容易理解的正则表达式。以下是支持的占位符:

These placeholders help writing regular expressions that are more readable for developers and easier
to understand. The following placeholders are supported:

+--------------+---------------------+--------------------------------------------------------------------------------------------------------+
| Placeholder  | Regular Expression  | Usage                                                                                                  |
+==============+=====================+========================================================================================================+
| /:module     | /([a-zA-Z0-9\_\-]+) | Matches a valid module name with alpha-numeric characters only                                         |
+--------------+---------------------+--------------------------------------------------------------------------------------------------------+
| /:controller | /([a-zA-Z0-9\_\-]+) | Matches a valid controller name with alpha-numeric characters only                                     |
+--------------+---------------------+--------------------------------------------------------------------------------------------------------+
| /:action     | /([a-zA-Z0-9\_]+)   | Matches a valid action name with alpha-numeric characters only                                         |
+--------------+---------------------+--------------------------------------------------------------------------------------------------------+
| /:params     | (/.*)*              | Matches a list of optional words separated by slashes. Use only this placeholder at the end of a route |
+--------------+---------------------+--------------------------------------------------------------------------------------------------------+
| /:namespace  | /([a-zA-Z0-9\_\-]+) | Matches a single level namespace name                                                                  |
+--------------+---------------------+--------------------------------------------------------------------------------------------------------+
| /:int        | /([0-9]+)           | Matches an integer parameter                                                                           |
+--------------+---------------------+--------------------------------------------------------------------------------------------------------+

控制器名称采用驼峰命名法。这就意味着(-) 和 (_)将会被移除并且下一个字母将会大写。some_controller将会被转化为SomeController。

Controller names are camelized, this means that characters (-) and (_) are removed and the next character
is uppercased. For instance, some_controller is converted to SomeController.

因为要使用add()方法添加很多路由规则,添加的顺序决定了它们匹配的顺序,最后添加路由规格优于最先添加的。在内部所有定义的规格都是按照相反顺序执行的,直到:doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` 匹配到一个URI规格并执行，后面的规则将会被忽略。

Since you can add many routes as you need using add(), the order in which routes are added indicate
their relevance, latest routes added have more relevance than first added. Internally, all defined routes
are traversed in reverse order until :doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` finds the
one that matches the given URI and processes it, while ignoring the rest.

参数名称Parameters with Names
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
下面例子演示了如何定义路由的参数名称：

The example below demonstrates how to define names to route parameters:

.. code-block:: php

    <?php

    $router->add(
        "/news/([0-9]{4})/([0-9]{2})/([0-9]{2})/:params",
        array(
            "controller" => "posts",
            "action"     => "show",
            "year"       => 1, // ([0-9]{4})
            "month"      => 2, // ([0-9]{2})
            "day"        => 3, // ([0-9]{2})
            "params"     => 4, // :params
        )
    );

在上面的例子中，路由没有定义controller和action。它们被posts和show固定替换了。用户觉察不到已经使用了默认路由。在控制器内部可以使用如下方式获得参数。	
	
In the above example, the route doesn't define a "controller" or "action" part. These parts are replaced
with fixed values ("posts" and "show"). The user will not know the controller that is really dispatched
by the request. Inside the controller, those named parameters can be accessed as follows:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function indexAction()
        {

        }

        public function showAction()
        {

            // Return "year" parameter
            $year = $this->dispatcher->getParam("year");

            // Return "month" parameter
            $month = $this->dispatcher->getParam("month");

            // Return "day" parameter
            $day = $this->dispatcher->getParam("day");

        }

    }

注意参数值是从分配器中获得的。之所以可以这样是因为它是最后与驱动应用程序交互的组件。此外,还有另外一个方法来创建命名参数:
	
Note that the values of the parameters are obtained from the dispatcher. This happens because it is the
component that finally interacts with the drivers of your application. Moreover, there is also another
way to create named parameters as part of the pattern:

.. code-block:: php

    <?php

    $router->add(
        "/documentation/{chapter}/{name}.{type:[a-z]+}",
        array(
            "controller" => "documentation",
            "action"     => "show"
        )
    );

我们可以通过如下方式获取值：	
	
You can access their values in the same way as before:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class DocumentationController extends Controller
    {

        public function showAction()
        {

            // Returns "name" parameter
            $name = $this->dispatcher->getParam("name");

            // Returns "type" parameter
            $type = $this->dispatcher->getParam("type");

        }

    }

短语法Short Syntax
^^^^^^^^^^^^^^^^^^^^^^^^
如果不想使用数组定义路由，可以使用如下方式，结果是一样的：

If you don't like using an array to define the route paths, an alternative syntax is also available.
The following examples produce the same result:

.. code-block:: php

    <?php

    // Short form
    $router->add("/posts/{year:[0-9]+}/{title:[a-z\-]+}", "Posts::show");

    // Array form
    $router->add(
        "/posts/([0-9]+)/([a-z\-]+)",
        array(
           "controller" => "posts",
           "action"     => "show",
           "year"       => 1,
           "title"      => 2,
        )
    );

数组和短语法混合使用Mixing Array and Short Syntax
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
数组和短语法混合使用去定义数组。在这个例子中命名参数根据它定义的位置被自动添加到路由路径中。

Array and short syntax can be mixed to define a route, in this case note that named parameters automatically
are added to the route paths according to the position on which they were defined:

.. code-block:: php

    <?php

    //First position must be skipped because it is used for 第一部分使用了命名参数应该被跳过
    //the named parameter 'country'
    $router->add('/news/{country:[a-z]{2}}/([a-z+])/([a-z\-+])',
        array(
            'section' => 2, //Positions start with 2
            'article' => 3
        )
    );

路由到模块Routing to Modules
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
可以定义路径中包含模块的路由。这适用于多模块的应用。可以定义一个包含module通配符的默认路由：

You can define routes whose paths include modules. This is specially suitable to multi-module applications.
It's possible define a default route that includes a module wildcard:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    $router = new Router(false);

    $router->add('/:module/:controller/:action/:params', array(
        'module'     => 1,
        'controller' => 2,
        'action'     => 3,
        'params'     => 4
    ));

在这种情况下路由中必须包含模块名。如下所示，URL: /admin/users/edit/sonny将会被解析为：	
	
In this case, the route always must have the module name as part of the URL. For example, the following
URL: /admin/users/edit/sonny, will be processed as:

+------------+---------------+
| Module     | admin         |
+------------+---------------+
| Controller | users         |
+------------+---------------+
| Action     | edit          |
+------------+---------------+
| Parameter  | sonny         |
+------------+---------------+

或者可以绑定特定的模块到路由中:

Or you can bind specific routes to specific modules:

.. code-block:: php

    <?php

    $router->add("/login", array(
        'module'     => 'backend',
        'controller' => 'login',
        'action'     => 'index',
    ));

    $router->add("/products/:action", array(
        'module'     => 'frontend',
        'controller' => 'products',
        'action'     => 1,
    ));

或者是绑定到特定的命名空间：	
	
Or bind them to specific namespaces:

.. code-block:: php

    <?php

    $router->add("/:namespace/login", array(
        'namespace'  => 1,
        'controller' => 'login',
        'action'     => 'index'
    ));

Namespaces/class必须被单独传递：	
	
Namespaces/class names must be passed separated:

.. code-block:: php

    <?php

    $router->add("/login", array(
        'namespace'  => 'Backend\Controllers',
        'controller' => 'login',
        'action'     => 'index'
    ));

限制 HTTP 请求传入方式 HTTP Method Restrictions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
使用add()添加路由方法后，会被应用到所有的http方法。我们可以限制路由到特定的方法中，在创建RESTful应用的时候非常有用：

When you add a route using simply add(), the route will be enabled for any HTTP method. Sometimes we can restrict a route to a specific method,
this is especially useful when creating RESTful applications:

.. code-block:: php

    <?php

    // This route only will be matched if the HTTP method is GET
    $router->addGet("/products/edit/{id}", "Products::edit");

    // This route only will be matched if the HTTP method is POST
    $router->addPost("/products/save", "Products::save");

    // This route will be matched if the HTTP method is POST or PUT
    $router->add("/products/update")->via(array("POST", "PUT"));

使用转换Using convertions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在将请求传递给分配器前转换方法可以将路由参数进行灵活的转换。下面示例演示如何使用：

Convertions allow to freely transform the route's parameters before passing them to the dispatcher, the following examples show how to use them:

.. code-block:: php

    <?php

    //The action name allows dashes, an action can be: /products/new-ipod-nano-4-generation
    $router
        ->add('/products/{slug:[a-z\-]+}', array(
            'controller' => 'products',
            'action'     => 'show'
        ))
        ->convert('slug', function($slug) {
            //Transform the slug removing the dashes
            return str_replace('-', '', $slug);
        });

路由分组Groups of Routes
^^^^^^^^^^^^^^^^^^^^^^^^^^^
路由中如果包含常用共同的路径可以进行分组方便管理：

If a set of routes have common paths they can be grouped to easily maintain them:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;
    use Phalcon\Mvc\Router\Group as RouterGroup;

    $router = new Router();

    //Create a group with a common module and controller
    $blog = new RouterGroup(array(
        'module'     => 'blog',
        'controller' => 'index'
    ));

    //All the routes start with /blog
    $blog->setPrefix('/blog');

    //Add a route to the group
    $blog->add('/save', array(
        'action' => 'save'
    ));

    //Add another route to the group
    $blog->add('/edit/{id}', array(
        'action' => 'edit'
    ));

    //This route maps to a controller different than the default
    $blog->add('/blog', array(
        'controller' => 'blog',
        'action'     => 'index'
    ));

    //Add the group to the router
    $router->mount($blog);

可以将路由分组放到单独文件中更加合理的组织应用，增加代码的重用性：	
	
You can move groups of routes to separate files in order to improve the organization and code reusing in the application:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router\Group as RouterGroup;

    class BlogRoutes extends RouterGroup
    {
        public function initialize()
        {
            //Default paths
            $this->setPaths(array(
                'module'    => 'blog',
                'namespace' => 'Blog\Controllers'
            ));

            //All the routes start with /blog
            $this->setPrefix('/blog');

            //Add a route to the group
            $this->add('/save', array(
                'action' => 'save'
            ));

            //Add another route to the group
            $this->add('/edit/{id}', array(
                'action' => 'edit'
            ));

            //This route maps to a controller different than the default
            $this->add('/blog', array(
                'controller' => 'blog',
                'action'     => 'index'
            ));

        }
    }

然后在路由中加载路由分组：	
	
Then mount the group in the router:

.. code-block:: php

    <?php

    //Add the group to the router
    $router->mount(new BlogRoutes());

匹配路由Matching Routes
---------------------------
需要传递给路由正确的URI。默认情况下从重写引擎模块的$_GET['_url'] 变量中取得路由URI，可以对phalcon应用一些的正则重写：

A valid URI must be passed to Router in order to let it checks the route that matches that given URI.
By default, the routing URI is taken from the $_GET['_url'] variable that is created by the rewrite engine
module. A couple of rewrite rules that work very well with Phalcon are:

.. code-block:: apacheconf

    RewriteEngine On
    RewriteCond   %{REQUEST_FILENAME} !-d
    RewriteCond   %{REQUEST_FILENAME} !-f
    RewriteRule   ^(.*)$ index.php?_url=/$1 [QSA,L]

下面的例子演示了如何独立使用路由组件：	
	
The following example shows how to use this component in stand-alone mode:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    // Creating a router
    $router = new Router();

    // Define routes here if any
    // ...

    // Taking URI from $_GET["_url"]
    $router->handle();

    // or Setting the URI value directly
    $router->handle("/employees/edit/17");

    // Getting the processed controller
    echo $router->getControllerName();

    // Getting the processed action
    echo $router->getActionName();

    //Get the matched route
    $route = $router->getMatchedRoute();

路由命名Naming Routes
--------------------------
添加到路由器中的路由会被当做对象储存在:doc:`Phalcon\\Mvc\\Router\\Route <../api/Phalcon_Mvc_Router_Route>`中。它封装了路由的细节。在应用中我们可以给路径一个特定的名称。当我们从路由中创建连接的时候非常有用。

Each route that is added to the router is stored internally as an object :doc:`Phalcon\\Mvc\\Router\\Route <../api/Phalcon_Mvc_Router_Route>`.
That class encapsulates all the details of each route. For instance, we can give a name to a path to identify it uniquely in our application.
This is especially useful if you want to create URLs from it.

.. code-block:: php

    <?php

    $route = $router->add("/posts/{year}/{title}", "Posts::show");

    $route->setName("show-posts");

    //or just

    $router->add("/posts/{year}/{title}", "Posts::show")->setName("show-posts");

使用:doc:`Phalcon\\Mvc\\Url <../api/Phalcon_Mvc_Url>`可以从名称创建路由连接：	
	
Then, using for example the component :doc:`Phalcon\\Mvc\\Url <../api/Phalcon_Mvc_Url>` we can build routes from its name:

.. code-block:: php

    <?php

    // returns /posts/2012/phalcon-1-0-released
    echo $url->get(array(
        "for"   => "show-posts",
        "year"  => "2012",
        "title" => "phalcon-1-0-released"
    ));

范例Usage Examples
--------------------------
下面例子是自定义路由：

The following are examples of custom routes:

.. code-block:: php

    <?php

    // matches "/system/admin/a/edit/7001"
    $router->add(
        "/system/:controller/a/:action/:params",
        array(
            "controller" => 1,
            "action"     => 2,
            "params"     => 3
        )
    );

    // matches "/es/news"
    $router->add(
        "/([a-z]{2})/:controller",
        array(
            "controller" => 2,
            "action"     => "index",
            "language"   => 1
        )
    );

    // matches "/es/news"
    $router->add(
        "/{language:[a-z]{2}}/:controller",
        array(
            "controller" => 2,
            "action"     => "index"
        )
    );

    // matches "/admin/posts/edit/100"
    $router->add(
        "/admin/:controller/:action/:int",
        array(
            "controller" => 1,
            "action"     => 2,
            "id"         => 3
        )
    );

    // matches "/posts/2010/02/some-cool-content"
    $router->add(
        "/posts/([0-9]{4})/([0-9]{2})/([a-z\-]+)",
        array(
            "controller" => "posts",
            "action"     => "show",
            "year"       => 1,
            "month"      => 2,
            "title"      => 4
        )
    );

    // matches "/manual/en/translate.adapter.html"
    $router->add(
        "/manual/([a-z]{2})/([a-z\.]+)\.html",
        array(
            "controller" => "manual",
            "action"     => "show",
            "language"   => 1,
            "file"       => 2
        )
    );

    // matches /feed/fr/le-robots-hot-news.atom
    $router->add(
        "/feed/{lang:[a-z]+}/{blog:[a-z\-]+}\.{type:[a-z\-]+}",
        "Feed::get"
    );

    // matches /api/v1/users/peter.json
    $router->add('/api/(v1|v2)/{method:[a-z]+}/{param:[a-z]+}\.(json|xml)',
        array(
            'controller' => 'api',
            'version'    => 1,
            'format'     => 4
        )
    );

.. highlights::
    注意在控制器和命名空间使用的字符，这些字符会被转换为类名并传递给文件系统，可能会让攻击者读取未授权文件。安全的正则表达式为/([a-zA-Z0-9\_\-]+)。

    Beware of characters allowed in regular expression for controllers and namespaces. As these
    become class names and in turn they're passed through the file system could be used by attackers to
    read unauthorized files. A safe regular expression is: /([a-zA-Z0-9\_\-]+)

默认行为Default Behavior
--------------------------
:doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>`有默认的路由行为。匹配如下格式/:controller/:action/:params

:doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` has a default behavior providing a very simple routing that
always expects a URI that matches the following pattern: /:controller/:action/:params

例如， *http://phalconphp.com/documentation/show/about.html*链接会被解析为如下格式：

For example, for a URL like this *http://phalconphp.com/documentation/show/about.html*, this router will translate it as follows:

+------------+---------------+
| Controller | documentation |
+------------+---------------+
| Action     | show          |
+------------+---------------+
| Parameter  | about.html    |
+------------+---------------+

如果不想使用默认路由规格在应用中，可以在创建的时候传入false值：

If you don't want use this routes as default in your application, you must create the router passing false as parameter:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    // Create the router without default routes
    $router = new Router(false);

设置默认路由Setting the default route
---------------------------------------
当访问应用的时候没有输入任何路径，'/'路由将会被使用并决定显示应用的哪个初始页面：

When your application is accessed without any route, the '/' route is used to determine what paths must be used to show the initial page
in your website/application:

.. code-block:: php

    <?php

    $router->add("/", array(
        'controller' => 'index',
        'action'     => 'index'
    ));

没有找到路径Not Found Paths
-------------------------------
如果没有任何匹配的路由，可以定义如下格式路由：

If none of the routes specified in the router are matched, you can define a group of paths to be used in this scenario:

.. code-block:: php

    <?php

    //Set 404 paths
    $router->notFound(array(
        "controller" => "index",
        "action"     => "route404"
    ));

设置默认路径Setting default paths
------------------------------------
可以为通用路径中的 module, controller, action 定义默认值。当一个路由缺少其中任何一项时，路由器可以自动用默认值填充：

It's possible to define default values for common paths like module, controller or action. When a route is missing any of
those paths they can be automatically filled by the router:

.. code-block:: php

    <?php

    //Setting a specific default
    $router->setDefaultModule('backend');
    $router->setDefaultNamespace('Backend\Controllers');
    $router->setDefaultController('index');
    $router->setDefaultAction('index');

    //Using an array
    $router->setDefaults(array(
        'controller' => 'index',
        'action'     => 'index'
    ));

处理结尾额外的斜杆Dealing with extra/trailing slashes
-------------------------------------------------------


Sometimes a route could be accessed with extra/trailing slashes and the end of the route, those extra slashes would lead to produce
a not-found status in the dispatcher. You can set up the router to automatically remove the slashes from the end of handled route:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    $router = new Router();

    //Remove trailing slashes automatically
    $router->removeExtraSlashes(true);

Or, you can modify specific routes to optionally accept trailing slashes:

.. code-block:: php

    <?php

    $router->add(
        '/{language:[a-z]{2}}/:controller[/]{0,1}',
        array(
            'controller' => 2,
            'action'     => 'index'
        )
    );

匹配回调函数Match Callbacks
-----------------------------
有时在特定条件下路由必须被匹配，可以使用beforeMatch回调函数添加任意的条件。如果函数返回false，路由匹配失败：

Sometimes, routes must be matched if they meet specific conditions, you can add arbitrary conditions to routes using the
'beforeMatch' callback, if this function return false, the route will be treaded as non-matched:

.. code-block:: php

    <?php

    $router->add('/login', array(
        'module'     => 'admin',
        'controller' => 'session'
    ))->beforeMatch(function($uri, $route) {
        //Check if the request was made with Ajax
        if ($_SERVER['HTTP_X_REQUESTED_WITH'] == 'xmlhttprequest') {
            return false;
        }
        return true;
    });

类中定义可以重用代码：	
	
You can re-use these extra conditions in classes:

.. code-block:: php

    <?php

    class AjaxFilter
    {
        public function check()
        {
            return $_SERVER['HTTP_X_REQUESTED_WITH'] == 'xmlhttprequest';
        }
    }

可以使用这个类替换匿名函数：	
	
And use this class instead of the anonymous function:

.. code-block:: php

    <?php

    $router->add('/get/info/{id}', array(
        'controller' => 'products',
        'action'     => 'info'
    ))->beforeMatch(array(new AjaxFilter(), 'check'));

限制主机名Hostname Constraints
----------------------------------
路由可以设置限制主机名，只有匹配主机名的才能应用特定的路由或者是路由群组：

The router allow to set hostname constraints, this means that specific routes or a group of routes can be restricted
to only match if the route also meets the hostname constraint:

.. code-block:: php

    <?php

    $router->add('/login', array(
        'module'     => 'admin',
        'controller' => 'session',
        'action'     => 'login'
    ))->setHostName('admin.company.com');

	
主机名也可以是正则表达式：	
	
Hostname can also be regular expressions:

.. code-block:: php

    <?php

    $router->add('/login', array(
        'module'     => 'admin',
        'controller' => 'session',
        'action'     => 'login'
    ))->setHostName('([a-z+]).company.com');

可以在路由分组中设置域名限制：	
	
In groups of routes you can set up a hostname constraint that apply for every route in the group:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router\Group as RouterGroup;

    //Create a group with a common module and controller
    $blog = new RouterGroup(array(
        'module'     => 'blog',
        'controller' => 'posts'
    ));

    //Hostname restriction
    $blog->setHostName('blog.mycompany.com');

    //All the routes start with /blog
    $blog->setPrefix('/blog');

    //Default route
    $blog->add('/', array(
        'action' => 'index'
    ));

    //Add a route to the group
    $blog->add('/save', array(
        'action' => 'save'
    ));

    //Add another route to the group
    $blog->add('/edit/{id}', array(
        'action' => 'edit'
    ));

    //Add the group to the router
    $router->mount($blog);

URI 来源 URI Sources
----------------------
默认情况下URI信息从$_GET['_url']变量中获得，它由重新引擎传递给phalcon，如果需要可以直接使用$_SERVER['REQUEST_URI']。

By default the URI information is obtained from the $_GET['_url'] variable, this is passed by the Rewrite-Engine to
Phalcon, you can also use $_SERVER['REQUEST_URI'] if required:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    ...

    $router->setUriSource(Router::URI_SOURCE_GET_URL); // use $_GET['_url'] (default)
    $router->setUriSource(Router::URI_SOURCE_SERVER_REQUEST_URI); // use $_SERVER['REQUEST_URI'] (default)

或者手动传递URI给handle方法：	
	
Or you can manually pass a URI to the 'handle' method:

.. code-block:: php

    <?php

    $router->handle('/some/route/to/handle');

测试路由Testing your routes
--------------------------------
因为路由组件没有依赖，可以创建如下文件去测试路由：

Since this component has no dependencies, you can create a file as shown below to test your routes:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    //These routes simulate real URIs
    $testRoutes = array(
        '/',
        '/index',
        '/index/index',
        '/index/test',
        '/products',
        '/products/index/',
        '/products/show/101',
    );

    $router = new Router();

    //Add here your custom routes
    //...

    //Testing each route
    foreach ($testRoutes as $testRoute) {

        //Handle the route
        $router->handle($testRoute);

        echo 'Testing ', $testRoute, '<br>';

        //Check if some route was matched
        if ($router->wasMatched()) {
            echo 'Controller: ', $router->getControllerName(), '<br>';
            echo 'Action: ', $router->getActionName(), '<br>';
        } else {
            echo 'The route wasn\'t matched by any route<br>';
        }
        echo '<br>';

    }

路由注解 Annotations Router
------------------------------
路由组件有一个和:doc:`annotations <annotations>`注解组件高度整合的变体。使用它我们可以在控制器中直接定义路由而不用再在注册服务中去定义了。

This component provides a variant that's integrated with the :doc:`annotations <annotations>` service. Using this strategy
you can write the routes directly in the controllers instead of adding them in the service registration:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router\Annotations as RouterAnnotations;

    $di['router'] = function() {

        //Use the annotations router
        $router = new RouterAnnotations(false);

        //Read the annotations from ProductsController if the uri starts with /api/products
        $router->addResource('Products', '/api/products');

        return $router;
    };

注解通过如下方式定义：	
	
The annotations can be defined in the following way:

.. code-block:: php

    <?php

    /**
     * @RoutePrefix("/api/products")
     */
    class ProductsController
    {

        /**
         * @Get("/")
         */
        public function indexAction()
        {

        }

        /**
         * @Get("/edit/{id:[0-9]+}", name="edit-robot")
         */
        public function editAction($id)
        {

        }

        /**
         * @Route("/save", methods={"POST", "PUT"}, name="save-robot")
         */
        public function saveAction()
        {

        }

        /**
         * @Route("/delete/{id:[0-9]+}", methods="DELETE",
         *      conversors={id="MyConversors::checkId"})
         */
        public function deleteAction($id)
        {

        }

        public function infoAction($id)
        {

        }

    }

只有方法被标注了有效的注解才能被作为路由。支持的注解如下：	
	
Only methods marked with valid annotations are used as routes. List of annotations supported:

+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Name         | Description                                                                                       | Usage                                                              |
+==============+===================================================================================================+====================================================================+
| RoutePrefix  | A prefix to be prepended to each route uri. This annotation must be placed at the class' docblock | @RoutePrefix("/api/products")                                      |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Route        | This annotation marks a method as a route. This annotation must be placed in a method docblock    | @Route("/api/products/show")                                       |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Get          | This annotation marks a method as a route restricting the HTTP method to GET                      | @Get("/api/products/search")                                       |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Post         | This annotation marks a method as a route restricting the HTTP method to POST                     | @Post("/api/products/save")                                        |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Put          | This annotation marks a method as a route restricting the HTTP method to PUT                      | @Put("/api/products/save")                                         |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Delete       | This annotation marks a method as a route restricting the HTTP method to DELETE                   | @Delete("/api/products/delete/{id}")                               |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Options      | This annotation marks a method as a route restricting the HTTP method to OPTIONS                  | @Option("/api/products/info")                                      |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+

支持如下参数：

For annotations that add routes, the following parameters are supported:

+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Name         | Description                                                                                       | Usage                                                              |
+==============+===================================================================================================+====================================================================+
| methods      | Define one or more HTTP method that route must meet with                                          | @Route("/api/products", methods={"GET", "POST"})                   |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| name         | Define a name for the route                                                                       | @Route("/api/products", name="get-products")                       |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| paths        | An array of paths like the one passed to Phalcon\\Mvc\\Router::add                                | @Route("/posts/{id}/{slug}", paths={module="backend"})             |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| conversors   | A hash of conversors to be applied to the parameters                                              | @Route("/posts/{id}/{slug}", conversors={id="MyConversor::getId"}) |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+

如果在模块中要映射路由到控制器最好使用addModuleResource方法：

If routes map to controllers in modules is better use the addModuleResource method:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router\Annotations as RouterAnnotations;

    $di['router'] = function() {

        //Use the annotations router
        $router = new RouterAnnotations(false);

        //Read the annotations from Backend\Controllers\ProductsController if the uri starts with /api/products
        $router->addModuleResource('backend', 'Products', '/api/products');

        return $router;
    };

注册路由实例Registering Router instance
-----------------------------------------
可以在服务注册的时候使用Phalcon的依赖注入完成路由器注册从而保证在控制器中全部可用。需要在启动代码里添加如下代码。（例如在/index.php或者使用`Phalcon Developer Tools <http://phalconphp.com/en/download/tools>`_ 添加至app/config/services.php ）

You can register router during service registration with Phalcon dependency injector to make it available inside controller.

You need to add code below in your bootstrap file (for example index.php or app/config/services.php if you use `Phalcon Developer Tools <http://phalconphp.com/en/download/tools>`_)

.. code-block:: php

    <?php

    /**
    * add routing capabilities
    */
    $di->set('router', function(){
        require __DIR__.'/../app/config/routes.php';
        return $router;
    });

需要创建app/config/routes.php并添加路由器初始化代码，例如：	
	
You need to create app/config/routes.php and add router initialization code, for example:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    $router = new Router();

    $router->add("/login", array(
        'controller' => 'login',
        'action'     => 'index',
    ));

    $router->add("/products/:action", array(
        'controller' => 'products',
        'action'     => 1,
    ));

    return $router;


实现自定义路由Implementing your own Router
--------------------------------------------
:doc:`Phalcon\\Mvc\\RouterInterface <../api/Phalcon_Mvc_RouterInterface>`在创建自定义路由的时候必须先被集成实现。

The :doc:`Phalcon\\Mvc\\RouterInterface <../api/Phalcon_Mvc_RouterInterface>` interface must be implemented to create your own router replacing
the one provided by Phalcon.

.. _PCRE regular expressions: http://www.php.net/manual/en/book.pcre.php
