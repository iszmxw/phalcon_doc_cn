Cookie 管理Cookies Management
==============================
Cookies 是一个将数据存储在客户端的有效途径，这样即使用户关闭了TA的浏览器也能获取这些数据。  :doc:`Phalcon\\Http\\Response\\Cookies <../api/Phalcon_Http_Response_Cookies>`作为全局的cookies包。 在请求执行的期间，Cookies存放于这个包里，并且在请求结束时会自动发送回给客户端。

Cookies_ are very useful way to store small pieces of data in the client that can be retrieved even
if the user closes his/her browser. :doc:`Phalcon\\Http\\Response\\Cookies <../api/Phalcon_Http_Response_Cookies>`
acts as a global bag for cookies. Cookies are stored in this bag during the request execution and are sent
automatically at the end of the request.

基本使用Basic Usage
--------------------------
你可以在应用中任何可以访问服务的部分，通用使用“cookies”服务来设置/获取cookie 。

You can set/get cookies by just accessing the 'cookies' service in any part of the application where services can be
accessed:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class SessionController extends Controller
    {
        public function loginAction()
        {
            //Check if the cookie has previously set
            if ($this->cookies->has('remember-me')) {

                //Get the cookie
                $rememberMe = $this->cookies->get('remember-me');

                //Get the cookie's value
                $value      = $rememberMe->getValue();

            }
        }

        public function startAction()
        {
            $this->cookies->set('remember-me', 'some value', time() + 15 * 86400);
        }
    }

Cookie 的加密和解密Encryption/Decryption of Cookies
-----------------------------------------------------------
默认情况下，cookie会在返回给客户端前自动加密并且在接收到后自动解密。 在保护机制下，即使未验证的用户在客户端（浏览器）查看了cookie的内容，也无妨。 即使这样，敏感的数据还是不应该存放到cookie。

By default, cookies are automatically encrypted before be sent to the client and decrypted when retrieved.
This protection allow unauthorized users to see the cookies' contents in the client (browser).
Although this protection, sensitive data should not be stored on cookies.

你可以通过以下方式禁用加密：

You can disable encryption in the following way:

.. code-block:: php

    <?php

    use Phalcon\Http\Response\Cookies;

    $di->set('cookies', function() {
        $cookies = new Cookies();
        $cookies->useEncryption(false);
        return $cookies;
    });

使用加密的话，必须在“crypt”服务中设置一个全局的key：	
	
In case of using encryption a global key must be set in the 'crypt' service:

.. code-block:: php

    <?php

    use Phalcon\Crypt;

    $di->set('crypt', function() {
        $crypt = new Crypt();
        $crypt->setKey('#1dj8$=dp?.ak//j1V$'); //Use your own key!
        return $crypt;
    });

.. highlights::

    将未加密且包含了复杂对象结构、结果集、服务信息等等的cookie数据发送给客户端， 可能会暴露应用内部的细节给外界，从而被黑客利用、发起攻击。 如果你不想使用加密，我们强烈建议你只返回基本的cookie数据，如数字或者小串的文字。

    Send cookies data without encryption to clients including complex objects structures, resultsets,
    service information, etc. could expose internal application details that could be used by an attacker
    to attack the application. If you do not want to use encryption, we highly recommend you only send very
    basic cookie data like numbers or small string literals.

.. _Cookies : http://en.wikipedia.org/wiki/HTTP_cookie
