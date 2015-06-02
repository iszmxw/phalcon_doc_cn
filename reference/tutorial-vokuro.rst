教程6: Vökuró
==================
Vökuró是另一个去学习phalcon的示例应用。Vökuró实现了安全特性，管理用户和权限。可以从 Github_ 克隆项目。

Vökuró is another sample application you can use to learn more about Phalcon.
Vökuró is a small website that shows how to implement a security features and
management of users and permissions. You can clone its code from Github_.

目录结构
----------
当克隆完项目后会看到如下目录结构：

Once you clone the project in your document root you'll see the following structure:

.. code-block:: bash

    invo/
        app/
            cache/
            config/
            controllers/
            forms/
            library/
            models/
            plugins/
            views/
        public/
            css/
            js/
        schemas/

项目结构和INVO类似，用浏览器访问http://localhost/vokuro 会看到如下所示：		
		
This project follows a quite similar structure to INVO. Once you open the application in your
browser http://localhost/vokuro you'll see something like this:

.. figure:: ../_static/img/vokuro-1.png
   :align: center

应用分为两部分，前端：用户可以注册，后端：用户可以管理注册用户。前端和后端在一个模块里面。
   
The application is divided into two parts, a frontend, where visitors can sign up the service
and a backend where administrative users can manage registered users. Both frontend and backend
are combined in a single module.

加载类和依赖关系
--------------------
项目使用Phalcon\\Loader加载控制器、模块、表单等。使用composer_ 去处理依赖关系。在终端中输入如下命令：

This project uses Phalcon\\Loader to load controllers, models, forms, etc. within the project and composer_
to load the project's dependencies. So, the first thing you have to do before execute Vökuró is
install its dependencies via composer_. Assuming you have it correctly installed, type the
following command in the console:

.. code-block:: bash

    cd vokuro
    composer install

Vökuró 使用swift去通知确认注册用户。composer.json如下所示：	
	
Vökuró sends emails to confirm the sign up of registered users using Swift,
the composer.json looks like:

.. code-block:: json

    {
        "require" : {
            "php" : ">=5.4.0",
            "ext-phalcon" : ">=2.0.0",
            "swiftmailer/swiftmailer" : "5.0.*",
            "amazonwebservices/aws-sdk-for-php" : "~1.0"
        }
    }

app/config/loader.php 中写有自动加载文件的配置，在文件最后有composer自动加载器去加载依赖类。	
	
Now, there is a file called app/config/loader.php where all the auto-loading stuff is set up. At the end of
this file you can see that the composer autoloader is included enabling the application to autoload
any of the classes in the downloaded dependencies:

.. code-block:: php

    <?php

    // ...

    // Use composer autoloader to load vendor classes
    require_once __DIR__ . '/../../vendor/autoload.php';

再次和INVO不同的是，vokuro使用命名空间去组织项目，(app/config/loader.php)这个文件看起来和INVO不太相同。
	
Moreover, Vökuró, unlike the INVO, utilizes namespaces for controllers and models
which is the recommended practice to structure a project. This way the autoloader looks slightly
different than the one we saw before (app/config/loader.php):

.. code-block:: php

    <?php

    $loader = new \Phalcon\Loader();

    $loader->registerNamespaces(array(
        'Vokuro\Models'      => $config->application->modelsDir,
        'Vokuro\Controllers' => $config->application->controllersDir,
        'Vokuro\Forms'       => $config->application->formsDir,
        'Vokuro'             => $config->application->libraryDir
    ));

    $loader->register();

    // ...

我们使用registerNamespaces替代了registerDirectories。每个命名空间指向了(app/config/config.php)文件中定义的目录。例如，命名空间Vokuro\\Controllers指向了app/controllers，在命名空间中应用需要的类定义如下：	
	
Instead of using registerDirectories, we use registerNamespaces. Every namespace points to a directory
defined in the configuration file (app/config/config.php). For instance the namespace Vokuro\\Controllers
points to app/controllers so all the classes required by the application within this namespace
requires it in its definition:

.. code-block:: php

    <?php

    namespace Vokuro\Controllers;

    class AboutController extends ControllerBase
    {

        // ...
    }


注册
------
首先检查用户是否在vokuro中注册。当用户点击注册按钮，sessioncontroller控制器被调用。signup方法被执行。

First, let's check how users are registered in Vökuró. When a user clicks the "Create an Account" button,
the controller SessionController is invoked and the action "signup" is executed:

.. code-block:: php

    <?php

    namespace Vokuro\Controllers;

    use Vokuro\Forms\SignUpForm;

    class RegisterController extends ControllerBase
    {
        public function signupAction()
        {
            $form = new SignUpForm();

            // ...

            $this->view->form = $form;
        }
    }

这个动作只传递一个表单SignUpForm实例给视图,展示给用户并让用户输入登录数据:	
	
This action simply pass a form instance of SignUpForm to the view, which itself is rendered to
allow the user enter the login details:

.. code-block:: html+jinja

    {{ form('class': 'form-search') }}

        <h2>Sign Up</h2>

        <p>{{ form.label('name') }}</p>
        <p>
            {{ form.render('name') }}
            {{ form.messages('name') }}
        </p>

        <p>{{ form.label('email') }}</p>
        <p>
            {{ form.render('email') }}
            {{ form.messages('email') }}
        </p>

        <p>{{ form.label('password') }}</p>
        <p>
            {{ form.render('password') }}
            {{ form.messages('password') }}
        </p>

        <p>{{ form.label('confirmPassword') }}</p>
        <p>
            {{ form.render('confirmPassword') }}
            {{ form.messages('confirmPassword') }}
        </p>

        <p>
            {{ form.render('terms') }} {{ form.label('terms') }}
            {{ form.messages('terms') }}
        </p>

        <p>{{ form.render('Sign Up') }}</p>

        {{ form.render('csrf', ['value': security.getToken()]) }}
        {{ form.messages('csrf') }}

        <hr>

    </form>

结论
----------

As we have seen, develop a RESTful API with Phalcon is easy. Later in the documentation we'll explain in detail how to
use micro applications and the :doc:`PHQL <phql>` language.

.. _Github: https://github.com/phalcon/vokuro
.. _composer: https://getcomposer.org/
