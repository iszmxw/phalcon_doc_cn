教程 2: INVO 简介
===================
在这篇教程中我们介绍一个更复杂的应用，以便于我们后续使用phalcon做更深入的开发。INVO是一个我们用来做案例的应用，用户使用这个小应用可以去产生单据，管理客户和产品，我们可以从 Github_ 克隆代码下来。

In this second tutorial, we'll explain a more complete application in order to deepen the development with Phalcon.
INVO is one of the applications we have created as samples. INVO is a small website that allows their users to
generate invoices, and do other tasks such as manage their customers and products. You can clone its code from Github_.

INVO使用 `Bootstrap`_ 作为前端框架，即使这个应用不能去生成单据，它作为一个理解框架是如何工作的例子还是非常好的。

Also, INVO was made with `Bootstrap`_ as client-side framework. Although the application does not generate
invoices, it still serves as an example to understand how the framework works.

项目结构
----------
当我们完成项目克隆后，在项目根目录下会看到如下文件结构：

Once you clone the project in your document root you'll see the following structure:

.. code-block:: bash

    invo/
        app/
            config/
            controllers/
            library/
            forms/
            models/
            plugins/
            views/
        public/
            bootstrap/
            css/
            js/
        schemas/
		
之前我们提到过phalcon并不限制应用开发的目录结构。这个项目提供了一个简单的MVC的结构和一个公共的public文件目录。
		
As you know, Phalcon does not impose a particular file structure for application development. This project
provides a simple MVC structure and a public document root.

用浏览器访问http://localhost/invo会有如下所示：

Once you open the application in your browser http://localhost/invo you'll see something like this:

.. figure:: ../_static/img/invo-1.png
   :align: center

这个应用分成了两个部分，一个对用户开放前端，用户可以接受和请求INVO消息。第二部分是管理后端，注册用户可以管理他的产品和客户。   
   
The application is divided into two parts, a frontend, that is a public part where visitors can receive information
about INVO and request contact information. The second part is the backend, an administrative area where a
registered user can manage his/her products and customers.

路由
-------
INVO使用 :doc:`Router <routing>` 组件中内置的标准路由。路由匹配按照这个格式/:controller/:action/:params进行。具体的是第一部分是控制器，第二部分是方法，后面的是参数。

INVO uses the standard route that is built-in with the :doc:`Router <routing>` component. These routes match the following
pattern: /:controller/:action/:params. This means that the first part of a URI is the controller, the second the
action and the rest are the parameters.

例如`/session/register` 会执行SessionController控制器中的registerAction方法。

The following route `/session/register` executes the controller SessionController and its action registerAction.

配置
------
INVO配置文件设置了应用的常用参数。这个文件在app/config/config.ini，在启动文件(public/index.php)中的开始部分加载了这个配置文件。

INVO has a configuration file that sets general parameters in the application. This file is located at
app/config/config.ini and it's loaded in the very first lines of the application bootstrap (public/index.php):

.. code-block:: php

    <?php

    use Phalcon\Config\Adapter\Ini as ConfigIni;

    // ...

    /**
     * Read the configuration
     */
    $config = new ConfigIni(APP_PATH . 'app/config/config.ini');

:doc:`Phalcon\\Config <config>` 允许我们以面向对象的方式操作文件。在这个应用中我们使用ini文件配置，其实有很多的配置文件适配器可以使用的。配置文件包含了如下设置：	
	
:doc:`Phalcon\\Config <config>` allows us to manipulate the file in an object-oriented way.
In this example, we're using a ini file as configuration, however, there are more adapters supported
for configuration files. The configuration file contains the following settings:

.. code-block:: ini

    [database]
    adapter  = Mysql
    host     = localhost
    username = root
    password =
    name     = invo

    [application]
    controllersDir = app/controllers/
    modelsDir      = app/models/
    viewsDir       = app/views/
    pluginsDir     = app/plugins/
    formsDir       = app/forms/
    libraryDir     = app/library/
    baseUri        = /invo/

Phalcon没有任何预定义的常量设置。配置分组帮助我们合理的组织配置选项，在这个配置文件中有“application”和“database”两个分组。

Phalcon hasn't any pre-defined convention settings. Sections help us to organize the options as appropriate.
In this file there are two sections to be used later "application" and "database".

自动加载器
-----------
在启动文件(public/index.php)的第二部分是自动加载器：

The second part that appears in the bootstrap file (public/index.php) is the autoloader:

.. code-block:: php

    <?php

    /**
     * Auto-loader configuration
     */
    require APP_PATH . 'app/config/loader.php';

自动加载器设置了应用以后会用到的类的目录。	
	
The autoloader registers a set of directories in which the application will look for
the classes that it eventually will need.

.. code-block:: php

    <?php

    $loader = new \Phalcon\Loader();

    /**
     * We're a registering a set of directories taken from the configuration file
     */
    $loader->registerDirs(
        array(
            APP_PATH . $config->application->controllersDir,
            APP_PATH . $config->application->pluginsDir,
            APP_PATH . $config->application->libraryDir,
            APP_PATH . $config->application->modelsDir,
            APP_PATH . $config->application->formsDir,
        )
    )->register();

注意上面的代码注册了我们在配置文件中指定的目录。唯一没有注册的目录就是视图目录，因为视图包含的是html和php代码而不是类。同样的我们使用了定义在启动文件(public/index.php)中的APP_PATH常量，使用这个常量我们引用项目的根目录：
	
Note that the above code has registered the directories that were defined in the configuration file. The only
directory that is not registered is the viewsDir, because it contains HTML + PHP files but no classes.
Also, note that we have using a constant called APP_PATH, this constant is defined in the bootstrap
(public/index.php) to allow us have a reference to the root of our project:

.. code-block:: php

    <?php

    // ...

    define('APP_PATH', realpath('..') . '/');

注册服务
-------------
另一个在启动文件中需要的文件是(app/config/services.php)。在这个文件中我们可以组织INVO用到的服务类。

Another file that is required in the bootstrap is (app/config/services.php). This file allow
us to organize the services that INVO does use.

.. code-block:: php

    <?php

    /**
     * Load application services
     */
    require APP_PATH . 'app/config/services.php';

注册服务的操作流程和上一个教程中的一样，利用一个闭包延迟加载所需的组件:

Service registration is achieved as in the previous tutorial, making use of a closure to lazily loads
the required components:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Url as UrlProvider;

    // ...

    /**
     * The URL component is used to generate all kind of urls in the application
     */
    $di->set('url', function() use ($config){
        $url = new UrlProvider();
        $url->setBaseUri($config->application->baseUri);
        return $url;
    });

后续我们会详细的再讨论。	
	
We will discuss this file in depth later

处理请求
-----------
如果我们跳到文件(public/index.php)的末尾，所有的请求会被Phalcon\\Mvc\\Application处理，它负责初始化并执行请求并让应用运行起来。

If we skip to the end of the file (public/index.php), the request is finally handled by Phalcon\\Mvc\\Application
which initializes and executes all that is necessary to make the application run:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Application;

    // ...

    $app = new Application($di);

    echo $app->handle()->getContent();

依赖注入
----------
看上面的第一行代码块,应用程序的类构造函数接收变量$di作为参数。这个变量是什么意思呢？Phalcon是个松耦合框架，所以我们需要一个像是胶水的组件将其他组件能够整合在一起运行。这个组件就是Phalcon\\DI。它是个实现依赖注入和服务定位的服务容器，实例化应用程序需要的所有组件。

Look at the first line of the code block above, the Application class constructor is receiving the variable
$di as an argument. What is the purpose of that variable? Phalcon is a highly decoupled framework,
so we need a component that acts as glue to make everything work together. That component is Phalcon\\DI.
It is a service container that also performs dependency injection and service location,
instantiating all components as they are needed by the application.

在容器中注册服务有很多方法。在INVO中大多数服务已经通过使用匿名函数或闭包完成注册。正是由于这一点,对象的实例化可以延迟加载执行,减少应用程序所需的资源。

There are many ways of registering services in the container. In INVO, most services have been registered using
anonymous functions/closures. Thanks to this, the objects are instantiated in a lazy way, reducing the resources needed
by the application.

例如,在下面的代码中会话服务被注册。这个匿名函数只有当应用程序需要访问会话数据时才会被调用：

For instance, in the following excerpt the session service is registered. The anonymous function will only be
called when the application requires access to the session data:

.. code-block:: php

    <?php

    use Phalcon\Session\Adapter\Files as Session;

    // ...

    // Start the session the first time a component requests the session service
    $di->set('session', function() {
        $session = new Session();
        $session->start();
        return $session;
    });

我们可以自由改变适配器去执行额外的初始化工作或者更多操作。注意：服务注册使用名字“session”。这是一个约定，通过名称让框架来在服务容器中识别是否有这个服务。	
	
Here, we have the freedom to change the adapter, perform additional initialization and much more. Note that the service
was registered using the name "session". This is a convention that will allow the framework to identify the active
service in the services container.

请求可能会使用很多的服务，如果去单独注册每一个服务，这将是一个繁琐的任务。出于这个原因，框架提供了一个Phalcon\\DI变体称为Phalcon\\DI\\FactoryDefault 它的功能就是去注册所有的服务来为我们提供了一个完整的框架。

A request can use many services and registering each service individually can be a cumbersome task. For that reason,
the framework provides a variant of Phalcon\\DI called Phalcon\\DI\\FactoryDefault whose task is to register
all services providing a full-stack framework.

.. code-block:: php

    <?php

    use Phalcon\DI\FactoryDefault;

    // ...

    // The FactoryDefault Dependency Injector automatically registers the
    // right services providing a full-stack framework
    $di = new FactoryDefault();

上面的代码注册了的大部分框架提供的标准组件。如果我们需要覆盖一些服务的定义，我们可以像上面操作一样单独去注册“session”或者是“url”。这就是为什么变量$di存在的原因。	
	
It registers the majority of services with components provided by the framework as standard. If we need to override
the definition of some service we could just set it again as we did above with "session" or "url".
This is the reason for the existence of the variable $di.

在下一章,我们将看到如何在INVO中实现身份验证和授权。

In next chapter, we will see how to authentication and authorization is implemented in INVO.

.. _Github: https://github.com/phalcon/invo
.. _Bootstrap: http://getbootstrap.com/
