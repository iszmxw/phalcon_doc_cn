调度控制器Dispatching Controllers
======================================
:doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>`是MVC应用中负责实例化 控制器和执行在这些控制器上必要动作的组件。理解它的操作和能力将能帮助我们获得更多Phalcon框架提供的服务。

:doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>` is the component responsible for instantiating controllers and executing the required actions
on them in an MVC application. Understanding its operation and capabilities helps us get more out of the services provided by the framework.

循环调度The Dispatch Loop
--------------------------------
在MVC流中，这是一个重要的处理环节，特别对于控制器这部分。这些处理 发生在控制调度器中。控制器的文件将会被依次读取、加载和实例化。然后指定的action将会被执行。 如果一个动作将这个流转发给了另一个控制器/动作，控制调度器将会再次启动。为了更好 解释这一点，以下示例怡到好处地说明了在 :doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>` 中的处理过程：

This is an important process that has much to do with the MVC flow itself, especially with the controller part. The work occurs within the controller
dispatcher. The controller files are read, loaded, and instantiated. Then the required actions are executed. If an action forwards the flow to another
controller/action, the controller dispatcher starts again. To better illustrate this, the following example shows approximately the process performed
within :doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>`:

.. code-block:: php

    <?php

    //Dispatch loop
    while (!$finished) {

        $finished = true;

        $controllerClass = $controllerName . "Controller";

        //Instantiating the controller class via autoloaders
        $controller = new $controllerClass();

        // Execute the action
        call_user_func_array(array($controller, $actionName . "Action"), $params);

        // '$finished' should be reloaded to check if the flow
        // was forwarded to another controller
        $finished = true;
    }

上面的代码缺少了验证，过滤器和额外的检查，但它演示了在调度器中正常的操作流。	
	
The code above lacks validations, filters and additional checks, but it demonstrates the normal flow of operation in the dispatcher.

循环调度事件Dispatch Loop Events
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
:doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>` 可以发送事件给当前的 EventsManager 。 事件会以“dispatch”类型被所触发。当返回false时有些事件可以终止当前激活的操作。已支持的事件如下：

:doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>` is able to send events to an :doc:`EventsManager <events>` if it is present. Events are triggered using the type "dispatch". Some events when returning boolean false could stop the active operation. The following events are supported:

+----------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+-----------------------+
| Event Name           | Triggered                                                                                                                                                                                                      | Can stop operation? | Triggered on          |
+======================+================================================================================================================================================================================================================+=====================+=======================+
| beforeDispatchLoop   | Triggered before entering in the dispatch loop. At this point the dispatcher don't know if the controller or the actions to be executed exist. The Dispatcher only knows the information passed by the Router. | Yes                 | Listeners             |
+----------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+-----------------------+
| beforeDispatch       | Triggered after entering in the dispatch loop. At this point the dispatcher don't know if the controller or the actions to be executed exist. The Dispatcher only knows the information passed by the Router.  | Yes                 | Listeners             |
+----------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+-----------------------+
| beforeExecuteRoute   | Triggered before executing the controller/action method. At this point the dispatcher has been initialized the controller and know if the action exist.                                                        | Yes                 | Listeners/Controllers |
+----------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+-----------------------+
| initialize           | Allow to globally initialize the controller in the request                                                                                                                                                     | No                  | Controllers           |
+----------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+-----------------------+
| afterExecuteRoute    | Triggered after executing the controller/action method. As operation cannot be stopped, only use this event to make clean up after execute the action                                                          | No                  | Listeners/Controllers |
+----------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+-----------------------+
| beforeNotFoundAction | Triggered when the action was not found in the controller                                                                                                                                                      | Yes                 | Listeners             |
+----------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+-----------------------+
| beforeException      | Triggered before the dispatcher throws any exception                                                                                                                                                           | Yes                 | Listeners             |
+----------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+-----------------------+
| afterDispatch        | Triggered after executing the controller/action method. As operation cannot be stopped, only use this event to make clean up after execute the action                                                          | Yes                 | Listeners             |
+----------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+-----------------------+
| afterDispatchLoop    | Triggered after exiting the dispatch loop                                                                                                                                                                      | No                  | Listeners             |
+----------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+-----------------------+

:doc:`INVO <tutorial-invo>`这篇导读说明了如何从通过结合 Acl 实现的一个安全过滤器中获得事件调度的好处

The :doc:`INVO <tutorial-invo>` tutorial shows how to take advantage of dispatching events implementing a security filter with :doc:`Acl <acl>`

以下例子演示了如何将侦听者绑定到组件上：

The following example demonstrates how to attach listeners to this component:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Dispatcher as MvcDispatcher;
    use Phalcon\Events\Manager as EventsManager;

    $di->set('dispatcher', function(){

        //Create an event manager
        $eventsManager = new EventsManager();

        //Attach a listener for type "dispatch"
        $eventsManager->attach("dispatch", function($event, $dispatcher) {
            //...
        });

        $dispatcher = new MvcDispatcher();

        //Bind the eventsManager to the view component
        $dispatcher->setEventsManager($eventsManager);

        return $dispatcher;

    }, true);

一个实例化的控制器会自动作为事件调度的侦听者，所以你可以实现回调函数：	
	
An instantiated controller automatically acts as a listener for dispatch events, so you can implement methods as callbacks:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function beforeExecuteRoute($dispatcher)
        {
            // Executed before every found action
        }

        public function afterExecuteRoute($dispatcher)
        {
            // Executed after every found action
        }

    }

转发到其他动作Forwarding to other actions
-----------------------------------------------
循环调度允许我们转发执行流到另一个控制器/动作。这对于检查用户是否可以 访问页面，将用户重定向到其他屏幕或简单地代码重用都非常有用。

The dispatch loop allows us to forward the execution flow to another controller/action. This is very useful to check if the user can
access to certain options, redirect users to other screens or simply reuse code.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function indexAction()
        {

        }

        public function saveAction($year, $postTitle)
        {

            // .. store some product and forward the user

            // Forward flow to the index action
            $this->dispatcher->forward(array(
                "controller" => "post",
                "action" => "index"
            ));
        }

    }

请注意制造一个“forward”并不等同于制造一个HTTP的重定向。尽管这两者表面上最终效果都一样。 “forward”不会重新加载当前页面，全部的重定向都只发生在一个请求里面，而HTTP重定向则需要两次请求 才能完成这个流程。	
	
Keep in mind that making a "forward" is not the same as making an HTTP redirect. Although they apparently got the same result.
The "forward" doesn't reload the current page, all the redirection occurs in a single request, while the HTTP redirect needs two requests
to complete the process.

更多转发示例：

More forwarding examples:

.. code-block:: php

    <?php

    // Forward flow to another action in the current controller
    $this->dispatcher->forward(array(
        "action" => "search"
    ));

    // Forward flow to another action in the current controller
    // passing parameters
    $this->dispatcher->forward(array(
        "action" => "search",
        "params" => array(1, 2, 3)
    ));

一个转发的动作可以接受以下参数：
	
A forward action accepts the following parameters:

+----------------+--------------------------------------------------------+
| Parameter      | Triggered                                              |
+================+========================================================+
| controller     | A valid controller name to forward to.                 |
+----------------+--------------------------------------------------------+
| action         | A valid action name to forward to.                     |
+----------------+--------------------------------------------------------+
| params         | An array of parameters for the action                  |
+----------------+--------------------------------------------------------+
| namespace      | A valid namespace name where the controller is part of |
+----------------+--------------------------------------------------------+

准备参数Preparing Parameters
--------------------------------
因为 :doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>`提供的钩子函数， 你可以简单地调整你的应用来匹配URL格式：
例如，你想把你的URL看起来像这样：http://example.com/controller/key1/value1/key2/value
默认下，参数会按URL传递的顺序传给对应的动作，你可以按期望来转换他们：

Thanks to the hooks points provided by :doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>` you can easily
adapt your application to any URL schema:

For example, you want your URLs look like: http://example.com/controller/key1/value1/key2/value

Parameters by default are passed as they come in the URL to actions, you can transform them to the desired schema:

.. code-block:: php

    <?php

    use Phalcon\Dispatcher;
    use Phalcon\Mvc\Dispatcher as MvcDispatcher;
    use Phalcon\Events\Manager as EventsManager;

    $di->set('dispatcher', function() {

        //Create an EventsManager
        $eventsManager = new EventsManager();

        //Attach a listener
        $eventsManager->attach("dispatch:beforeDispatchLoop", function($event, $dispatcher) {

            $keyParams = array();
            $params    = $dispatcher->getParams();

            //Use odd parameters as keys and even as values
            foreach ($params as $number => $value) {
                if ($number & 1) {
                    $keyParams[$params[$number - 1]] = $value;
                }
            }

            //Override parameters
            $dispatcher->setParams($keyParams);
        });

        $dispatcher = new MvcDispatcher();
        $dispatcher->setEventsManager($eventsManager);

        return $dispatcher;
    });

如果期望的链接是这样： http://example.com/controller/key1:value1/key2:value，那么就需要以下这样的代码：	
	
If the desired schema is: http://example.com/controller/key1:value1/key2:value, the following code is required:

.. code-block:: php

    <?php

    use Phalcon\Dispatcher;
    use Phalcon\Mvc\Dispatcher as MvcDispatcher;
    use Phalcon\Events\Manager as EventsManager;

    $di->set('dispatcher', function() {

        //Create an EventsManager
        $eventsManager = new EventsManager();

        //Attach a listener
        $eventsManager->attach("dispatch:beforeDispatchLoop", function($event, $dispatcher) {

            $keyParams = array();
            $params    = $dispatcher->getParams();

            //Explode each parameter as key,value pairs
            foreach ($params as $number => $value) {
                $parts                = explode(':', $value);
                $keyParams[$parts[0]] = $parts[1];
            }

            //Override parameters
            $dispatcher->setParams($keyParams);
        });

        $dispatcher = new MvcDispatcher();
        $dispatcher->setEventsManager($eventsManager);

        return $dispatcher;
    });

获取参数Getting Parameters
------------------------------
当路由提供了命名的参数变量，你就可以在控制器、视图或者任何一个继承了 :doc:`Phalcon\\DI\\Injectable <../api/Phalcon_DI_Injectable>`的组件中获得这些参数。

When a route provides named parameters you can receive them in a controller, a view or any other component that extends
:doc:`Phalcon\\DI\\Injectable <../api/Phalcon_DI_Injectable>`.

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

            // Get the post's title passed in the URL as parameter
            // or prepared in an event
            $title = $this->dispatcher->getParam("title");

            // Get the post's year passed in the URL as parameter
            // or prepared in an event also filtering it
            $year = $this->dispatcher->getParam("year", "int");
        }

    }

准备行动Preparing actions
-----------------------------
你也可以为动作定义一个调度前的映射表。

You can also define an arbitrary schema for actions before be dispatched.

转换动作名Camelize action names
^^^^^^^^^^^^^^^^^^^^^
如果原始链接是：http://example.com/admin/products/show-latest-products， 例如你想把’show-latest-products’转换成’ShowLatestProducts’， 需要以下代码：

If the original URL is: http://example.com/admin/products/show-latest-products,
and for example you want to camelize 'show-latest-products' to 'ShowLatestProducts',
the following code is required:

.. code-block:: php

    <?php

    use Phalcon\Text;
    use Phalcon\Mvc\Dispatcher as MvcDispatcher;
    use Phalcon\Events\Manager as EventsManager;

    $di->set('dispatcher', function() {

        //Create an EventsManager
        $eventsManager = new EventsManager();

        //Camelize actions
        $eventsManager->attach("dispatch:beforeDispatchLoop", function($event, $dispatcher) {
            $dispatcher->setActionName(Text::camelize($dispatcher->getActionName()));
        });

        $dispatcher = new MvcDispatcher();
        $dispatcher->setEventsManager($eventsManager);

        return $dispatcher;
    });

删除遗留的扩展名Remove legacy extensions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果原始链接总是包含一个’.php’扩展名：

If the original URL always contains a '.php' extension:

http://example.com/admin/products/show-latest-products.php
http://example.com/admin/products/index.php

你可以在调度对应的控制器/动作组前将它删除：

You can remove it before dispatch the controller/action combination:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Dispatcher as MvcDispatcher;
    use Phalcon\Events\Manager as EventsManager;

    $di->set('dispatcher', function() {

        //Create an EventsManager
        $eventsManager = new EventsManager();

        //Remove extension before dispatch
        $eventsManager->attach("dispatch:beforeDispatchLoop", function($event, $dispatcher) {

            //Remove extension
            $action = preg_replace('/\.php$/', '', $dispatcher->getActionName());

            //Override action
            $dispatcher->setActionName($action);
        });

        $dispatcher = new MvcDispatcher();
        $dispatcher->setEventsManager($eventsManager);

        return $dispatcher;
    });

注入模型实例Inject model instances
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在这个实例中，开发人员想要观察动作接收到的参数以便可以动态注入模型实例。

In this example, the developer wants to inspect the parameters that an action will receive in order to dynamically
inject model instances.

控制器看起来像这样：

The controller looks like:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {
        /**
         * Shows posts
         *
         * @param \Posts $post
         */
        public function showAction(Posts $post)
        {
            $this->view->post = $post;
        }
    }

‘showAction’方法接收到一个 Posts 模型的实例，开发人员可以在调度动作和准备映射参数前进行观察：	
	
Method 'showAction' receives an instance of the model \Posts, the developer could inspect this
before dispatch the action preparing the parameter accordingly:

.. code-block:: php

    <?php

    use Phalcon\Text;
    use Phalcon\Mvc\Dispatcher as MvcDispatcher;
    use Phalcon\Events\Manager as EventsManager;

    $di->set('dispatcher', function() {

        //Create an EventsManager
        $eventsManager = new EventsManager();

        $eventsManager->attach("dispatch:beforeDispatchLoop", function($event, $dispatcher) {

            //Possible controller class name
            $controllerName =   Text::camelize($dispatcher->getControllerName()) . 'Controller';

            //Possible method name
            $actionName = $dispatcher->getActionName() . 'Action';

            try {

                //Get the reflection for the method to be executed
                $reflection = new \ReflectionMethod($controllerName, $actionName);

                //Check parameters
                foreach ($reflection->getParameters() as $parameter) {

                    //Get the expected model name
                    $className = $parameter->getClass()->name;

                    //Check if the parameter expects a model instance
                    if (is_subclass_of($className, 'Phalcon\Mvc\Model')) {

                        $model = $className::findFirstById($dispatcher->getParams()[0]);

                        //Override the parameters by the model instance
                        $dispatcher->setParams(array($model));
                    }
                }

            } catch (\Exception $e) {
                //An exception has occurred, maybe the class or action does not exist?
            }

        });

        $dispatcher = new MvcDispatcher();
        $dispatcher->setEventsManager($eventsManager);

        return $dispatcher;
    });

上面示例出于学术目的已经作了简化。 开发人员可以在执行动作前注入任何类型的依赖或者模型，以进行提高和强化。	
	
The above example has been simplified for academic purposes.
A developer can improve it to inject any kind of dependency or model in actions before be executed.

处理 Not-Found 错误Handling Not-Found Exceptions
-----------------------------------------------------
使用 :doc:`EventsManager <events>` ，可以在调度器找不到对应的控制器/动作组时而抛出异常前，插入一个钩子：

Using the :doc:`EventsManager <events>` it's possible to insert a hook point before the dispatcher throws an exception
when the controller/action combination wasn't found:

.. code-block:: php

    <?php

    use Phalcon\Dispatcher;
    use Phalcon\Mvc\Dispatcher as MvcDispatcher;
    use Phalcon\Events\Manager as EventsManager;
    use Phalcon\Mvc\Dispatcher\Exception as DispatchException;

    $di->set('dispatcher', function() {

        //Create an EventsManager
        $eventsManager = new EventsManager();

        //Attach a listener
        $eventsManager->attach("dispatch:beforeException", function($event, $dispatcher, $exception) {

            //Handle 404 exceptions
            if ($exception instanceof DispatchException) {
                $dispatcher->forward(array(
                    'controller' => 'index',
                    'action'     => 'show404'
                ));
                return false;
            }

            //Alternative way, controller or action doesn't exist
            if ($event->getType() == 'beforeException') {
                switch ($exception->getCode()) {
                    case \Phalcon\Dispatcher::EXCEPTION_HANDLER_NOT_FOUND:
                    case \Phalcon\Dispatcher::EXCEPTION_ACTION_NOT_FOUND:
                        $dispatcher->forward(array(
                            'controller' => 'index',
                            'action'     => 'show404'
                        ));
                        return false;
                }
            }
        });

        $dispatcher = new \Phalcon\Mvc\Dispatcher();

        //Bind the EventsManager to the dispatcher
        $dispatcher->setEventsManager($eventsManager);

        return $dispatcher;

    }, true);

当然，这个方法也可以移至独立的插件类中，使得在循环调度产生异常时可以有超过一个类执行需要的动作：	
	
Of course, this method can be moved onto independent plugin classes, allowing more than one class
take actions when an exception is produced in the dispatch loop:


.. code-block:: php

    <?php

    use Phalcon\Events\Event;
    use Phalcon\Mvc\Dispatcher;
    use Phalcon\Mvc\Dispatcher\Exception as DispatchException;

    class ExceptionsPlugin
    {
        public function beforeException(Event $event, Dispatcher $dispatcher, $exception)
        {

            //Handle 404 exceptions
            if ($exception instanceof DispatchException) {
                $dispatcher->forward(array(
                    'controller' => 'index',
                    'action'     => 'show404'
                ));
                return false;
            }

            //Handle other exceptions
            $dispatcher->forward(array(
                'controller' => 'index',
                'action'     => 'show503'
            ));

            return false;
        }
    }

.. highlights::

    仅仅当异常产生于调度器或者异常产生于被执行的动作时才会通知’beforeException’里面的事件。 侦听者或者控制器事件中产生的异常则会重定向到最近的try/catch。

    Only exceptions produced by the dispatcher and exceptions produced in the executed action
    are notified in the 'beforeException' events. Exceptions produced in listeners or
    controller events are redirected to the latest try/catch.

自定义调度器Implementing your own Dispatcher
----------------------------------------------------
为了创建自定义调度器，必须实现 :doc:`Phalcon\\Mvc\\DispatcherInterface <../api/Phalcon_Mvc_DispatcherInterface>`接口， 从而替换Phalcon框架默认提供的调度器。

The :doc:`Phalcon\\Mvc\\DispatcherInterface <../api/Phalcon_Mvc_DispatcherInterface>` interface must be implemented to create your own dispatcher
replacing the one provided by Phalcon.
