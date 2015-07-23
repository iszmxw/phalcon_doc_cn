单元测试Unit testing
=========================
编写适当的测试可以帮助写作更好的软件。如果你设置适当的测试用例可以消除大多数的功能缺陷和更好地去维护软件。

Writing proper tests can assist in writing better software. If you set up proper test cases you can eliminate most
functional bugs and better maintain your software.

整合 PHPunit 到 phalcon Integrating PHPunit with phalcon
------------------------------------------------------------
如果你没有安装phpunit,你可以通过使用下面的composer命令:

If you don't already have phpunit installed, you can do it by using the following composer command:

.. code-block:: bash

  composer require phpunit/phpunit


或者手动添加到composer.json：  
  
or by manually adding it to composer.json:

.. code-block:: json

  {
      "require-dev": {
          "phpunit/phpunit": "~4.5"
      }
  }


当phpunit安装好后，在根目录创建叫做“test”的目录：  
  
Once phpunit is installed create a directory called 'tests' in your root directory:

.. code-block:: bash

  app/
  public/
  tests/

接下来，需要一个启动文件来启动单元测试。  
  
Next, we need a 'helper' file to bootstrap the application for unit testing.

PHPunit 辅助文件 The PHPunit helper file
----------------------------------------------
引导应用程序运行测试，需要一个helper文件。我们准备了一个示例文件。把它放到test目录并命名为TestHelper.php。

A helper file is required to bootstrap the application for running the tests. We have prepared a sample file. Put the
file in your tests/ directory as TestHelper.php.




.. code-block:: php

  <?php
  use Phalcon\DI,
      Phalcon\DI\FactoryDefault;

  ini_set('display_errors',1);
  error_reporting(E_ALL);

  define('ROOT_PATH', __DIR__);
  define('PATH_LIBRARY', __DIR__ . '/../app/library/');
  define('PATH_SERVICES', __DIR__ . '/../app/services/');
  define('PATH_RESOURCES', __DIR__ . '/../app/resources/');

  set_include_path(
      ROOT_PATH . PATH_SEPARATOR . get_include_path()
  );

  // required for phalcon/incubator
  include __DIR__ . "/../vendor/autoload.php";

  // use the application autoloader to autoload the classes
  // autoload the dependencies found in composer
  $loader = new \Phalcon\Loader();

  $loader->registerDirs(array(
      ROOT_PATH
  ));

  $loader->register();

  $di = new FactoryDefault();
  DI::reset();

  // add any needed services to the DI here

  DI::setDefault($di);


如果您需要从自己的类库测试任何组件,将它们添加到自动加载器或从主程序中使用自动加载器。  
  
Should you need to test any components from your own library, add them to the autoloader or use the autoloader from your
main application.

为了帮助您构建单元测试,我们做了一些抽象类可以用来引导单元测试。去@ https://github.com/phalcon/incubator.查看。

To help you build the unit tests, we made a few abstract classes you can use to bootstrap the unit tests themselves.
These files exist in the Phalcon incubator @ https://github.com/phalcon/incubator.

可以使用incubator库将他们添加到composer中。

You can use the incubator library by adding it as a dependency:

.. code-block:: bash

  composer require phalcon/incubator


或者手动添加：  
  
or by manually adding it to composer.json:

.. code-block:: json

  {
      "require": {
          "phalcon/incubator": "dev-master"
      }
  }

或者clone上面的库使用上面的链接。  
  
You can also clone the repository using the repo link above.

PHPunit.xml 文件 PHPunit.xml file
-------------------------------------
Now, create a phpunit file:

.. code-block:: xml

  <?xml version="1.0" encoding="UTF-8"?>
  <phpunit bootstrap="./TestHelper.php"
           backupGlobals="false"
           backupStaticAttributes="false"
           verbose="true"
           colors="false"
           convertErrorsToExceptions="true"
           convertNoticesToExceptions="true"
           convertWarningsToExceptions="true"
           processIsolation="false"
           stopOnFailure="false"
           syntaxCheck="true">
      <testsuite name="Phalcon - Testsuite">
          <directory>./</directory>
      </testsuite>
  </phpunit>

更改 phpunit.xml 去满足需求，并保存在tests/目录下。 
  
Modify the phpunit.xml to fit your needs and save it in tests/.

将会执行在tests/目录下任何的测试。

This will run any tests under the tests/ directory.

简单的单元测试 Sample unit test
-----------------------------------
运行单元测试,您需要定义它们。自动加载器将确保适当的文件，然后phpunit将为您运行测试。

To run any unit tests you need to define them. The autoloader will make sure the proper files are loaded so all you
need to do is create the files and phpunit will run the tests for you.

这个例子不包含一个配置文件,然而,大多数测试用例需要一个。你可以将它添加到依赖注入容器去获取 UnitTestCase文件。

This example does not contain a config file, most test cases however, do need one. You can add it to the DI to get the UnitTestCase file.

首先需要在/tests目录创建一个UnitTestCase.php文件：

First create a base unit test called UnitTestCase.php in your /tests directory:

.. code-block:: php

  <?php
  use Phalcon\DI,
      \Phalcon\Test\UnitTestCase as PhalconTestCase;

  abstract class UnitTestCase extends PhalconTestCase {

      /**
       * @var \Voice\Cache
       */
      protected $_cache;

      /**
       * @var \Phalcon\Config
       */
      protected $_config;

      /**
       * @var bool
       */
      private $_loaded = false;

      public function setUp(Phalcon\DiInterface $di = NULL, Phalcon\Config $config = NULL) {

          // Load any additional services that might be required during testing
          $di = DI::getDefault();

          // get any DI components here. If you have a config, be sure to pass it to the parent
          parent::setUp($di);

          $this->_loaded = true;
      }

      /**
       * Check if the test case is setup properly
       * @throws \PHPUnit_Framework_IncompleteTestError;
       */
      public function __destruct() {
          if(!$this->_loaded) {
              throw new \PHPUnit_Framework_IncompleteTestError('Please run parent::setUp().');
          }
      }
  }

在名称空间独立单元测试总是一个好主意。对于这个测试,我们将创建名称空间“测试”。 所以创建一个文件叫做 \tests\Test\UnitTest.php
  
It's always a good idea to separate your Unit tests in namespaces. For this test we will create the namespace
'Test'. So create a file called \tests\Test\UnitTest.php:

.. code-block:: php

  <?php

  namespace Test;

  /**
   * Class UnitTest
   */
  class UnitTest extends \UnitTestCase {

      public function testTestCase() {

          $this->assertEquals('works',
              'works',
              'This is OK'
          );

          $this->assertEquals('works',
              'works1',
              'This will fail'
          );
      }
  }

 在\tests目录命令行中执行'phpunit'。会得到如下结果： 
  
Now when you execute 'phpunit' in your command-line from the \tests directory you will get the following output:

.. code-block:: bash

  $ phpunit
  PHPUnit 3.7.23 by Sebastian Bergmann.

  Configuration read from /private/var/www/tests/phpunit.xml

  Time: 3 ms, Memory: 3.25Mb

  There was 1 failure:

  1) Test\UnitTest::testTestCase
  This will fail
  Failed asserting that two strings are equal.
  --- Expected
  +++ Actual
  @@ @@
  -'works'
  +'works1'

  /private/var/www/tests/Test/UnitTest.php:25

  FAILURES!
  Tests: 1, Assertions: 2, Failures: 1.

现在可以编写我们的单元测试了。这里有篇很好的教程http://blog.stevensanderson.com/2009/08/24/writing-great-unit-tests-best-and-worst-practises/。如果不熟悉建议去读phpunit的官方教程。 
  
Now you can start building your unit tests. You can view a good guide here (we also recommend reading the
PHPunit documentation if you're not familiar with PHPunit):

http://blog.stevensanderson.com/2009/08/24/writing-great-unit-tests-best-and-worst-practises/
