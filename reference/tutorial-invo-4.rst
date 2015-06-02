教程5：自定义 INVO
============================
为了详细解释INVO，我们将解释如何定制INVO添加UI元素并根据控制器执行和修改标题。

To finish the detailed explanation of INVO we are going to explain how to customize INVO adding UI elements
and changing the title according to the controller executed.

用户组件
---------------
所有应用程序的UI元素和视觉风格大部分是通过 `Bootstrap`_ 实现了。一些元素,比如导航栏根据应用程序的状态变化。例如,在右上角的“登录/注册”链接在一个用户登录到应用程序后变为“注销”。

All the UI elements and visual style of the application has been achieved mostly through `Bootstrap`_.
Some elements, such as the navigation bar changes according to the state of the application. For example, in the
upper right corner, the link "Log in / Sign Up" changes to "Log out" if an user is logged into the application.

应用中这部分功能在"Elements" (app/library/Elements.php)中实现。

This part of the application is implemented in the component "Elements" (app/library/Elements.php).

.. code-block:: php

    <?php

    use Phalcon\Mvc\User\Component;

    class Elements extends Component
    {

        public function getMenu()
        {
            //...
        }

        public function getTabs()
        {
            //...
        }

    }

这个类扩展了Phalcon\\Mvc\\User\\Component。它不但强加扩展了这个组件,而且有助于更快地获得应用程序服务。现在,我们要在服务容器中注册我们的第一个用户组件:	
	
This class extends the Phalcon\\Mvc\\User\\Component. It is not imposed to extend a component with this class, but
it helps to get access more quickly to the application services. Now, we are going to register
our first user component in the services container:

.. code-block:: php

    <?php

    //Register an user component
    $di->set('elements', function(){
        return new Elements();
    });

作为控制器，视图内插件或组件，该组件可以访问容器中的一个属性来达到访问服务的目的：
	
As controllers, plugins or components within a view, this component also has access to the services registered
in the container and by just accessing an attribute with the same name as a previously registered service:

.. code-block:: html+php

    <div class="navbar navbar-fixed-top">
        <div class="navbar-inner">
            <div class="container">
                <a class="btn btn-navbar" data-toggle="collapse" data-target=".nav-collapse">
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </a>
                <a class="brand" href="#">INVO</a>
                {{ elements.getMenu() }}
            </div>
        </div>
    </div>

    <div class="container">
        {{ content() }}
        <hr>
        <footer>
            <p>&copy; Company 2014</p>
        </footer>
    </div>

重要的部分是：	
	
The important part is:

.. code-block:: html+php

    {{ elements.getMenu() }}


动态改变标题
----------------
当我们浏览的时候会发现标题动态的变化表明我们所在的页面。这主要通过每个控制器中初始化实现:

When you browse between one option and another will see that the title changes dynamically indicating where
we are currently working. This is achieved in each controller initializer:

.. code-block:: php

    <?php

    class ProductsController extends ControllerBase
    {

        public function initialize()
        {
            //Set the document title
            $this->tag->setTitle('Manage your product types');
            parent::initialize();
        }

        //...

    }

注意, parent::initialize()也被调用了，这将会添加更多的数据到标题中:	
	
Note, that the method parent::initialize() is also called, it adds more data to the title:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class ControllerBase extends Controller
    {

        protected function initialize()
        {
            //Prepend the application name to the title
            $this->tag->prependTitle('INVO | ');
        }

        //...
    }

最后标题被输出到主视图上(app/views/index.phtml)：
	
Finally, the title is printed in the main view (app/views/index.phtml):

.. code-block:: html+php

    <!DOCTYPE html>
    <html>
        <head>
            <?php echo $this->tag->getTitle() ?>
        </head>
        <!-- ... -->
    </html>

.. _Bootstrap: http://getbootstrap.com/
