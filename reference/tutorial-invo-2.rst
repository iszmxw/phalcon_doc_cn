教程 3: 加强INVO的安全
==========================
在这一节,我们继续讲解INVO的结构,我们会谈到关于身份验证实现、通过事件机制完成授权、插件和一个由Phalcon管理的访问控制列表(ACL)。

In this chapter, we continue explaining how INVO is structured, we'll talk
about the implementation of authentication, authorization using events and plugins and
an access control list (ACL) managed by Phalcon.

登录到应用程序
----------------
“登录”功能将使我们可以使用后端控制器。分离前端和后端控制器是合乎逻辑的。所有的控制器都位于同一目录(app/controllers/)。

A "log in" facility will allow us to work on backend controllers. The separation between backend controllers and
frontend ones is only logical. All controllers are located in the same directory (app/controllers/).

要想登录进入系统，用户必须有一个有效的用户名和密码。用户信息存储在INVO的数据库的“users”数据表中。

To enter the system, users must have a valid username and password. Users are stored in the table "users"
in the database "invo".

在我们开始一个会话之前,我们需要配置应用连接到数据库。在服务容器中创建一个名为“db”并且包含有连接信息的服务。我们使用自动加载器从配置文件中加载参数并配置好一个服务:

Before we can start a session, we need to configure the connection to the database in the application. A service
called "db" is set up in the service container with the connection information. As with the autoloader, we are
again taking parameters from the configuration file in order to configure a service:

.. code-block:: php

    <?php

    use Phalcon\Db\Adapter\Pdo\Mysql as DbAdapter;

    // ...

    // Database connection is created based on parameters defined in the configuration file
    $di->set('db', function() use ($config) {
        return new DbAdapter(array(
            "host"     => $config->database->host,
            "username" => $config->database->username,
            "password" => $config->database->password,
            "dbname"   => $config->database->name
        ));
    });

在这里我们返回MySQL连接适配器的一个实例。如果需要我们可以额外添加一个日志记录服务、分析器或更改适配器设置，按照自己的需求修改。	
	
Here, we return an instance of the MySQL connection adapter. If needed, you could do extra actions such as adding a
logger, a profiler or change the adapter, setting it up as you want.

以下简单的表单(app/views/session/index.volt)用于让用户填写登录信息。已经删除一些HTML代码让例子看着更加简洁:

The following simple form (app/views/session/index.volt) requests the login information. We've removed
some HTML code to make the example more concise:

.. code-block:: html+jinja

    {{ form('session/start') }}
        <fieldset>
            <div>
                <label for="email">Username/Email</label>
                <div>
                    {{ text_field('email') }}
                </div>
            </div>
            <div>
                <label for="password">Password</label>
                <div>
                    {{ password_field('password') }}
                </div>
            </div>
            <div>
                {{ submit_button('Login') }}
            </div>
        </fieldset>
    </form>

与前面的教程中直接使用php相比我们现在开始使用 :doc:`Volt <volt>`。这是一个内置的模板引擎，Volt在 Jinja_ 启发下完成的，它提供一个更简单和友好的语法来创建模板。我们熟悉volt模板引擎不会花费太多时间。	
	
Instead of using raw PHP as the previous tutorial, we started to use :doc:`Volt <volt>`. This is a built-in
template engine inspired in Jinja_ providing a simpler and friendly syntax to create templates.
It will not take too long before you become familiar with Volt.

(app/controllers/SessionController.php)中的SessionController::startAction函数验证用户提交的数据并且检查用户在数据库中是否有效:

The SessionController::startAction function (app/controllers/SessionController.php) has the task of validating the
data entered in the form including checking for a valid user in the database:

.. code-block:: php

    <?php

    class SessionController extends ControllerBase
    {

        // ...

        private function _registerSession($user)
        {
            $this->session->set('auth', array(
                'id'    => $user->id,
                'name'  => $user->name
            ));
        }

        /**
         * This action authenticate and logs an user into the application
         *
         */
        public function startAction()
        {
            if ($this->request->isPost()) {

                $email      = $this->request->getPost('email');
                $password   = $this->request->getPost('password');

                $user = Users::findFirst(array(
                    "(email = :email: OR username = :email:) AND password = :password: AND active = 'Y'",
                    'bind' => array('email' => $email, 'password' => sha1($password))
                ));
                if ($user != false) {
                    $this->_registerSession($user);
                    $this->flash->success('Welcome ' . $user->name);
                    return $this->forward('invoices/index');
                }

                $this->flash->error('Wrong email/password');
            }

            return $this->forward('session/index');
        }
    }

为了简单起见,我们使用“sha1_”算法加密用户密码然后保存在数据库中。然而，sha1算法不推荐在正式的应用中使用，推荐使用":doc:`bcrypt <security>`"。	
	
For the sake of simplicity, we have used "sha1_" to store the password hashes in the database, however, this algorithm is
not recommended in real applications, use ":doc:`bcrypt <security>`" instead.

注意,这里有多个控制器的公共属性可以访问:$this->flash, $this->request or $this->session。这些服务在之前的服务容器中被定义(app/config/services.php)。当这些服务被第一次访问后,他们就注入到了当前控制器中并作为其一部分。

Note that multiple public attributes are accessed in the controller like: $this->flash, $this->request or $this->session.
These are services defined in the services container from earlier (app/config/services.php).
When they're accessed the first time, they are injected as part of the controller.

这些服务是“共享”的,这意味着我们总是能够访问同样的服务实例而不用去管我们在哪些地方调用过。

These services are "shared", which means that we are always accessing the same instance regardless of the place
where we invoke them.

例如,在下面代码中我们调用“session”服务,然后我们将用户标识存储在变量“auth”中:

For instance, here we invoke the "session" service and then we store the user identity in the variable "auth":

.. code-block:: php

    <?php

    $this->session->set('auth', array(
        'id'    => $user->id,
        'name'  => $user->name
    ));

本节另一个重要方面是如何验证用户的身份真实有效，首先我们验证请求是否是POST方法传过来的:	
	
Another important aspect of this section is how the user is validated as a valid one,
first we validate whether the request has been made using method POST:

.. code-block:: php

    <?php

    if ($this->request->isPost()) {

然后从表单接受参数：	
	
Then, we receive the parameters from the form:

.. code-block:: php

    <?php

    $email = $this->request->getPost('email');
    $password = $this->request->getPost('password');

然后检查是否有这个用户名或者邮件的用户并且密码相同：	
	
Now, we have to check if there is one user with the same username or email and password:

.. code-block:: php

    <?php

    $user = Users::findFirst(array(
        "(email = :email: OR username = :email:) AND password = :password: AND active = 'Y'",
        'bind' => array('email' => $email, 'password' => sha1($password))
    ));

注意，这里使用了'绑定参数'，占位符:email: 和 :password: 通过参数“bind”完成绑定，并会被实际的值替换。安全的替换这些值可以防止SQL注入。
	
Note, the use of 'bound parameters', placeholders :email: and :password: are placed where values should be,
then the values are 'bound' using the parameter 'bind'. This safely replaces the values for those
columns without having the risk of a SQL injection.

如果用户有效就会把他注册到会话中，然后引导用户到控制面板：

If the user is valid we register it in session and forwards him/her to the dashboard:

.. code-block:: php

    <?php

    if ($user != false) {
        $this->_registerSession($user);
        $this->flash->success('Welcome ' . $user->name);
        return $this->forward('invoices/index');
    }

如果用户不存在会返回到用户登录页面:	
	
If the user does not exist we forward the user back again to action where the form is displayed:

.. code-block:: php

    <?php

    return $this->forward('session/index');


加强后端
------------
后端是一个私有领域只有注册用户才可以访问。因此,必须检查只有注册用户访问这些控制器。如果不登录到应用程序中,尝试访问产品控制器(私有)我们将看到如下结果:

The backend is a private area where only registered users have access. Therefore, it is necessary
to check that only registered users have access to these controllers. If you aren't logged
into the application and you try to access, for example, the products controller (which is private)
you will see a screen like this:

.. figure:: ../_static/img/invo-2.png
   :align: center

每次有人试图访问任何控制器/动作,应用验证当前会话用户的角色判断是否能够访问它,如果没有权限就会显示一个如上的消息并重定向到主页。   
   
Every time someone attempts to access any controller/action, the application verifies that the
current role (in session) has access to it, otherwise it displays a message like the above and
forwards the flow to the home page.

现在让我们看看应用程序如何实现这一点。首先要知道的是有一个叫 :doc:`分配器dispatching <dispatching>` 的组件。:doc:`路由 <routing>` 通知它去负责加载适当的控制器和执行相应的动作方法。

Now let's find out how the application accomplishes this. The first thing to know is that
there is a component called :doc:`Dispatcher <dispatching>`. It is informed about the route
found by the :doc:`Routing <routing>` component. Then, it is responsible for loading the
appropriate controller and execute the corresponding action method.

正常情况下,框架会自动创建分配器。在我们的例子中,我们要在执行所需的动作之前检查用户是否可以访问它，为了达到这个目标,我们需要在启动文件中用一个匿名函数替代这个组件:

Normally, the framework creates the Dispatcher automatically. In our case, we want to perform a verification
before executing the required action, checking if the user has access to it or not. To achieve this, we have
replaced the component by creating a function in the bootstrap:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Dispatcher;

    // ...

    /**
     * MVC dispatcher
     */
    $di->set('dispatcher', function() {

        // ...

        $dispatcher = new Dispatcher;

        return $dispatcher;
    });

我们现在可以完全控制应用程序中的分配器。框架中的许多组件能够触发让我们修改其内部操作流的事件。因为有依赖项注入器组件充当粘合剂,一个新的名为 :doc:`事件管理器 <events>` 的组件使我们能够拦截一个组件的事件,并将事件路由给它的侦听器。	
	
We now have total control over the Dispatcher used in the application. Many components in the framework trigger
events that allow us to modify their internal flow of operation. As the Dependency Injector component acts as glue
for components, a new component called :doc:`EventsManager <events>` allows us to intercept the events produced
by a component, routing the events to listeners.

事件管理器
^^^^^^^^^^^^
:doc:`事件管理器 <events>` 允许我们将侦听器附加到一个特定类型的事件。我们现在感兴趣的是“分配”事件。下面的代码过滤了所有由分配器产生的事件:

An :doc:`EventsManager <events>` allows us to attach listeners to a particular type of event. The type that
interests us now is "dispatch". The following code filters all events produced by the Dispatcher:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Dispatcher;
    use Phalcon\Events\Manager as EventsManager;

    $di->set('dispatcher', function() {

        $eventsManager = new EventsManager;

        /**
         * Check if the user is allowed to access certain action using the SecurityPlugin
         */
        $eventsManager->attach('dispatch:beforeDispatch', new SecurityPlugin);

        /**
         * Handle exceptions and not-found exceptions using NotFoundPlugin
         */
        $eventsManager->attach('dispatch:beforeException', new NotFoundPlugin);

        $dispatcher = new Dispatcher;
        $dispatcher->setEventsManager($eventsManager);

        return $dispatcher;
    });

当"beforeDispatch"事件被触发时，下面的插件将会被通知：	
	
When an event called "beforeDispatch" is triggered the following plugin will be notified:

.. code-block:: php

    <?php

    /**
     * Check if the user is allowed to access certain action using the SecurityPlugin
     */
    $eventsManager->attach('dispatch:beforeDispatch', new SecurityPlugin);

当"beforeException"事件被触发时，另一个插件将会被通知：	
	
When a "beforeException" is triggered then other plugin is notified:

.. code-block:: php

    <?php

    /**
     * Handle exceptions and not-found exceptions using NotFoundPlugin
     */
    $eventsManager->attach('dispatch:beforeException', new NotFoundPlugin);

安全插件在(app/plugins/SecurityPlugin.php)这个目录。这个类实现了"beforeDispatch"这个方法。方法的名称和上面分配器里面的事件名称一样。
	
SecurityPlugin is a class located at (app/plugins/SecurityPlugin.php). This class implements the method
"beforeDispatch". This is the same name as one of the events produced in the Dispatcher:

.. code-block:: php

    <?php

    use Phalcon\Acl;
    use Phalcon\Events\Event;
    use Phalcon\Mvc\User\Plugin;
    use Phalcon\Mvc\Dispatcher;

    class SecurityPlugin extends Plugin
    {

        // ...

        public function beforeDispatch(Event $event, Dispatcher $dispatcher)
        {
            // ...
        }

    }

钩子事件总是获得包含了产生事件的上下文信息($event)作为第一个参数，第二个参数是产生该事件的对象本身($dispatcher)。这不是强制性的，插件类是Phalcon\\Mvc\\User\\Plugin的扩展,这样做更容易获得应用程序中可用的服务。	
	
The hook events always receive a first parameter that contains contextual information of the event produced ($event)
and a second one that is the object that produced the event itself ($dispatcher). It is not mandatory that
plugins extend the class Phalcon\\Mvc\\User\\Plugin, but by doing this they gain easier access to the services
available in the application.

现在我们在当前会话中验证用户角色,使用ACL列表检查用户是否有访问权限。如果没有则重定向到主页:

Now, we're verifying the role in the current session, checking if the user has access using the ACL list.
If the user does not have access we redirect to the home screen as explained before:

.. code-block:: php

    <?php

    use Phalcon\Acl;
    use Phalcon\Events\Event;
    use Phalcon\Mvc\User\Plugin;
    use Phalcon\Mvc\Dispatcher;

    class Security extends Plugin
    {

        // ...

        public function beforeExecuteRoute(Event $event, Dispatcher $dispatcher)
        {

            //Check whether the "auth" variable exists in session to define the active role
            $auth = $this->session->get('auth');
            if (!$auth) {
                $role = 'Guests';
            } else {
                $role = 'Users';
            }

            //Take the active controller/action from the dispatcher
            $controller = $dispatcher->getControllerName();
            $action = $dispatcher->getActionName();

            //Obtain the ACL list
            $acl = $this->getAcl();

            //Check if the Role have access to the controller (resource)
            $allowed = $acl->isAllowed($role, $controller, $action);
            if ($allowed != Acl::ALLOW) {

                //If he doesn't have access forward him to the index controller
                $this->flash->error("You don't have access to this module");
                $dispatcher->forward(
                    array(
                        'controller' => 'index',
                        'action'     => 'index'
                    )
                );

                //Returning "false" we tell to the dispatcher to stop the current operation
                return false;
            }

        }

    }

创建ACL列表
^^^^^^^^^^^^^^
在上面的例子中我们通过使用$this->getAcl()方法获得ACL。这种方法也是在插件中实现的。现在我们要逐步解释我们如何建立访问控制列表(ACL):

In the above example we have obtained the ACL using the method $this->_getAcl(). This method is also
implemented in the Plugin. Now we are going to explain step-by-step how we built the access control list (ACL):

.. code-block:: php

    <?php

    use Phalcon\Acl;
    use Phalcon\Acl\Role;
    use Phalcon\Acl\Adapter\Memory as AclList;

    // Create the ACL
    $acl = new AclList();

    // The default action is DENY access
    $acl->setDefaultAction(Acl::DENY);

    // Register two roles, Users is registered users
    // and guests are users without a defined identity
    $roles = array(
        'users'  => new Role('Users'),
        'guests' => new Role('Guests')
    );
    foreach ($roles as $role) {
        $acl->addRole($role);
    }

现在我们为每个区域分别定义资源。控制器名称为资源名，控制器中的方法就是访问该资源的权限名:	
	
Now, we define the resources for each area respectively. Controller names are resources and their actions are
accesses for the resources:

.. code-block:: php

    <?php

    use Phalcon\Acl\Resource;

    // ...

    // Private area resources (backend)
    $privateResources = array(
      'companies'    => array('index', 'search', 'new', 'edit', 'save', 'create', 'delete'),
      'products'     => array('index', 'search', 'new', 'edit', 'save', 'create', 'delete'),
      'producttypes' => array('index', 'search', 'new', 'edit', 'save', 'create', 'delete'),
      'invoices'     => array('index', 'profile')
    );
    foreach ($privateResources as $resource => $actions) {
        $acl->addResource(new Resource($resource), $actions);
    }

    // Public area resources (frontend)
    $publicResources = array(
       'index'      => array('index'),
       'about'      => array('index'),
       'register'   => array('index'),
       'errors'     => array('show404', 'show500'),
       'session'    => array('index', 'register', 'start', 'end'),
       'contact'    => array('index', 'send')
    );
    foreach ($publicResources as $resource => $actions) {
        $acl->addResource(new Resource($resource), $actions);
    }

ACL现在能够识别所有的控制器及其相关的动作。“Users”角色的用户应该能够访问所有的前端和后端资源。“Guests”的角色只有访问公共区域:	
	
The ACL now have knowledge of the existing controllers and their related actions. Role "Users" has access to
all the resources of both frontend and backend. The role "Guests" only has access to the public area:

.. code-block:: php

    <?php

    // Grant access to public areas to both users and guests
    foreach ($roles as $role) {
        foreach ($publicResources as $resource => $actions) {
            $acl->allow($role->getName(), $resource, '*');
        }
    }

    // Grant access to private area only to role Users
    foreach ($privateResources as $resource => $actions) {
        foreach ($actions as $action) {
            $acl->allow('Users', $resource, $action);
        }
    }

Hooray!，ACL控制列表完成了，在下一节我们会看到在phalcon中CRUD是如何实现的以及我们如何自定义它们。	
	
Hooray!, the ACL is now complete. In next chapter, we will see how a CRUD is implemented in Phalcon and how you
can customize it.

.. _jinja: http://jinja.pocoo.org/
.. _sha1: http://php.net/manual/en/function.sha1.php
.. _bcrypt: http://stackoverflow.com/questions/4795385/how-do-you-use-bcrypt-for-hashing-passwords-in-php
