命令行应用Command Line Applications
========================================
CLI应用即是运行在命令行窗体上的应用。 主要用来实现后台任务， 命令行工具等。

CLI applications are executed from the command line. They are useful to create cron jobs, scripts, command utilities and more.

结构Structure
---------------------
最小结构的CLI程序如下：

A minimal structure of a CLI application will look like this:

* app/config/config.php
* app/tasks/MainTask.php
* app/cli.php <-- main bootstrap file

创建引导Creating a Bootstrap
----------------------------------
普通的MVC程序中，启动文件用来启动整个应用。和web应用不同, 此处应用中我们使用cli.php来作为启动文件。

As in regular MVC applications, a bootstrap file is used to bootstrap the application. Instead of the index.php bootstrapper
in web applications, we use a cli.php file for bootstrapping the application.

下面是一个简单的启动文件示例：

Below is a sample bootstrap that is being used for this example.

.. code-block:: php

   <?php
    
    use Phalcon\DI\FactoryDefault\CLI as CliDI,
        Phalcon\CLI\Console as ConsoleApp;
    
    define('VERSION', '1.0.0');
    
    //Using the CLI factory default services container
    $di = new CliDI();
    
    // Define path to application directory
    defined('APPLICATION_PATH')
    || define('APPLICATION_PATH', realpath(dirname(__FILE__)));
    
    /**
     * Register the autoloader and tell it to register the tasks directory
     */
    $loader = new \Phalcon\Loader();
    $loader->registerDirs(
        array(
            APPLICATION_PATH . '/tasks'
        )
    );
    $loader->register();
    
    // Load the configuration file (if any) 
    if(is_readable(APPLICATION_PATH . '/config/config.php')) {
        $config = include APPLICATION_PATH . '/config/config.php';
        $di->set('config', $config);
    }    
    
    //Create a console application
    $console = new ConsoleApp();
    $console->setDI($di);
    
    /**
    * Process the console arguments
    */
    $arguments = array();
    foreach($argv as $k => $arg) {
        if($k == 1) {
            $arguments['task'] = $arg;
        } elseif($k == 2) {
            $arguments['action'] = $arg;
        } elseif($k >= 3) {
           $arguments['params'][] = $arg;
        }
    }

    // define global constants for the current task and action
    define('CURRENT_TASK', (isset($argv[1]) ? $argv[1] : null));
    define('CURRENT_ACTION', (isset($argv[2]) ? $argv[2] : null));
    
    try {
        // handle incoming arguments
        $console->handle($arguments);
    }
    catch (\Phalcon\Exception $e) {
        echo $e->getMessage();
        exit(255);
    }

上面的代码可以使用如下方式执行：	
	
This piece of code can be run using:

.. code-block:: bash

    $ php app/cli.php
   
    This is the default task and the default action
    
    
任务Tasks
----------------
这里的任务同于web应用中的控制器。 任一 CLI 应用程序都至少包含一个mainTask 及一个 mainAction， 每个任务至少有一个mainAction, 这样在使用者未明确的 指定action时 此mainAction就会执行。

Tasks work similar to controllers. Any CLI application needs at least a mainTask and a mainAction and every task needs
to have a mainAction which will run if no action is given explicitly.

下面即是一个mainTask的例子（ app/tasks/MainTask.php ）：

Below is an example of the app/tasks/MainTask.php file

.. code-block:: php

    <?php

    class MainTask extends \Phalcon\CLI\Task
    {
        public function mainAction() {
             echo "\nThis is the default task and the default action \n";
        }
    }


处理动作参数Processing action parameters
---------------------------------------------
CLI应用中， 开发者也可以在action中处理传递过来的参数， 下面的例子中已经对传递过来的参数进行了处理。

It's possible to pass parameters to actions, the code for this is already present in the sample bootstrap.

如果你使用下面的参数和动作运行应用程序:

If you run the the application with the following parameters and action:


.. code-block:: php

    <?php

    class MainTask extends \Phalcon\CLI\Task
    {
        public function mainAction() {
             echo "\nThis is the default task and the default action \n";
        }
        
        /**
        * @param array $params
        */
       public function testAction(array $params) {
           echo sprintf('hello %s', $params[0]) . PHP_EOL;
           echo sprintf('best regards, %s', $params[1]) . PHP_EOL;
       }
    }
	
	
.. code-block:: bash

   $ php app/cli.php main test world universe
   
   hello world
   best regards, universe
    

链中运行任务Running tasks in a chain
---------------------------------------
CLI应用中可以在一个action中执行另一action. 要实现这个需要在 DI 中设置console.

It's also possible to run tasks in a chain if it's required. To accomplish this you must add the console itself
to the DI:

.. code-block:: php
    
    <?php
    
    $di->setShared('console', $console);
     
    try {
        // handle incoming arguments
        $console->handle($arguments);
    }
    
然后开发者即可在一个action中使用用其它的action了. 下面即是例子：	
	
Then you can use the console inside of any task. Below is an example of a modified MainTask.php:

.. code-block:: php
    
    <?php
    
    class MainTask extends \Phalcon\CLI\Task 
    {
        public function mainAction() {
            echo "\nThis is the default task and the default action \n";
    
            $this->console->handle(array(
               'task' => 'main',
               'action' => 'test'
            ));
        }
    
        public function testAction() {
            echo '\nI will get printed too!\n';
        }
    }
    
当然， 通过扩展 \\Phalcon\\CLI\\Task 来实现如上操作会是一个更好主意。	
	
However, it's a better idea to extend \\Phalcon\\CLI\\Task and implement this kind of logic there.

