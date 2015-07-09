HTTP 请求环境Request Environment
========================================
每个HTTP请求（通常有浏览器发出）都包含了如头信息，文件，变量等。基于web的应用程序需要去解析这些请求并返回正确信息。:doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>`封装了请求的信息，让我们可以以面向对象的方式去访问这些请求。

Every HTTP request (usually originated by a browser) contains additional information regarding the request such as header data,
files, variables, etc. A web based application needs to parse that information so as to provide the correct
response back to the requester. :doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>` encapsulates the
information of the request, allowing you to access it in an object-oriented way.

.. code-block:: php

    <?php

    use Phalcon\Http\Request;

    // Getting a request instance
    $request = new Request();

    // Check whether the request was made with method POST
    if ($request->isPost() == true) {

        // Check whether the request was made with Ajax
        if ($request->isAjax() == true) {
            echo "Request was made using POST and AJAX";
        }
    }

获取值Getting Values
------------------------------
PHP根据请求类型自动将全局变量填充到$_GET 和 $_POST变量中。这些变量包含了URL请求或者表单请求的参数信息。 这些信息可能会包含恶意的字符或代码从而引起`SQL injection`_ SQL注入或者是`Cross Site Scripting (XSS)`_ 跨站攻击。

PHP automatically fills the superglobal arrays $_GET and $_POST depending on the type of the request. These arrays
contain the values present in forms submitted or the parameters sent via the URL. The variables in the arrays are
never sanitized and can contain illegal characters or even malicious code, which can lead to `SQL injection`_ or
`Cross Site Scripting (XSS)`_ attacks.

通过:doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>`可以访问储存在$_REQUEST,$_GET 和 $_POST数据中的变量，并使用filter服务（默认是:doc:`Phalcon\\Filter <filter>`）过滤清理变量，如下所示： 

:doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>` allows you to access the values stored in the $_REQUEST,
$_GET and $_POST arrays and sanitize or filter them with the 'filter' service, (by default
:doc:`Phalcon\\Filter <filter>`). The following examples offer the same behavior:

.. code-block:: php

    <?php

    use Phalcon\Filter;

    // Manually applying the filter
    $filter = new Filter();
    $email  = $filter->sanitize($_POST["user_email"], "email");

    // Manually applying the filter to the value
    $filter = new Filter();
    $email  = $filter->sanitize($request->getPost("user_email"), "email");

    // Automatically applying the filter
    $email = $request->getPost("user_email", "email");

    // Setting a default value if the param is null
    $email = $request->getPost("user_email", "email", "some@example.com");

    // Setting a default value if the param is null without filtering
    $email = $request->getPost("user_email", null, "some@example.com");


控制器中访问请求Accessing the Request from Controllers
----------------------------------------------------------
最常见的是在控制器中访问请求参数。从控制器访问:doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>` 对象需要使用控制器的$this->request开放属性：

The most common place to access the request environment is in an action of a controller. To access the
:doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>` object from a controller you will need to use
the $this->request public property of the controller:

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

            // Check if request has made with POST
            if ($this->request->isPost() == true) {

                // Access POST data
                $customerName = $this->request->getPost("name");
                $customerBorn = $this->request->getPost("born");

            }

        }

    }

文件上传Uploading Files
---------------------------
另一个常见任务是文件上传。:doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>`提供了面向对象方式去完成文件上传：

Another common task is file uploading. :doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>` offers
an object-oriented way to achieve this task:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {

        public function uploadAction()
        {
            // Check if the user has uploaded files
            if ($this->request->hasFiles() == true) {

                // Print the real file names and sizes
                foreach ($this->request->getUploadedFiles() as $file) {

                    //Print file details
                    echo $file->getName(), " ", $file->getSize(), "\n";

                    //Move the file into the application
                    $file->moveTo('files/' . $file->getName());
                }
            }
        }

    }

由Phalcon\\Http\\Request::getUploadedFiles() 返回的每个对象都是	:doc:`Phalcon\\Http\\Request\\File <../api/Phalcon_Http_Request_File>`类的实例。用$_FILES全局变量同样可以。 :doc:`Phalcon\\Http\\Request\\File <../api/Phalcon_Http_Request_File>` 只是封装了请求中的文件的相关信息。
	
Each object returned by Phalcon\\Http\\Request::getUploadedFiles() is an instance of the
:doc:`Phalcon\\Http\\Request\\File <../api/Phalcon_Http_Request_File>` class. Using the $_FILES superglobal
array offers the same behavior. :doc:`Phalcon\\Http\\Request\\File <../api/Phalcon_Http_Request_File>` encapsulates
only the information related to each file uploaded with the request.

使用头信息Working with Headers
----------------------------------
就像上面说的那样，请求的头信息中包含了必要的信息决定了我们该给用户返回哪些信息。下面代码说明如何使用：

As mentioned above, request headers contain useful information that allow us to send the proper response back to
the user. The following examples show usages of that information:

.. code-block:: php

    <?php

    // get the Http-X-Requested-With header
    $requestedWith = $request->getHeader("HTTP_X_REQUESTED_WITH");
    if ($requestedWith == "XMLHttpRequest") {
        echo "The request was made with Ajax";
    }

    // Same as above
    if ($request->isAjax()) {
        echo "The request was made with Ajax";
    }

    // Check the request layer
    if ($request->isSecureRequest() == true) {
        echo "The request was made using a secure layer";
    }

    // Get the servers's ip address. ie. 192.168.0.100
    $ipAddress   = $request->getServerAddress();

    // Get the client's ip address ie. 201.245.53.51
    $ipAddress   = $request->getClientAddress();

    // Get the User Agent (HTTP_USER_AGENT)
    $userAgent   = $request->getUserAgent();

    // Get the best acceptable content by the browser. ie text/xml
    $contentType = $request->getAcceptableContent();

    // Get the best charset accepted by the browser. ie. utf-8
    $charset     = $request->getBestCharset();

    // Get the best language accepted configured in the browser. ie. en-us
    $language    = $request->getBestLanguage();


.. _SQL injection: http://en.wikipedia.org/wiki/SQL_injection
.. _Cross Site Scripting (XSS): http://en.wikipedia.org/wiki/Cross-site_scripting
