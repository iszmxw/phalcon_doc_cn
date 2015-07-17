过滤与清理Filtering and Sanitizing
=====================================
清理用户输入是软件开发中很重要的一个环节。信任或者忽略对用户输入数据作清理可能会导致 对应用内容（主要是用户数据），甚至你应用所处在的服务器的非法访问。

Sanitizing user input is a critical part of software development. Trusting or neglecting to sanitize user input could lead to unauthorized
access to the content of your application, mainly user data, or even the server your application is hosted on.

.. figure:: ../_static/img/sql.png
   :align: center

`Full image (from xkcd)`_

此:doc:`Phalcon\\Filter <../api/Phalcon_Filter>` 组件提供了一系列通用可用的过滤器和数据清理助手。它提供了围绕于PHP过滤扩展的面向对象包装。

The :doc:`Phalcon\\Filter <../api/Phalcon_Filter>` component provides a set of commonly used filters and data sanitizing helpers. It provides object-oriented wrappers around the PHP filter extension.

清理数据Sanitizing data
----------------------------
清理是指从一个值中移除特定字符的过程，此过程对用户和应用不是必须，也不是他们想得到的。 通过清理输入，我们确保了应用的完整性和正确性。

Sanitizing is the process which removes specific characters from a value, that are not required or desired by the user or application.
By sanitizing input we ensure that application integrity will be intact.

.. code-block:: php

    <?php

    use Phalcon\Filter;

    $filter = new Filter();

    // returns "someone@example.com"
    $filter->sanitize("some(one)@exa\mple.com", "email");

    // returns "hello"
    $filter->sanitize("hello<<", "string");

    // returns "100019"
    $filter->sanitize("!100a019", "int");

    // returns "100019.01"
    $filter->sanitize("!100a019.01a", "float");


在控制器中使用清理Sanitizing from Controllers
----------------------------------------------------
当接收到GET或POST的数据时（通过请求对象），你可以在控制器中访问一个 :doc:`Phalcon\\Filter <../api/Phalcon_Filter>` 对象。 第一个参数是等待获得变量的名字，第二个参数是将应用在此变量的过滤器。

You can access a :doc:`Phalcon\\Filter <../api/Phalcon_Filter>` object from your controllers when accessing GET or POST input data
(through the request object). The first parameter is the name of the variable to be obtained; the second is the filter to be applied on it.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class ProductsController extends Controller
    {

        public function indexAction()
        {

        }

        public function saveAction()
        {

            // Sanitizing price from input
            $price = $this->request->getPost("price", "double");

            // Sanitizing email from input
            $email = $this->request->getPost("customerEmail", "email");

        }

    }

过滤动作参数Filtering Action Parameters
------------------------------------------
接下来的示例演示了在一个控制器的动作中如何清理动作的参数：

The next example shows you how to sanitize the action parameters within a controller action:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class ProductsController extends Controller
    {

        public function indexAction()
        {

        }

        public function showAction($productId)
        {
            $productId = $this->filter->sanitize($productId, "int");
        }

    }

过滤数据Filtering data
---------------------------
此外， :doc:`Phalcon\\Filter <../api/Phalcon_Filter>` 也提供了可以进行删除或者修改输入数据以满足我们需要的格式的过滤器。

In addition to sanitizing, :doc:`Phalcon\\Filter <../api/Phalcon_Filter>` also provides filtering by removing or modifying input data to
the format we expect.

.. code-block:: php

    <?php

    use Phalcon\Filter;

    $filter = new Filter();

    // returns "Hello"
    $filter->sanitize("<h1>Hello</h1>", "striptags");

    // returns "Hello"
    $filter->sanitize("  Hello   ", "trim");


内置过滤器类型Types of Built-in Filters
--------------------------------------------
以下是该容器提供的内置过滤器：

The following are the built-in filters provided by this component:

+-----------+---------------------------------------------------------------------------+
| Name      | Description                                                               |
+===========+===========================================================================+
| string    | Strip tags and escapes HTML entities, including single and double quotes. |
+-----------+---------------------------------------------------------------------------+
| email     | Remove all characters except letters, digits and !#$%&*+-/=?^_`{\|}~@.[]. |
+-----------+---------------------------------------------------------------------------+
| int       | Remove all characters except digits, plus and minus sign.                 |
+-----------+---------------------------------------------------------------------------+
| float     | Remove all characters except digits, dot, plus and minus sign.            |
+-----------+---------------------------------------------------------------------------+
| alphanum  | Remove all characters except [a-zA-Z0-9]                                  |
+-----------+---------------------------------------------------------------------------+
| striptags | Applies the strip_tags_ function                                          |
+-----------+---------------------------------------------------------------------------+
| trim      | Applies the trim_ function                                                |
+-----------+---------------------------------------------------------------------------+
| lower     | Applies the strtolower_ function                                          |
+-----------+---------------------------------------------------------------------------+
| upper     | Applies the strtoupper_ function                                          |
+-----------+---------------------------------------------------------------------------+

创建过滤器Creating your own Filters
---------------------------------------
你可以将你自己的过滤器添加到  :doc:`Phalcon\\Filter <../api/Phalcon_Filter>` 。过滤器的方法可以是匿名函数：

You can add your own filters to :doc:`Phalcon\\Filter <../api/Phalcon_Filter>`. The filter function could be an anonymous function:

.. code-block:: php

    <?php

    use Phalcon\Filter;

    $filter = new Filter();

    //Using an anonymous function
    $filter->add('md5', function($value) {
        return preg_replace('/[^0-9a-f]/', '', $value);
    });

    //Sanitize with the "md5" filter
    $filtered = $filter->sanitize($possibleMd5, "md5");

或者，如果你愿意，你可以在类中实现过滤器：	
	
Or, if you prefer, you can implement the filter in a class:

.. code-block:: php

    <?php

    use Phalcon\Filter;

    class IPv4Filter
    {

        public function filter($value)
        {
            return filter_var($value, FILTER_VALIDATE_IP, FILTER_FLAG_IPV4);
        }

    }

    $filter = new Filter();

    //Using an object
    $filter->add('ipv4', new IPv4Filter());

    //Sanitize with the "ipv4" filter
    $filteredIp = $filter->sanitize("127.0.0.1", "ipv4");

复杂的过滤与清理Complex Sanitizing and Filtering
-------------------------------------------------
你可以使用PHP本身提供的优秀过滤器扩展。请查看对应的文档：  `Data Filtering at PHP Documentation`_

PHP itself provides an excellent filter extension you can use. Check out its documentation: `Data Filtering at PHP Documentation`_

自定义过滤器Implementing your own Filter
------------------------------------------
如需创建你自己的过滤器并代替Phalcon提供的过滤器，你需要实现 :doc:`Phalcon\\FilterInterface <../api/Phalcon_FilterInterface>` 接口。

The :doc:`Phalcon\\FilterInterface <../api/Phalcon_FilterInterface>` interface must be implemented to create your own filtering service
replacing the one provided by Phalcon.

.. _Full image (from xkcd): http://xkcd.com/327/
.. _Data Filtering at PHP Documentation: http://www.php.net/manual/en/book.filter.php
.. _strip_tags: http://www.php.net/manual/en/function.strip-tags.php
.. _trim: http://www.php.net/manual/en/function.trim.php
.. _strtolower: http://www.php.net/manual/en/function.strtolower.php
.. _strtoupper: http://www.php.net/manual/en/function.strtoupper.php
