闪存消息Flashing Messages
=============================
闪存消息用于通知用户关于他/她产生的动作状态，或者简单地为用户显示一此信息。 这类消息可以使用这个组件来生成。

Flash messages are used to notify the user about the state of actions he/she made or simply show information to the users.
These kind of messages can be generated using this component.

适配器Adapters
--------------------
这个组件使用了适配器来定义消息传递给Flasher后的行为：

This component makes use of adapters to define the behavior of the messages after being passed to the Flasher:

+---------+-----------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| Adapter | Description                                                                                   | API                                                                        |
+=========+===============================================================================================+============================================================================+
| Direct  | Directly outputs the messages passed to the flasher                                           | :doc:`Phalcon\\Flash\\Direct <../api/Phalcon_Flash_Direct>`                |
+---------+-----------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| Session | Temporarily stores the messages in session, then messages can be printed in the next request  | :doc:`Phalcon\\Flash\\Session <../api/Phalcon_Flash_Session>`              |
+---------+-----------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+

使用Usage
-----------------
通常闪存消息都是来自服务容器的请求， 如果你正在使用 :doc:`Phalcon\\DI\\FactoryDefault <../api/Phalcon_DI_FactoryDefault>`， 那么  :doc:`Phalcon\\Flash\\Direct <../api/Phalcon_Flash_Direct>`将会作为 “flash” 服务自动注册：

Usually the Flash Messaging service is requested from the services container,
if you're using :doc:`Phalcon\\DI\\FactoryDefault <../api/Phalcon_DI_FactoryDefault>`
then :doc:`Phalcon\\Flash\\Direct <../api/Phalcon_Flash_Direct>` is automatically registered as "flash" service:

.. code-block:: php

    <?php

    use Phalcon\Flash\Direct as FlashDirect;

    //Set up the flash service
    $di->set('flash', function() {
        return new FlashDirect();
    });

这样的话，你便可以在控制器或者视图中通过在必要的片段中注入此服务来使用它：		
	
This way, you can use it in controllers or views by injecting the service in the required scope:

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
            $this->flash->success("The post was correctly saved!");
        }

    }

	
目前已支持的有四种内置消息类型：	
	
There are four built-in message types supported:

.. code-block:: php

    <?php

    $this->flash->error("too bad! the form had errors");
    $this->flash->success("yes!, everything went very smoothly");
    $this->flash->notice("this a very important information");
    $this->flash->warning("best check yo self, you're not looking too good.");

你可以用你自己的类型来添加消息：	
	
You can add messages with your own types:

.. code-block:: php

    <?php

    $this->flash->message("debug", "this is debug message, you don't say");

输出信息Printing Messages
-----------------------------
发送给flash服务的消息将会自动格式成html：

Messages sent to the flash service are automatically formatted with html:

.. code-block:: html

    <div class="errorMessage">too bad! the form had errors</div>
    <div class="successMessage">yes!, everything went very smoothly</div>
    <div class="noticeMessage">this a very important information</div>
    <div class="warningMessage">best check yo self, you're not looking too good.</div>

正如你看到的，CSS的类将会自动添加到div中。这些类允许你定义消息在浏览器上的图形表现。 此CSS类可以被重写，例如，如果你正在使用Twitter的bootstrap，对应的类可以这样配置：	
	
As you can see, CSS classes are added automatically to the DIVs. These classes allow you to define the graphical presentation
of the messages in the browser. The CSS classes can be overridden, for example, if you're using Twitter bootstrap, classes can be configured as:

.. code-block:: php

    <?php

    use Phalcon\Flash\Direct as FlashDirect;

    //Register the flash service with custom CSS classes
    $di->set('flash', function(){
        $flash = new FlashDirect(array(
            'error'   => 'alert alert-error',
            'success' => 'alert alert-success',
            'notice'  => 'alert alert-info',
        ));
        return $flash;
    });

然后消息会是这样输出：	
	
Then the messages would be printed as follows:

.. code-block:: html

    <div class="alert alert-error">too bad! the form had errors</div>
    <div class="alert alert-success">yes!, everything went very smoothly</div>
    <div class="alert alert-info">this a very important information</div>

隐式刷送与会话Implicit Flush vs. Session
-----------------------------------------------
依赖于发送消息的适配器，它可以立即产生输出，也可以先临时将消息存放于会话中随后再显示。 你何时应该使用哪个？这通常依赖于你在发送消息后重定向的类型。例如， 如果你用了“转发”则不需要将消息存放于会话中，但如果你用的是一个HTTP重定向，那么则需要存放于会话中：

Depending on the adapter used to send the messages, it could be producing output directly, or be temporarily storing the messages in session to be shown later.
When should you use each? That usually depends on the type of redirection you do after sending the messages. For example,
if you make a "forward" is not necessary to store the messages in session, but if you do a HTTP redirect then, they need to be stored in session:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class ContactController extends Controller
    {

        public function indexAction()
        {

        }

        public function saveAction()
        {

            //store the post

            //Using direct flash
            $this->flash->success("Your information was stored correctly!");

            //Forward to the index action
            return $this->dispatcher->forward(array("action" => "index"));
        }

    }

或者使用一个HTTP重定向：	
	
Or using a HTTP redirection:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class ContactController extends Controller
    {

        public function indexAction()
        {

        }

        public function saveAction()
        {

            //store the post

            //Using session flash
            $this->flashSession->success("Your information was stored correctly!");

            //Make a full HTTP redirection
            return $this->response->redirect("contact/index");
        }
    }

在这种情况下，你需要手动在交互的视图上打印消息：	
	
In this case you need to manually print the messages in the corresponding view:

.. code-block:: html+php

    <!-- app/views/contact/index.phtml -->

    <p><?php $this->flashSession->output() ?></p>

“flashSession”属性是先前在依赖注入容器中设置的闪存。 为了能成功使用flashSession消息者，你需要先启动 session 。	
	
The attribute 'flashSession' is how the flash was previously set into the dependency injection container.
You need to start the :doc:`session <session>` first to successfully use the flashSession messenger.
