MVC 应用MVC Applications
==============================
在Phalcon，策划MVC操作背后的全部困难工作通常都可以 通过 :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>`做到。这个组件封装了全部后端所需要的复杂 操作，实例化每一个需要用到的组件并与项目整合在一起，从而使得MVC模式可以如期地运行。

All the hard work behind orchestrating the operation of MVC in Phalcon is normally done by
:doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>`. This component encapsulates all the complex
operations required in the background, instantiating every component needed and integrating it with the
project, to allow the MVC pattern to operate as desired.

单模块或多模块应用Single or Multi Module Applications
----------------------------------------------------------
通过这个组件，你可以运行各式各样的MVC结构：

With this component you can run various types of MVC structures:

单模块Single Module
^^^^^^^^^^^^^^^^^^^^^^
单一的MVC应用仅仅包含了一个模块。可以使用命名空间，但不是必需的。 这样类型的应用可能会有以下文件目录结构：

Single MVC applications consist of one module only. Namespaces can be used but are not necessary.
An application like this would have the following file structure:

.. code-block:: php

    single/
        app/
            controllers/
            models/
            views/
        public/
            css/
            img/
            js/

如果未使用命名空间，以下的启动文件可用于编排MVC工作流：			
			
If namespaces are not used, the following bootstrap file could be used to orchestrate the MVC flow:

.. code-block:: php

    <?php

    use Phalcon\Loader;
    use Phalcon\Mvc\View;
    use Phalcon\Mvc\Application;
    use Phalcon\DI\FactoryDefault;

    $loader = new Loader();

    $loader->registerDirs(
        array(
            '../apps/controllers/',
            '../apps/models/'
        )
    )->register();

    $di = new FactoryDefault();

    // Registering the view component
    $di->set('view', function() {
        $view = new View();
        $view->setViewsDir('../apps/views/');
        return $view;
    });

    try {

        $application = new Application($di);

        echo $application->handle()->getContent();

    } catch (\Exception $e) {
        echo $e->getMessage();
    }

如果使用了命名空间，则可以使用以下启动文件（译者注：主要区别在于使用$loader的方式）：	
	
If namespaces are used, the following bootstrap can be used:

.. code-block:: php

    <?php

    use Phalcon\Loader;
    use Phalcon\Mvc\View;
    use Phalcon\Mvc\Dispatcher;
    use Phalcon\Mvc\Application;
    use Phalcon\DI\FactoryDefault;

    $loader = new Loader();

    // Use autoloading with namespaces prefixes
    $loader->registerNamespaces(
        array(
            'Single\Controllers' => '../apps/controllers/',
            'Single\Models'      => '../apps/models/',
        )
    )->register();

    $di = new FactoryDefault();

    // Register the dispatcher setting a Namespace for controllers
    $di->set('dispatcher', function() {
        $dispatcher = new Dispatcher();
        $dispatcher->setDefaultNamespace('Single\Controllers');
        return $dispatcher;
    });

    // Registering the view component
    $di->set('view', function() {
        $view = new View();
        $view->setViewsDir('../apps/views/');
        return $view;
    });

    try {

        $application = new Application($di);

        echo $application->handle()->getContent();

    } catch(\Exception $e){
        echo $e->getMessage();
    }


多模块Multi Module
^^^^^^^^^^^^^^^^^^^^^^^^^
多模块的应用使用了相同的文档根目录但拥有多个模块。在这种情况下，可以使用以下的文件目录结构：

A multi-module application uses the same document root for more than one module. In this case the following file structure can be used:

.. code-block:: php

    multiple/
      apps/
        frontend/
           controllers/
           models/
           views/
           Module.php
        backend/
           controllers/
           models/
           views/
           Module.php
      public/
        css/
        img/
        js/

在apps/下的每一个目录都有自己的MVC结构。Module.php文件代表了各个模块不同的配置，如自动加载器和自定义服务：		
		
Each directory in apps/ have its own MVC structure. A Module.php is present to configure specific settings of each module like autoloaders or custom services:

.. code-block:: php

    <?php

    namespace Multiple\Backend;

    use Phalcon\Loader;
    use Phalcon\Mvc\View;
    use Phalcon\Mvc\Dispatcher;
    use Phalcon\Mvc\ModuleDefinitionInterface;

    class Module implements ModuleDefinitionInterface
    {

        /**
         * Register a specific autoloader for the module
         */
        public function registerAutoloaders()
        {

            $loader = new Loader();

            $loader->registerNamespaces(
                array(
                    'Multiple\Backend\Controllers' => '../apps/backend/controllers/',
                    'Multiple\Backend\Models'      => '../apps/backend/models/',
                )
            );

            $loader->register();
        }

        /**
         * Register specific services for the module
         */
        public function registerServices($di)
        {

            //Registering a dispatcher
            $di->set('dispatcher', function() {
                $dispatcher = new Dispatcher();
                $dispatcher->setDefaultNamespace("Multiple\Backend\Controllers");
                return $dispatcher;
            });

            //Registering the view component
            $di->set('view', function() {
                $view = new View();
                $view->setViewsDir('../apps/backend/views/');
                return $view;
            });
        }

    }

还需要一个指定的启动文件来加载多模块的MVC架构：	
	
A special bootstrap file is required to load the a multi-module MVC architecture:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;
    use Phalcon\Mvc\Application;
    use Phalcon\DI\FactoryDefault;

    $di = new FactoryDefault();

    //Specify routes for modules
    $di->set('router', function () {

        $router = new Router();

        $router->setDefaultModule("frontend");

        $router->add("/login", array(
            'module'     => 'backend',
            'controller' => 'login',
            'action'     => 'index',
        ));

        $router->add("/admin/products/:action", array(
            'module'     => 'backend',
            'controller' => 'products',
            'action'     => 1,
        ));

        $router->add("/products/:action", array(
            'controller' => 'products',
            'action'     => 1,
        ));

        return $router;
    });

    try {

        //Create an application
        $application = new Application($di);

        // Register the installed modules
        $application->registerModules(
            array(
                'frontend' => array(
                    'className' => 'Multiple\Frontend\Module',
                    'path'      => '../apps/frontend/Module.php',
                ),
                'backend'  => array(
                    'className' => 'Multiple\Backend\Module',
                    'path'      => '../apps/backend/Module.php',
                )
            )
        );

        //Handle the request
        echo $application->handle()->getContent();

    } catch(\Exception $e){
        echo $e->getMessage();
    }

如果你想在启动文件保持模块的配置，你可以使用匿名函数来注册对应的模块：	
	
If you want to maintain the module configuration in the bootstrap file you can use an anonymous function to register the
module:

.. code-block:: php

    <?php

    use Phalcon\Mvc\View;

    //Creating a view component
    $view = new View();

    //Set options to view component
    //...

    // Register the installed modules
    $application->registerModules(
        array(
            'frontend' => function($di) use ($view) {
                $di->setShared('view', function() use ($view) {
                    $view->setViewsDir('../apps/frontend/views/');
                    return $view;
                });
            },
            'backend' => function($di) use ($view) {
                $di->setShared('view', function() use ($view) {
                    $view->setViewsDir('../apps/backend/views/');
                    return $view;
                });
            }
        )
    );

当 :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` 有多个模块注册时，通常每个都是需要的，以便每一个被匹配到的路由都能返回一个有效的模块。每个已经注册的模块都有一个相关的类来提供建立和启动自身的函数。 而每个模块定义的类都必须实现registerAutoloaders()和registerServices()这两个方法，这两个函数会在模块即被执行时被:doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` 调用。	
	
When :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` have modules registered, always is
necessary that every matched route returns a valid module. Each registered module has an associated class
offering functions to set the module itself up. Each module class definition must implement two
methods: registerAutoloaders() and registerServices(), they will be called by
:doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` according to the module to be executed.

理解默认行为Understanding the default behavior
-------------------------------------------------
如果你已经看过了 tutorial 或者已经通过 Phalcon Devtools 生成了代码， 你将很容易识别以下的启动文件：

If you've been following the :doc:`tutorial <tutorial>` or have generated the code using :doc:`Phalcon Devtools <tools>`,
you may recognize the following bootstrap file:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Application;

    try {

        // Register autoloaders
        //...

        // Register services
        //...

        // Handle the request
        $application = new Application($di);

        echo $application->handle()->getContent();

    } catch (\Exception $e) {
        echo "Exception: ", $e->getMessage();
    }

控制器中全部核心的工作都会在handle()被回调时触发执行。	
	
The core of all the work of the controller occurs when handle() is invoked:

.. code-block:: php

    <?php

    echo $application->handle()->getContent();

手动启动Manual bootstrapping
----------------------------------
如果你不想使用 :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>`，以上的代码可以改成这样：

If you do not wish to use :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>`, the code above can be changed as follows:

.. code-block:: php

    <?php

    // Get the 'router' service
    $router = $di['router'];

    $router->handle();

    $view = $di['view'];

    $dispatcher = $di['dispatcher'];

    // Pass the processed router parameters to the dispatcher
    $dispatcher->setControllerName($router->getControllerName());
    $dispatcher->setActionName($router->getActionName());
    $dispatcher->setParams($router->getParams());

    // Start the view
    $view->start();

    // Dispatch the request
    $dispatcher->dispatch();

    // Render the related views
    $view->render(
        $dispatcher->getControllerName(),
        $dispatcher->getActionName(),
        $dispatcher->getParams()
    );

    // Finish the view
    $view->finish();

    $response = $di['response'];

    // Pass the output of the view to the response
    $response->setContent($view->getContent());

    // Send the request headers
    $response->sendHeaders();

    // Print the response
    echo $response->getContent();

以下代码替换了 :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>`，虽然缺少了视图组件， 但却更适合Rest风格的API接口：	
	
The following replacement of :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` lacks of a view component making
it suitable for Rest APIs:

.. code-block:: php

    <?php

    // Get the 'router' service
    $router = $di['router'];

    $router->handle();

    $dispatcher = $di['dispatcher'];

    // Pass the processed router parameters to the dispatcher
    $dispatcher->setControllerName($router->getControllerName());
    $dispatcher->setActionName($router->getActionName());
    $dispatcher->setParams($router->getParams());

    // Dispatch the request
    $dispatcher->dispatch();

    //Get the returned value by the latest executed action
    $response = $dispatcher->getReturnedValue();

    //Check if the action returned is a 'response' object
    if ($response instanceof Phalcon\Http\ResponseInterface) {

        //Send the request
        $response->send();
    }

另外一个修改就是在分发器中对抛出异常的捕捉可以将请求转发到其他的操作：	
	
Yet another alternative that catch exceptions produced in the dispatcher forwarding to other actions consequently:

.. code-block:: php

    <?php

    // Get the 'router' service
    $router = $di['router'];

    $router->handle();

    $dispatcher = $di['dispatcher'];

    // Pass the processed router parameters to the dispatcher
    $dispatcher->setControllerName($router->getControllerName());
    $dispatcher->setActionName($router->getActionName());
    $dispatcher->setParams($router->getParams());

    try {

        // Dispatch the request
        $dispatcher->dispatch();

    } catch (Exception $e) {

        //An exception has occurred, dispatch some controller/action aimed for that

        // Pass the processed router parameters to the dispatcher
        $dispatcher->setControllerName('errors');
        $dispatcher->setActionName('action503');

        // Dispatch the request
        $dispatcher->dispatch();

    }

    //Get the returned value by the latest executed action
    $response = $dispatcher->getReturnedValue();

    //Check if the action returned is a 'response' object
    if ($response instanceof Phalcon\Http\ResponseInterface) {

        //Send the request
        $response->send();
    }

尽管上面的代码比使用 :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` 而需要的代码远远要累赘得很， 但它为启动你的应用提供了一个可修改、可定制化的途径。 因为根据你的项目需要，你可以想对实例什么和不实例化什么进行完全的控制，或者想用你自己的组件来替代那些确定和必须的组件从而扩展默认的功能。	
	
Although the above implementations are a lot more verbose than the code needed while using :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>`,
it offers an alternative in bootstrapping your application. Depending on your needs, you might want to have full control of what
should be instantiated or not, or replace certain components with those of your own to extend the default functionality.

应用事件Application Events
-------------------------------
:doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` 可以把事件发送到 EventsManager （如果它激活的话）。 事件将被当作”application”类型被消费掉。目前已支持的事件如下：

:doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` is able to send events to the :doc:`EventsManager <events>`
(if it is present). Events are triggered using the type "application". The following events are supported:

+---------------------+--------------------------------------------------------------+
| Event Name          | Triggered                                                    |
+=====================+==============================================================+
| boot                | Executed when the application handles its first request      |
+---------------------+--------------------------------------------------------------+
| beforeStartModule   | Before initialize a module, only when modules are registered |
+---------------------+--------------------------------------------------------------+
| afterStartModule    | After initialize a module, only when modules are registered  |
+---------------------+--------------------------------------------------------------+
| beforeHandleRequest | Before execute the dispatch loop                             |
+---------------------+--------------------------------------------------------------+
| afterHandleRequest  | After execute the dispatch loop                              |
+---------------------+--------------------------------------------------------------+

以下示例演示了如何将侦听器绑定到组件：

The following example demonstrates how to attach listeners to this component:

.. code-block:: php

    <?php

    use Phalcon\Events\Manager as EventsManager;

    $eventsManager = new EventsManager();

    $application->setEventsManager($eventsManager);

    $eventsManager->attach(
        "application",
        function($event, $application) {
            // ...
        }
    );

外部资源	
	
External Resources
------------------
* `MVC examples on Github <https://github.com/phalcon/mvc>`_
