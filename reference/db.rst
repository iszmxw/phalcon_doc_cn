数据库抽象层Database Abstraction Layer
============================================
:doc:`Phalcon\\Db <../api/Phalcon_Db>` 是 :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 背后的一个组件，它为框架提供了强大的model层。它是一个完全由C语言写的独立的高级抽象层的数据库系统。

:doc:`Phalcon\\Db <../api/Phalcon_Db>` is the component behind :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` that powers the model layer
in the framework. It consists of an independent high-level abstraction layer for database systems completely written in C.

这个组件提供了比传统模式的更容易上手的数据库操作。

This component allows for a lower level database manipulation than using traditional models.

.. highlights::

    这个指引不是一个完整的包含所有方法和它们的参数的文档。 查看完整的文档参考，请访问 :doc:`API <../api/index>`

    This guide is not intended to be a complete documentation of available methods and their arguments. Please visit the :doc:`API <../api/index>`
    for a complete reference.

数据库适配器Database Adapters
----------------------------------
这个组件利用了这些适配器去封装特定的数据库的详细操作。Phalcon使用 PDO_ 去连接这些数据库。下面这些是我们支持的数据库引擎：

This component makes use of adapters to encapsulate specific database system details. Phalcon uses PDO_ to connect to databases. The following
database engines are supported:

+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| Name       | Description                                                                                                                                                                                                                          | API                                                                                     |
+============+======================================================================================================================================================================================================================================+=========================================================================================+
| MySQL      | Is the world's most used relational database management system (RDBMS) that runs as a server providing multi-user access to a number of databases                                                                                    | :doc:`Phalcon\\Db\\Adapter\\Pdo\\Mysql <../api/Phalcon_Db_Adapter_Pdo_Mysql>`           |
+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| PostgreSQL | PostgreSQL is a powerful, open source relational database system. It has more than 15 years of active development and a proven architecture that has earned it a strong reputation for reliability, data integrity, and correctness. | :doc:`Phalcon\\Db\\Adapter\\Pdo\\Postgresql <../api/Phalcon_Db_Adapter_Pdo_Postgresql>` |
+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| SQLite     | SQLite is a software library that implements a self-contained, serverless, zero-configuration, transactional SQL database engine                                                                                                     | :doc:`Phalcon\\Db\\Adapter\\Pdo\\Sqlite <../api/Phalcon_Db_Adapter_Pdo_Sqlite>`         |
+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| Oracle     | Oracle is an object-relational database management system produced and marketed by Oracle Corporation.                                                                                                                               | :doc:`Phalcon\\Db\\Adapter\\Pdo\\Oracle <../api/Phalcon_Db_Adapter_Pdo_Oracle>`         |
+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+

自定义适配器Implementing your own adapters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果你想创建自己的适配器或者扩展现有的适配器，这个 :doc:`Phalcon\\Db\\AdapterInterface <../api/Phalcon_Db_AdapterInterface>`  接口必须被实现。

The :doc:`Phalcon\\Db\\AdapterInterface <../api/Phalcon_Db_AdapterInterface>` interface must be implemented in order to create your own
database adapters or extend the existing ones.

数据库“接口”封装Database Dialects
-----------------------------------
Phalcon把每个数据库引擎的具体操作封装成“接口”，这些“接口”提供了提供通用的功能和SQL生成的适配器。 (译者注：这里的“接口”是指Phalcon把一些常用的数据库操作封装成类的方法，例如检查数据库中表是否存在，不再需要麻烦的手动写SQL，可以把调用tableExists方法去查询)

Phalcon encapsulates the specific details of each database engine in dialects. Those provide common functions and SQL generator to the adapters.

+------------+-----------------------------------------------------+--------------------------------------------------------------------------------+
| Name       | Description                                         | API                                                                            |
+============+=====================================================+================================================================================+
| MySQL      | SQL specific dialect for MySQL database system      | :doc:`Phalcon\\Db\\Dialect\\Mysql <../api/Phalcon_Db_Dialect_Mysql>`           |
+------------+-----------------------------------------------------+--------------------------------------------------------------------------------+
| PostgreSQL | SQL specific dialect for PostgreSQL database system | :doc:`Phalcon\\Db\\Dialect\\Postgresql <../api/Phalcon_Db_Dialect_Postgresql>` |
+------------+-----------------------------------------------------+--------------------------------------------------------------------------------+
| SQLite     | SQL specific dialect for SQLite database system     | :doc:`Phalcon\\Db\\Dialect\\Sqlite <../api/Phalcon_Db_Dialect_Sqlite>`         |
+------------+-----------------------------------------------------+--------------------------------------------------------------------------------+
| Oracle     | SQL specific dialect for Oracle database system     | :doc:`Phalcon\\Db\\Dialect\\Oracle <../api/Phalcon_Db_Dialect_Oracle>`         |
+------------+-----------------------------------------------------+--------------------------------------------------------------------------------+

自定义“接口”Implementing your own dialects
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果你想创建自己的“接口”或者扩展现有的“接口”，你需要实现这个接口：:doc:`Phalcon\\Db\\DialectInterface <../api/Phalcon_Db_DialectInterface>` 

The :doc:`Phalcon\\Db\\DialectInterface <../api/Phalcon_Db_DialectInterface>` interface must be implemented in order to create your own database dialects or extend the existing ones.

连接数据库Connecting to Databases
-------------------------------------
为了建立连接，实例化适配器类是必须的。它只接收一个包含连接参数的数组。 下面的例子展示了，传递必要参数和可选项的参数去连接数据库：

To create a connection it's necessary instantiate the adapter class. It only requires an array with the connection parameters. The example
below shows how to create a connection passing both required and optional parameters:

.. code-block:: php

    <?php

    // Required
    $config = array(
        "host" => "127.0.0.1",
        "username" => "mike",
        "password" => "sigma",
        "dbname" => "test_db"
    );

    // Optional
    $config["persistent"] = false;

    // Create a connection
    $connection = new \Phalcon\Db\Adapter\Pdo\Mysql($config);

.. code-block:: php

    <?php

    // Required
    $config = array(
        "host" => "localhost",
        "username" => "postgres",
        "password" => "secret1",
        "dbname" => "template"
    );

    // Optional
    $config["schema"] = "public";

    // Create a connection
    $connection = new \Phalcon\Db\Adapter\Pdo\Postgresql($config);

.. code-block:: php

    <?php

    // Required
    $config = array(
        "dbname" => "/path/to/database.db"
    );

    // Create a connection
    $connection = new \Phalcon\Db\Adapter\Pdo\Sqlite($config);

.. code-block:: php

    <?php

    // Basic configuration
    $config = array(
        'username' => 'scott',
        'password' => 'tiger',
        'dbname' => '192.168.10.145/orcl',
    );

    // Advanced configuration
    $config = array(
        'dbname' => '(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=localhost)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=xe)(FAILOVER_MODE=(TYPE=SELECT)(METHOD=BASIC)(RETRIES=20)(DELAY=5))))',
        'username' => 'scott',
        'password' => 'tiger',
        'charset' => 'AL32UTF8',
    );

    // Create a connection
    $connection = new \Phalcon\Db\Adapter\Pdo\Oracle($config);

设置额外的 PDO 选项Setting up additional PDO options
----------------------------------------------------------
你可以在连接的时候，通过传递’options’参数，设置PDO选项：

You can set PDO options at connection time by passing the parameters 'options':

.. code-block:: php

    <?php

    // Create a connection with PDO options
    $connection = new \Phalcon\Db\Adapter\Pdo\Mysql(array(
        "host" => "localhost",
        "username" => "root",
        "password" => "sigma",
        "dbname" => "test_db",
        "options" => array(
            PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES \'UTF8\'",
            PDO::ATTR_CASE => PDO::CASE_LOWER
        )
    ));

查找行Finding Rows
-------------------------
:doc:`Phalcon\\Db <../api/Phalcon_Db>` 提供了几种方法去查询行。在这个例子中，SQL语句是必须符合数据库的SQL语法的：

:doc:`Phalcon\\Db <../api/Phalcon_Db>` provides several methods to query rows from tables. The specific SQL syntax of the target database engine is required in this case:

.. code-block:: php

    <?php

    $sql = "SELECT id, name FROM robots ORDER BY name";

    // Send a SQL statement to the database system
    $result = $connection->query($sql);

    // Print each robot name
    while ($robot = $result->fetch()) {
       echo $robot["name"];
    }

    // Get all rows in an array
    $robots = $connection->fetchAll($sql);
    foreach ($robots as $robot) {
       echo $robot["name"];
    }

    // Get only the first row
    $robot = $connection->fetchOne($sql);

默认情况下，这些调用会建立一个数组，数组中包含以字段名和以数字下标为key的值。你可以改变这种行为通过使用 Phalcon\\Db\\Result::setFetchMode() 。这个方法接受一个常量值，确定哪些类型的指标是被要求的。	
	
By default these calls create arrays with both associative and numeric indexes. You can change this behavior by using Phalcon\\Db\\Result::setFetchMode(). This method receives a constant, defining which kind of index is required.

+--------------------------+-----------------------------------------------------------+
| Constant                 | Description                                               |
+==========================+===========================================================+
| Phalcon\\Db::FETCH_NUM   | Return an array with numeric indexes                      |
+--------------------------+-----------------------------------------------------------+
| Phalcon\\Db::FETCH_ASSOC | Return an array with associative indexes                  |
+--------------------------+-----------------------------------------------------------+
| Phalcon\\Db::FETCH_BOTH  | Return an array with both associative and numeric indexes |
+--------------------------+-----------------------------------------------------------+
| Phalcon\\Db::FETCH_OBJ   | Return an object instead of an array                      |
+--------------------------+-----------------------------------------------------------+

.. code-block:: php

    <?php

    $sql = "SELECT id, name FROM robots ORDER BY name";
    $result = $connection->query($sql);

    $result->setFetchMode(Phalcon\Db::FETCH_NUM);
    while ($robot = $result->fetch()) {
       echo $robot[0];
    }

这个 Phalcon\\Db::query() 方法返回一个 :doc:`Phalcon\\Db\\Result\\Pdo <../api/Phalcon_Db_Result_Pdo>`实例。这些对象封装了凡是涉及到返回的结果集的功能，例如遍历，寻找特定行，计算总行数等等	
	
The Phalcon\\Db::query() returns an instance of :doc:`Phalcon\\Db\\Result\\Pdo <../api/Phalcon_Db_Result_Pdo>`. These objects encapsulate all the functionality related to the returned resultset i.e. traversing, seeking specific records, count etc.

.. code-block:: php

    <?php

    $sql = "SELECT id, name FROM robots";
    $result = $connection->query($sql);

    // Traverse the resultset
    while ($robot = $result->fetch()) {
       echo $robot["name"];
    }

    // Seek to the third row
    $result->seek(2);
    $robot = $result->fetch();

    // Count the resultset
    echo $result->numRows();

绑定参数Binding Parameters
---------------------------------
在 :doc:`Phalcon\\Db <../api/Phalcon_Db>` 中支持绑定参数。虽然使用绑定参数会有很少性能的损失，但是我们鼓励你使用这个方法 去消除(译者注：是消除，不是减少，因为使用参数绑定可以彻底解决SQL注入问题)SQL注入攻击的可能性。 字符串和占位符都支持，就像下面展示的那样，绑定参数可以简单地实现：

Bound parameters is also supported in :doc:`Phalcon\\Db <../api/Phalcon_Db>`. Although there is a minimal performance impact by using
bound parameters, you are encouraged to use this methodology so as to eliminate the possibility of your code being subject to SQL
injection attacks. Both string and positional placeholders are supported. Binding parameters can simply be achieved as follows:

.. code-block:: php

    <?php

    // Binding with numeric placeholders
    $sql    = "SELECT * FROM robots WHERE name = ? ORDER BY name";
    $result = $connection->query($sql, array("Wall-E"));

    // Binding with named placeholders
    $sql     = "INSERT INTO `robots`(name`, year) VALUES (:name, :year)";
    $success = $connection->query($sql, array("name" => "Astro Boy", "year" => 1952));

插入、更新、删除行Inserting/Updating/Deleting Rows
--------------------------------------------------------
去插入，更新或者删除行，你可以使用原生SQL操作，或者使用类中预设的方法

To insert, update or delete rows, you can use raw SQL or use the preset functions provided by the class:

.. code-block:: php

    <?php

    // Inserting data with a raw SQL statement
    $sql     = "INSERT INTO `robots`(`name`, `year`) VALUES ('Astro Boy', 1952)";
    $success = $connection->execute($sql);

    //With placeholders
    $sql     = "INSERT INTO `robots`(`name`, `year`) VALUES (?, ?)";
    $success = $connection->execute($sql, array('Astro Boy', 1952));

    // Generating dynamically the necessary SQL
    $success = $connection->insert(
       "robots",
       array("Astro Boy", 1952),
       array("name", "year")
    );

    // Generating dynamically the necessary SQL (another syntax)
    $success = $connection->insertAsDict(
       "robots",
       array(
          "name" => "Astro Boy",
          "year" => 1952
       )
    );

    // Updating data with a raw SQL statement
    $sql     = "UPDATE `robots` SET `name` = 'Astro boy' WHERE `id` = 101";
    $success = $connection->execute($sql);

    //With placeholders
    $sql     = "UPDATE `robots` SET `name` = ? WHERE `id` = ?";
    $success = $connection->execute($sql, array('Astro Boy', 101));

    // Generating dynamically the necessary SQL
    $success = $connection->update(
       "robots",
       array("name"),
       array("New Astro Boy"),
       "id = 101" //Warning! In this case values are not escaped
    );

    // Generating dynamically the necessary SQL (another syntax)
    $success = $connection->updateAsDict(
       "robots",
       array(
          "name" => "New Astro Boy"
       ),
       "id = 101" //Warning! In this case values are not escaped
    );

    //With escaping conditions
    $success = $connection->update(
       "robots",
       array("name"),
       array("New Astro Boy"),
       array(
          'conditions' => 'id = ?',
          'bind' => array(101),
          'bindTypes' => array(PDO::PARAM_INT) //optional parameter
       )
    );
    $success = $connection->updateAsDict(
       "robots",
       array(
          "name" => "New Astro Boy"
       ),
       array(
          'conditions' => 'id = ?',
          'bind' => array(101),
          'bindTypes' => array(PDO::PARAM_INT) //optional parameter
       )
    );

    // Deleting data with a raw SQL statement
    $sql     = "DELETE `robots` WHERE `id` = 101";
    $success = $connection->execute($sql);

    //With placeholders
    $sql     = "DELETE `robots` WHERE `id` = ?";
    $success = $connection->execute($sql, array(101));

    // Generating dynamically the necessary SQL
    $success = $connection->delete("robots", "id = ?", array(101));

事务与嵌套事务Transactions and Nested Transactions
-------------------------------------------------------
PDO支持事务工作。在事务里面执行数据操作, 在大多数数据库系统上, 往往可以提高数据库的性能：

Working with transactions is supported as it is with PDO. Perform data manipulation inside transactions
often increase the performance on most database systems:

.. code-block:: php

    <?php

    try {

        //Start a transaction
        $connection->begin();

        //Execute some SQL statements
        $connection->execute("DELETE `robots` WHERE `id` = 101");
        $connection->execute("DELETE `robots` WHERE `id` = 102");
        $connection->execute("DELETE `robots` WHERE `id` = 103");

        //Commit if everything goes well
        $connection->commit();

    } catch(Exception $e) {
        //An exception has occurred rollback the transaction
        $connection->rollback();
    }

除了标准的事务，Phalcon\\Db提供了内置支持 `nested transactions`_(如果数据库系统支持的话)。 当你第二次调用begin()方法，一个嵌套的事务就被创建了：	
	
In addition to standard transactions, Phalcon\\Db provides built-in support for `nested transactions`_
(if the database system used supports them). When you call begin() for a second time a nested transaction
is created:

.. code-block:: php

    <?php

    try {

        //Start a transaction
        $connection->begin();

        //Execute some SQL statements
        $connection->execute("DELETE `robots` WHERE `id` = 101");

        try {

            //Start a nested transaction
            $connection->begin();

            //Execute these SQL statements into the nested transaction
            $connection->execute("DELETE `robots` WHERE `id` = 102");
            $connection->execute("DELETE `robots` WHERE `id` = 103");

            //Create a save point
            $connection->commit();

        } catch(Exception $e) {
            //An error has occurred, release the nested transaction
            $connection->rollback();
        }

        //Continue, executing more SQL statements
        $connection->execute("DELETE `robots` WHERE `id` = 104");

        //Commit if everything goes well
        $connection->commit();

    } catch(Exception $e) {
        //An exception has occurred rollback the transaction
        $connection->rollback();
    }

数据库事件Database Events
-------------------------------
:doc:`Phalcon\\Db <../api/Phalcon_Db>` 可以发送事件到一个 :doc:`EventsManager <events>` 中，如果它存在的话。 一些事件当返回布尔值false可以停止操作。我们支持下面这些事件：

:doc:`Phalcon\\Db <../api/Phalcon_Db>` is able to send events to a :doc:`EventsManager <events>` if it's present.
Some events when returning boolean false could stop the active operation. The following events are supported:

+---------------------+-----------------------------------------------------------+---------------------+
| Event Name          | Triggered                                                 | Can stop operation? |
+=====================+===========================================================+=====================+
| afterConnect        | After a successfully connection to a database system      | No                  |
+---------------------+-----------------------------------------------------------+---------------------+
| beforeQuery         | Before send a SQL statement to the database system        | Yes                 |
+---------------------+-----------------------------------------------------------+---------------------+
| afterQuery          | After send a SQL statement to database system             | No                  |
+---------------------+-----------------------------------------------------------+---------------------+
| beforeDisconnect    | Before close a temporal database connection               | No                  |
+---------------------+-----------------------------------------------------------+---------------------+
| beginTransaction    | Before a transaction is going to be started               | No                  |
+---------------------+-----------------------------------------------------------+---------------------+
| rollbackTransaction | Before a transaction is rollbacked                        | No                  |
+---------------------+-----------------------------------------------------------+---------------------+
| commitTransaction   | Before a transaction is committed                         | No                  |
+---------------------+------------------------------------------------------------+--------------------+

绑定一个EventsManager给一个连接是很简单的， Phalcon\\Db 将触发这些类型为“db”的事件：

Bind an EventsManager to a connection is simple, Phalcon\\Db will trigger the events with the type "db":

.. code-block:: php

    <?php

    use Phalcon\Events\Manager as EventsManager,
        \Phalcon\Db\Adapter\Pdo\Mysql as Connection;

    $eventsManager = new EventsManager();

    //Listen all the database events
    $eventsManager->attach('db', $dbListener);

    $connection = new Connection(array(
        "host" => "localhost",
        "username" => "root",
        "password" => "secret",
        "dbname" => "invo"
    ));

    //Assign the eventsManager to the db adapter instance
    $connection->setEventsManager($eventsManager);

数据库事件中，停止操作是非常有用的，例如：如果你想要实现一个注入检查器，在发送SQL到数据库前触发：	
	
Stop SQL operations are very useful if for example you want to implement some last-resource SQL injector checker:

.. code-block:: php

    <?php

    $eventsManager->attach('db:beforeQuery', function($event, $connection) {

        //Check for malicious words in SQL statements
        if (preg_match('/DROP|ALTER/i', $connection->getSQLStatement())) {
            // DROP/ALTER operations aren't allowed in the application,
            // this must be a SQL injection!
            return false;
        }

        //It's ok
        return true;
    });

分析 SQL 语句Profiling SQL Statements
---------------------------------------------
:doc:`Phalcon\\Db <../api/Phalcon_Db>`包含了一个性能分析组件，叫 :doc:`Phalcon\\Db\\Profiler <../api/Phalcon_Db_Profiler>`，它被用于分析数据库的操作性能以便诊断性能问题，并发现瓶颈。 使用 doc:`Phalcon\\Db\\Profiler <../api/Phalcon_Db_Profiler>` 来分析数据库真的很简单:

:doc:`Phalcon\\Db <../api/Phalcon_Db>` includes a profiling component called :doc:`Phalcon\\Db\\Profiler <../api/Phalcon_Db_Profiler>`, that is used to analyze the performance of database operations so as to diagnose performance problems and discover bottlenecks.

Database profiling is really easy With :doc:`Phalcon\\Db\\Profiler <../api/Phalcon_Db_Profiler>`:

.. code-block:: php

    <?php

    use Phalcon\Events\Manager as EventsManager,
        Phalcon\Db\Profiler as DbProfiler;

    $eventsManager = new EventsManager();

    $profiler = new DbProfiler();

    //Listen all the database events
    $eventsManager->attach('db', function($event, $connection) use ($profiler) {
        if ($event->getType() == 'beforeQuery') {
            //Start a profile with the active connection
            $profiler->startProfile($connection->getSQLStatement());
        }
        if ($event->getType() == 'afterQuery') {
            //Stop the active profile
            $profiler->stopProfile();
        }
    });

    //Assign the events manager to the connection
    $connection->setEventsManager($eventsManager);

    $sql = "SELECT buyer_name, quantity, product_name "
         . "FROM buyers "
         . "LEFT JOIN products ON buyers.pid = products.id";

    // Execute a SQL statement
    $connection->query($sql);

    // Get the last profile in the profiler
    $profile = $profiler->getLastProfile();

    echo "SQL Statement: ", $profile->getSQLStatement(), "\n";
    echo "Start Time: ", $profile->getInitialTime(), "\n";
    echo "Final Time: ", $profile->getFinalTime(), "\n";
    echo "Total Elapsed Time: ", $profile->getTotalElapsedSeconds(), "\n";

你也可以基于 :doc:`Phalcon\\Db\\Profiler <../api/Phalcon_Db_Profiler>` 建立你自己的分析器类，以记录SQL语句发送到数据库的实时统计：	
	
You can also create your own profile class based on :doc:`Phalcon\\Db\\Profiler <../api/Phalcon_Db_Profiler>` to record real time statistics of the statements sent to the database system:

.. code-block:: php

    <?php

    use Phalcon\Events\Manager as EventsManager,
        Phalcon\Db\Profiler as Profiler,
        Phalcon\Db\Profiler\Item as Item;

    class DbProfiler extends Profiler
    {

        /**
         * Executed before the SQL statement will sent to the db server
         */
        public function beforeStartProfile(Item $profile)
        {
            echo $profile->getSQLStatement();
        }

        /**
         * Executed after the SQL statement was sent to the db server
         */
        public function afterEndProfile(Item $profile)
        {
            echo $profile->getTotalElapsedSeconds();
        }

    }

    //Create an EventsManager
    $eventsManager = new EventsManager();

    //Create a listener
    $dbProfiler = new DbProfiler();

    //Attach the listener listening for all database events
    $eventsManager->attach('db', $dbProfiler);

记录 SQL 语句Logging SQL Statements
-----------------------------------------
使用例如 :doc:`Phalcon\\Db <../api/Phalcon_Db>` 的高级抽象组件操作数据库，被发送到数据库中执行的原生SQL语句是难以获知的。使用 :doc:`Phalcon\\Logger <../api/Phalcon_Logger>` 和 :doc:`Phalcon\\Db <../api/Phalcon_Db>` 来配合使用，可以在数据库抽象层上提供记录的功能。

Using high-level abstraction components such as :doc:`Phalcon\\Db <../api/Phalcon_Db>` to access a database, it is difficult to understand which statements are sent to the database system. :doc:`Phalcon\\Logger <../api/Phalcon_Logger>` interacts with :doc:`Phalcon\\Db <../api/Phalcon_Db>`, providing logging capabilities on the database abstraction layer.

.. code-block:: php

    <?php

    use Phalcon\Logger,
        Phalcon\Events\Manager as EventsManager,
        Phalcon\Logger\Adapter\File as FileLogger;

    $eventsManager = new EventsManager();

    $logger = new FileLogger("app/logs/db.log");

    //Listen all the database events
    $eventsManager->attach('db', function($event, $connection) use ($logger) {
        if ($event->getType() == 'beforeQuery') {
            $logger->log($connection->getSQLStatement(), Logger::INFO);
        }
    });

    //Assign the eventsManager to the db adapter instance
    $connection->setEventsManager($eventsManager);

    //Execute some SQL statement
    $connection->insert(
        "products",
        array("Hot pepper", 3.50),
        array("name", "price")
    );

如上操作，文件 *app/logs/db.log* 将包含像下面这样的信息：	
	
As above, the file *app/logs/db.log* will contain something like this:

.. code-block:: php

    [Sun, 29 Apr 12 22:35:26 -0500][DEBUG][Resource Id #77] INSERT INTO products
    (name, price) VALUES ('Hot pepper', 3.50)


自定义日志记录器Implementing your own Logger
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
你可以实现你自己的日志类来记录数据库的所有操作，通过创建一个实现了”log”方法的类。 这个方法需要接受一个字符串作为第一个参数。你可以把日志类的对象传递给Phalcon\\Db::setLogger()， 这样执行SQL时将调用这个对象的log方法去记录。

You can implement your own logger class for database queries, by creating a class that implements a single method called "log".
The method needs to accept a string as the first argument. You can then pass your logging object to Phalcon\\Db::setLogger(),
and from then on any SQL statement executed will call that method to log the results.

获取数据库表与视图信息Describing Tables/Views
-------------------------------------------------
Phalcon\Db 也提供了方法去检索详细的表和视图信息：

:doc:`Phalcon\\Db <../api/Phalcon_Db>` also provides methods to retrieve detailed information about tables and views:

.. code-block:: php

    <?php

    // Get tables on the test_db database
    $tables = $connection->listTables("test_db");

    // Is there a table 'robots' in the database?
    $exists = $connection->tableExists("robots");

    // Get name, data types and special features of 'robots' fields
    $fields = $connection->describeColumns("robots");
    foreach ($fields as $field) {
        echo "Column Type: ", $field["Type"];
    }

    // Get indexes on the 'robots' table
    $indexes = $connection->describeIndexes("robots");
    foreach ($indexes as $index) {
        print_r($index->getColumns());
    }

    // Get foreign keys on the 'robots' table
    $references = $connection->describeReferences("robots");
    foreach ($references as $reference) {
        // Print referenced columns
        print_r($reference->getReferencedColumns());
    }

一个表的详细描述信息和MYSQL的describe命令返回的信息非常相似，它包含以下信息：	
	
A table description is very similar to the MySQL describe command, it contains the following information:

+-------+----------------------------------------------------+
| Index | Description                                        |
+=======+====================================================+
| Field | Field's name                                       |
+-------+----------------------------------------------------+
| Type  | Column Type                                        |
+-------+----------------------------------------------------+
| Key   | Is the column part of the primary key or an index? |
+-------+----------------------------------------------------+
| Null  | Does the column allow null values?                 |
+-------+----------------------------------------------------+

对于被支持的数据库系统，获取视图的信息的方法也被实现了：

Methods to get information about views are also implemented for every supported database system:

.. code-block:: php

    <?php

    // Get views on the test_db database
    $tables = $connection->listViews("test_db");

    // Is there a view 'robots' in the database?
    $exists = $connection->viewExists("robots");

创建/修改/删除表Creating/Altering/Dropping Tables
----------------------------------------------------
不同的数据库系统（MySQL,Postgresql等）通过了CREATE, ALTER 或 DROP命令提供了用于创建，修改或删除表的功能。但是不同的数据库语法不同。 :doc:`Phalcon\\Db <../api/Phalcon_Db>` 提供了统一的接口来改变表，而不需要区分基于目标存储系统上的SQL语法。

Different database systems (MySQL, Postgresql etc.) offer the ability to create, alter or drop tables with the use of
commands such as CREATE, ALTER or DROP. The SQL syntax differs based on which database system is used.
:doc:`Phalcon\\Db <../api/Phalcon_Db>` offers a unified interface to alter tables, without the need to
differentiate the SQL syntax based on the target storage system.

创建数据库表Creating Tables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
下面这个例子展示了怎么建立一个表：

The following example shows how to create a table:

.. code-block:: php

    <?php

    use \Phalcon\Db\Column as Column;

    $connection->createTable(
        "robots",
        null,
        array(
           "columns" => array(
                new Column("id",
                    array(
                        "type"          => Column::TYPE_INTEGER,
                        "size"          => 10,
                        "notNull"       => true,
                        "autoIncrement" => true,
                    )
                ),
                new Column("name",
                    array(
                        "type"    => Column::TYPE_VARCHAR,
                        "size"    => 70,
                        "notNull" => true,
                    )
                ),
                new Column("year",
                    array(
                        "type"    => Column::TYPE_INTEGER,
                        "size"    => 11,
                        "notNull" => true,
                    )
                )
            )
        )
    );

Phalcon\\Db::createTable()接受一个描述数据库表相关的数组。字段被定义成class :doc:`Phalcon\\Db\\Column <../api/Phalcon_Db_Column>` 。 下表列出了可用于定义字段的选项：	
	
Phalcon\\Db::createTable() accepts an associative array describing the table. Columns are defined with the class
:doc:`Phalcon\\Db\\Column <../api/Phalcon_Db_Column>`. The table below shows the options available to define a column:

+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| Option          | Description                                                                                                                                | Optional |
+=================+============================================================================================================================================+==========+
| "type"          | Column type. Must be a Phalcon\\Db\\Column constant (see below for a list)                                                                 | No       |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "primary"       | True if the column is part of the table's primary                                                                                          | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "size"          | Some type of columns like VARCHAR or INTEGER may have a specific size                                                                      | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "scale"         | DECIMAL or NUMBER columns may be have a scale to specify how many decimals should be stored                                                | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "unsigned"      | INTEGER columns may be signed or unsigned. This option does not apply to other types of columns                                            | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "notNull"       | Column can store null values?                                                                                                              | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "autoIncrement" | With this attribute column will filled automatically with an auto-increment integer. Only one column in the table can have this attribute. | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "bind"          | One of the BIND_TYPE_* constants telling how the column must be binded before save it                                                      | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "first"         | Column must be placed at first position in the column order                                                                                | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "after"         | Column must be placed after indicated column                                                                                               | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+

Phalcon\Db 支持下面的数据库字段类型:

Phalcon\\Db supports the following database column types:

* Phalcon\\Db\\Column::TYPE_INTEGER
* Phalcon\\Db\\Column::TYPE_DATE
* Phalcon\\Db\\Column::TYPE_VARCHAR
* Phalcon\\Db\\Column::TYPE_DECIMAL
* Phalcon\\Db\\Column::TYPE_DATETIME
* Phalcon\\Db\\Column::TYPE_CHAR
* Phalcon\\Db\\Column::TYPE_TEXT

传入Phalcon\Db::createTable() 的相关数组可能含有的下标：

The associative array passed in Phalcon\\Db::createTable() can have the possible keys:

+--------------+----------------------------------------------------------------------------------------------------------------------------------------+----------+
| Index        | Description                                                                                                                            | Optional |
+==============+========================================================================================================================================+==========+
| "columns"    | An array with a set of table columns defined with :doc:`Phalcon\\Db\\Column <../api/Phalcon_Db_Column>`                                | No       |
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+----------+
| "indexes"    | An array with a set of table indexes defined with :doc:`Phalcon\\Db\\Index <../api/Phalcon_Db_Index>`                                  | Yes      |
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+----------+
| "references" | An array with a set of table references (foreign keys) defined with :doc:`Phalcon\\Db\\Reference <../api/Phalcon_Db_Reference>`        | Yes      |
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+----------+
| "options"    | An array with a set of table creation options. These options often relate to the database system in which the migration was generated. | Yes      |
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+----------+

修改数据库表Altering Tables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
随着你的应用的增长，作为一个重构的一部分，或者增加新功能，你也许需要修改你的数据库。 因为不是所有的数据库允许你修改已存在的字段或者添加字段在2个已存在的字段之间。所以 Phalcon\Db 会受到数据库系统的这些限制。

As your application grows, you might need to alter your database, as part of a refactoring or adding new features.
Not all database systems allow to modify existing columns or add columns between two existing ones. :doc:`Phalcon\\Db <../api/Phalcon_Db>`
is limited by these constraints.

.. code-block:: php

    <?php

    use Phalcon\Db\Column as Column;

    // Adding a new column
    $connection->addColumn("robots", null,
        new Column("robot_type", array(
            "type"    => Column::TYPE_VARCHAR,
            "size"    => 32,
            "notNull" => true,
            "after"   => "name"
        ))
    );

    // Modifying an existing column
    $connection->modifyColumn("robots", null, new Column("name", array(
        "type" => Column::TYPE_VARCHAR,
        "size" => 40,
        "notNull" => true,
    )));

    // Deleting the column "name"
    $connection->dropColumn("robots", null, "name");


删除数据库表Dropping Tables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
删除数据库表的例子:

Examples on dropping tables:

.. code-block:: php

    <?php

    // Drop table robot from active database
    $connection->dropTable("robots");

    //Drop table robot from database "machines"
    $connection->dropTable("robots", "machines");

.. _PDO: http://www.php.net/manual/en/book.pdo.php
.. _`nested transactions`: http://en.wikipedia.org/wiki/Nested_transaction
