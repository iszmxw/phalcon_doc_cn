安全Security
=====================
该组件可以帮助开发人员在共同安全方面的一些工作，比如密码散列和跨站点请求伪造(CSRF)保护。

This component aids the developer in common security tasks such as password hashing and Cross-Site Request Forgery protection (CSRF).

密码散列Password Hashing
-------------------------------
将密码以纯文本方式保存是一个糟糕的安全实践。任何可以访问数据库的人可立即访问所有用户账户，因此能够从事未经授权的活动。为解决这个问题，许多应用程序使用熟悉的 单向散列方法比如“md5_”和“sha1_”。然而，硬件每天都在发展，变得更快，这些算法在蛮力攻击面前变得脆弱不堪。这些攻击也被称为“彩虹表（`rainbow tables`_）”。

Storing passwords in plain text is a bad security practice. Anyone with access to the database will immediately have access to all user
accounts thus being able to engage in unauthorized activities. To combat that, many applications use the familiar one way hashing methods
“md5_” and “sha1_”. However, hardware evolves each day, and becomes faster, these algorithms are becoming vulnerable
to brute force attacks. These attacks are also known as `rainbow tables`_.

为了解决这个问题我们可以使用哈希算法 bcrypt。为什么 bcrypt_？ 由于其“Eksblowfish_”键设置算法，我们可以使密码加密如同我们想要的“慢”。慢的算法使计算 Hash背后的真实密码的过程非常困难甚至不可能。这可以在一个很长一段时间内免遭可能的彩虹表攻击。

To solve this problem we can use hash algorithms as bcrypt_. Why bcrypt? Thanks to its “Eksblowfish_” key setup algorithm
we can make the password encryption as “slow” as we want. Slow algorithms make the process to calculate the real
password behind a hash extremely difficult if not impossible. This will protect your for a long time from a
possible attack using rainbow tables.

这个组件使您能够以一个简单的方法使用该算法：

This component gives you the ability to use this algorithm in a simple way:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class UsersController extends Controller
    {

        public function registerAction()
        {

            $user = new Users();

            $login = $this->request->getPost('login');
            $password = $this->request->getPost('password');

            $user->login = $login;

            //Store the password hashed
            $user->password = $this->security->hash($password);

            $user->save();
        }

    }

我们保存一个用默认因子散列的密码。更高的因子可以使密码更加可靠。我们可以用如下的方法检查密码是否正确:	
	
We saved the password hashed with a default work factor. A higher work factor will make the password less vulnerable as
its encryption will be slow. We can check if the password is correct as follows:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class SessionController extends Controller
    {

        public function loginAction()
        {

            $login = $this->request->getPost('login');
            $password = $this->request->getPost('password');

            $user = Users::findFirstByLogin($login);
            if ($user) {
                if ($this->security->checkHash($password, $user->password)) {
                    //The password is valid
                }
            }

            //The validation has failed
        }

    }

Salt使用PHP的 openssl_random_pseudo_bytes 函数的伪随机字节生成的，所以需要加载扩展 openssl_。	
	
The salt is generated using pseudo-random bytes with the PHP's function openssl_random_pseudo_bytes_ so is required to have the openssl_ extension loaded.

防止跨站点请求伪造攻击Cross-Site Request Forgery (CSRF) protection
-----------------------------------------------------------------------
这是另一个常见的web站点和应用程序攻击。如用户注册或添加注释的这类表单就很容易受到这种攻击。

This is another common attack against web sites and applications. Forms designed to perform tasks such as user registration or adding comments
are vulnerable to this attack.

可以想到的方式防止表单值发送自外部应用程序。为了解决这个问题，我们为每个表单生成一个一次性随机令牌（`random nonce`_），在会话中添加令牌，然后一旦表单数据提交到 程序之后，将提交的数据中的的令牌和存储在会中的令牌进行比较，来验证是否合法。

The idea is to prevent the form values from being sent outside our application. To fix this, we generate a `random nonce`_ (token) in each
form, add the token in the session and then validate the token once the form posts data back to our application by comparing the stored
token in the session to the one submitted by the form:

.. code-block:: html+php

    <?php echo Tag::form('session/login') ?>

        <!-- login and password inputs ... -->

        <input type="hidden" name="<?php echo $this->security->getTokenKey() ?>"
            value="<?php echo $this->security->getToken() ?>"/>

    </form>

在控制器的动作中可以检查CSRF令牌是否有效:	
	
Then in the controller's action you can check if the CSRF token is valid:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class SessionController extends Controller
    {

        public function loginAction()
        {
            if ($this->request->isPost()) {
                if ($this->security->checkToken()) {
                    //The token is ok
                }
            }
        }

    }

记得添加一个会话适配器到依赖注入器中，否则令牌检查是行不通的:	
	
Remember to add a session adapter to your Dependency Injector, otherwise the token check won't work:

.. code-block:: php

    $di->setShared('session', function() {
        $session = new Phalcon\Session\Adapter\Files();
        $session->start();
        return $session;
    });

同时也建议为表单添加一个 captcha_ ，以完全避免这种攻击的风险。	
	
Adding a captcha_ to the form is also recommended to completely avoid the risks of this attack.

设置组件Setting up the component
-------------------------------------
该组件自动在服务容器中注册为“security”,你亦可以重新注册它并为它设置参数:

This component is automatically registered in the services container as 'security', you can re-register it
to setup its options:

.. code-block:: php

    <?php

    use Phalcon\Security;

    $di->set('security', function(){

        $security = new Security();

        //Set the password hashing factor to 12 rounds
        $security->setWorkFactor(12);

        return $security;
    }, true);

外部资源External Resources
-------------------------------
`Vökuró <http://vokuro.phalconphp.com>`_ , 是一个使用的安全组件避免CSRF和密码散列的示例应用程序  [`Github <https://github.com/phalcon/vokuro>`_]

* `Vökuró <http://vokuro.phalconphp.com>`_, is a sample application that uses the Security component for avoid CSRF and password hashing, [`Github <https://github.com/phalcon/vokuro>`_]

.. _sha1 : http://php.net/manual/en/function.sha1.php
.. _md5 : http://php.net/manual/en/function.md5.php
.. _openssl_random_pseudo_bytes : http://php.net/manual/en/function.openssl-random-pseudo-bytes.php
.. _openssl : http://php.net/manual/en/book.openssl.php
.. _captcha : http://www.google.com/recaptcha
.. _`random nonce`: http://en.wikipedia.org/wiki/Cryptographic_nonce
.. _bcrypt : http://en.wikipedia.org/wiki/Bcrypt
.. _Eksblowfish : http://en.wikipedia.org/wiki/Bcrypt#Algorithm

.. _`rainbow tables`: http://en.wikipedia.org/wiki/Rainbow_table
