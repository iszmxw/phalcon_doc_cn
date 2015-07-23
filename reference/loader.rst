通用类加载器Universal Class Loader
=====================================
:doc:`Phalcon\\Loader <../api/Phalcon_Loader>`是一个允许我们根据预先定义好的规则自动加载类的组件。因为这个组件是有C语言编写的所以在读取和解析外部PHP时候性能非常高。

:doc:`Phalcon\\Loader <../api/Phalcon_Loader>` is a component that allows you to load project classes automatically,
based on some predefined rules. Since this component is written in C, it provides the lowest overhead in
reading and interpreting external PHP files.

这个行为依赖于PHP的`autoloading classes`_能力。如果一个类在代码任何一部位没有被使用，一个特定的处理过程将会架在它。:doc:`Phalcon\\Loader <../api/Phalcon_Loader>`就是那个特殊的过程组件。整体性能得到了提升，只有当文件再被使用的时候才会被加载，这项技术叫做`lazy initialization`_。

The behavior of this component is based on the PHP's capability of `autoloading classes`_. If a class that does
not exist is used in any part of the code, a special handler will try to load it.
:doc:`Phalcon\\Loader <../api/Phalcon_Loader>` serves as the special handler for this operation.
By loading classes on a need to load basis, the overall performance is increased since the only file
reads that occur are for the files needed. This technique is called `lazy initialization`_.

通过该组件可以从其他项目或提供者加载文件。这个自动加载器是兼容`PSR-0 <https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md>`_。

With this component you can load files from other projects or vendors, this autoloader is `PSR-0 <https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md>`_ compliant.

:doc:`Phalcon\\Loader <../api/Phalcon_Loader>` 提供了四个选项去自动加载类。可以灵活的使用一个或者多个组合。

:doc:`Phalcon\\Loader <../api/Phalcon_Loader>` offers four options to autoload classes. You can use them one at a time or combine them.

注册命名空间Registering Namespaces
------------------------------------
如果使用名称空间组织代码或外部库,registerNamespaces()提供了半自动的机制。它需要一个关联数组作为参数,键名为命名空间的前缀，键值为命名空间类所在的目录。当加载程序试图找到的类时命名空间路径将会被目录路径取代。记得在路径的最后一行添加一个/斜线。

If you're organizing your code using namespaces, or external libraries do so, the registerNamespaces() provides the autoloading mechanism. It
takes an associative array, which keys are namespace prefixes and their values are directories where the classes are located in. The namespace
separator will be replaced by the directory separator when the loader try to find the classes. Remember always to add a trailing slash at
the end of the paths.

.. code-block:: php

    <?php

    // Creates the autoloader
    $loader = new \Phalcon\Loader();

    //Register some namespaces
    $loader->registerNamespaces(
        array(
           "Example\Base"    => "vendor/example/base/",
           "Example\Adapter" => "vendor/example/adapter/",
           "Example"         => "vendor/example/",
        )
    );

    // register autoloader
    $loader->register();

    // The required class will automatically include the
    // file vendor/example/adapter/Some.php
    $some = new Example\Adapter\Some();

注册前缀Registering Prefixes
------------------------------
这种策略类似于名称空间的策略。它需要一个关联数组,键名为命名空间的前缀，键值为命名空间类所在的目录。当加载程序试图找到的类时带下划线_的命名空间路径将会被目录路径取代。记得在路径的最后一行添加一个/斜线。

This strategy is similar to the namespaces strategy. It takes an associative array, which keys are prefixes and their values are directories
where the classes are located in. The namespace separator and the "_" underscore character will be replaced by the directory separator when
the loader try to find the classes. Remember always to add a trailing slash at the end of the paths.

.. code-block:: php

    <?php

    // Creates the autoloader
    $loader = new \Phalcon\Loader();

    //Register some prefixes
    $loader->registerPrefixes(
        array(
           "Example_Base"     => "vendor/example/base/",
           "Example_Adapter"  => "vendor/example/adapter/",
           "Example_"         => "vendor/example/",
        )
    );

    // register autoloader
    $loader->register();

    // The required class will automatically include the
    // file vendor/example/adapter/Some.php
    $some = new Example_Adapter_Some();

注册文件夹Registering Directories
--------------------------------------
第三种方法是注册目录,指定类要从哪个目录加载。因为性能原因不推荐使用这个方法,因为Phalcon在每个文件夹中需要执行大量的文件数据,寻找与类相同的名称的文件。注册目录关联顺序是很重要的。记得在路径的最后一行添加一个/斜线。

The third option is to register directories, in which classes could be found. This option is not recommended in terms of performance,
since Phalcon will need to perform a significant number of file stats on each folder, looking for the file with the same name as the class.
It's important to register the directories in relevance order. Remember always add a trailing slash at the end of the paths.

.. code-block:: php

    <?php

    // Creates the autoloader
    $loader = new \Phalcon\Loader();

    // Register some directories
    $loader->registerDirs(
        array(
            "library/MyComponent/",
            "library/OtherComponent/Other/",
            "vendor/example/adapters/",
            "vendor/example/"
        )
    );

    // register autoloader
    $loader->register();

    // The required class will automatically include the file from
    // the first directory where it has been located
    // i.e. library/OtherComponent/Other/Some.php
    $some = new Some();

注册类名Registering Classes
--------------------------------
最后一个方法是注册类名和路径。将会非常的有用在当项目的文件夹使用路径和类名不容易获得文件的时候。这是最快的自动加载的方法。然而随着应用程序规模的增长,更多的类/文件需要被添加到这个自动加载器重,维护这个类的列表将会非常繁琐所以不建议使用。

The last option is to register the class name and its path. This autoloader can be very useful when the folder convention of the
project does not allow for easy retrieval of the file using the path and the class name. This is the fastest method of autoloading.
However the more your application grows, the more classes/files need to be added to this autoloader, which will effectively make
maintenance of the class list very cumbersome and it is not recommended.

.. code-block:: php

    <?php

    // Creates the autoloader
    $loader = new \Phalcon\Loader();

    // Register some classes
    $loader->registerClasses(
        array(
            "Some"         => "library/OtherComponent/Other/Some.php",
            "Example\Base" => "vendor/example/adapters/Example/BaseClass.php",
        )
    );

    // register autoloader
    $loader->register();

    // Requiring a class will automatically include the file it references
    // in the associative array
    // i.e. library/OtherComponent/Other/Some.php
    $some = new Some();

额外的扩展名Additional file extensions
----------------------------------------
 "prefixes", "namespaces" 或者 "directories" 自动加载策略将会在文件末尾自动添加php尾缀。如果使用不同的尾缀，需要通过"setExtensions"设置。文件就会按照设置的检测：

Some autoloading strategies such as  "prefixes", "namespaces" or "directories" automatically append the "php" extension at the end of the checked file. If you
are using additional extensions you could set it with the method "setExtensions". Files are checked in the order as it were defined:

.. code-block:: php

    <?php

     // Creates the autoloader
    $loader = new \Phalcon\Loader();

    //Set file extensions to check
    $loader->setExtensions(array("php", "inc", "phb"));

修改当前策略Modifying current strategies
---------------------------------------------
额外的自动数据加载可以通过以下方式添加到现有数据中：

Additional auto-loading data can be added to existing values in the following way:

.. code-block:: php

    <?php

    // Adding more directories
    $loader->registerDirs(
        array(
            "../app/library/",
            "../app/plugins/"
        ),
        true
    );

第二个传递true,将会将新值和现有值进行合并。	
	
Passing "true" as second parameter will merge the current values with new ones in any strategy.

安全层Security Layer
-------------------------
Phalcon\\Loader 提供安全层自动过滤类名，防止加载未授权文件。代码如下所示：

Phalcon\\Loader offers a security layer sanitizing by default class names avoiding possible inclusion of unauthorized files.
Consider the following example:

.. code-block:: php

    <?php

    //Basic autoloader
    spl_autoload_register(function($className) {
        if (file_exists($className . '.php')) {
            require $className . '.php';
        }
    });

上面代码缺少必要的安全检查。如果在自动加载器中函数出错，将会是一个被恶意拼接的字符串被包含并执行。	
	
The above auto-loader lacks of any security check, if by mistake in a function that launch the auto-loader,
a malicious prepared string is used as parameter this would allow to execute any file accessible by the application:

.. code-block:: php

    <?php

    //This variable is not filtered and comes from an insecure source
    $className = '../processes/important-process';

    //Check if the class exists triggering the auto-loader
    if (class_exists($className)) {
        //...
    }

如果'../processes/important-process.php'是个合理的文件。一个外部未授权用户将能执行这个文件。	
	
If '../processes/important-process.php' is a valid file, an external user could execute the file without
authorization.

为了避免类似的攻击，Phalcon\\Loader自动从类名中移除不合法的字符串降低被攻击的风险。

To avoid these or most sophisticated attacks, Phalcon\\Loader removes any invalid character from the class name
reducing the possibility of being attacked.

自动加载事件Autoloading Events
------------------------------------
如下代码所示，使用自动加载器EventsManager让我们获得调试信息的考虑流程操作：

In the following example, the EventsManager is working with the class loader, allowing us to obtain debugging information regarding the flow of operation:

.. code-block:: php

    <?php

    $eventsManager = new \Phalcon\Events\Manager();

    $loader = new \Phalcon\Loader();

    $loader->registerNamespaces(array(
       'Example\\Base' => 'vendor/example/base/',
       'Example\\Adapter' => 'vendor/example/adapter/',
       'Example' => 'vendor/example/'
    ));

    //Listen all the loader events
    $eventsManager->attach('loader', function($event, $loader) {
        if ($event->getType() == 'beforeCheckPath') {
            echo $loader->getCheckedPath();
        }
    });

    $loader->setEventsManager($eventsManager);

    $loader->register();

	
一些事件当返回布尔false可以停止action操作。支持以下事件:	
	
Some events when returning boolean false could stop the active operation. The following events are supported:

+------------------+---------------------------------------------------------------------------------------------------------------------+---------------------+
| Event Name       | Triggered                                                                                                           | Can stop operation? |
+==================+=====================================================================================================================+=====================+
| beforeCheckClass | Triggered before starting the autoloading process                                                                   | Yes                 |
+------------------+---------------------------------------------------------------------------------------------------------------------+---------------------+
| pathFound        | Triggered when the loader locate a class                                                                            | No                  |
+------------------+---------------------------------------------------------------------------------------------------------------------+---------------------+
| afterCheckClass  | Triggered after finish the autoloading process. If this event is launched the autoloader didn't find the class file | No                  |
+------------------+-----------------------------------------------------------+---------------------------------------------------------+---------------------+

注意事项Troubleshooting
------------------------------
在使用通用类加载器的时候需要注意以下几点：

Some things to keep in mind when using the universal autoloader:

* 自动加载流程区分大小写，加载在代码中定义的类名
* namespaces/prefixes自动加载策略要比目录加载策略快。
* 如果有类似APC_字节码缓存器安装。则文件将从缓存中获得（将会执行文件的隐式缓存）

* Auto-loading process is case-sensitive, the class will be loaded as it is written in the code
* Strategies based on namespaces/prefixes are faster than the directories strategy
* If a cache bytecode like APC_ is installed this will used to retrieve the requested file (an implicit caching of the file is performed)

.. _autoloading classes: http://www.php.net/manual/en/language.oop5.autoload.php
.. _lazy initialization: http://en.wikipedia.org/wiki/Lazy_initialization
.. _APC: http://php.net/manual/en/book.apc.php
