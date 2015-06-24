使用视图Using Views
========================
视图代表了应用程序中的用户界面. 视图通常是在 HTML 文件里嵌入 PHP 代码，这些代码仅仅是用来展示数据。 视图的任务是当应用程序发生请求时，提供数据给 web 浏览器或者其他工具。

Views represent the user interface of your application. Views are often HTML files with embedded PHP code that perform tasks
related solely to the presentation of the data. Views handle the job of providing data to the web browser or other tool that
is used to make requests from your application.

Phalcon\Mvc\View 和 Phalcon\Mvc\View\Simple 负责管理你的MVC应用程序的视图(View)层。

:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` and :doc:`Phalcon\\Mvc\\View\\Simple <../api/Phalcon_Mvc_View_Simple>`
are responsible for the managing the view layer of your MVC application.

集成视图到控制器Integrating Views with Controllers
--------------------------------------------------------
当某个控制器已经完成了它的周期，Phalcon自动将执行传递到视图组件。视图组件将在视图文件夹中寻找一个文件夹名与最后一个控制器名相同,文件命名与最后一个动作相同的文件执行。例如，如果请求的URL *http://127.0.0.1/blog/posts/show/301*, Phalcon将如下所示的方式按解析URL:

Phalcon automatically passes the execution to the view component as soon as a particular controller has completed its cycle. The view component
will look in the views folder for a folder named as the same name of the last controller executed and then for a file named as the last action
executed. For instance, if a request is made to the URL *http://127.0.0.1/blog/posts/show/301*, Phalcon will parse the URL as follows:

+-------------------+-----------+
| Server Address    | 127.0.0.1 |
+-------------------+-----------+
| Phalcon Directory | blog      |
+-------------------+-----------+
| Controller        | posts     |
+-------------------+-----------+
| Action            | show      |
+-------------------+-----------+
| Parameter         | 301       |
+-------------------+-----------+

调度程序将寻找一个“PostsController”控制器及其“showAction”动作。对于这个示例的一个简单的控制器文件：

The dispatcher will look for a "PostsController" and its action "showAction". A simple controller file for this example:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function indexAction()
        {

        }

        public function showAction($postId)
        {
            // Pass the $postId parameter to the view
            $this->view->setVar("postId", $postId);
        }

    }

setVar允许我们创建视图变量，这样可以在视图模板中使用它们。上面的示例演示了如何传递 $postId 参数到相应的视图模板。	
	
The setVar allows us to create view variables on demand so that they can be used in the view template. The example above demonstrates
how to pass the $postId parameter to the respective view template.

分层渲染Hierarchical Rendering
------------------------------------
:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`支持文件的层次结构，在Phalcon中是默认的视图渲染组件。这个层次结构允许通用的布局点(常用的视图)和以控制器命名的文件夹中定义各自的视图模板
该组件使用默认PHP本身作为模板引擎，因此视图应该以.phtml作为拓展名。如果视图目录是 *app/views* ，视图组件会自动找到这三个视图文件。

:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` supports a hierarchy of files and is the default component for view rendering in Phalcon.
This hierarchy allows for common layout points (commonly used views), as well as controller named folders defining respective view templates.

This component uses by default PHP itself as the template engine, therefore views should have the .phtml extension.
If the views directory is  *app/views* then view component will find automatically for these 3 view files.

+-------------------+-------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Name              | File                          | Description                                                                                                                                                                                                              |
+===================+===============================+==========================================================================================================================================================================================================================+
| Action View       | app/views/posts/show.phtml    | This is the view related to the action. It only will be shown when the "show" action was executed.                                                                                                                       |
+-------------------+-------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Controller Layout | app/views/layouts/posts.phtml | This is the view related to the controller. It only will be shown for every action executed within the controller "posts". All the code implemented in the layout will be reused for all the actions in this controller. |
+-------------------+-------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Main Layout       | app/views/index.phtml         | This is main action it will be shown for every controller or action executed within the application.                                                                                                                     |
+-------------------+-------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

你不需要实现上面提到的所有文件。在文件的层次结构中  :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` 将简单地移动到下一个视图级别。如果这三个视图文件被实现，他们将被按下面方式处理:

You are not required to implement all of the files mentioned above. :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` will simply move to the
next view level in the hierarchy of files. If all three view files are implemented, they will be processed as follows:

.. code-block:: html+php

    <!-- app/views/posts/show.phtml -->

    <h3>This is show view!</h3>

    <p>I have received the parameter <?php echo $postId ?></p>

.. code-block:: html+php

    <!-- app/views/layouts/posts.phtml -->

    <h2>This is the "posts" controller layout!</h2>

    <?php echo $this->getContent() ?>

.. code-block:: html+php

    <!-- app/views/index.phtml -->
    <html>
        <head>
            <title>Example</title>
        </head>
        <body>

            <h1>This is main layout!</h1>

            <?php echo $this->getContent() ?>

        </body>
    </html>

注意方法 *$this->getContent()* 被调用的这行。这种方法指示  :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` 在这里注入前面视图层次结构执行的内容。在上面的示例中，输出将会是：	
	
Note the lines where the method *$this->getContent()* was called. This method instructs :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`
on where to inject the contents of the previous view executed in the hierarchy. For the example above, the output will be:

.. figure:: ../_static/img/views-1.png
   :align: center

请求生成的HTML的将为：   
   
The generated HTML by the request will be:

.. code-block:: html+php

    <!-- app/views/index.phtml -->
    <html>
        <head>
            <title>Example</title>
        </head>
        <body>

            <h1>This is main layout!</h1>

            <!-- app/views/layouts/posts.phtml -->

            <h2>This is the "posts" controller layout!</h2>

            <!-- app/views/posts/show.phtml -->

            <h3>This is show view!</h3>

            <p>I have received the parameter 101</p>

        </body>
    </html>

使用模版 Using Templates
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
模板视图可以用来分享共同的视图代码。他们作为控制器的布局，所以你需要放在布局目录。

Templates are views that can be used to share common view code. They act as controller layouts, so you need to place them in the
layouts directory.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {
        public function initialize()
        {
            $this->view->setTemplateAfter('common');
        }

        public function lastAction()
        {
            $this->flash->notice("These are the latest posts");
        }
    }

.. code-block:: html+php

    <!-- app/views/index.phtml -->
    <!DOCTYPE html>
    <html>
        <head>
            <title>Blog's title</title>
        </head>
        <body>
            <?php echo $this->getContent() ?>
        </body>
    </html>

.. code-block:: html+php

    <!-- app/views/layouts/common.phtml -->

    <ul class="menu">
        <li><a href="/">Home</a></li>
        <li><a href="/articles">Articles</a></li>
        <li><a href="/contact">Contact us</a></li>
    </ul>

    <div class="content"><?php echo $this->getContent() ?></div>

.. code-block:: html+php

    <!-- app/views/layouts/posts.phtml -->

    <h1>Blog Title</h1>

    <?php echo $this->getContent() ?>

.. code-block:: html+php

    <!-- app/views/posts/last.phtml -->

    <article>
        <h2>This is a title</h2>
        <p>This is the post content</p>
    </article>

    <article>
        <h2>This is another title</h2>
        <p>This is another post content</p>
    </article>

最终的输出如下:	
	
The final output will be the following:

.. code-block:: html+php

    <!-- app/views/index.phtml -->
    <!DOCTYPE html>
    <html>
        <head>
            <title>Blog's title</title>
        </head>
        <body>

            <!-- app/views/layouts/common.phtml -->

            <ul class="menu">
                <li><a href="/">Home</a></li>
                <li><a href="/articles">Articles</a></li>
                <li><a href="/contact">Contact us</a></li>
            </ul>

            <div class="content">

                <!-- app/views/layouts/posts.phtml -->

                <h1>Blog Title</h1>

                <!-- app/views/posts/last.phtml -->

                <article>
                    <h2>This is a title</h2>
                    <p>This is the post content</p>
                </article>

                <article>
                    <h2>This is another title</h2>
                    <p>This is another post content</p>
                </article>

            </div>

        </body>
    </html>

控制渲染级别Control Rendering Levels
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如上所述，:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`支持视图分层。你可能需要控制视图组件的渲染级别。方法 Phalcon\Mvc\\View::setRenderLevel()提供这个功能。

As seen above, :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` supports a view hierarchy. You might need to control the level of rendering
produced by the view component. The method Phalcon\Mvc\\View::setRenderLevel() offers this functionality.

这种方法可以从控制器调用或是从上级视图层干涉渲染过程。

This method can be invoked from the controller or from a superior view layer to interfere with the rendering process.

.. code-block:: php

    <?php

    use Phalcon\Mvc\View;
    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function indexAction()
        {

        }

        public function findAction()
        {

            // This is an Ajax response so it doesn't generate any kind of view
            $this->view->setRenderLevel(View::LEVEL_NO_RENDER);

            //...
        }

        public function showAction($postId)
        {
            // Shows only the view related to the action
            $this->view->setRenderLevel(View::LEVEL_ACTION_VIEW);
        }

    }

	
可用的渲染级别:	
	
The available render levels are:

+-----------------------+--------------------------------------------------------------------------+-------+
| Class Constant        | Description                                                              | Order |
+=======================+==========================================================================+=======+
| LEVEL_NO_RENDER       | 表明要避免产生任何形式的显示。  						                   |       |
+-----------------------+--------------------------------------------------------------------------+-------+
| LEVEL_ACTION_VIEW     | 生成显示到视图关联的动作。									           | 1     |
+-----------------------+--------------------------------------------------------------------------+-------+
| LEVEL_BEFORE_TEMPLATE | 生成显示到控制器模板布局之前。                                           | 2     |
+-----------------------+--------------------------------------------------------------------------+-------+
| LEVEL_LAYOUT          | 生成显示到控制器布局。								                   | 3     |
+-----------------------+--------------------------------------------------------------------------+-------+
| LEVEL_AFTER_TEMPLATE  | 生成显示到控制器模板布局后。    										   | 4     |
+-----------------------+--------------------------------------------------------------------------+-------+
| LEVEL_MAIN_LAYOUT     | 生成显示到主布局。文件： views/index.phtml						       | 5     |
+-----------------------+--------------------------------------------------------------------------+-------+

关闭渲染级别Disabling render levels
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
你可以永久或暂时禁用渲染级别。如果不在整个应用程序使用，可以永久禁用一个级别：

You can permanently or temporarily disable render levels. A level could be permanently disabled if it isn't used at all in the whole application:

.. code-block:: php

    <?php

    use Phalcon\Mvc\View;

    $di->set('view', function(){

        $view = new View();

        //Disable several levels
        $view->disableLevel(array(
            View::LEVEL_LAYOUT      => true,
            View::LEVEL_MAIN_LAYOUT => true
        ));

        return $view;

    }, true);

或者在某些应用程序的一部分暂时或禁用:	
	
Or disable temporarily in some part of the application:

.. code-block:: php

    <?php

    use Phalcon\Mvc\View;
    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function indexAction()
        {

        }

        public function findAction()
        {
            $this->view->disableLevel(View::LEVEL_MAIN_LAYOUT);
        }

    }

选择视图Picking Views
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如上所述, 当 :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` 由 :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>`视图渲染的是最后的一个相关的控制器和执行动作。你可以使用 Phalcon\\Mvc\\View::pick() 方法覆盖它。

As mentioned above, when :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` is managed by :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>`
the view rendered is the one related with the last controller and action executed. You could override this by using the Phalcon\\Mvc\\View::pick() method:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class ProductsController extends Controller
    {

        public function listAction()
        {
            // Pick "views-dir/products/search" as view to render
            $this->view->pick("products/search");

            // Pick "views-dir/books/list" as view to render
            $this->view->pick(array('books'));

            // Pick "views-dir/products/search" as view to render
            $this->view->pick(array(1 => 'search'));
        }
    }

关闭视图Disabling the view
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果你的控制器不在视图里产生(或没有)任何输出，你可以禁用视图组件来避免不必要的处理：

If your controller doesn't produce any output in the view (or not even have one) you may disable the view component
avoiding unnecessary processing:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class UsersController extends Controller
    {

        public function closeSessionAction()
        {
            //Close session
            //...

            //An HTTP Redirect
            $this->response->redirect('index/index');

            //Disable the view to avoid rendering
            $this->view->disable();
        }

    }

你可以返回一个“response”的对象，避免手动禁用视图:	
	
You can return a 'response' object to avoid disable the view manually:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class UsersController extends Controller
    {

        public function closeSessionAction()
        {
            //Close session
            //...

            //An HTTP Redirect
            return $this->response->redirect('index/index');
        }

    }

简单渲染Simple Rendering
---------------------------
:doc:`Phalcon\\Mvc\\View\\Simple <../api/Phalcon_Mvc_View_Simple>` 是 :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` 的大多数的设计思想，但缺少文件的层次结构是它们的主要区别。

:doc:`Phalcon\\Mvc\\View\\Simple <../api/Phalcon_Mvc_View_Simple>` is an alternative component to :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`.
It keeps most of the philosophy of :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` but lacks of a hierarchy of files which is, in fact,
the main feature of its counterpart.

该组件允许开发人员控制渲染视图时，视图所在位置。 此外，该组件可以利用从视图中继承的可用的模板引擎。比如 :doc:`Volt <volt>`和其他的一些模板引擎。

This component allows the developer to have control of when a view is rendered and its location.
In addition, this component can leverage of view inheritance available in template engines such
as :doc:`Volt <volt>` and others.

默认使用该的组件必须替换服务容器：

The default component must be replaced in the service container:

.. code-block:: php

    <?php

    use Phalcon\Mvc\View\Simple as SimpleView;

    $di->set('view', function() {

        $view = new SimpleView();

        $view->setViewsDir('../app/views/');

        return $view;

    }, true);

自动渲染必须在 :doc:`Phalcon\Mvc\Application <applications>`被禁用 (如果需要):	
	
Automatic rendering must be disabled in :doc:`Phalcon\\Mvc\\Application <applications>` (if needed):

.. code-block:: php

    <?php

    use Phalcon\Mvc\Application;

    try {

        $application = new Application($di);

        $application->useImplicitView(false);

        echo $application->handle()->getContent();

    } catch (\Exception $e) {
        echo $e->getMessage();
    }

渲染一个视图必须显式地调用render方法来指定你想显示的视图的相对路径：	
	
To render a view it's necessary to call the render method explicitly indicating the relative path to the view you want to display:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends \Controller
    {

        public function indexAction()
        {
            //Render 'views-dir/index.phtml'
            echo $this->view->render('index');

            //Render 'views-dir/posts/show.phtml'
            echo $this->view->render('posts/show');

            //Render 'views-dir/index.phtml' passing variables
            echo $this->view->render('index', array('posts' => Posts::find()));

            //Render 'views-dir/posts/show.phtml' passing variables
            echo $this->view->render('posts/show', array('posts' => Posts::find()));
        }

    }

使用局部模版Using Partials
---------------------------------
局部模板是把渲染过程分解成更简单、更好管理的、可以重用不同部分的应用程序块的另一种方式。你可以移动渲染特定响应的代码块到自己的文件。

Partial templates are another way of breaking the rendering process into simpler more manageable chunks that can be reused by different
parts of the application. With a partial, you can move the code for rendering a particular piece of a response to its own file.

使用局部模板的一种方法是把它们作为相等的子例程：作为一种移动细节视图，这样您的代码可以更容易地被理解。例如，您可能有一个视图看起来像这样：

One way to use partials is to treat them as the equivalent of subroutines: as a way to move details out of a view so that your code
can be more easily understood. For example, you might have a view that looks like this:

.. code-block:: html+php

    <div class="top"><?php $this->partial("shared/ad_banner") ?></div>

    <div class="content">
        <h1>Robots</h1>

        <p>Check out our specials for robots:</p>
        ...
    </div>

    <div class="footer"><?php $this->partial("shared/footer") ?></div>

方法 partial() 也接受一个只存在于局部范围的变量/参数的数组作为第二个参数:	
	
Method partial() does accept a second parameter as an array of variables/parameters that only will exists in the scope of the partial:

.. code-block:: html+php

    <?php $this->partial("shared/ad_banner", array('id' => $site->id, 'size' => 'big')) ?>

控制器传值给视图Transfer values from the controller to views
---------------------------------------------------------------
:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` 可以在每个控制器中使用视图变量 ($this->view)。 你可以在控制器动作中使用视图对象的setVar()方法直接设置视图变量。

:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` is available in each controller using the view variable ($this->view). You can
use that object to set variables directly to the view from a controller action by using the setVar() method.

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
            //Pass all the posts to the views
            $this->view->setVar("posts", Posts::find());

            //Using the magic setter
            $this->view->posts = Posts::find();

            //Passing more than one variable at the same time
            $this->view->setVars(array(
                'title' => $post->title,
                'content' => $post->content
            ));
        }

    }

名为setvar()的第一参数值的变量将在视图中创建的，并且可以被使用。变量可以是任何类型：从一个简单的字符串，整数等等，变为更复杂的结构，如数组，集合。	
	
A variable with the name of the first parameter of setVar() will be created in the view, ready to be used. The variable can be of any type,
from a simple string, integer etc. variable to a more complex structure such as array, collection etc.

.. code-block:: html+php

    <div class="post">
    <?php

      foreach ($posts as $post) {
        echo "<h1>", $post->title, "</h1>";
      }

    ?>
    </div>

在视图中使用模型Using models in the view layer
----------------------------------------------------
应用模型在视图层也是可用的。:doc:`Phalcon\\Loader <../api/Phalcon_Loader>` 将在运行时实例化模型:

Application models are always available at the view layer. The :doc:`Phalcon\\Loader <../api/Phalcon_Loader>` will instantiate them at
runtime automatically:

.. code-block:: html+php

    <div class="categories">
    <?php

        foreach (Categories::find("status = 1") as $category) {
           echo "<span class='category'>", $category->name, "</span>";
        }

    ?>
    </div>

尽管你可以执行模型处理操作，如在视图层 insert() 或 update()，但这是不推荐，因为在一个错误或异常发生时，它不可能将执行流程转发给另一个控制器。	
	
Although you may perform model manipulation operations such as insert() or update() in the view layer, it is not recommended since
it is not possible to forward the execution flow to another controller in the case of an error or an exception.

缓存视图片段Caching View Fragments
----------------------------------------
有时当你开发动态网站和一些区域不会经常更新，请求的输出是完全相同的。:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`提供缓存全部或部分的渲染输出来提高性能。

Sometimes when you develop dynamic websites and some areas of them are not updated very often, the output is exactly
the same between requests. :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` offers caching a part or the whole
rendered output to increase performance.

将 :doc:`Phalcon\\\Mvc\\View <../api/Phalcon_Mvc_View>` 配合 :doc:`Phalcon\\Cache <cache>`  能提供一种更简单的方法缓存输出片段。你可以手动设置缓存处理程序或一个全局处理程序。

:doc:`Phalcon\\\Mvc\\View <../api/Phalcon_Mvc_View>` integrates with :doc:`Phalcon\\Cache <cache>` to provide an easier way
to cache output fragments. You could manually set the cache handler or set a global handler:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function showAction()
        {
            //Cache the view using the default settings
            $this->view->cache(true);
        }

        public function showArticleAction()
        {
            // Cache this view for 1 hour
            $this->view->cache(array(
                "lifetime" => 3600
            ));
        }

        public function resumeAction()
        {
            //Cache this view for 1 day with the key "resume-cache"
            $this->view->cache(
                array(
                    "lifetime" => 86400,
                    "key"      => "resume-cache",
                )
            );
        }

        public function downloadAction()
        {
            //Passing a custom service
            $this->view->cache(
                array(
                    "service"  => "myCache",
                    "lifetime" => 86400,
                    "key"      => "resume-cache",
                )
            );
        }

    }

当我们没有定义缓存的关键组件，这个组件会自动创建一个经过 md5 的当前渲染的视图名。它是定义每个关键动作的一个良好实践，这样你可以很容易地识别与每个视图关联的缓存。	
	
When we do not define a key to the cache, the component automatically creates one using a md5_ hash of the name of the view currently being rendered.
It is a good practice to define a key for each action so you can easily identify the cache associated with each view.

当视图组件需要缓存的东西时，就会请求缓存服务的服务容器。 这个服务的服务名称约定为”viewCache”：

When the View component needs to cache something it will request a cache service from the services container.
The service name convention for this service is "viewCache":

.. code-block:: php

    <?php

    use Phalcon\Cache\Frontend\Output as OutputFrontend;
    use Phalcon\Cache\Backend\Memcache as MemcacheBackend;

    //Set the views cache service
    $di->set('viewCache', function() {

        //Cache data for one day by default
        $frontCache = new OutputFrontend(array(
            "lifetime" => 86400
        ));

        //Memcached connection settings
        $cache = new MemcacheBackend($frontCache, array(
            "host" => "localhost",
            "port" => "11211"
        ));

        return $cache;
    });

.. highlights::

	前端 Phalcon\Cache\Frontend\Output 和服务 ‘viewCache’ 必须在服务容器（DI）注册为总是开放（不是共享 not shared）

    The frontend must always be Phalcon\\Cache\\Frontend\\Output and the service 'viewCache' must be registered as
    always open (not shared) in the services container (DI)

在视图中使用视图缓存也是有用的，以防止控制器执行过程所产生的数据被显示。	
	
When using views, caching can be used to prevent controllers from needing to generate view data on each request.

为了实现这一点，我们必须确定每个缓存键是独一无二的。 首先，我们验证缓存不存在或是否过期，再去计算/查询并在视图中显示数据:

To achieve this we must identify uniquely each cache with a key. First we verify that the cache does not exist or has
expired to make the calculations/queries to display data in the view:

.. code-block:: html+php

    <?php

    use Phalcon\Mvc\Controller;

    class DownloadController extends Controller
    {

        public function indexAction()
        {

            //Check whether the cache with key "downloads" exists or has expired
            if ($this->view->getCache()->exists('downloads')) {

                //Query the latest downloads
                $latest = Downloads::find(array(
                    'order' => 'created_at DESC'
                ));

                $this->view->latest = $latest;
            }

            //Enable the cache with the same key "downloads"
            $this->view->cache(array(
                'key' => 'downloads'
            ));
        }

    }

`PHP alternative site`_ 是实现缓存片段的一个例子。	
	
The `PHP alternative site`_ is an example of implementing the caching of fragments.

模版引擎Template Engines
----------------------------
模板引擎可以帮助设计者不使用复杂的语法创建视图。Phalcon包含一个强大的和快速的模板引擎，它被叫做叫 :doc:`Volt <volt>`。

Template Engines help designers to create views without the use of a complicated syntax. Phalcon includes a powerful and fast templating engine
called :doc:`Volt <volt>`.

此外,  :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` 允许你使用除了简单的PHP或者Volt外其它的模板引擎。

Additionally, :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` allows you to use other template engines instead of plain PHP or Volt.

用不同的模板引擎，通常需要使用外部PHP库并且引入复杂的文本解析来为用户生成最终的输出解析。这通常会增加一些你的应用程序的资源耗费。

Using a different template engine, usually requires complex text parsing using external PHP libraries in order to generate the final output
for the user. This usually increases the number of resources that your application will use.

如果一个外部模板引擎被使用，:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`提供完全相同的视图渲染等级，仍然可以尝试在这些模板内访问的更多的API。

If an external template engine is used, :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` provides exactly the same view hierarchy and it's
still possible to access the API inside these templates with a little more effort.

该组件使用的适配器，这些适配器帮助 Phalcon 与外部模板引擎以一个统一的方式对话，让我们看看如何整合。

This component uses adapters, these help Phalcon to speak with those external template engines in a unified way, let's see how to do that integration.

创建模版引擎Creating your own Template Engine Adapter
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
有很多模板引擎，你可能想整合或建立一个自己的。开始使用一个外部的模板引擎的第一步是创建一个适配器。

There are many template engines, which you might want to integrate or create one of your own. The first step to start using an external template engine is create an adapter for it.

模板引擎的适配器是一个类，作为 :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`和模板引擎本身之间的桥梁。 通常它只需要实现两个方法: __construct() and render()。首先接收 :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`和应用程序使用的DI容器来创建引擎适配器实例。

A template engine adapter is a class that acts as bridge between :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` and the template engine itself.
Usually it only needs two methods implemented: __construct() and render(). The first one receives the :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`
instance that creates the engine adapter and the DI container used by the application.

方法render()接受一个到视图文件的绝对路径和视图参数，设置使用$this->view->setVar()。必要的时候，你可以读入或引入它。

The method render() accepts an absolute path to the view file and the view parameters set using $this->view->setVar(). You could read or require it
when it's necessary.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Engine;

    class MyTemplateAdapter extends Engine
    {

        /**
         * Adapter constructor
         *
         * @param \Phalcon\Mvc\View $view
         * @param \Phalcon\DI $di
         */
        public function __construct($view, $di)
        {
            //Initialize here the adapter
            parent::__construct($view, $di);
        }

        /**
         * Renders a view using the template engine
         *
         * @param string $path
         * @param array $params
         */
        public function render($path, $params)
        {

            // Access view
            $view    = $this->_view;

            // Access options
            $options = $this->_options;

            //Render the view
            //...
        }

    }

替换模版引擎Changing the Template Engine
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
你可以想下面一样从控制器更换或者添加更多的模板引擎：

You can replace or add more a template engine from the controller as follows:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function indexAction()
        {
            // Set the engine
            $this->view->registerEngines(
                array(
                    ".my-html" => "MyTemplateAdapter"
                )
            );
        }

        public function showAction()
        {
            // Using more than one template engine
            $this->view->registerEngines(
                array(
                    ".my-html" => 'MyTemplateAdapter',
                    ".phtml"   => 'Phalcon\Mvc\View\Engine\Php'
                )
            );
        }

    }

你可以完全更换模板引擎或同时使用多个模板引擎。方法 \Phalcon\\Mvc\\View::registerEngines()接受一个包含定义模板引擎数据的数组。每个引擎的键名是一个区别于其他引擎的拓展名。模板文件和特定的引擎关联必须有这些扩展名。	
	
You can replace the template engine completely or use more than one template engine at the same time. The method \Phalcon\\Mvc\\View::registerEngines()
accepts an array containing data that define the template engines. The key of each engine is an extension that aids in distinguishing one from another.
Template files related to the particular engine must have those extensions.

\Phalcon\\Mvc\\View::registerEngines() 会按照相关顺序定义模板引擎执行。如果:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`发现具有相同名称但不同的扩展，它只会使第一个。

The order that the template engines are defined with \Phalcon\\Mvc\\View::registerEngines() defines the relevance of execution. If
:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` finds two views with the same name but different extensions, it will only render the first one.

如果你想在应用程序的每个请求中注册一个或一组模板引擎。你可以在创建视图时注册服务：

If you want to register a template engine or a set of them for each request in the application. You could register it when the view service is created:

.. code-block:: php

    <?

    use Phalcon\Mvc\View;

    //Setting up the view component
    $di->set('view', function() {

        $view = new View();

        //A trailing directory separator is required
        $view->setViewsDir('../app/views/');

        $view->registerEngines(array(
            ".my-html" => 'MyTemplateAdapter'
        ));

        return $view;

    }, true);

在 `Phalcon Incubator <https://github.com/phalcon/incubator/tree/master/Library/Phalcon/Mvc/View/Engine>`_ 有一些适配器可用于数个模板引擎	
	
There are adapters available for several template engines on the `Phalcon Incubator <https://github.com/phalcon/incubator/tree/master/Library/Phalcon/Mvc/View/Engine>`_

注入服务到视图Injecting services in View
-----------------------------------------
每个视图执行内部包含一个 :doc:`Phalcon\\DI\\Injectable <../api/Phalcon_DI_Injectable>` 实例, 提供方便地方式访问应用程序的服务容器。

Every view executed is included inside a :doc:`Phalcon\\DI\\Injectable <../api/Phalcon_DI_Injectable>` instance, providing easy access
to the application's service container.

下面的示例演示如何用一个框架约定好的URL服务写一个 jQuery `ajax request`_ 。 “url” (通常是 :doc:`Phalcon\\Mvc\\Url <url>`) 服务被注入在视图由相同名称的属性访问：

The following example shows how to write a jQuery `ajax request`_ using a url with the framework conventions.
The service "url" (usually :doc:`Phalcon\\Mvc\\Url <url>`) is injected in the view by accessing a property with the same name:

.. code-block:: html+php

    <script type="text/javascript">

    $.ajax({
        url: "<?php echo $this->url->get("cities/get") ?>"
    })
    .done(function() {
        alert("Done!");
    });

    </script>

独立的组件Stand-Alone Component
----------------------------------
在Phalcon的所有部件都可以作为胶水 *glue* 组件单独使用，因为它们彼此松散耦合:

All the components in Phalcon can be used as *glue* components individually because they are loosely coupled to each other:

分层渲染Hierarchical Rendering
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如下所示，可以单独使用 :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`：

Using :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` in a stand-alone mode can be demonstrated below

.. code-block:: php

    <?php

    use Phalcon\Mvc\View;

    $view = new View();

    //A trailing directory separator is required
    $view->setViewsDir("../app/views/");

    // Passing variables to the views, these will be created as local variables
    $view->setVar("someProducts", $products);
    $view->setVar("someFeatureEnabled", true);

    //Start the output buffering
    $view->start();

    //Render all the view hierarchy related to the view products/list.phtml
    $view->render("products", "list");

    //Finish the output buffering
    $view->finish();

    echo $view->getContent();

使用短的语法也可以:	
	
A short syntax is also available:

.. code-block:: php

    <?php

    use Phalcon\Mvc\View;

    $view = new View();

    echo $view->getRender('products', 'list',
        array(
            "someProducts"       => $products,
            "someFeatureEnabled" => true
        ),
        function($view) {
            //Set any extra options here
            $view->setViewsDir("../app/views/");
            $view->setRenderLevel(View::LEVEL_LAYOUT);
        }
    );

简单渲染Simple Rendering
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如下所示，以单独使用 :doc:`Phalcon\\Mvc\\View\\Simple <../api/Phalcon_Mvc_View_Simple>`：

Using :doc:`Phalcon\\Mvc\\View\\Simple <../api/Phalcon_Mvc_View_Simple>` in a stand-alone mode can be demonstrated below:

.. code-block:: php

    <?php

    use Phalcon\Mvc\View\Simple as SimpleView;

    $view = new SimpleView();

    //A trailing directory separator is required
    $view->setViewsDir("../app/views/");

    // Render a view and return its contents as a string
    echo $view->render("templates/welcomeMail");

    // Render a view passing parameters
    echo $view->render("templates/welcomeMail", array(
        'email'   => $email,
        'content' => $content
    ));

视图事件View Events
------------------------
如果事件管理器（EventsManager）存在，:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` 和 :doc:`Phalcon\\Mvc\\View\\Simple <../api/Phalcon_Mvc_View_Simple>`  能够发送事件到 EventsManager。事件触发使用的“view”类型。当返回布尔值false，一些事件可以停止运行。以下是被支持的事件：

:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` and :doc:`Phalcon\\Mvc\\View\\Simple <../api/Phalcon_Mvc_View_Simple>` are able to send
events to an :doc:`EventsManager <events>` if it is present. Events are triggered using the type "view". Some events when returning
boolean false could stop the active operation. The following events are supported:

+----------------------+------------------------------------------------------------+---------------------+
| Event Name           | Triggered                                                  | Can stop operation? |
+======================+============================================================+=====================+
| beforeRender         | Triggered before starting the render process               | Yes                 |
+----------------------+------------------------------------------------------------+---------------------+
| beforeRenderView     | Triggered before rendering an existing view                | Yes                 |
+----------------------+------------------------------------------------------------+---------------------+
| afterRenderView      | Triggered after rendering an existing view                 | No                  |
+----------------------+------------------------------------------------------------+---------------------+
| afterRender          | Triggered after completing the render process              | No                  |
+----------------------+------------------------------------------------------------+---------------------+
| notFoundView         | Triggered when a view was not found                        | No                  |
+----------------------+------------------------------------------------------------+---------------------+

下面的例子演示了如何将监听器附加到该组件：

The following example demonstrates how to attach listeners to this component:

.. code-block:: php

    <?php

    use Phalcon\Mvc\View;
    use Phalcon\Events\Manager as EventsManager;

    $di->set('view', function() {

        //Create an events manager
        $eventsManager = new EventsManager();

        //Attach a listener for type "view"
        $eventsManager->attach("view", function($event, $view) {
            echo $event->getType(), ' - ', $view->getActiveRenderPath(), PHP_EOL;
        });

        $view = new View();
        $view->setViewsDir("../app/views/");

        //Bind the eventsManager to the view component
        $view->setEventsManager($eventsManager);

        return $view;

    }, true);

下面的示例演示如何创建一个插件  Tidy_ ，清理/修复的渲染过程中产生的HTML：	
	
The following example shows how to create a plugin that clean/repair the HTML produced by the render process using Tidy_:

.. code-block:: php

    <?php

    class TidyPlugin
    {

        public function afterRender($event, $view)
        {

            $tidyConfig = array(
                'clean'          => true,
                'output-xhtml'   => true,
                'show-body-only' => true,
                'wrap'           => 0,
            );

            $tidy = tidy_parse_string($view->getContent(), $tidyConfig, 'UTF8');
            $tidy->cleanRepair();

            $view->setContent((string) $tidy);
        }

    }

    //Attach the plugin as a listener
    $eventsManager->attach("view:afterRender", new TidyPlugin());

.. _this Github repository: https://github.com/bobthecow/mustache.php
.. _ajax request: http://api.jquery.com/jQuery.ajax/
.. _Tidy: http://www.php.net/manual/en/book.tidy.php
.. _md5: http://php.net/manual/en/function.md5.php
.. _PHP alternative site: https://github.com/phalcon/php-site
