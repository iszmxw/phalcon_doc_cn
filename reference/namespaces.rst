使用命名空间Working with Namespaces
========================================
Namespaces 可以用来避免类名的冲突，比如如果在一个应用中有两个控制器使用同样的名称，那么可以用namespace来区分他们。 另外命名空间在创建组件或者模块的时候也是非常有用的。

Namespaces_ can be used to avoid class name collisions; this means that if you have two controllers in an application with the same name,
a namespace can be used to differentiate them. Namespaces are also useful for creating bundles or modules.

设置框架Setting up the framework
----------------------------------------
使用名称空间对是否能够加载正确的控制器有影响。调整框架以便于使用命名空间需要执行下面一个或所有的任务:

Using namespaces has some implications when loading the appropriate controller. To adjust the framework behavior to namespaces is necessary
to perform one or all of the following tasks:

使用自动加载去配置命名空间，例如 Phalcon\\Loader:

Use an autoload strategy that takes into account the namespaces, for example with Phalcon\\Loader:

.. code-block:: php

    <?php

    $loader->registerNamespaces(
        array(
           'Store\Admin\Controllers' => "../bundles/admin/controllers/",
           'Store\Admin\Models'      => "../bundles/admin/models/",
        )
    );

在路由路径中作为一个独立的参数定义：	
	
Specify it in the routes as a separate parameter in the route's paths:

.. code-block:: php

    <?php

    $router->add(
        '/admin/users/my-profile',
        array(
            'namespace'  => 'Store\Admin',
            'controller' => 'Users',
            'action'     => 'profile',
        )
    );

作为路由的一部分传递：	
	
Passing it as part of the route:

.. code-block:: php

    <?php

    $router->add(
        '/:namespace/admin/users/my-profile',
        array(
            'namespace'  => 1,
            'controller' => 'Users',
            'action'     => 'profile',
        )
    );

如果你在应用程序中每一个控制器使用相同的名称空间,那么可以分配器中定义一个默认名称空间,通过这样做,不需要在路由器的路径指定一个完整的类名:	
	
If you are only working with the same namespace for every controller in your application, then you can define a default namespace
in the Dispatcher, by doing this, you don't need to specify a full class name in the router path:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Dispatcher;

    //Registering a dispatcher
    $di->set('dispatcher', function() {
        $dispatcher = new Dispatcher();
        $dispatcher->setDefaultNamespace('Store\Admin\Controllers');
        return $dispatcher;
    });

控制器加入命名空间Controllers in Namespaces
-----------------------------------------------
下面的例子演示了使用命名空间实现一个控制器：

The following example shows how to implement a controller that use namespaces:

.. code-block:: php

    <?php

    namespace Store\Admin\Controllers;

    use Phalcon\Mvc\Controller;

    class UsersController extends Controller
    {

        public function indexAction()
        {

        }

        public function profileAction()
        {

        }

    }

模型加入命名空间Models in Namespaces
-----------------------------------------
下面的例子演示了使用命名空间实现一个数据模型：

Take the following into consideration when using models in namespaces:

.. code-block:: php

    <?php

    namespace Store\Models;

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

    }

如果模型有关系，则也需要包含命名空间：	
	
If models have relationships they must include the namespace too:

.. code-block:: php

    <?php

    namespace Store\Models;

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {
        public function initialize()
        {
            $this->hasMany('id', 'Store\Models\Parts', 'robots_id', array(
                'alias' => 'parts'
            ));
        }
    }

在PHQL语句中也必须包含命名空间语句：	
	
In PHQL you must write the statements including namespaces:

.. code-block:: php

    <?php

    $phql = 'SELECT r.* FROM Store\Models\Robots r JOIN Store\Models\Parts p';

.. _Namespaces: http://php.net/manual/en/language.namespaces.php
