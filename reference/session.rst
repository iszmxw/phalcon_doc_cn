使用 Session 存储数据 Storing data in Session
===============================================
:doc:`Phalcon\\Session <../api/Phalcon_Session>` 提供对session数据面向对象的封装。使用这个组件替换原始session数据的原因有：

The :doc:`Phalcon\\Session <../api/Phalcon_Session>` provides object-oriented wrappers to access session data.

Reasons to use this component instead of raw-sessions:

* 可以从同一个域名的不同应用中分离session数据
* 在应用程序中设置/获取session数据时进行截获
* 根据应用需求改变session的适配器

* You can easily isolate session data across applications on the same domain
* Intercept where session data is set/get in your application
* Change the session adapter according to the application needs

启动会话Starting the Session
----------------------------------
一些应用程序是session密集使用型的,几乎任何执行操作都需要访问session数据。也有session访问不密集型的。由于有服务容器,我们可以确保只要必须要访问session时才能正常访问到数据:

Some applications are session-intensive, almost any action that performs requires access to session data. There are others who access session data casually.
Thanks to the service container, we can ensure that the session is accessed only when it's clearly needed:

.. code-block:: php

    <?php

    use Phalcon\Session\Adapter\Files as Session;

    //Start the session the first time when some component request the session service
    $di->setShared('session', function() {
        $session = new Session();
        $session->start();
        return $session;
    });

Session 的存储与读取 Storing/Retrieving data in Session
----------------------------------------------------------
从一个控制器,一个视图或任何其他组件，只要是由 :doc:`Phalcon\\DI\\Injectable <../api/Phalcon_DI_Injectable>`扩展来的，使用以下方式你可以访问会话服务和存储和检索数据:

From a controller, a view or any other component that extends :doc:`Phalcon\\DI\\Injectable <../api/Phalcon_DI_Injectable>` you can access the session service
and store items and retrieve them in the following way:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class UserController extends Controller
    {

        public function indexAction()
        {
            //Set a session variable
            $this->session->set("user-name", "Michael");
        }

        public function welcomeAction()
        {

            //Check if the variable is defined
            if ($this->session->has("user-name")) {

                //Retrieve its value
                $name = $this->session->get("user-name");
            }
        }

    }

Sessions 的删除和销毁Removing/Destroying Sessions
-------------------------------------------------------
可以删除指定的数据或者是销毁整个session数据：

It's also possible remove specific variables or destroy the whole session:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class UserController extends Controller
    {

        public function removeAction()
        {
            //Remove a session variable
            $this->session->remove("user-name");
        }

        public function logoutAction()
        {
            //Destroy the whole session
            $this->session->destroy();
        }

    }

隔离不同应用的会话数据Isolating Session Data between Applications
--------------------------------------------------------------------------
有时用户可以使用相同的应用程序两次,在同一台服务器上,在同一会话。当然,如果我们在会话中使用的变量,我们希望在每一个应用程序有单独的会话数据(即使相同的代码和变量名相同)。为了解决这个问题,可以在应用程序中为每个会话变量添加一个前缀:

Sometimes a user can use the same application twice, on the same server, in the same session. Surely, if we use variables in session,
we want that every application have separate session data (even though the same code and same variable names). To solve this, you can add a
prefix for every session variable created in a certain application:

.. code-block:: php

    <?php

    use Phalcon\Session\Adapter\Files as Session;

    //Isolating the session data
    $di->set('session', function(){

        //All variables created will prefixed with "my-app-1"
        $session = new Session(
            array(
                'uniqueId' => 'my-app-1'
            )
        );

        $session->start();

        return $session;
    });

会话袋Session Bags
------------------------
:doc:`Phalcon\\Session\\Bag <../api/Phalcon_Session_Bag>` 是一个可以分离会话数据到namespaces的组件。使用这个组件可以轻易的创建会话数据分组。只需要将变量设置为bag中的值就可以自动保存在会话中：

:doc:`Phalcon\\Session\\Bag <../api/Phalcon_Session_Bag>` is a component that helps separating session data into "namespaces".
Working by this way you can easily create groups of session variables into the application. By only setting the variables in the "bag",
it's automatically stored in session:

.. code-block:: php

    <?php

    use Phalcon\Session\Bag as SessionBag;

    $user       = new SessionBag('user');
    $user->setDI($di);
    $user->name = "Kimbra Johnson";
    $user->age  = 22;


组件的持久数据Persistent Data in Components
------------------------------------------------
继承自 :doc:`Phalcon\\DI\\Injectable <../api/Phalcon_DI_Injectable>` 的控制器、组件或者是类可以注入到 :doc:`Phalcon\\Session\\Bag <../api/Phalcon_Session_Bag>` 中。这个类可以分离每个类中的变量。使用这个方法可以在请求之间持久化保存数据。

Controller, components and classes that extends :doc:`Phalcon\\DI\\Injectable <../api/Phalcon_DI_Injectable>` may inject
a :doc:`Phalcon\\Session\\Bag <../api/Phalcon_Session_Bag>`. This class isolates variables for every class.
Thanks to this you can persist data between requests in every class in an independent way.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class UserController extends Controller
    {

        public function indexAction()
        {
            // Create a persistent variable "name"
            $this->persistent->name = "Laura";
        }

        public function welcomeAction()
        {
            if (isset($this->persistent->name))
            {
                echo "Welcome, ", $this->persistent->name;
            }
        }

    }

在一个组件中：	
	
In a component:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class Security extends Component
    {

        public function auth()
        {
            // Create a persistent variable "name"
            $this->persistent->name = "Laura";
        }

        public function getAuthName()
        {
            return $this->persistent->name;
        }

    }

添加到session中($this->session)的数据整个应用中都可以访问，但是持久化的数据($this->persistent)只有在当前类的作用域中才可以访问。	
	
The data added to the session ($this->session) are available throughout the application, while persistent ($this->persistent)
can only be accessed in the scope of the current class.

自定义适配器Implementing your own adapters
---------------------------------------------
:doc:`Phalcon\\Session\\AdapterInterface <../api/Phalcon_Session_AdapterInterface>`接口必须被继承实现如果想要定义自己的会话适配器。 `Phalcon Incubator <https://github.com/phalcon/incubator/tree/master/Library/Phalcon/Session/Adapter>`_在这个中介绍了更多的适配器。

The :doc:`Phalcon\\Session\\AdapterInterface <../api/Phalcon_Session_AdapterInterface>` interface must be implemented in order to create your own session adapters or extend the existing ones.

There are more adapters available for this components in the `Phalcon Incubator <https://github.com/phalcon/incubator/tree/master/Library/Phalcon/Session/Adapter>`_
