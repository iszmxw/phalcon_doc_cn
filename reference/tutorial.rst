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

接下来我们注册一个根URI，这样所有在这个例子中由phalcon生成的URIs都会包含“tutorial”目录。这步非常重要，因为在下面的操作中我们会用到 :doc:`\Phalcon\\Tag <../api/Phalcon_Tag>` 去生成超链接。
	
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

在引导文件代码的最后部分我们看到 :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` ，它的作用是初始化请求环境，路由进来的请求，然后分配解析到的动作，最后汇聚所有的返回结果在所有流程结束后将结果返回。	

In the last part of this file, we find :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>`. Its purpose
is to initialize the request environment, route the incoming request, and then dispatch any discovered actions;
it aggregates any responses and returns them when the process is complete.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Application;

    // ...

    $application = new Application($di);

    echo $application->handle()->getContent();

正如我们刚才看到的这样，启动文件代码非常短，我们不用再去引入其他附件文件，就已经创建了一个不到30行代码的灵活的MVC应用。	
	
As you can see, the bootstrap file is very short and we do not need to include any additional files. We have set
ourselves a flexible MVC application in less than 30 lines of code.

创建一个控制器
^^^^^^^^^^^^^^^^
在请求中不指明控制器或动作的时候，默认情况下Phalcon将寻找一个名为“Index”的控制器。index控制器(app/controllers/IndexController.php)代码如下所示：

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

控制器的类名必须包含“Controller”尾缀，控制器中的动作actions方法必须包含“Action”尾缀。如果从浏览器中访问这个应用会有如下的显示；	
	
The controller classes must have the suffix "Controller" and controller actions must have the suffix "Action". If you access the application from your browser, you should see something like this:

.. figure:: ../_static/img/tutorial-1.png
    :align: center

恭喜！Phalcon已经可以带你去飞了！

Congratulations, you're flying with Phalcon!

输出到视图
^^^^^^^^^^^
从控制器直接输出数据到屏幕上有时是必要的,但是这样操作不可取的,因为大多数纯粹MVC主义者在社区将证明。一切想要显示的数据必须传递给视图,然后视图负责在屏幕上输出数据。Phalcon会在目录中查找最后执行的一个控制中最后一个动作方法同名的视图文件，在这个例子中就是这个文件(app/views/index/index.phtml):

Sending output to the screen from the controller is at times necessary but not desirable as most purists in the MVC community will attest. Everything must be passed to the view that is responsible for outputting data on screen. Phalcon will look for a view with the same name as the last executed action inside a directory named as the last executed controller. In our case (app/views/index/index.phtml):

.. code-block:: php

    <?php echo "<h1>Hello!</h1>";

现在我们的控制器(app/controllers/IndexController.php)中动作方法定义为空：
	
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

浏览器仍旧可以正常输出内容。:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` 这个静态组件在动作执行完毕后将会被创建。可以从 :doc:`视图使用说明 <views>` 学习更多。	

The browser output should remain the same. The :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` static component is automatically created when the action execution has ended. Learn more about :doc:`views usage here <views>` .

设计一个注册表单
^^^^^^^^^^^^^^^^^^
现在我们修改下index.phtml这个文件，添加一个指向名为“signup”控制器的链接，目的是让用户能在我们的应用中注册。

Now we will change the index.phtml view file, to add a link to a new controller named "signup". The goal is to allow users to sign up within our application.

.. code-block:: php

    <?php

    echo "<h1>Hello!</h1>";

    echo $this->tag->linkTo("signup", "Sign Up Here!");

生成的html代码是一个带有指向新控制器的a标签。	
	
The generated HTML code displays an anchor ("a") HTML tag linking to a new controller:

.. code-block:: html

    <h1>Hello!</h1> <a href="/tutorial/signup">Sign Up Here!</a>

生成html标签我们使用 :doc:`\Phalcon\\Tag <../api/Phalcon_Tag>` 类。这是一个实用程序类,允许我们在框架约定的范围内构建HTML标签。这个类也是一个被依赖注入容器注册的服务，我们可以用$this->tag去调用它。
	
To generate the tag we use the class :doc:`\Phalcon\\Tag <../api/Phalcon_Tag>`. This is a utility class that allows
us to build HTML tags with framework conventions in mind. As this class is a also a service registered in the DI
we use $this->tag to access it.

更多关于html生成的说明可以参考 :doc:`found here <tags>`

A more detailed article regarding HTML generation can be :doc:`found here <tags>`

.. figure:: ../_static/img/tutorial-2.png
    :align: center
	
下面代码是注册控制器(app/controllers/SignupController.php):

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

空index控制器指向了一个带有注册表单的视图文件(app/views/signup/index.phtml):	
	
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

在浏览器中访问会看到如下显示：	
	
Viewing the form in your browser will show something like this:

.. figure:: ../_static/img/tutorial-3.png
    :align: center

:doc:`Phalcon\\Tag <../api/Phalcon_Tag>` 提供了很多去创建表单元素的方法。
	
:doc:`Phalcon\\Tag <../api/Phalcon_Tag>` also provides useful methods to build form elements.

Phalcon\Tag::form 方法只接受一个指向控制器/方法的相对uri作为实例参数。

The Phalcon\\Tag::form method receives only one parameter for instance, a relative uri to a controller/action in
the application.

通过单击“Send”按钮，您将注意到框架抛出了一个异常，这表明我们在signup控制器中没有“register”这个动作。我们的 public/index.php 文件抛出了这个异常：

By clicking the "Send" button, you will notice an exception thrown from the framework, indicating that we are missing the "register" action in the controller "signup". Our public/index.php file throws this exception:

    PhalconException: Action "register" was not found on controller "signup"

实现了该方法将移除异常：	

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

如果你再点击“Send”按钮,我们将看到一个空白页。我们输入的用户名和邮件应该已经被存储在数据库中了。根据MVC的规范要求,与数据库交互必须通过数据模型来完成，这样可以确保代码都是以面向对象的方式来实现。	
	
If you click the "Send" button again, you will see a blank page. The name and email input provided by the user should be stored in a database. According to MVC guidelines, database interactions must be done through models so as to ensure clean object-oriented code.

创建模型
^^^^^^^^^^
Phalcon提供了第一个完全用C语言编写的PHP ORM。它简化了开发的复杂性。

Phalcon brings the first ORM for PHP entirely written in C-language. Instead of increasing the complexity of development, it simplifies it.

在创建我们的第一个模型之前，我们需要在Phalcon以外创建一个数据库表，用来存储注册用户的信息，可以这样定义：

Before creating our first model, we need to create a database table outside of Phalcon to map it to. A simple table to store registered users can be defined like this:

.. code-block:: sql

    CREATE TABLE `users` (
      `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
      `name` varchar(70) NOT NULL,
      `email` varchar(70) NOT NULL,
      PRIMARY KEY (`id`)
    );

模型应该位于 app/models 目录 (app/models/Users.php). 这个模型对应“users”表:	
	
A model should be located in the app/models directory (app/models/Users.php). The model maps to the "users" table:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Users extends Model
    {

    }

设置数据库连接
^^^^^^^^^^^^^^^^
为了能够使用一个数据库连接，并通过这个连接让我们的数据模型访问数据，我们需要在引导文件的代码中设置好。数据库连接也是我们在应用中能够在不同组件中调用的一个服务：

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

使用正确的数据库参数设置好后，我们的模型就能在应用中使用了。

With the correct database parameters, our models are ready to work and interact with the rest of the application.

使用模型保存数据
^^^^^^^^^^^^^^^^^
下一个步骤是从表单中接收数据并存储在数据表中。

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


然后我们实例化用户类，它对应于一个用户记录。类的公共属性映射到用户表中的记录的字段。在新记录中设置相应的值并调用save()将在数据库中存储的数据记录。save()方法返回一个布尔值，表示存储的数据是否成功。	
	
We then instantiate the Users class, which corresponds to a User record. The class public properties map to the fields
of the record in the users table. Setting the relevant values in the new record and calling save() will store the data in the database for that record. The save() method returns a boolean value which indicates whether the storing of the data was successful or not.

ORM自动转义输入以防止SQL注入，所以我们只需要将请求数据传递给save()方法。

The ORM automatically escapes the input preventing SQL injections so we only need to pass the request to the save method.

如果字段定义为not null(非空必需)，附加的自动验证会验证字段。如果我们不在注册表单中输入任何必填的数据，我们的屏幕将会看起来像这样：

Additional validation happens automatically on fields that are defined as not null (required). If we don't enter any of the required fields in the sign up form our screen will look like this:

.. figure:: ../_static/img/tutorial-4.png
    :align: center

结束语
--------
这是一个非常简单的教程，正如你所看到的，使用Phalcon很容易开始构建应用程序。Phalcon是一个在你的web服务器上没有干扰、易于开发、特性优良的扩展。我们邀请你继续阅读手册，这样你就可以发现Phalcon提供的附加功能!

This is a very simple tutorial and as you can see, it's easy to start building an application using Phalcon.
The fact that Phalcon is an extension on your web server has not interfered with the ease of development or
features available. We invite you to continue reading the manual so that you can discover additional features offered by Phalcon!

.. _anonymous function: http://php.net/manual/en/functions.anonymous.php

