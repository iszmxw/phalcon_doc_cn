使用控制器Using Controllers
==============================
控制器提供了一堆可以被调用的方法，即：action。action是控制器中用于处理请求的方法。默认情况下，全部控制器public的方法都会映射到action并且可以通过URL访问。action负责解释请求和创建响应。 通常，响应是以渲染的视图格式被创建，但也存在其他的方式来创建（译者注：如AJAX请求返回JSON格式的数据）。

The controllers provide a number of methods that are called actions. Actions are methods on a controller that handle requests. By default all
public methods on a controller map to actions and are accessible by an URL. Actions are responsible for interpreting the request and creating
the response. Usually responses are in the form of a rendered view, but there are other ways to create responses as well.

例如，当你访问一个类似这样的URL时：http://localhost/blog/posts/show/2012/the-post-title，Phalcon默认会这样分解各个部分：

For instance, when you access an URL like this: http://localhost/blog/posts/show/2012/the-post-title Phalcon by default will decompose each
part like this:

+------------------------+----------------+
| **Phalcon 目录**       | blog           |
+------------------------+----------------+
| **控制器**             | posts          |
+------------------------+----------------+
| **Action**             | show           |
+------------------------+----------------+
| **参数**               | 2012           |
+------------------------+----------------+
| **参数**               | the-post-title |
+------------------------+----------------+

这时，PostsController将会处理这个请求。在一个项目中，没有强制指定放置控制器的地方，这些控制器都可以 通过使用 autoloaders 来加载，所以你可以根据需要自由组件你的控制器。

In this case, the PostsController will handle this request. There is no a special location to put controllers in an application, they
could be loaded using :doc:`autoloaders <loader>`, so you're free to organize your controllers as you need.

控制器类必须以“Controller”为后缀，action则须以“Action”为后缀。一个控制器类的例子如下：

Controllers must have the suffix "Controller" while actions the suffix "Action". A sample of a controller is as follows:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function indexAction()
        {

        }

        public function showAction($year, $postTitle)
        {

        }

    }

额外的URI参数定义为action的参数，以致这些参数可以简单地通过本地变量来获取。控制器可以选择继承 :doc:`Phalcon\\Mvc\\Controller <../api/Phalcon_Mvc_Controller>` 。如果继承此基类，你的控制器类则能轻松访问应用的各种服务。	
	
Additional URI parameters are defined as action parameters, so that they can be easily accessed using local variables. A controller can
optionally extend :doc:`Phalcon\\Mvc\\Controller <../api/Phalcon_Mvc_Controller>`. By doing this, the controller can have easy access to
the application services.

没有默认缺省值的参数视为必须参数处理。可以像PHP那样为参数设定一个默认值：

Parameters without a default value are handled as required. Setting optional values for parameters is done as usual in PHP:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function indexAction()
        {

        }

        public function showAction($year = 2012, $postTitle = 'some default title')
        {

        }

    }

参数将会按路由传递和函数定义一样的顺序来赋值。你可以使用以下根据参数名称的方式来获取任意一个参数：	
	
Parameters are assigned in the same order as they were passed in the route. You can get an arbitrary parameter from its name in the following way:

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
            $year       = $this->dispatcher->getParam('year');
            $postTitle  = $this->dispatcher->getParam('postTitle');
        }

    }


调度循环Dispatch Loop
------------------------
调度循环将会在分发器执行，直到没有action需要执行为止。在上面的例子中，只有一个action 被执行到。现在让我们来看下“forward”（转发）怎样才能在循环调度里提供一个更加复杂的操作流，从而将执行转发到 另一个controller/action。

The dispatch loop will be executed within the Dispatcher until there are no actions left to be executed. In the above example only one
action was executed. Now we'll see how "forward" can provide a more complex flow of operation in the dispatch loop, by forwarding
execution to a different controller/action.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function indexAction()
        {

        }

        public function showAction($year, $postTitle)
        {
            $this->flash->error("You don't have permission to access this area");

            // Forward flow to another action
            $this->dispatcher->forward(array(
                "controller" => "users",
                "action"     => "signin"
            ));
        }

    }

如果用户没有访问某个action的权限，那么请求将会被转发到Users控制器的signin行为。	
	
If users don't have permissions to access a certain action then will be forwarded to the Users controller, signin action.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class UsersController extends Controller
    {

        public function indexAction()
        {

        }

        public function signinAction()
        {

        }

    }

对于“forwards”转发的次数没有限制，只要不会形成循环重定向即可，否则就意味着你的应用将会停止（译者注：如果浏览器发现一个请求循环重定向时，会终止请求）。 如果在循环调度里面没有其他action可以分发，分发器将会自动调用由 :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` 管理的MVC的视图层。	
	
There is no limit on the "forwards" you can have in your application, so long as they do not result in circular references, at which point
your application will halt. If there are no other actions to be dispatched by the dispatch loop, the dispatcher will automatically invoke
the view layer of the MVC that is managed by :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`.

初始化控制器Initializing Controllers
---------------------------------------
:doc:`Phalcon\\Mvc\\Controller <../api/Phalcon_Mvc_Controller>`提供了初始化的函数，它会最先执行，并优于任何控制器的其他action。不推荐使用“__construct”方法。

:doc:`Phalcon\\Mvc\\Controller <../api/Phalcon_Mvc_Controller>` offers the initialize method, which is executed first, before any
action is executed on a controller. The use of the "__construct" method is not recommended.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public $settings;

        public function initialize()
        {
            $this->settings = array(
                "mySetting" => "value"
            );
        }

        public function saveAction()
        {
            if ($this->settings["mySetting"] == "value") {
                //...
            }
        }

    }

.. highlights::

    “initialize”仅仅会在事件“beforeExecuteRoute”成功执行后才会被调用。这样可以避免在初始化中的应用逻辑不会在未验证的情况下执行不了。
	
    Method 'initialize' is only called if the event 'beforeExecuteRoute' is executed with success. This avoid
    that application logic in the initializer cannot be executed without authorization.

如果你想在紧接着创建控制器对象的后面执行一些初始化的逻辑，你要实现“onConstruct”方法：	
	
If you want to execute some initialization logic just after build the controller object you can implement the
method 'onConstruct':

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function onConstruct()
        {
            //...
        }
    }

.. highlights::

    需要注意的是，即使待执行的action在控制器不存在，或者用户没有 访问到它（根据开发人员提供的自定义控制器接入），“onConstruct”都会被执行。

    Be aware that method 'onConstruct' is executed even if the action to be executed not exists
    in the controller or the user does not have access to it (according to custom control access
    provided by developer).

注入服务Injecting Services
-----------------------------
如果控制器继承于 :doc:`Phalcon\\Mvc\\Controller <../api/Phalcon_Mvc_Controller>` ，那么它可以轻松访问 应用的服务容器。例如，如果我们类似这样注册了一个服务：

If a controller extends :doc:`Phalcon\\Mvc\\Controller <../api/Phalcon_Mvc_Controller>` then it has easy access to the service
container in application. For example, if we have registered a service like this:

.. code-block:: php

    <?php

    use Phalcon\DI;

    $di = new DI();

    $di->set('storage', function() {
        return new Storage('/some/directory');
    }, true);

那么，我们可以通常多种方式来访问这个服务：	
	
Then, we can access to that service in several ways:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class FilesController extends Controller
    {

        public function saveAction()
        {

            //Injecting the service by just accessing the property with the same name
            $this->storage->save('/some/file');

            //Accessing the service from the DI
            $this->di->get('storage')->save('/some/file');

            //Another way to access the service using the magic getter
            $this->di->getStorage()->save('/some/file');

            //Another way to access the service using the magic getter
            $this->getDi()->getStorage()->save('/some/file');

            //Using the array-syntax
            $this->di['storage']->save('/some/file');
        }

    }

如果你是把Phalcon作为全能(Full-Stack)框架来使用，你可以阅读框架中  :doc:`by default <di>` 提供的服务。	
	
If you're using Phalcon as a full-stack framework, you can read the services provided :doc:`by default <di>` in the framework.

请求与响应Request and Response
---------------------------------
假设框架预先提供了一系列的注册的服务。我们这里将解释如何和HTTP环境进行关联和交互。 “request”服务包含了一个 :doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>` 的实例， “response”服务则包含了一个 :doc:`Phalcon\\Http\\Response <../api/Phalcon_Http_Response>` 的实例，用来表示将要返回给客户端的内容。

Assuming that the framework provides a set of pre-registered services. We explain how to interact with the HTTP environment.
The "request" service contains an instance of :doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>` and the "response"
contains a :doc:`Phalcon\\Http\\Response <../api/Phalcon_Http_Response>` representing what is going to be sent back to the client.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function indexAction()
        {

        }

        public function saveAction()
        {
            // Check if request has made with POST
            if ($this->request->isPost() == true) {
                // Access POST data
                $customerName = $this->request->getPost("name");
                $customerBorn = $this->request->getPost("born");
            }
        }

    }

响应对象通常不会直接使用，但在action的执行前会被创建，有时候 - 如在 一个afterDispatch事件中 - 它对于直接访问响应非常有帮助：	
	
The response object is not usually used directly, but is built up before the execution of the action, sometimes - like in
an afterDispatch event - it can be useful to access the response directly:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function indexAction()
        {

        }

        public function notFoundAction()
        {
            // Send a HTTP 404 response header
            $this->response->setStatusCode(404, "Not Found");
        }

    }

如需学习了解HTTP环境更多内容，请查看专题： :doc:`request <request>` 和 :doc:`response <response>`。	
	
Learn more about the HTTP environment in their dedicated articles :doc:`request <request>` and :doc:`response <response>`.

会话数据Session Data
-----------------------
会话可以帮助我们在多个请求中保持久化的数据。你可以从任何控制器中访问 :doc:`Phalcon\\Session\\Bag <../api/Phalcon_Session_Bag>` 以便封装需要进行持久化的数据。

Sessions help us maintain persistent data between requests. You could access a :doc:`Phalcon\\Session\\Bag <../api/Phalcon_Session_Bag>`
from any controller to encapsulate data that need to be persistent.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class UserController extends Controller
    {

        public function indexAction()
        {
            $this->persistent->name = "Michael";
        }

        public function welcomeAction()
        {
            echo "Welcome, ", $this->persistent->name;
        }

    }

在控制器中使用服务Using Services as Controllers
---------------------------------------------------
服务可以是控制器，控制器类通常会从服务容器中请求。据于此， 任何一个用其名字注册的类都可以轻易地替换一个控制器：

Services may act as controllers, controllers classes are always requested from the services container. Accordingly,
any other class registered with its name can easily replace a controller:

.. code-block:: php

    <?php

    //Register a controller as a service
    $di->set('IndexController', function() {
        $component = new Component();
        return $component;
    });

	    //Register a namespaced controller as a service
	    $di->set('Backend\Controllers\IndexController', function() {
	        $component = new Component();
	        return $component;
	    });

创建基控制器Creating a Base Controller
-------------------------------------------
对于某些应用特性如访问控制列表（ACL），翻译，缓存，和模板引擎一般对于 控制器都是通用的。在这种情况下，我们鼓励创建一个 “基控制器”，从而确保你的代码遵循 DRY_。 基控制器可以是一个简单的类，然后继承于 :doc:`Phalcon\\Mvc\\Controller <../api/Phalcon_Mvc_Controller>`，并封装 全部控制器都有的通用功能操作。反过来，你的控制器则继承这个“基控制器”以便可以直接使用通用功能操作。

Some application features like access control lists, translation, cache, and template engines are often common to many
controllers. In cases like these the creation of a "base controller" is encouraged to ensure your code stays DRY_. A base
controller is simply a class that extends the :doc:`Phalcon\\Mvc\\Controller <../api/Phalcon_Mvc_Controller>` and encapsulates
the common functionality that all controllers must have. In turn, your controllers extend the "base controller" and have
access to the common functionality.

这个基类可以放置在任何一个地方，但出于代码组织的便利我们推荐应该放置在控制器的目录下， 如：apps/controllers/ControllerBase.php。我们可以在启动文件直接require这个文件，也可以使用自动加载：

This class could be located anywhere, but for organizational conventions we recommend it to be in the controllers folder,
e.g. apps/controllers/ControllerBase.php. We may require this file directly in the bootstrap file or cause to be
loaded using any autoloader:

.. code-block:: php

    <?php

    require "../app/controllers/ControllerBase.php";

对通用组件（action，方法，和类属性等）也在这个基类文件里面：	
	
The implementation of common components (actions, methods, properties etc.) resides in this file:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class ControllerBase extends Controller
    {

      /**
       * This action is available for multiple controllers
       */
      public function someAction()
      {

      }

    }

现在，其他全部的控制都继承于ControllerBase，然后便可访问通用组件（如上面讲到的的）：	
	
Any other controller now inherits from ControllerBase, automatically gaining access to the common components (discussed above):

.. code-block:: php

    <?php

    class UsersController extends ControllerBase
    {

    }

控制器中的事件Events in Controllers
------------------------------------
控制器会自动作为 :doc:`dispatcher <dispatching>` 事件的侦听者，使用这些事件并实现实现这些方法后， 你便可以实现对应被执行的action的before/after钩子函数：

Controllers automatically act as listeners for :doc:`dispatcher <dispatching>` events, implementing methods with those event names allow
you to implement hook points before/after the actions are executed:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function beforeExecuteRoute($dispatcher)
        {
            // This is executed before every found action
            if ($dispatcher->getActionName() == 'save') {

                $this->flash->error("You don't have permission to save posts");

                $this->dispatcher->forward(array(
                    'controller' => 'home',
                    'action'     => 'index'
                ));

                return false;
            }
        }

        public function afterExecuteRoute($dispatcher)
        {
            // Executed after every found action
        }

    }

.. _DRY: http://en.wikipedia.org/wiki/Don't_repeat_yourself
