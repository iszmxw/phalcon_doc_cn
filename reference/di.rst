依赖注入与服务定位器Dependency Injection/Service Location
************************************************************
接下来的例子有些长，但解释了为什么我们使用依赖注入与服务定位器. 首先，假设我们正在开发一个组件，叫SomeComponent，它执行的内容现在还不重要。 我们的组件需要依赖数据库的连接。

The following example is a bit lengthy, but it attempts to explain why Phalcon uses service location and dependency injection.
First, let's pretend we are developing a component called SomeComponent. This performs a task that is not important now.
Our component has some dependency that is a connection to a database.

在下面第一个例子中，数据库的连接是在组件内部建立的。这种方法是不实用的；事实上这样做的话，我们不能改变创建数据库连接的参数或者选择不同的数据库系统，因为连接是当组件被创建时建立的。

In this first example, the connection is created inside the component. This approach is impractical; due to the fact
we cannot change the connection parameters or the type of database system because the component only works as created.

.. code-block:: php

    <?php

    class SomeComponent
    {

        /**
         * The instantiation of the connection is hardcoded inside
         * the component, therefore it's difficult replace it externally
         * or change its behavior
         */
        public function someDbTask()
        {
            $connection = new Connection(array(
                "host"      => "localhost",
                "username"  => "root",
                "password"  => "secret",
                "dbname"    => "invo"
            ));

            // ...
        }

    }

    $some = new SomeComponent();
    $some->someDbTask();

为了解决这样的情况，我们建立一个setter，在使用前注入独立外部依赖。现在，看起来似乎是一个不错的解决办法：	
	
To solve this, we have created a setter that injects the dependency externally before using it. For now, this seems to be
a good solution:

.. code-block:: php

    <?php

    class SomeComponent
    {

        protected $_connection;

        /**
         * Sets the connection externally
         */
        public function setConnection($connection)
        {
            $this->_connection = $connection;
        }

        public function someDbTask()
        {
            $connection = $this->_connection;

            // ...
        }

    }

    $some = new SomeComponent();

    //Create the connection
    $connection = new Connection(array(
        "host"      => "localhost",
        "username"  => "root",
        "password"  => "secret",
        "dbname"    => "invo"
    ));

    //Inject the connection in the component
    $some->setConnection($connection);

    $some->someDbTask();

想一下，假设我们使用这个组件在应用内的好几个地方都用到，然而我们在注入连接实例时还需要建立好几次数据的连接实例。 如果我们可以获取到数据库的连接实例而不用每次都要创建新的连接实例，使用某种全局注册表可以解决这样的问题：	
	
Now consider that we use this component in different parts of the application and
then we will need to create the connection several times before passing it to the component.
Using some kind of global registry where we obtain the connection instance and not have
to create it again and again could solve this:

.. code-block:: php

    <?php

    class Registry
    {

        /**
         * Returns the connection
         */
        public static function getConnection()
        {
           return new Connection(array(
                "host"      => "localhost",
                "username"  => "root",
                "password"  => "secret",
                "dbname"    => "invo"
            ));
        }

    }

    class SomeComponent
    {

        protected $_connection;

        /**
         * Sets the connection externally
         */
        public function setConnection($connection)
        {
            $this->_connection = $connection;
        }

        public function someDbTask()
        {
            $connection = $this->_connection;

            // ...
        }

    }

    $some = new SomeComponent();

    //Pass the connection defined in the registry
    $some->setConnection(Registry::getConnection());

    $some->someDbTask();

	
现在，让我们设想一下，我们必须实现2个方法，第一个方法是总是创建一个新的连接，第二方法是总是使用一个共享连接：	
	
	
Now, let's imagine that we must implement two methods in the component, the first always needs to create a new connection and the second always needs to use a shared connection:

.. code-block:: php

    <?php

    class Registry
    {

        protected static $_connection;

        /**
         * Creates a connection
         */
        protected static function _createConnection()
        {
            return new Connection(array(
                "host"      => "localhost",
                "username"  => "root",
                "password"  => "secret",
                "dbname"    => "invo"
            ));
        }

        /**
         * Creates a connection only once and returns it
         */
        public static function getSharedConnection()
        {
            if (self::$_connection===null){
                $connection = self::_createConnection();
                self::$_connection = $connection;
            }
            return self::$_connection;
        }

        /**
         * Always returns a new connection
         */
        public static function getNewConnection()
        {
            return self::_createConnection();
        }

    }

    class SomeComponent
    {

        protected $_connection;

        /**
         * Sets the connection externally
         */
        public function setConnection($connection)
        {
            $this->_connection = $connection;
        }

        /**
         * This method always needs the shared connection
         */
        public function someDbTask()
        {
            $connection = $this->_connection;

            // ...
        }

        /**
         * This method always needs a new connection
         */
        public function someOtherDbTask($connection)
        {

        }

    }

    $some = new SomeComponent();

    //This injects the shared connection
    $some->setConnection(Registry::getSharedConnection());

    $some->someDbTask();

    //Here, we always pass a new connection as parameter
    $some->someOtherDbTask(Registry::getNewConnection());

	
到目前为止，我们已经看到依赖注入怎么解决我们的问题了。把依赖作为参数来传递，而不是建立在内部建立它们，这使我们的应用更加容易维护和更加解耦。不管怎么样，长期来说，这种形式的依赖注入有一些缺点。	
	
	
So far we have seen how dependency injection solved our problems. Passing dependencies as arguments instead
of creating them internally in the code makes our application more maintainable and decoupled. However, in the long-term,
this form of dependency injection has some disadvantages.

例如，如果这个组件有很多依赖， 我们需要创建多个参数的setter方法​​来传递依赖关系，或者建立一个多个参数的构造函数来传递它们，另外在使用组件前还要每次都创建依赖，这让我们的代码像这样不易维护：

For instance, if the component has many dependencies, we will need to create multiple setter arguments to pass
the dependencies or create a constructor that passes them many arguments, additionally creating dependencies
before using the component, every time, makes our code not as maintainable as we would like:

.. code-block:: php

    <?php

    //Create the dependencies or retrieve them from the registry
    $connection = new Connection();
    $session    = new Session();
    $fileSystem = new FileSystem();
    $filter     = new Filter();
    $selector   = new Selector();

    //Pass them as constructor parameters
    $some = new SomeComponent($connection, $session, $fileSystem, $filter, $selector);

    // ... or using setters

    $some->setConnection($connection);
    $some->setSession($session);
    $some->setFileSystem($fileSystem);
    $some->setFilter($filter);
    $some->setSelector($selector);

假设我们必须在应用的不同地方使用和创建这些对象。如果当你永远不需要任何依赖实例时，你需要去删掉构造函数的参数，或者去删掉注入的setter。为了解决这样的问题，我们再次回到全局注册表创建组件。不管怎么样，在创建对象之前，它增加了一个新的抽象层：	
	
Think if we had to create this object in many parts of our application. In the future, if we do not require any of the dependencies,
we need to go through the entire code base to remove the parameter in any constructor or setter where we injected the code. To solve this,
we return again to a global registry to create the component. However, it adds a new layer of abstraction before creating
the object:

.. code-block:: php

    <?php

    class SomeComponent
    {

        // ...

        /**
         * Define a factory method to create SomeComponent instances injecting its dependencies
         */
        public static function factory()
        {

            $connection = new Connection();
            $session    = new Session();
            $fileSystem = new FileSystem();
            $filter     = new Filter();
            $selector   = new Selector();

            return new self($connection, $session, $fileSystem, $filter, $selector);
        }

    }

瞬间，我们又回到刚刚开始的问题了，我们再次创建依赖实例在组件内部！我们可以继续前进，找出一个每次能奏效的方法去解决这个问题。但似乎一次又一次，我们又回到了不实用的例子中。	
	
Now we find ourselves back where we started, we are again building the dependencies inside of the component! We must seek a solution that
keeps us from repeatedly falling into bad practices.

一个实用和优雅的解决方法，是为依赖实例提供一个容器。这个容器担任全局的注册表，就像我们刚才看到的那样。使用依赖实例的容器作为一个桥梁来获取依赖实例，使我们能够降低我们的组件的复杂性：

A practical and elegant way to solve these problems is using a container for dependencies. The containers act as the global registry that
we saw earlier. Using the container for dependencies as a bridge to obtain the dependencies allows us to reduce the complexity
of our component:

.. code-block:: php

    <?php

    class SomeComponent
    {

        protected $_di;

        public function __construct($di)
        {
            $this->_di = $di;
        }

        public function someDbTask()
        {

            // Get the connection service
            // Always returns a new connection
            $connection = $this->_di->get('db');

        }

        public function someOtherDbTask()
        {

            // Get a shared connection service,
            // this will return the same connection everytime
            $connection = $this->_di->getShared('db');

            //This method also requires an input filtering service
            $filter = $this->_di->get('filter');

        }

    }

    $di = new Phalcon\DI();

    //Register a "db" service in the container
    $di->set('db', function() {
        return new Connection(array(
            "host"      => "localhost",
            "username"  => "root",
            "password"  => "secret",
            "dbname"    => "invo"
        ));
    });

    //Register a "filter" service in the container
    $di->set('filter', function() {
        return new Filter();
    });

    //Register a "session" service in the container
    $di->set('session', function() {
        return new Session();
    });

    //Pass the service container as unique parameter
    $some = new SomeComponent($di);

    $some->someDbTask();

这个组件现在可以很简单的获取到它所需要的服务，服务采用延迟加载的方式，只有在需要使用的时候才初始化，这也节省了服务器资源。这个组件现在是高度解耦。例如，我们可以替换掉创建连接的方式，它们的行为或它们的任何其他方面，也不会影响该组件。	
	
The component can now simply access the service it requires when it needs it, if it does not require a service it is not even initialized,
saving resources. The component is now highly decoupled. For example, we can replace the manner in which connections are created,
their behavior or any other aspect of them and that would not affect the component.

实现方法Our approach
=======================
Phalcon\\DI 是一个实现依赖注入和定位服务的组件，而且它本身就是一个装载它们的容器。因为Phalcon是高度解构的，整合框架的不同组件，使用Phalcon\\DI是必不可少的。开发者也可以使用这个组件去注入依赖和管理的应用程序中来自不同类的全局实例。

Phalcon\\DI is a component implementing Dependency Injection and Location of services and it's itself a container for them.

Since Phalcon is highly decoupled, Phalcon\\DI is essential to integrate the different components of the framework. The developer can
also use this component to inject dependencies and manage global instances of the different classes used in the application.

基本上，这个组件实现了 `Inversion of Control`_ (控制器反转)的模式。使用这种模式，组件的对象不用再使用setter或者构造函数去接受依赖实例，而是使用请求服务的依赖注入。这减少了总的复杂性，因为在组件内，只有一个方法去获取所需的依赖实例。

Basically, this component implements the `Inversion of Control`_ pattern. Applying this, the objects do not receive their dependencies
using setters or constructors, but requesting a service dependency injector. This reduces the overall complexity since there is only
one way to get the required dependencies within a component.

另外，该模式增加了代码的可测试性，从而使其不易出错。

Additionally, this pattern increases testability in the code, thus making it less prone to errors.

使用容器注册服务Registering services in the Container
============================================================
框架本身或者开发者都可以注册服务。当一个组件A需要组件B(或者它的类的实例) 去操作，它可以通过容器去请求组件B，而不是创建一个新的组件B实例。

The framework itself or the developer can register services. When a component A requires component B (or an instance of its class) to operate, it
can request component B from the container, rather than creating a new instance component B.

这个工作方法给我们提供了许多优势：

This way of working gives us many advantages:

* 我们可以很容易的使用一个我们自己建立的或者是第三方的组件去替换原有的组件。
* 我们完全控制对象的初始化，这让我们在传递它们的实例到组件之前，根据需要设置这些对象。
* 我们可以在一个结构化的和统一组件内获取全局实例。

* We can easily replace a component with one created by ourselves or a third party.
* We have full control of the object initialization, allowing us to set these objects, as needed before delivering them to components.
* We can get global instances of components in a structured and unified way

服务可以使用不同方式去定义：

Services can be registered using several types of definitions:

.. code-block:: php

    <?php

    use Phalcon\Http\Request;

    //Create the Dependency Injector Container
    $di = new Phalcon\DI();

    //By its class name
    $di->set("request", 'Phalcon\Http\Request');

    //Using an anonymous function, the instance will be lazy loaded
    $di->set("request", function() {
        return new Request();
    });

    //Registering an instance directly
    $di->set("request", new Request());

    //Using an array definition
    $di->set("request", array(
        "className" => 'Phalcon\Http\Request'
    ));

使用数组的方式去注册服务也是可以的：	
	
The array syntax is also allowed to register services:

.. code-block:: php

    <?php

    use Phalcon\Http\Request;

    //Create the Dependency Injector Container
    $di = new Phalcon\DI();

    //By its class name
    $di["request"] = 'Phalcon\Http\Request';

    //Using an anonymous function, the instance will be lazy loaded
    $di["request"] = function() {
        return new Request();
    };

    //Registering an instance directly
    $di["request"] = new Request();

    //Using an array definition
    $di["request"] = array(
        "className" => 'Phalcon\Http\Request'
    );

在上面的例子中，当框架需要访问request服务的内容，它会在容器里面查找名为‘request’的服务。 在容器中将返回所需要的服务的实例。当有需要时，开发者可能最终需要替换这个组件。	
	
In the examples above, when the framework needs to access the request data, it will ask for the service identified as ‘request’ in the container.
The container in turn will return an instance of the required service. A developer might eventually replace a component when he/she needs.

每个方法（在上面的例子证明）用于设置/注册服务方面具都具有优势和劣势。这是由开发者和特别的要求决定具体使用哪个。

Each of the methods (demonstrated in the examples above) used to set/register a service has advantages and disadvantages. It is up to the
developer and the particular requirements that will designate which one is used.

通过字符串设置一个服务是很简单，但是缺乏灵活性。通过数组设置服务提供了更加灵活的方式，但是使代码更复杂。匿名函数是上述两者之间的一个很好的平衡，但是会导致比预期的更多维护。

Setting a service by a string is simple, but lacks flexibility. Setting services using an array offers a lot more flexibility, but makes the
code more complicated. The lambda function is a good balance between the two, but could lead to more maintenance than one would expect.

Phalcon\\DI 对每个储存的服务提供了延迟加载。除非开发者选择直接实例化一个对象并将其存储在容器中，任何储存在里面的对象(通过数组，字符串等等设置的)都将延迟加载，即只要当使用到时才实例化。

Phalcon\\DI offers lazy loading for every service it stores. Unless the developer chooses to instantiate an object directly and store it
in the container, any object stored in it (via array, string, etc.) will be lazy loaded i.e. instantiated only when requested.

简单的注册Simple Registration
--------------------------------
就像你之前看到的那样，这里有几种方法去注册服务。下面是简单调用的例子：

As seen before, there are several ways to register services. These we call simple:

字符串String
^^^^^^^^^^^^^^^^
使用字符串注册服务需要一个有效的类名称，它将返回指定的类对象，如果类还没有加载的话，将使用自动加载器实例化对象。这种类型不允许向构造函数指定参数：

This type expects the name of a valid class, returning an object of the specified class, if the class is not loaded it will be instantiated using an auto-loader.
This type of definition does not allow to specify arguments for the class constructor or parameters:

.. code-block:: php

    <?php

    // return new Phalcon\Http\Request();
    $di->set('request', 'Phalcon\Http\Request');

对象Object
^^^^^^^^^^^^
这种类型注册服务需要一个对象。实际上，这个服务不再需要初始化，因为它已经是一个对象，可以说，这是不是一个真正的依赖注入，但是如果你想强制总是返回相同的对象/值，使用这种方式还是有用的:

This type expects an object. Due to the fact that object does not need to be resolved as it is
already an object, one could say that it is not really a dependency injection,
however it is useful if you want to force the returned dependency to always be
the same object/value:

.. code-block:: php

    <?php

    use Phalcon\Http\Request;

    // return new Phalcon\Http\Request();
    $di->set('request', new Request());

闭包与匿名函数Closures/Anonymous functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
这个方法提供了更加自由的方式去注册依赖，但是如果你想从外部改变实例化的参数而不用改变注册服务的代码，这是很困难的：

This method offers greater freedom to build the dependency as desired, however, it is difficult to
change some of the parameters externally without having to completely change the definition of dependency:

.. code-block:: php

    <?php

    use Phalcon\Db\Adapter\Pdo\Mysql as PdoMysql;

    $di->set("db", function() {
        return new PdoMysql(array(
             "host"     => "localhost",
             "username" => "root",
             "password" => "secret",
             "dbname"   => "blog"
        ));
    });

这些限制是可以克服的，通过传递额外的变量到闭包函数里面：	
	
Some of the limitations can be overcome by passing additional variables to the closure's environment:

.. code-block:: php

    <?php

    use Phalcon\Db\Adapter\Pdo\Mysql as PdoMysql;

    //Using the $config variable in the current scope
    $di->set("db", function() use ($config) {
        return new PdoMysql(array(
             "host"     => $config->host,
             "username" => $config->username,
             "password" => $config->password,
             "dbname"   => $config->name
        ));
    });

复杂的注册Complex Registration
------------------------------------
如果要求不用实例化/解析服务，就可以改变定义服务的话，我们需要使用数组的方式去定义服务。使用数组去定义服务可以更加详细：

If it is required to change the definition of a service without instantiating/resolving the service,
then, we need to define the services using the array syntax. Define a service using an array definition
can be a little more verbose:

.. code-block:: php

    <?php

    use \Phalcon\Logger\Adapter\File as LoggerFile;

    //Register a service 'logger' with a class name and its parameters
    $di->set('logger', array(
        'className' => 'Phalcon\Logger\Adapter\File',
        'arguments' => array(
            array(
                'type'  => 'parameter',
                'value' => '../apps/logs/error.log'
            )
        )
    ));

    //Using an anonymous function
    $di->set('logger', function() {
        return new LoggerFile('../apps/logs/error.log');
    });

上面两种注册服务的方式的结果是一样的。然而，使用数组定义的话，在需要的时候可以变更注册服务的参数：	
	
Both service registrations above produce the same result. The array definition however, allows for alteration of the service parameters if needed:

.. code-block:: php

    <?php

    //Change the service class name
    $di->getService('logger')->setClassName('MyCustomLogger');

    //Change the first parameter without instantiating the logger
    $di->getService('logger')->setParameter(0, array(
        'type'  => 'parameter',
        'value' => '../apps/logs/error.log'
    ));

除了使用数组的语法注册服务，你还可以使用以下三种类型的依赖注入：	
	
In addition by using the array syntax you can use three types of dependency injection:

构造函数注入Constructor Injection
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
这个注入方式是通过传递依赖/参数到类的构造函数。让我们假设我们有下面的组件：

This injection type passes the dependencies/arguments to the class constructor.
Let's pretend we have the following component:

.. code-block:: php

    <?php

    namespace SomeApp;

    use Phalcon\Http\Response;

    class SomeComponent
    {

        protected $_response;

        protected $_someFlag;

        public function __construct(Response $response, $someFlag)
        {
            $this->_response = $response;
            $this->_someFlag = $someFlag;
        }

    }

这个服务可以这样被注入：	
	
The service can be registered this way:

.. code-block:: php

    <?php

    $di->set('response', array(
        'className' => 'Phalcon\Http\Response'
    ));

    $di->set('someComponent', array(
        'className' => 'SomeApp\SomeComponent',
        'arguments' => array(
            array('type' => 'service', 'name' => 'response'),
            array('type' => 'parameter', 'value' => true)
        )
    ));

reponse服务(Phalcon\Http\Response)作为第一个参数传递给构造函数，与此同时，一个布尔类型的值(true)作为第二个参数传递。	
	
The service "response" (Phalcon\\Http\\Response) is resolved to be passed as the first argument of the constructor,
while the second is a boolean value (true) that is passed as it is.

设值注入Setter Injection
^^^^^^^^^^^^^^^^^^^^^^^^^^
类中可能有setter去注入可选的依赖，前面那个class可以修改成通过setter来注入依赖的方式：

Classes may have setters to inject optional dependencies, our previous class can be changed to accept the dependencies with setters:

.. code-block:: php

    <?php

    namespace SomeApp;

    use Phalcon\Http\Response;

    class SomeComponent
    {

        protected $_response;

        protected $_someFlag;

        public function setResponse(Response $response)
        {
            $this->_response = $response;
        }

        public function setFlag($someFlag)
        {
            $this->_someFlag = $someFlag;
        }

    }

用setter方式来注入的服务可以通过下面的方式来注册：	
	
A service with setter injection can be registered as follows:

.. code-block:: php

    <?php

    $di->set('response', array(
        'className' => 'Phalcon\Http\Response'
    ));

    $di->set('someComponent', array(
        'className' => 'SomeApp\SomeComponent',
        'calls'     => array(
            array(
                'method'    => 'setResponse',
                'arguments' => array(
                    array('type' => 'service', 'name' => 'response'),
                )
            ),
            array(
                'method'    => 'setFlag',
                'arguments' => array(
                    array('type' => 'parameter', 'value' => true)
                )
            )
        )
    ));

属性注入Properties Injection
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
这是一个不太常用的方式，这种方式的注入是通过类的public属性来注入：

A less common strategy is to inject dependencies or parameters directly into public attributes of the class:

.. code-block:: php

    <?php

    namespace SomeApp;

    use Phalcon\Http\Response;

    class SomeComponent
    {

        public $response;

        public $someFlag;

    }

通过属性注入的服务，可以像下面这样注册：	
	
A service with properties injection can be registered as follows:

.. code-block:: php

    <?php

    $di->set('response', array(
        'className' => 'Phalcon\Http\Response'
    ));

    $di->set('someComponent', array(
        'className'     => 'SomeApp\SomeComponent',
        'properties'    => array(
            array(
                'name'  => 'response',
                'value' => array('type' => 'service', 'name' => 'response')
            ),
            array(
                'name'  => 'someFlag',
                'value' => array('type' => 'parameter', 'value' => true)
            )
        )
    ));

支持包括下面的参数类型：	
	
Supported parameter types include the following:

+-------------+----------------------------------------------------------+-------------------------------------------------------------------------------------+
| Type        | Description                                              | Example                                                                             |
+=============+==========================================================+=====================================================================================+
| parameter   | 表示一个文本值作为参数传递过去                           | array('type' => 'parameter', 'value' => 1234)                                       |
+-------------+----------------------------------------------------------+-------------------------------------------------------------------------------------+
| service     | 表示作为服务                                             | array('type' => 'service', 'name' => 'request')                                     |
+-------------+----------------------------------------------------------+-------------------------------------------------------------------------------------+
| instance    | 表示必须动态生成的对象                                   | array('type' => 'instance', 'className' => 'DateTime', 'arguments' => array('now')) |
+-------------+----------------------------------------------------------+-------------------------------------------------------------------------------------+

解析一个定义复杂的服务也许性能上稍微慢于先前看到的简单定义。但是，这提供了一个更强大的方式来定义和注入服务。

Resolving a service whose definition is complex may be slightly slower than simple definitions seen previously. However,
these provide a more robust approach to define and inject services.

混合不同类型的定义是可以的，每个人可以应用需要决定什么样的注册服务的方式是最适当的。

Mixing different types of definitions is allowed, everyone can decide what is the most appropriate way to register the services
according to the application needs.

解析服务Resolving Services
==================
从容器中获取一个服务是一件简单的事情，只要通过“get”方法就可以。这将返回一个服务的新实例：

Obtaining a service from the container is a matter of simply calling the “get” method. A new instance of the service will be returned:

.. code-block:: php

    <?php $request = $di->get("request");

或者通过魔术方法的方式获取：	
	
Or by calling through the magic method:

.. code-block:: php

    <?php

    $request = $di->getRequest();

或者通过访问数组的方式获取：	
	
Or using the array-access syntax:

.. code-block:: php

    <?php

    $request = $di['request'];

参数可以传递到构造函数中，通过添加一个数组的参数到get方法中：	
	
Arguments can be passed to the constructor by adding an array parameter to the method "get":

.. code-block:: php

    <?php

    // new MyComponent("some-parameter", "other")
    $component = $di->get("MyComponent", array("some-parameter", "other"));

事件Events
^^^^^^^^^^^^^^^^^
:doc:`Phalcon\\Di <../api/Phalcon_DI>` 能够向存在的:doc:`EventsManager <events>`发送事件。事件被"di"类型触发。事件返回布尔false值的会组织这个操作。下面是支持的事件：

:doc:`Phalcon\\Di <../api/Phalcon_DI>` is able to send events to an :doc:`EventsManager <events>` if it is present.
Events are triggered using the type "di". Some events when returning boolean false could stop the active operation.
The following events are supported:

+----------------------+---------------------------------------------------------------------------------------------------------------------------------+---------------------+--------------------+
| Event Name           | Triggered                                                                                                                       | Can stop operation? | Triggered on       |
+======================+=================================================================================================================================+=====================+====================+
| beforeServiceResolve | Triggered before resolve service. Listeners receive the service name and the parameters passed to it.                           | No                  | Listeners          |
+----------------------+---------------------------------------------------------------------------------------------------------------------------------+---------------------+--------------------+
| afterServiceResolve  | Triggered after resolve service. Listeners receive the service name, instance, and the parameters passed to it.                 | No                  | Listeners          |
+----------------------+---------------------------------------------------------------------------------------------------------------------------------+---------------------+--------------------+

共享服务Shared services
============================
服务可以注册成“shared”类型的服务，这意味着这个服务将使用 singletons_ [单例模式]运行， 一旦服务被首次解析后，这个实例将被保存在容器中，之后的每次请求都在容器中查找并返回这个实例。

Services can be registered as "shared" services this means that they always will act as singletons_. Once the service is resolved for the first time
the same instance of it is returned every time a consumer retrieve the service from the container:

.. code-block:: php

    <?php

    use Phalcon\Session\Adapter\Files as SessionFiles;

    //Register the session service as "always shared"
    $di->setShared('session', function() {
        $session = new SessionFiles();
        $session->start();
        return $session;
    });

    $session = $di->get('session'); // Locates the service for the first time
    $session = $di->getSession(); // Returns the first instantiated object

另一种方式去注册一个“shared”类型的服务是，传递“set”服务的时候，把true作为第三个参数传递过去：	
	
An alternative way to register shared services is to pass "true" as third parameter of "set":

.. code-block:: php

    <?php

    //Register the session service as "always shared"
    $di->set('session', function() {
        //...
    }, true);

如果一个服务不是注册成“shared”类型，而你又想从DI中获取服务的“shared”实例，你可以使用getShared方法：	
	
If a service isn't registered as shared and you want to be sure that a shared instance will be accessed every time
the service is obtained from the DI, you can use the 'getShared' method:

.. code-block:: php

    <?php

    $request = $di->getShared("request");

单独操作服务Manipulating services individually
=================================================
一旦服务被注册到服务容器中，你可以单独操作它：

Once a service is registered in the service container, you can retrieve it to manipulate it individually:

.. code-block:: php

    <?php

    use Phalcon\Http\Request;

    //Register the "register" service
    $di->set('request', 'Phalcon\Http\Request');

    //Get the service
    $requestService = $di->getService('request');

    //Change its definition
    $requestService->setDefinition(function() {
        return new Request();
    });

    //Change it to shared
    $requestService->setShared(true);

    //Resolve the service (return a Phalcon\Http\Request instance)
    $request = $requestService->resolve();

通过服务容器实例化类Instantiating classes via the Service Container
=======================================================================
当你从服务容器中请求一个服务，如果找不到具有相同名称的服务，它将尝试去加载以这个服务为名称的类。利用这个的行为， 我们可以代替任意一个类，通过简单的利用服务的名称来注册：

When you request a service to the service container, if it can't find out a service with the same name it'll try to load a class with
the same name. With this behavior we can replace any class by another simply by registering a service with its name:

.. code-block:: php

    <?php

    //Register a controller as a service
    $di->set('IndexController', function() {
        $component = new Component();
        return $component;
    }, true);

    //Register a controller as a service
    $di->set('MyOtherComponent', function() {
        //Actually returns another component
        $component = new AnotherComponent();
        return $component;
    });

    //Create an instance via the service container
    $myComponent = $di->get('MyOtherComponent');

你可以利用这种方式，通过服务容器来总是实例化你的类(即是他们没有注册为服务)， DI会回退到一个有效的自动加载类中，去加载这个类。通过这样做，以后你可以轻松替换任意的类通过为它实现一个定义。	
	
You can take advantage of this, always instantiating your classes via the service container (even if they aren't registered as services). The DI will
fallback to a valid autoloader to finally load the class. By doing this, you can easily replace any class in the future by implementing a definition
for it.

自动注入 DIAutomatic Injecting of the DI itself
================================================
如果一个类或者组件需要用到DI服务，你需要在你的类中实现  :doc:`Phalcon\\DI\\InjectionAwareInterface <../api/Phalcon_DI_InjectionAwareInterface>` 接口， 这样就可以在实例化这个类的对象时自动注入DI的服务:

If a class or component requires the DI itself to locate services, the DI can automatically inject itself to the instances it creates,
to do this, you need to implement the :doc:`Phalcon\\DI\\InjectionAwareInterface <../api/Phalcon_DI_InjectionAwareInterface>` in your classes:

.. code-block:: php

    <?php

    use Phalcon\DI\InjectionAwareInterface;

    class MyClass implements InjectionAwareInterface
    {

        protected $_di;

        public function setDi($di)
        {
            $this->_di = $di;
        }

        public function getDi()
        {
            return $this->_di;
        }

    }

按照上面这样，一旦服务被解析，$di对象将自动传递到setDi()方法：	
	
Then once the service is resolved, the $di will be passed to setDi automatically:

.. code-block:: php

    <?php

    //Register the service
    $di->set('myClass', 'MyClass');

    //Resolve the service (NOTE: $myClass->setDi($di) is automatically called)
    $myClass = $di->get('myClass');

避免服务解析Avoiding service resolution
=========================================
一些服务是用于应用的每个请求中，通过消除解析服务的过程的方式，可以使得服务解析在性能上会有小小的提升：

Some services are used in each of the requests made to the application, eliminate the process of resolving the service
could add some small improvement in performance.

.. code-block:: php

    <?php

    //Resolve the object externally instead of using a definition for it:
    $router = new MyRouter();

    //Pass the resolved object to the service registration
    $di->set('router', $router);

使用文件组织服务Organizing services in files
=================================================
你可以更好的组织你的应用，通过移动注册的服务到独立的文件里面，而不是全部写在应用的引导文件中：

You can better organize your application by moving the service registration to individual files instead of
doing everything in the application's bootstrap:

.. code-block:: php

    <?php

    $di->set('router', function() {
        return include "../app/config/routes.php";
    });

这样，在文件(”../app/config/routes.php”)中，返回已解析的对象：	
	
Then in the file ("../app/config/routes.php") return the object resolved:

.. code-block:: php

    <?php

    $router = new MyRouter();

    $router->post('/login');

    return $router;

使用静态的方式访问注入器Accessing the DI in a static way
============================================================
If needed you can access the latest DI created in a static function in the following way:

.. code-block:: php

    <?php

    use Phalcon\DI;

    class SomeComponent
    {

        public static function someMethod()
        {
            //Get the session service
            $session = DI::getDefault()->getSession();
        }

    }

注入器默认工厂Factory Default DI
=====================================
尽管Phalcon的解耦性质为我们提供了很大的自由度和灵活性，也许我们只是单纯的想使用它作为一个全栈框架。 为了达到这点，框架提供了变种的 Phalcon\DI 叫 Phalcon\DI\FactoryDefault 。这个类会自动注册相应的服务，并捆绑在一起作为一个全栈框架。

Although the decoupled character of Phalcon offers us great freedom and flexibility, maybe we just simply want to use it as a full-stack
framework. To achieve this, the framework provides a variant of Phalcon\\DI called Phalcon\\DI\\FactoryDefault. This class automatically
registers the appropriate services bundled with the framework to act as full-stack.

.. code-block:: php

    <?php

    use Phalcon\DI\FactoryDefault;

    $di = new FactoryDefault();

服务名称约定Service Name Conventions
==========================================
尽管你可以用你喜欢的名字来注册服务，但是Phalcon有一些命名约定，这些约定让你在需要的时候，可以获得正确的（内置）服务。

Although you can register services with the names you want, Phalcon has a several naming conventions that allow it to get the
the correct (built-in) service when you need it.

+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| Service Name        | Description                                 | Default                                                                                            | Shared |
+=====================+=============================================+====================================================================================================+========+
| dispatcher          | Controllers Dispatching Service             | :doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>`                                    | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| router              | Routing Service                             | :doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>`                                            | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| url                 | URL Generator Service                       | :doc:`Phalcon\\Mvc\\Url <../api/Phalcon_Mvc_Url>`                                                  | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| request             | HTTP Request Environment Service            | :doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>`                                        | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| response            | HTTP Response Environment Service           | :doc:`Phalcon\\Http\\Response <../api/Phalcon_Http_Response>`                                      | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| cookies             | HTTP Cookies Management Service             | :doc:`Phalcon\\Http\\Response\\Cookies <../api/Phalcon_Http_Response_Cookies>`                     | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| filter              | Input Filtering Service                     | :doc:`Phalcon\\Filter <../api/Phalcon_Filter>`                                                     | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| flash               | Flash Messaging Service                     | :doc:`Phalcon\\Flash\\Direct <../api/Phalcon_Flash_Direct>`                                        | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| flashSession        | Flash Session Messaging Service             | :doc:`Phalcon\\Flash\\Session <../api/Phalcon_Flash_Session>`                                      | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| session             | Session Service                             | :doc:`Phalcon\\Session\\Adapter\\Files <../api/Phalcon_Session_Adapter_Files>`                     | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| eventsManager       | Events Management Service                   | :doc:`Phalcon\\Events\\Manager <../api/Phalcon_Events_Manager>`                                    | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| db                  | Low-Level Database Connection Service       | :doc:`Phalcon\\Db <../api/Phalcon_Db>`                                                             | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| security            | Security helpers                            | :doc:`Phalcon\\Security <../api/Phalcon_Security>`                                                 | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| crypt               | Encrypt/Decrypt data                        | :doc:`Phalcon\\Crypt <../api/Phalcon_Crypt>`                                                       | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| tag                 | HTML generation helpers                     | :doc:`Phalcon\\Tag <../api/Phalcon_Tag>`                                                           | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| escaper             | Contextual Escaping                         | :doc:`Phalcon\\Escaper <../api/Phalcon_Escaper>`                                                   | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| annotations         | Annotations Parser                          | :doc:`Phalcon\\Annotations\\Adapter\\Memory <../api/Phalcon_Annotations_Adapter_Memory>`           | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| modelsManager       | Models Management Service                   | :doc:`Phalcon\\Mvc\\Model\\Manager <../api/Phalcon_Mvc_Model_Manager>`                             | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| modelsMetadata      | Models Meta-Data Service                    | :doc:`Phalcon\\Mvc\\Model\\MetaData\\Memory <../api/Phalcon_Mvc_Model_MetaData_Memory>`            | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| transactionManager  | Models Transaction Manager Service          | :doc:`Phalcon\\Mvc\\Model\\Transaction\\Manager <../api/Phalcon_Mvc_Model_Transaction_Manager>`    | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| modelsCache         | Cache backend for models cache              | None                                                                                               | -      |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| viewsCache          | Cache backend for views fragments           | None                                                                                               | -      |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+

自定义注入器Implementing your own DI
==========================================
如果你要创建一个自定义注入器或者继承一个已有的，接口 :doc:`Phalcon\\DiInterface <../api/Phalcon_DiInterface>`  必须被实现。

The :doc:`Phalcon\\DiInterface <../api/Phalcon_DiInterface>` interface must be implemented to create your own DI replacing the one provided by Phalcon or extend the current one.

.. _`Inversion of Control`: http://en.wikipedia.org/wiki/Inversion_of_control
.. _Singletons: http://en.wikipedia.org/wiki/Singleton_pattern
