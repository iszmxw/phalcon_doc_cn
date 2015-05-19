教程1：通过例子学习
=====================================================
虽然是第一个教程，我们会通过从头做一个有简单注册表单的应用来阐述框架的流程行为。如果对phalcon自动生成基础架构代码感兴趣可以参考 :doc:`开发者工具<tools>`

Throughout this first tutorial, we'll walk you through the creation of an application with a simple registration
form from the ground up. We will also explain the basic aspects of the framework's behavior. If you are interested
in automatic code generation tools for Phalcon, you can check our :doc:`developer tools <tools>`.

检查phalcon安装是否正确
--------------------------
如果我们已经完成了phalcon的安装，检查phpinfo() 的输出，查找是否有phalcon相关板块输出，或者是执行下面代码：

We'll assume you have Phalcon installed already. Check your phpinfo() output for a section referencing "Phalcon"
or execute the code snippet below:

.. code-block:: php

    <?php print_r(get_loaded_extensions()); ?>

phalcon扩展应该会出现在如下所示的列表中	
	
The Phalcon extension should appear as part of the output:

.. code-block:: php

    Array
    (
        [0] => Core
        [1] => libxml
        [2] => filter
        [3] => SPL
        [4] => standard
        [5] => phalcon
        [6] => pdo_mysql
    )

创建项目 
-----------------------------
学习这个教程最好的方式就是按照说明一步步来，可以从 `这里 <https://github.com/phalcon/tutorial>`_ 下载到完整代码

The best way to use this guide is to follow each step in turn. You can get the complete code
`here <https://github.com/phalcon/tutorial>`_.

文件结构
^^^^^^^^^^^^^^
phalcon没有强制在开发过程中使用一个特定的文件目录结构。基于phalcon的松耦合模式，我们可以使用我们自己感觉合适的目录接口来进行基于phalcon的开发。

Phalcon does not impose a particular file structure for application development. Due to the fact that it is
loosely coupled, you can implement Phalcon powered applications with a file structure you are most comfortable using.

基于现在教程，建议采用如下的简单目录结构：

For the purposes of this tutorial and as a starting point, we suggest this very simple structure:

.. code-block:: php

    tutorial/
      app/
        controllers/
        models/
        views/
      public/
        css/
        img/
        js/

注意我们不需要Library库目录，因为框架已经被加载到内存中了，随时都可以使用。		
		
Note that you don't need any "library" directory related to Phalcon. The framework is available in memory,
ready for you to use.

优美的 URLs 
^^^^^^^^^^^^
我们将在这个教程使用优美的友好URLs。友好的URLs有利于SEO和方便用户记忆。phalcon支持现在流行的web服务提供的重写模块。使用友好的URLs并不是必须的，不用我们照样可以顺利开发。

We'll use pretty (friendly) URLs for this tutorial. Friendly URLs are better for SEO as well as being easy for users to remember. Phalcon supports rewrite modules provided by the most popular web servers. Making your application's URLs friendly is not a requirement and you can just as easily develop without them.

在这个例子中我们将使用Apache的重写模块。在/tutorial/.htaccess 文件中写入如下的重写规则：

In this example we'll use the rewrite module for Apache. Let's create a couple of rewrite rules in the /tutorial/.htaccess file:

.. code-block:: apacheconf

    #/tutorial/.htaccess
    <IfModule mod_rewrite.c>
        RewriteEngine on
        RewriteRule  ^$ public/    [L]
        RewriteRule  (.*) public/$1 [L]
    </IfModule>

所有的请求都会被重写到public/目录并作为根目录。这步操作就是保证内部资源不被公开浏览并且减少安全威胁。
	
All requests to the project will be rewritten to the public/ directory making it the document root. This step ensures that the internal project folders remain hidden from public viewing and thus eliminates security threats of this kind.

下面第二组规则检查请求的文件是否存在，如果存在则web服务没有必要对其进行重写。

The second set of rules will check if the requested file exists and, if it does, it doesn't have to be rewritten by the web server module:

.. code-block:: apacheconf

    #/tutorial/public/.htaccess
    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php?_url=/$1 [QSA,L]
    </IfModule>

引导程序Bootstrap
^^^^^^^^^^^^^^^^^^
第一个要被创建文件就是引导文件。这个文件非常重要的，它是应用的唯一入口，在引导文件里面我们可以初始化组件并且能够控制应用的流程动作行为。

The first file you need to create is the bootstrap file. This file is very important; since it serves
as the base of your application, giving you control of all aspects of it. In this file you can implement
initialization of components as well as application behavior.

tutorial/public/index.php 引导文件内容如下所示：

The tutorial/public/index.php file should look like:

.. code-block:: php

    <?php

    use Phalcon\Loader;
    use Phalcon\Mvc\View;
    use Phalcon\Mvc\Url as UrlProvider;
    use Phalcon\Mvc\Application;
    use Phalcon\DI\FactoryDefault;

    try {

        // Register an autoloader
        $loader = new Loader();
        $loader->registerDirs(array(
            '../app/controllers/',
            '../app/models/'
        ))->register();

        // Create a DI
        $di = new FactoryDefault();

        // Setup the view component
        $di->set('view', function(){
            $view = new View();
            $view->setViewsDir('../app/views/');
            return $view;
        });

        // Setup a base URI so that all generated URIs include the "tutorial" folder
        $di->set('url', function(){
            $url = new UrlProvider();
            $url->setBaseUri('/tutorial/');
            return $url;
        });

        // Handle the request
        $application = new Application($di);

        echo $application->handle()->getContent();

    } catch(\Exception $e) {
         echo "PhalconException: ", $e->getMessage();
    }

自动加载器Autoloaders
^^^^^^^^^^^^^^^^^^^^^
在引导程序的第一步我们注册一个自动加载器。自动加载器在应用中被用于将类加载为控制器和数据模型。例如我们可以注册一个或多个控制器目录以增加应用的灵活性。在现在的这个例子中我们使用Phalcon\\Loader这个组件。

The first part that we find in the bootstrap is registering an autoloader. This will be used to load classes as controllers and models in the application. For example we may register one or more directories of controllers increasing the flexibility of the application. In our example we have used the component Phalcon\\Loader.

使用自动加载器可以通过多种方法来加载类，但是在这个例子中我们选择从预定义的目录中加载类：

With it, we can load classes using various strategies but for this example we have chosen to locate classes based on predefined directories:

.. code-block:: php

    <?php

    use Phalcon\Loader;

    // ...

    $loader = new Loader();
    $loader->registerDirs(
        array(
            '../app/controllers/',
            '../app/models/'
        )
    )->register();

依赖管理
^^^^^^^^^^
使用phalcon之前我们首先要理解一个重要的概念 :doc:`依赖注入容器 <di>` 这个概念听来也许很复杂但实际是非常简单实用的。

A very important concept that must be understood when working with Phalcon is its :doc:`dependency injection container <di>`. It may sound complex but is actually very simple and practical.

一个服务容器就像一个袋子一样，里面存放着我们应用能够调用的全局服务。每当框架需要调用一个组件的时候都会以之前约定好名称去向服务容器请求该服务。因为phalcon是一个高度解耦的框架，Phalcon\\DI依赖注入容器作用就像是胶水一样将不同的组件透明的粘合在一起协同工作。

A service container is a bag where we globally store the services that our application will use to function. Each time the framework requires a component, it will ask the container using an agreed upon name for the service. Since Phalcon is a highly decoupled framework, Phalcon\\DI acts as glue facilitating the integration of the different components achieving their work together in a transparent manner.

.. code-block:: php

    <?php

    use Phalcon\DI\FactoryDefault;

    // ...

    // Create a DI
    $di = new FactoryDefault();

:doc:`Phalcon\\DI\\FactoryDefault <../api/Phalcon\_DI_FactoryDefault>` 是Phalcon\\DI的一个变体，为了让开发更简单，这个容器注册phalcon内置的大部分的组件。这样我们就不用一个个去注册这些常用组件了。当然后续使用其他的工厂服务也是没有问题的。	
	
:doc:`Phalcon\\DI\\FactoryDefault <../api/Phalcon\_DI_FactoryDefault>` is a variant of Phalcon\\DI. To make things easier,
it has registered most of the components that come with Phalcon. Thus we should not register them one by one.
Later there will be no problem in replacing a factory service.

在接下来的部分，我们注册了“view”服务向框架指明要从那个目录加载我们的视图文件。因为视图和类不一致，所以我们无法通过一个自动加载器加载视图。

In the next part, we register the "view" service indicating the directory where the framework will find the views files.
As the views do not correspond to classes, they cannot be charged with an autoloader.

服务可以用几种不同的方式来注册，在这个例子中我们使用 `anonymous function`_:

Services can be registered in several ways, but for our tutorial we'll use an `anonymous function`_:

.. code-block:: php

    <?php

    use Phalcon\Mvc\View;

    // ...

    // Setup the view component
    $di->set('view', function() {
        $view = new View();
        $view->setViewsDir('../app/views/');
        return $view;
    });


	
Next we register a base URI so that all URIs generated by Phalcon include the "tutorial" folder we setup earlier.
This will become important later on in this tutorial when we use the class :doc:`\Phalcon\\Tag <../api/Phalcon_Tag>`
to generate a hyperlink.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Url as UrlProvider;

    // ...

    // Setup a base URI so that all generated URIs include the "tutorial" folder
    $di->set('url', function(){
        $url = new UrlProvider();
        $url->setBaseUri('/tutorial/');
        return $url;
    });

In the last part of this file, we find :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>`. Its purpose
is to initialize the request environment, route the incoming request, and then dispatch any discovered actions;
it aggregates any responses and returns them when the process is complete.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Application;

    // ...

    $application = new Application($di);

    echo $application->handle()->getContent();

As you can see, the bootstrap file is very short and we do not need to include any additional files. We have set
ourselves a flexible MVC application in less than 30 lines of code.

Creating a Controller
^^^^^^^^^^^^^^^^^^^^^
By default Phalcon will look for a controller named "Index". It is the starting point when no controller or
action has been passed in the request. The index controller (app/controllers/IndexController.php) looks like:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class IndexController extends Controller
    {

        public function indexAction()
        {
            echo "<h1>Hello!</h1>";
        }

    }

The controller classes must have the suffix "Controller" and controller actions must have the suffix "Action". If you access the application from your browser, you should see something like this:

.. figure:: ../_static/img/tutorial-1.png
    :align: center

Congratulations, you're flying with Phalcon!

Sending output to a view
^^^^^^^^^^^^^^^^^^^^^^^^
Sending output to the screen from the controller is at times necessary but not desirable as most purists in the MVC community will attest. Everything must be passed to the view that is responsible for outputting data on screen. Phalcon will look for a view with the same name as the last executed action inside a directory named as the last executed controller. In our case (app/views/index/index.phtml):

.. code-block:: php

    <?php echo "<h1>Hello!</h1>";

Our controller (app/controllers/IndexController.php) now has an empty action definition:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class IndexController extends Controller
    {

        public function indexAction()
        {

        }

    }

The browser output should remain the same. The :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` static component is automatically created when the action execution has ended. Learn more about :doc:`views usage here <views>` .

Designing a sign up form
^^^^^^^^^^^^^^^^^^^^^^^^
Now we will change the index.phtml view file, to add a link to a new controller named "signup". The goal is to allow users to sign up within our application.

.. code-block:: php

    <?php

    echo "<h1>Hello!</h1>";

    echo $this->tag->linkTo("signup", "Sign Up Here!");

The generated HTML code displays an anchor ("a") HTML tag linking to a new controller:

.. code-block:: html

    <h1>Hello!</h1> <a href="/tutorial/signup">Sign Up Here!</a>

To generate the tag we use the class :doc:`\Phalcon\\Tag <../api/Phalcon_Tag>`. This is a utility class that allows
us to build HTML tags with framework conventions in mind. As this class is a also a service registered in the DI
we use $this->tag to access it.

A more detailed article regarding HTML generation can be :doc:`found here <tags>`

.. figure:: ../_static/img/tutorial-2.png
    :align: center

Here is the Signup controller (app/controllers/SignupController.php):

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class SignupController extends Controller
    {

        public function indexAction()
        {

        }

    }

The empty index action gives the clean pass to a view with the form definition (app/views/signup/index.phtml):

.. code-block:: html+php

    <h2>Sign up using this form</h2>

    <?php echo $this->tag->form("signup/register"); ?>

     <p>
        <label for="name">Name</label>
        <?php echo $this->tag->textField("name") ?>
     </p>

     <p>
        <label for="email">E-Mail</label>
        <?php echo $this->tag->textField("email") ?>
     </p>

     <p>
        <?php echo $this->tag->submitButton("Register") ?>
     </p>

    </form>

Viewing the form in your browser will show something like this:

.. figure:: ../_static/img/tutorial-3.png
    :align: center

:doc:`Phalcon\\Tag <../api/Phalcon_Tag>` also provides useful methods to build form elements.

The Phalcon\\Tag::form method receives only one parameter for instance, a relative uri to a controller/action in
the application.

By clicking the "Send" button, you will notice an exception thrown from the framework, indicating that we are missing the "register" action in the controller "signup". Our public/index.php file throws this exception:

    PhalconException: Action "register" was not found on controller "signup"

Implementing that method will remove the exception:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class SignupController extends Controller
    {

        public function indexAction()
        {

        }

        public function registerAction()
        {

        }

    }

If you click the "Send" button again, you will see a blank page. The name and email input provided by the user should be stored in a database. According to MVC guidelines, database interactions must be done through models so as to ensure clean object-oriented code.

Creating a Model
^^^^^^^^^^^^^^^^
Phalcon brings the first ORM for PHP entirely written in C-language. Instead of increasing the complexity of development, it simplifies it.

Before creating our first model, we need to create a database table outside of Phalcon to map it to. A simple table to store registered users can be defined like this:

.. code-block:: sql

    CREATE TABLE `users` (
      `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
      `name` varchar(70) NOT NULL,
      `email` varchar(70) NOT NULL,
      PRIMARY KEY (`id`)
    );

A model should be located in the app/models directory (app/models/Users.php). The model maps to the "users" table:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Users extends Model
    {

    }

Setting a Database Connection
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In order to be able to use a database connection and subsequently access data through our models, we need to specify it in our bootstrap process. A database connection is just another service that our application has that can be used for several components:

.. code-block:: php

    <?php

    use Phalcon\Loader;
    use Phalcon\DI\FactoryDefault;
    use Phalcon\Mvc\View;
    use Phalcon\Mvc\Application;
    use Phalcon\Mvc\Url as UrlProvider;
    use Phalcon\Db\Adapter\Pdo\Mysql as DbAdapter;

    try {

        // Register an autoloader
        $loader = new Loader();
        $loader->registerDirs(array(
            '../app/controllers/',
            '../app/models/'
        ))->register();

        // Create a DI
        $di = new FactoryDefault();

        // Setup the database service
        $di->set('db', function(){
            return new DbAdapter(array(
                "host"     => "localhost",
                "username" => "root",
                "password" => "secret",
                "dbname"   => "test_db"
            ));
        });

        // Setup the view component
        $di->set('view', function(){
            $view = new View();
            $view->setViewsDir('../app/views/');
            return $view;
        });

        // Setup a base URI so that all generated URIs include the "tutorial" folder
        $di->set('url', function(){
            $url = new UrlProvider();
            $url->setBaseUri('/tutorial/');
            return $url;
        });

        //Handle the request
        $application = new Application($di);

        echo $application->handle()->getContent();

    } catch(\Exception $e) {
         echo "Exception: ", $e->getMessage();
    }

With the correct database parameters, our models are ready to work and interact with the rest of the application.

Storing data using models
^^^^^^^^^^^^^^^^^^^^^^^^^
Receiving data from the form and storing them in the table is the next step.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class SignupController extends Controller
    {

        public function indexAction()
        {

        }

        public function registerAction()
        {

            $user = new Users();

            //Store and check for errors
            $success = $user->save($this->request->getPost(), array('name', 'email'));

            if ($success) {
                echo "Thanks for registering!";
            } else {
                echo "Sorry, the following problems were generated: ";
                foreach ($user->getMessages() as $message) {
                    echo $message->getMessage(), "<br/>";
                }
            }

            $this->view->disable();
        }
    }


We then instantiate the Users class, which corresponds to a User record. The class public properties map to the fields
of the record in the users table. Setting the relevant values in the new record and calling save() will store the data in the database for that record. The save() method returns a boolean value which indicates whether the storing of the data was successful or not.

The ORM automatically escapes the input preventing SQL injections so we only need to pass the request to the save method.

Additional validation happens automatically on fields that are defined as not null (required). If we don't enter any of the required fields in the sign up form our screen will look like this:

.. figure:: ../_static/img/tutorial-4.png
    :align: center

Conclusion
----------
This is a very simple tutorial and as you can see, it's easy to start building an application using Phalcon.
The fact that Phalcon is an extension on your web server has not interfered with the ease of development or
features available. We invite you to continue reading the manual so that you can discover additional features offered by Phalcon!

.. _anonymous function: http://php.net/manual/en/functions.anonymous.php

