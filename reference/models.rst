使用模型Working with Models
=============================
模型代表了应用程序中的信息（数据）和处理数据的规则。模型主要用于管理与相应数据库表进行交互的规则。 大多数情况中，在应用程序中，数据库中每个表将对应一个模型。 应用程序中的大部分业务逻辑都将集中在模型里。

A model represents the information (data) of the application and the rules to manipulate that data. Models are primarily used for managing
the rules of interaction with a corresponding database table. In most cases, each table in your database will correspond to one model in
your application. The bulk of your application's business logic will be concentrated in the models.

:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 是 Phalcon 应用程序中所有模型的基类。它保证了数据库的独立性，基本的 CURD 操作， 高级的查询功能，多表关联等功能。 Phalcon\Mvc\Model 不需要直接使用 SQL 语句，因为它的转换方法，会动态的调用相应的数据库引擎进行处理。

:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` is the base for all models in a Phalcon application. It provides database independence, basic
CRUD functionality, advanced finding capabilities, and the ability to relate models to one another, among other services.
:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` avoids the need of having to use SQL statements because it translates methods dynamically
to the respective database engine operations.

.. highlights::

    模型是数据库的高级抽象层。如果您想进行低层次的数据库操作，您可以查看 :doc:`Phalcon\\Db <../api/Phalcon_Db>` 组件文档。

    Models are intended to work on a database high layer of abstraction. If you need to work with databases at a lower level check out the
    :doc:`Phalcon\\Db <../api/Phalcon_Db>` component documentation.

创建模型Creating Models
-------------------------
模型是一个继承自 :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 的一个类。 它必须放到 models 文件夹。一个模型文件必须包含一个类， 同时它的类名必须符合驼峰命名法：

A model is a class that extends from :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`. It must be placed in the models directory. A model
file must contain a single class; its class name should be in camel case notation:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

    }

上面的例子显示了 “Robots” 模型的实现。 需要注意的是 Robots 继承自 :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 。 因此，Robots 模型拥有了大量继承自该组件功能，包括基本的数据库 CRUD (Create, Read, Update, Delete) 操作，数据验证以及复杂的搜索支持，并且可以同时关联多个模型。	
	
The above example shows the implementation of the "Robots" model. Note that the class Robots inherits from :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`.
This component provides a great deal of functionality to models that inherit it, including basic database
CRUD (Create, Read, Update, Delete) operations, data validation, as well as sophisticated search support and the ability to relate multiple models
with each other.

.. highlights::

    如果使用 PHP 5.4/5.5 建议在模型中预先定义好所有的列，这样可以减少模型内存的开销以及内存分配。

    If you're using PHP 5.4/5.5 it is recommended you declare each column that makes part of the model in order to save
    memory and reduce the memory allocation.

默认情况下，模型 “Robots” 对应的是数据库表 “robots”， 如果想映射到其他数据库表，可以使用 getSource() 方法：	
	
By default, the model "Robots" will refer to the table "robots". If you want to manually specify another name for the mapping table,
you can use the getSource() method:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function getSource()
        {
            return "the_robots";
        }

    }

模型 Robots 现在映射到了 “the_robots” 表。initialize() 方法可以帮助在模型中建立自定义行为，例如指定不同的数据库表。 initialize() 方法在请求期间只被调用一次。	
	
The model Robots now maps to "the_robots" table. The initialize() method aids in setting up the model with a custom behavior i.e. a different table.
The initialize() method is only called once during the request.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function initialize()
        {
            $this->setSource("the_robots");
        }

    }

initialize() 方法在请求期间仅会被调用一次，目的是为应用中所有该模型的实例进行初始化。如果需要为每一个实例在创建的时候单独进行初始化， 可以使用 ‘onConstruct’ 事件：	
	
The initialize() method is only called once during the request, it's intended to perform initializations that apply for
all instances of the model created within the application. If you want to perform initialization tasks for every instance
created you can 'onConstruct':

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function onConstruct()
        {
            //...
        }

    }

公共属性对比设置与取值 Setters/GettersPublic properties vs. Setters/Getters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
模型可以通过公共属性的方式实现，意味着模型的所有属性在实例化该模型的地方可以无限制的读取和更新。

Models can be implemented with properties of public scope, meaning that each property can be read/updated
from any part of the code that has instantiated that model class without any restrictions:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {
        public $id;

        public $name;

        public $price;
    }

通过使用 getters/setters 方法，可以控制哪些属性可以公开访问，并且对属性值执行不同的形式的转换，同时可以保存在模型中的数据添加相应的验证规则。	
	
By using getters and setters you can control which properties are visible publicly perform various transformations
to the data (which would be impossible otherwise) and also add validation rules to the data stored in the object:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {
        protected $id;

        protected $name;

        protected $price;

        public function getId()
        {
            return $this->id;
        }

        public function setName($name)
        {
            //The name is too short?
            if (strlen($name) < 10) {
                throw new \InvalidArgumentException('The name is too short');
            }
            $this->name = $name;
        }

        public function getName()
        {
            return $this->name;
        }

        public function setPrice($price)
        {
            //Negative prices aren't allowed
            if ($price < 0) {
                throw new \InvalidArgumentException('Price can\'t be negative');
            }
            $this->price = $price;
        }

        public function getPrice()
        {
            //Convert the value to double before be used
            return (double) $this->price;
        }
    }

公共属性的方式可以在开发中降低复杂度。而 getters/setters 的实现方式可以显著的增强应用的可测试性、扩展性和可维护性。 开发人员可以自己决定哪一种策略更加适合自己开发的应用。ORM同时兼容这两种方法。	
	
Public properties provide less complexity in development. However getters/setters can heavily increase the testability,
extensibility and maintainability of applications. Developers can decide which strategy is more appropriate for the
application they are creating. The ORM is compatible with both schemes of defining properties.

模型放入命名空间Models in Namespaces
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
命名空间可以用来避免类名的冲突。ORM通过类名来映射相应的表名。比如 ‘Robots’：

Namespaces can be used to avoid class name collision. The mapped table is taken from the class name, in this case 'Robots':

.. code-block:: php

    <?php

    namespace Store\Toys;

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

    }

	
理解记录对象Understanding Records To Objects
--------------------------------
每个模型的实例对应一条数据表中的记录。可以方便的通过读取对象的属性来访问相应的数据。比如， 一个表 “robots” 有如下数据：

Every instance of a model represents a row in the table. You can easily access record data by reading object properties. For example,
for a table "robots" with the records:

.. code-block:: bash

    mysql> select * from robots;
    +----+------------+------------+------+
    | id | name       | type       | year |
    +----+------------+------------+------+
    |  1 | Robotina   | mechanical | 1972 |
    |  2 | Astro Boy  | mechanical | 1952 |
    |  3 | Terminator | cyborg     | 2029 |
    +----+------------+------------+------+
    3 rows in set (0.00 sec)

你可以通过主键找到某一条记录并且打印它的名称：	
	
You could find a certain record by its primary key and then print its name:

.. code-block:: php

    <?php

    // Find record with id = 3
    $robot = Robots::findFirst(3);

    // Prints "Terminator"
    echo $robot->name;

一旦记录被加载到内存中之后，你可以修改它的数据并保存所做的修改：	
	
Once the record is in memory, you can make modifications to its data and then save changes:

.. code-block:: php

    <?php

    $robot       = Robots::findFirst(3);
    $robot->name = "RoboCop";
    $robot->save();

如上所示，不需要写任何SQL语句。 :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 为web应用提供了高层数据库抽象。	
	
As you can see, there is no need to use raw SQL statements. :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` provides high database
abstraction for web applications.

查找记录Finding Records
-----------------------------
:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 为数据查询提供了多种方法。下面的例子将演示如何从一个模型中查找一条或者多条记录：

:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` also offers several methods for querying records. The following examples will show you
how to query one or more records from a model:

.. code-block:: php

    <?php

    // How many robots are there?
    $robots = Robots::find();
    echo "There are ", count($robots), "\n";

    // How many mechanical robots are there?
    $robots = Robots::find("type = 'mechanical'");
    echo "There are ", count($robots), "\n";

    // Get and print virtual robots ordered by name
    $robots = Robots::find(array(
        "type = 'virtual'",
        "order" => "name"
    ));
    foreach ($robots as $robot) {
        echo $robot->name, "\n";
    }

    // Get first 100 virtual robots ordered by name
    $robots = Robots::find(array(
        "type = 'virtual'",
        "order" => "name",
        "limit" => 100
    ));
    foreach ($robots as $robot) {
       echo $robot->name, "\n";
    }

.. highlights::

    如果想通过额外的数据（用户输入数据）查询记录必须要使用`Binding Parameters`_。

    If you want find record by external data (such as user input) or variable data you must use `Binding Parameters`_.

你可以使用 findFirst() 方法获取第一条符合查询条件的结果：	
	
You could also use the findFirst() method to get only the first record matching the given criteria:

.. code-block:: php

    <?php

    // What's the first robot in robots table?
    $robot = Robots::findFirst();
    echo "The robot name is ", $robot->name, "\n";

    // What's the first mechanical robot in robots table?
    $robot = Robots::findFirst("type = 'mechanical'");
    echo "The first mechanical robot name is ", $robot->name, "\n";

    // Get first virtual robot ordered by name
    $robot = Robots::findFirst(array("type = 'virtual'", "order" => "name"));
    echo "The first virtual robot name is ", $robot->name, "\n";

find() 和 findFirst() 方法都接受关联数组作为查询条件：	
	
Both find() and findFirst() methods accept an associative array specifying the search criteria:

.. code-block:: php

    <?php

    $robot = Robots::findFirst(array(
        "type = 'virtual'",
        "order" => "name DESC",
        "limit" => 30
    ));

    $robots = Robots::find(array(
        "conditions" => "type = ?1",
        "bind"       => array(1 => "virtual")
    ));

可用的查询选项如下	
	
The available query options are:

+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| Parameter   | Description                                                                                                                                                                                        | Example                                                                 |
+=============+====================================================================================================================================================================================================+=========================================================================+
| conditions  | Search conditions for the find operation. Is used to extract only those records that fulfill a specified criterion. By default Phalcon\\Mvc\\Model assumes the first parameter are the conditions. | "conditions" => "name LIKE 'steve%'"                                    |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| columns     | Return specific columns instead of the full columns in the model. When using this option an incomplete object is returned                                                                          | "columns" => "id, name"                                                 |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| bind        | Bind is used together with options, by replacing placeholders and escaping values thus increasing security                                                                                         | "bind" => array("status" => "A", "type" => "some-time")                 |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| bindTypes   | When binding parameters, you can use this parameter to define additional casting to the bound parameters increasing even more the security                                                         | "bindTypes" => array(Column::BIND_PARAM_STR, Column::BIND_PARAM_INT)    |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| order       | Is used to sort the resultset. Use one or more fields separated by commas.                                                                                                                         | "order" => "name DESC, status"                                          |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| limit       | Limit the results of the query to results to certain range                                                                                                                                         | "limit" => 10 / "limit" => array("number" => 10, "offset" => 5)         |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| group       | Allows to collect data across multiple records and group the results by one or more columns                                                                                                        | "group" => "name, status"                                               |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| for_update  | With this option, :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` reads the latest available data, setting exclusive locks on each row it reads                                              | "for_update" => true                                                    |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| shared_lock | With this option, :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` reads the latest available data, setting shared locks on each row it reads                                                 | "shared_lock" => true                                                   |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| cache       | Cache the resultset, reducing the continuous access to the relational system                                                                                                                       | "cache" => array("lifetime" => 3600, "key" => "my-find-key")            |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| hydration   | Sets the hydration strategy to represent each returned record in the result                                                                                                                        | "hydration" => Resultset::HYDRATE_OBJECTS                               |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+

如果你愿意，除了使用数组作为查询参数外，还可以通过一种面向对象的方式来创建查询：

If you prefer, there is also available a way to create queries in an object-oriented way, instead of using an array of parameters:

.. code-block:: php

    <?php

    $robots = Robots::query()
        ->where("type = :type:")
        ->andWhere("year < 2000")
        ->bind(array("type" => "mechanical"))
        ->order("name")
        ->execute();

静态方法 query() 返回一个对IDE自动完成友好的 :doc:`Phalcon\\Mvc\\Model\\Criteria <../api/Phalcon_Mvc_Model_Criteria>`对象。		
		
The static method query() returns a :doc:`Phalcon\\Mvc\\Model\\Criteria <../api/Phalcon_Mvc_Model_Criteria>` object that is friendly with IDE autocompleters.

所有查询在内部都以 PHQL 查询的方式处理。PHQL是一个高层的、面向对象的类SQL语言。通过PHQL语言你可以使用更多的比如join其他模型、定义分组、添加聚集等特性。

All the queries are internally handled as :doc:`PHQL <phql>` queries. PHQL is a high-level, object-oriented and SQL-like language.
This language provide you more features to perform queries like joining other models, define groupings, add aggregations etc.

最后，还有一个 findFirstBy<property-name>() 方法。这个方法扩展了前面提及的 “findFirst()” 方法。它允许您利用方法名中的属性名称，通过将要搜索的该字段的内容作为参数传给它，来快速从一个表执行检索操作。

Lastly, there is the findFirstBy<property-name>() method. This method expands on the "findFirst()" method mentioned earlier. It allows you to quickly perform a
retrieval from a table by using the property name in the method itself and passing it a parameter that contains the data you want to search for in that column.

还是用上面用过的 Robots 模型来举例说明：

An example is in order, so taking our Robots model mentioned earlier :

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {
        public $id;

        public $name;

        public $price;
    }

我们这里有3个属性：$id, $name 和 $price。因此，我们以想要查询第一个名称为 ‘Terminator’ 的记录为例，可以这样写：	
	
We have three properties to work with here. $id, $name and $price. So, let's say you want to retrieve the first record in the table with the name
'Terminator'. This could be written like so :

.. code-block:: php

    <?php

    $name  = "Terminator";
    $robot = Robots::findFirstByName($name);

    if($robot){
        $this->flash->success("The first robot with the name " . $name . " cost " . $robot->price ".");
    }else{
        $this->flash->error("There were no robots found in our table with the name " . $name ".");
    }

请注意我们在方法调用中用的是 ‘Name’，并向它传递了变量 $name，$name 的值就是我们想要找的记录的名称。另外注意，当我们的查询找到了符合的记录后，这个记录的其他属性也都是可用的。	
	
Notice that we used 'Name' in the method call and passed the variable $name to it, which contains the name we are looking for in our table. Notice also that
when we find a match with our query, all the other properties are available to us as well.

模型结果集Model Resultsets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
findFirst() 方法直接返回一个被调用对象的实例（如果有结果返回的话），而 find() 方法返回一个 :doc:`Phalcon\\Mvc\\Model\\Resultset\\Simple <../api/Phalcon_Mvc_Model_Resultset_Simple>` 对象。这个对象也封装进了所有结果集的功能，比如遍历、查找特定的记录、统计等等。

While findFirst() returns directly an instance of the called class (when there is data to be returned), the find() method returns a
:doc:`Phalcon\\Mvc\\Model\\Resultset\\Simple <../api/Phalcon_Mvc_Model_Resultset_Simple>`. This is an object that encapsulates all the functionality
a resultset has like traversing, seeking specific records, counting, etc.

这些对象比一般数组功能更强大。最大的特点是 :doc:`Phalcon\\Mvc\\Model\\Resultset <../api/Phalcon_Mvc_Model_Resultset>` 每时每刻只有一个结果在内存中。这对操作大数据量时的内存管理相当有帮助。

These objects are more powerful than standard arrays. One of the greatest features of the :doc:`Phalcon\\Mvc\\Model\\Resultset <../api/Phalcon_Mvc_Model_Resultset>`
is that at any time there is only one record in memory. This greatly helps in memory management especially when working with large amounts of data.

.. code-block:: php

    <?php

    // Get all robots
    $robots = Robots::find();

    // Traversing with a foreach
    foreach ($robots as $robot) {
        echo $robot->name, "\n";
    }

    // Traversing with a while
    $robots->rewind();
    while ($robots->valid()) {
        $robot = $robots->current();
        echo $robot->name, "\n";
        $robots->next();
    }

    // Count the resultset
    echo count($robots);

    // Alternative way to count the resultset
    echo $robots->count();

    // Move the internal cursor to the third robot
    $robots->seek(2);
    $robot = $robots->current();

    // Access a robot by its position in the resultset
    $robot = $robots[5];

    // Check if there is a record in certain position
    if (isset($robots[3])) {
       $robot = $robots[3];
    }

    // Get the first record in the resultset
    $robot = $robots->getFirst();

    // Get the last record
    $robot = $robots->getLast();

Phalcon 的结果集模拟了可滚动的游标，你可以通过位置，或者内部指针去访问任何一条特定的记录。注意有一些数据库系统不支持滚动游标，这就使得查询会被重复执行， 以便回放光标到最开始的位置，然后获得相应的记录。类似地，如果多次遍历结果集，那么必须执行相同的查询次数。	
	
Phalcon's resultsets emulate scrollable cursors, you can get any row just by accessing its position, or seeking the internal pointer
to a specific position. Note that some database systems don't support scrollable cursors, this forces to re-execute the query
in order to rewind the cursor to the beginning and obtain the record at the requested position. Similarly, if a resultset
is traversed several times, the query must be executed the same number of times.

将大数据量的查询结果存储在内存会消耗很多资源，正因为如此，分成每32行一块从数据库中获得结果集，以减少重复执行查询请求的次数，在一些情况下也节省内存。

Storing large query results in memory could consume many resources, because of this, resultsets are obtained
from the database in chunks of 32 rows reducing the need for re-execute the request in several cases also saving memory.

注意结果集可以序列化后保存在一个后端缓存里面。 :doc:`Phalcon\\Cache <cache>`可以用来实现这个。但是，序列化数据会导致 :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 将从数据库检索到的所有数据以一个数组的方式保存，因此在这样执行的地方会消耗更多的内存。

Note that resultsets can be serialized and stored in a cache backend. :doc:`Phalcon\\Cache <cache>` can help with that task. However,
serializing data causes :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` to retrieve all the data from the database in an array,
thus consuming more memory while this process takes place.

.. code-block:: php

    <?php

    // Query all records from model parts
    $parts = Parts::find();

    // Store the resultset into a file
    file_put_contents("cache.txt", serialize($parts));

    // Get parts from file
    $parts = unserialize(file_get_contents("cache.txt"));

    // Traverse the parts
    foreach ($parts as $part) {
       echo $part->id;
    }

过滤结果集Filtering Resultsets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
过滤数据最有效的方法是设置一些查询条件，数据库会利用表的索引快速返回数据。Phalcon 额外的允许你通过任何数据库不支持的方式过滤数据。

The most efficient way to filter data is setting some search criteria, databases will use indexes set on tables to return data faster.
Phalcon additionally allows you to filter the data using PHP using any resource that is not available in the database:

.. code-block:: php

    <?php

    $customers = Customers::find()->filter(function($customer) {

        //Return only customers with a valid e-mail
        if (filter_var($customer->email, FILTER_VALIDATE_EMAIL)) {
            return $customer;
        }

    });

绑定参数Binding Parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在 :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 中也支持绑定参数。即使使用绑定参数对性能有一点很小的影响，还是强烈建议您使用这种方法，以消除代码受SQL注入攻击的可能性。 绑定参数支持字符串和整数占位符。实现方法如下：

Bound parameters are also supported in :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`. Although there is a minimal performance
impact by using bound parameters, you are encouraged to use this methodology so as to eliminate the possibility of your code being subject
to SQL injection attacks. Both string and integer placeholders are supported. Binding parameters can simply be achieved as follows:

.. code-block:: php

    <?php

    // Query robots binding parameters with string placeholders
    $conditions = "name = :name: AND type = :type:";

    //Parameters whose keys are the same as placeholders
    $parameters = array(
        "name" => "Robotina",
        "type" => "maid"
    );

    //Perform the query
    $robots = Robots::find(array(
        $conditions,
        "bind" => $parameters
    ));

    // Query robots binding parameters with integer placeholders
    $conditions = "name = ?1 AND type = ?2";
    $parameters = array(1 => "Robotina", 2 => "maid");
    $robots     = Robots::find(array(
        $conditions,
        "bind" => $parameters
    ));

    // Query robots binding parameters with both string and integer placeholders
    $conditions = "name = :name: AND type = ?1";

    //Parameters whose keys are the same as placeholders
    $parameters = array(
        "name" => "Robotina",
        1      => "maid"
    );

    //Perform the query
    $robots = Robots::find(array(
        $conditions,
        "bind" => $parameters
    ));

当使用数字占位符,您需要定义它们为整数即1或2。使用“1”或“2”被认为是字符串而不是数字,所以占位符不能成功地替换了。	
	
When using numeric placeholders, you will need to define them as integers i.e. 1 or 2. In this case "1" or "2" are considered strings
and not numbers, so the placeholder could not be successfully replaced.

使用PDO_字符串会被自动转义。这个函数考虑了连接字符集,因此推荐在数据库中的连接参数或配置中定义正确的字符集,定义一个错误的字符集将会在存储或检索数据产生错误的结果。

Strings are automatically escaped using PDO_. This function takes into account the connection charset, so its recommended to define
the correct charset in the connection parameters or in the database configuration, as a wrong charset will produce undesired effects
when storing or retrieving data.

另外你可以设置参数“bindTypes”,这个可以根据参数类型进行绑定:

Additionally you can set the parameter "bindTypes", this allows defining how the parameters should be bound according to its data type:

.. code-block:: php

    <?php

    use Phalcon\Db\Column;

    //Bind parameters
    $parameters = array(
        "name" => "Robotina",
        "year" => 2008
    );

    //Casting Types
    $types = array(
        "name" => Column::BIND_PARAM_STR,
        "year" => Column::BIND_PARAM_INT
    );

    // Query robots binding parameters with string placeholders
    $robots = Robots::find(array(
        "name = :name: AND year = :year:",
        "bind"      => $parameters,
        "bindTypes" => $types
    ));

.. highlights::

    因为默认绑定类型为\\Phalcon\\Db\\Column::BIND_PARAM_STR,所以如果所有列的类型都是字符就不需要再指定“bindTypes”参数。

    Since the default bind-type is \\Phalcon\\Db\\Column::BIND_PARAM_STR, there is no need to specify the
    "bindTypes" parameter if all of the columns are of that type.

绑定参数可用于所有查询方法,如find(),findFirst(),count(),sum(),average()等。
	
Bound parameters are available for all query methods such as find() and findFirst() but also the calculation
methods like count(), sum(), average() etc.

初始化/准备获取记录Initializing/Preparing fetched records
-----------------------------------------------------------------
普通的流程是从数据库获取数据并处理后供程序后续使用。但是我们也可以在数据模型中实现'afterFetch'这个方法，它将在实例执行后立刻被调用被分配相应的数据:

May be the case that after obtaining a record from the database is necessary to initialise the data before
being used by the rest of the application. You can implement the method 'afterFetch' in a model, this event
will be executed just after create the instance and assign the data to it:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public $id;

        public $name;

        public $status;

        public function beforeSave()
        {
            //Convert the array into a string
            $this->status = join(',', $this->status);
        }

        public function afterFetch()
        {
            //Convert the string to an array
            $this->status = explode(',', $this->status);
        }
    }

如果公共属性和getter/setter一块使用,当访问这些属性的时候就可以完成初始化:	
	
If you use getters/setters instead of/or together with public properties, you can initialize the field once it is
accessed:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {
        public $id;

        public $name;

        public $status;

        public function getStatus()
        {
            return explode(',', $this->status);
        }

    }

模型关系Relationships between Models
----------------------------------------
有四种类型的关系:一对一,一对多,多对一和多对多。关系可能是单向或双向,每个可以是简单的(一个对一个模型)或更复杂的(模型)的组合。模型管理器管理这些关系的外键约束,这些定义保证了引用的完整性，让我们容易和快速访问一个模型的相关记录。通过关系的实现, 很容易以统一的方式从相互关联的模型中访问每一条记录。

There are four types of relationships: one-on-one, one-to-many, many-to-one and many-to-many. The relationship may be
unidirectional or bidirectional, and each can be simple (a one to one model) or more complex (a combination of models).
The model manager manages foreign key constraints for these relationships, the definition of these helps referential
integrity as well as easy and fast access of related records to a model. Through the implementation of relations,
it is easy to access data in related models from each record in a uniform way.

单项关系Unidirectional relationships
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Unidirectional relations are those that are generated in relation to one another but not vice versa.

双向关系Bidirectional relations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The bidirectional relations build relationships in both models and each model defines the inverse relationship of the other.

定义关系Defining relationships
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在Phalcon中关系必须在模型的initialize()中定义。belongsTo(),hasOne(),hasMany()和hasManyToMany()定义了从当前模型中一个或多个字段之间的关系到另一个模型中。上述每种方法都需要三个参数:本地字段，引用模型,引用字段。

In Phalcon, relationships must be defined in the initialize() method of a model. The methods belongsTo(), hasOne(),
hasMany() and hasManyToMany() define the relationship between one or more fields from the current model to fields in
another model. Each of these methods requires 3 parameters: local fields, referenced model, referenced fields.

+---------------+----------------------------+
| Method        | Description                |
+===============+============================+
| hasMany       | Defines a 1-n relationship |
+---------------+----------------------------+
| hasOne        | Defines a 1-1 relationship |
+---------------+----------------------------+
| belongsTo     | Defines a n-1 relationship |
+---------------+----------------------------+
| hasManyToMany | Defines a n-n relationship |
+---------------+----------------------------+

下面是作为我们演示关系的例子的三个表:

The following schema shows 3 tables whose relations will serve us as an example regarding relationships:

.. code-block:: sql

    CREATE TABLE `robots` (
        `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
        `name` varchar(70) NOT NULL,
        `type` varchar(32) NOT NULL,
        `year` int(11) NOT NULL,
        PRIMARY KEY (`id`)
    );

    CREATE TABLE `robots_parts` (
        `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
        `robots_id` int(10) NOT NULL,
        `parts_id` int(10) NOT NULL,
        `created_at` DATE NOT NULL,
        PRIMARY KEY (`id`),
        KEY `robots_id` (`robots_id`),
        KEY `parts_id` (`parts_id`)
    );

    CREATE TABLE `parts` (
        `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
        `name` varchar(70) NOT NULL,
        PRIMARY KEY (`id`)
    );

* The model "Robots" has many "RobotsParts".
* The model "Parts" has many "RobotsParts".
* The model "RobotsParts" belongs to both "Robots" and "Parts" models as a many-to-one relation.
* The model "Robots" has a relation many-to-many to "Parts" through "RobotsParts"

看下面的EER图更加直观：

Check the EER diagram to understand better the relations:

.. figure:: ../_static/img/eer-1.png
   :align: center

模型和之间的关系实现如下：   
   
The models with their relations could be implemented as follows:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {
        public $id;

        public $name;

        public function initialize()
        {
            $this->hasMany("id", "RobotsParts", "robots_id");
        }

    }

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Parts extends Model
    {

        public $id;

        public $name;

        public function initialize()
        {
            $this->hasMany("id", "RobotsParts", "parts_id");
        }

    }

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class RobotsParts extends Model
    {

        public $id;

        public $robots_id;

        public $parts_id;

        public function initialize()
        {
            $this->belongsTo("robots_id", "Robots", "id");
            $this->belongsTo("parts_id", "Parts", "id");
        }

    }

第一个参数表示本地模型中字段关系;第二个参数表示引用模型的名称，第三个参数是引用模型的字段名。你也可以使用数组来定义多个字段的关系。	
	
The first parameter indicates the field of the local model used in the relationship; the second indicates the name
of the referenced model and the third the field name in the referenced model. You could also use arrays to define multiple fields in the relationship.

多对多的关系需要3模型并且定义关系中涉及到的属性:

Many to many relationships require 3 models and define the attributes involved in the relationship:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {
        public $id;

        public $name;

        public function initialize()
        {
            $this->hasManyToMany(
                "id",
                "RobotsParts",
                "robots_id", "parts_id",
                "Parts",
                "id"
            );
        }

    }

使用关系Taking advantage of relationships
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
当显式地定义模型之间的关系,它很容易为一个特定的记录找到相关的记录。

When explicitly defining the relationships between models, it is easy to find related records for a particular record.

.. code-block:: php

    <?php

    $robot = Robots::findFirst(2);
    foreach ($robot->robotsParts as $robotPart) {
        echo $robotPart->parts->name, "\n";
    }

Phalcon使用魔术方法__set/__get /__call存储或检索使用了关系的相关数据。	
	
Phalcon uses the magic methods __set/__get/__call to store or retrieve related data using relationships.

通过访问一个名称相同的关系中的属性将检索所有相关记录。

By accessing an attribute with the same name as the relationship will retrieve all its related record(s).

.. code-block:: php

    <?php

    $robot = Robots::findFirst();
    $robotsParts = $robot->robotsParts; // all the related records in RobotsParts

同样可以使用getter	
	
Also, you can use a magic getter:

.. code-block:: php

    <?php

    $robot = Robots::findFirst();
    $robotsParts = $robot->getRobotsParts(); // all the related records in RobotsParts
    $robotsParts = $robot->getRobotsParts(array('limit' => 5)); // passing parameters

如果调用一个有get前缀的方法:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 会返回一个findFirst()/find() 结果，下面的示例对比了使用魔术方法和不使用去获取数据：
	
If the called method has a "get" prefix :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` will return a
findFirst()/find() result. The following example compares retrieving related results with using magic methods
and without:

.. code-block:: php

    <?php

    $robot       = Robots::findFirst(2);

    // Robots model has a 1-n (hasMany)
    // relationship to RobotsParts then
    $robotsParts = $robot->robotsParts;

    // Only parts that match conditions
    $robotsParts = $robot->getRobotsParts("created_at = '2012-03-15'");

    // Or using bound parameters
    $robotsParts = $robot->getRobotsParts(array(
        "created_at = :date:",
        "bind" => array("date" => "2012-03-15")
    ));

    $robotPart   = RobotsParts::findFirst(1);

    // RobotsParts model has a n-1 (belongsTo)
    // relationship to RobotsParts then
    $robot = $robotPart->robots;

手动获取相关记录	
	
Getting related records manually:

.. code-block:: php

    <?php

    $robot       = Robots::findFirst(2);

    // Robots model has a 1-n (hasMany)
    // relationship to RobotsParts, then
    $robotsParts = RobotsParts::find("robots_id = '" . $robot->id . "'");

    // Only parts that match conditions
    $robotsParts = RobotsParts::find(
        "robots_id = '" . $robot->id . "' AND created_at = '2012-03-15'"
    );

    $robotPart   = RobotsParts::findFirst(1);

    // RobotsParts model has a n-1 (belongsTo)
    // relationship to RobotsParts then
    $robot = Robots::findFirst("id = '" . $robotPart->robots_id . "'");

前缀用于find()/findFirst()相关记录。根据关系类型决定使用find还是findFirst
	
The prefix "get" is used to find()/findFirst() related records. Depending on the type of relation it will use
'find' or 'findFirst':

+---------------------+----------------------------------------------------------------------------------------------------------------------------+------------------------+
| Type                | Description                                                                                                                | Implicit Method        |
+=====================+============================================================================================================================+========================+
| Belongs-To          | Returns a model instance of the related record directly                                                                    | findFirst              |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+------------------------+
| Has-One             | Returns a model instance of the related record directly                                                                    | findFirst              |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+------------------------+
| Has-Many            | Returns a collection of model instances of the referenced model                                                            | find                   |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+------------------------+
| Has-Many-to-Many    | Returns a collection of model instances of the referenced model, it implicitly does 'inner joins' with the involved models | (complex query)        |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+------------------------+

还可以使用“count”前缀返回一个表示相关记录的统计整数:

You can also use "count" prefix to return an integer denoting the count of the related records:

.. code-block:: php

    <?php

    $robot = Robots::findFirst(2);
    echo "The robot has ", $robot->countRobotsParts(), " parts\n";

关系别名Aliasing Relationships
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
为了解释别名如何使用，让我们看下面例子：

To explain better how aliases work, let's check the following example:

表“robots_similar”定义了类似机器人的信息:

Table "robots_similar" has the function to define what robots are similar to others:

.. code-block:: bash

    mysql> desc robots_similar;
    +-------------------+------------------+------+-----+---------+----------------+
    | Field             | Type             | Null | Key | Default | Extra          |
    +-------------------+------------------+------+-----+---------+----------------+
    | id                | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
    | robots_id         | int(10) unsigned | NO   | MUL | NULL    |                |
    | similar_robots_id | int(10) unsigned | NO   |     | NULL    |                |
    +-------------------+------------------+------+-----+---------+----------------+
    3 rows in set (0.00 sec)

robots_id表和similar_robots_id表都和Robots模型有关系：	
	
Both "robots_id" and "similar_robots_id" have a relation to the model Robots:

.. figure:: ../_static/img/eer-2.png
   :align: center

定义他们之前关系的数据模型如下所示：   
   
A model that maps this table and its relationships is the following:

.. code-block:: php

    <?php

    class RobotsSimilar extends Phalcon\Mvc\Model
    {

        public function initialize()
        {
            $this->belongsTo('robots_id', 'Robots', 'id');
            $this->belongsTo('similar_robots_id', 'Robots', 'id');
        }

    }

因为他们的关系都指向了同一个数据模型(Robots)，获取关联数据就会不清楚：	
	
Since both relations point to the same model (Robots), obtain the records related to the relationship could not be clear:

.. code-block:: php

    <?php

    $robotsSimilar = RobotsSimilar::findFirst();

    //Returns the related record based on the column (robots_id)
    //Also as is a belongsTo it's only returning one record
    //but the name 'getRobots' seems to imply that return more than one
    $robot = $robotsSimilar->getRobots();

    //but, how to get the related record based on the column (similar_robots_id)
    //if both relationships have the same name?

可以使用别名重命名关系，这样就解决了这个问题：	
	
The aliases allow us to rename both relationships to solve these problems:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class RobotsSimilar extends Model
    {

        public function initialize()
        {
            $this->belongsTo('robots_id', 'Robots', 'id', array(
                'alias' => 'Robot'
            ));
            $this->belongsTo('similar_robots_id', 'Robots', 'id', array(
                'alias' => 'SimilarRobot'
            ));
        }

    }

使用别名可以轻松的获取相关数据：	
	
With the aliasing we can get the related records easily:

.. code-block:: php

    <?php

    $robotsSimilar = RobotsSimilar::findFirst();

    //Returns the related record based on the column (robots_id)
    $robot = $robotsSimilar->getRobot();
    $robot = $robotsSimilar->robot;

    //Returns the related record based on the column (similar_robots_id)
    $similarRobot = $robotsSimilar->getSimilarRobot();
    $similarRobot = $robotsSimilar->similarRobot;

魔术方法 Getters 对比显示方法Magic Getters vs. Explicit methods
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
大多数IDE和编辑器有自动提示完成代码的功能，但是使用魔法getter方法时不能推断出正确的类型, 而不是使用魔法的getter方法我们可以显式地定义这些方法和相应的docblocks帮助IDE生成更好的自动提示完成代码:

Most IDEs and editors with auto-completion capabilities can not infer the correct types when using magic getters,
instead of use the magic getters you can optionally define those methods explicitly with the corresponding
docblocks helping the IDE to produce a better auto-completion:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public $id;

        public $name;

        public function initialize()
        {
            $this->hasMany("id", "RobotsParts", "robots_id");
        }

        /**
         * Return the related "robots parts"
         *
         * @return \RobotsParts[]
         */
        public function getRobotsParts($parameters=null)
        {
            return $this->getRelated('RobotsParts', $parameters);
        }

    }

虚拟外键Virtual Foreign Keys
----------------------------------
默认情况下,不像数据库外键的关系那样,如果插入/更新一个无效值到相关模型中,Phalcon不会产生一个验证消息。在定义一个关系的时候可以通过添加第四个参数修改这个行为。

By default, relationships do not act like database foreign keys, that is, if you try to insert/update a value without having a valid
value in the referenced model, Phalcon will not produce a validation message. You can modify this behavior by adding a fourth parameter
when defining a relationship.

修改下RobotsPart模型来演示这个功能:

The RobotsPart model can be changed to demonstrate this feature:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class RobotsParts extends Model
    {

        public $id;

        public $robots_id;

        public $parts_id;

        public function initialize()
        {
            $this->belongsTo("robots_id", "Robots", "id", array(
                "foreignKey" => true
            ));

            $this->belongsTo("parts_id", "Parts", "id", array(
                "foreignKey" => array(
                    "message" => "The part_id does not exist on the Parts model"
                )
            ));
        }

    }

如果改变一个belongsTo()作为外键的关系,在进行插入/更新相关模型操作时它将验证这些字段值是否有效。类似地,如果一个hasMany()/hasOne()被加了外键约束，它将验证记录不能被删除，如果记录还在存在一个相关模型中。	
	
If you alter a belongsTo() relationship to act as foreign key, it will validate that the values inserted/updated on those fields have a
valid value on the referenced model. Similarly, if a hasMany()/hasOne() is altered it will validate that the records cannot be deleted
if that record is used on a referenced model.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Parts extends Model
    {

        public function initialize()
        {
            $this->hasMany("id", "RobotsParts", "parts_id", array(
                "foreignKey" => array(
                    "message" => "The part cannot be deleted because other robots are using it"
                )
            ));
        }

    }

级联与限制动作Cascade/Restrict actions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
默认关系作为虚拟外键限制创建/更新/删除记录来保持数据的完整性:

Relationships that act as virtual foreign keys by default restrict the creation/update/deletion of records
to maintain the integrity of data:

.. code-block:: php

    <?php

    namespace Store\Models;

    use Phalcon\Mvc\Model;
    use Phalcon\Mvc\Model\Relation;

    class Robots extends Model
    {

        public $id;

        public $name;

        public function initialize()
        {
            $this->hasMany('id', 'Store\\Models\Parts', 'robots_id', array(
                'foreignKey' => array(
                    'action' => Relation::ACTION_CASCADE
                )
            ));
        }

    }

上面的代码设置如果主记录(robot)被删除则删除所有被引用的记录(parts)。	
	
The above code set up to delete all the referenced records (parts) if the master record (robot) is deleted.

生成运算Generating Calculations
------------------------------------
:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`计算或者统计函数像是 COUNT, SUM, MAX, MIN or AVG可以直接使用。

Calculations (or aggregations) are helpers for commonly used functions of database systems such as COUNT, SUM, MAX, MIN or AVG.
:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` allows to use these functions directly from the exposed methods.

Count examples:

.. code-block:: php

    <?php

    // How many employees are?
    $rowcount = Employees::count();

    // How many different areas are assigned to employees?
    $rowcount = Employees::count(array("distinct" => "area"));

    // How many employees are in the Testing area?
    $rowcount = Employees::count("area = 'Testing'");

    // Count employees grouping results by their area
    $group = Employees::count(array("group" => "area"));
    foreach ($group as $row) {
       echo "There are ", $row->rowcount, " in ", $row->area;
    }

    // Count employees grouping by their area and ordering the result by count
    $group = Employees::count(array(
        "group" => "area",
        "order" => "rowcount"
    ));

    // Avoid SQL injections using bound parameters
    $group = Employees::count(array(
        "type > ?0",
        "bind" => array($type)
    ));

求和例子：
	
Sum examples:

.. code-block:: php

    <?php

    // How much are the salaries of all employees?
    $total = Employees::sum(array("column" => "salary"));

    // How much are the salaries of all employees in the Sales area?
    $total = Employees::sum(array(
        "column"     => "salary",
        "conditions" => "area = 'Sales'"
    ));

    // Generate a grouping of the salaries of each area
    $group = Employees::sum(array(
        "column" => "salary",
        "group"  => "area"
    ));
    foreach ($group as $row) {
       echo "The sum of salaries of the ", $row->area, " is ", $row->sumatory;
    }

    // Generate a grouping of the salaries of each area ordering
    // salaries from higher to lower
    $group = Employees::sum(array(
        "column" => "salary",
        "group"  => "area",
        "order"  => "sumatory DESC"
    ));

    // Avoid SQL injections using bound parameters
    $group = Employees::sum(array(
        "conditions" => "area > ?0",
        "bind" => array($area)
    ));

求平均例子：	
	
Average examples:

.. code-block:: php

    <?php

    // What is the average salary for all employees?
    $average = Employees::average(array("column" => "salary"));

    // What is the average salary for the Sales's area employees?
    $average = Employees::average(array(
        "column"     => "salary",
        "conditions" => "area = 'Sales'"
    ));

    // Avoid SQL injections using bound parameters
    $average = Employees::average(array(
        "column"     => "age",
        "conditions" => "area > ?0",
        "bind"       => array($area)
    ));

最大/最小例子：	
	
Max/Min examples:

.. code-block:: php

    <?php

    // What is the oldest age of all employees?
    $age = Employees::maximum(array("column" => "age"));

    // What is the oldest of employees from the Sales area?
    $age = Employees::maximum(array(
        "column"     => "age",
        "conditions" => "area = 'Sales'"
    ));

    // What is the lowest salary of all employees?
    $salary = Employees::minimum(array("column" => "salary"));

Hydration模式Hydration Modes
---------------------------------
正如上面提到的,结果集是一个完整的集合对象,这意味着每一个返回的结果是一个对象，它代表数据库中的一行。这些对象可以被修改并再次持久保存在数据库中:

As mentioned above, resultsets are collections of complete objects, this means that every returned result is an object
representing a row in the database. These objects can be modified and saved again to persistence:

.. code-block:: php

    <?php

    // Manipulating a resultset of complete objects
    foreach (Robots::find() as $robot) {
        $robot->year = 2000;
        $robot->save();
    }

有时获取的数据只需要以只读模式呈现给用户,在这种情况下变更记录的呈现方式可以解决这个问题。这种变更对象在结果集中的呈现方式的方法被叫做“Hydration模式”:	
	
Sometimes records are obtained only to be presented to a user in read-only mode, in these cases it may be useful
to change the way in which records are represented to facilitate their handling. The strategy used to represent objects
returned in a resultset is called 'hydration mode':

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\Resultset;

    $robots = Robots::find();

    //Return every robot as an array
    $robots->setHydrateMode(Resultset::HYDRATE_ARRAYS);

    foreach ($robots as $robot) {
        echo $robot['year'], PHP_EOL;
    }

    //Return every robot as an stdClass
    $robots->setHydrateMode(Resultset::HYDRATE_OBJECTS);

    foreach ($robots as $robot) {
        echo $robot->year, PHP_EOL;
    }

    //Return every robot as a Robots instance
    $robots->setHydrateMode(Resultset::HYDRATE_RECORDS);

    foreach ($robots as $robot) {
        echo $robot->year, PHP_EOL;
    }

Hydration模式也可以被作为参数传递给“find”:	
	
Hydration mode can also be passed as a parameter of 'find':

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\Resultset;

    $robots = Robots::find(array(
        'hydration' => Resultset::HYDRATE_ARRAYS
    ));

    foreach ($robots as $robot) {
        echo $robot['year'], PHP_EOL;
    }

创建与更新记录Creating Updating/Records
----------------------------------------
Phalcon\\Mvc\\Model::save()方法允许您创建/更新记录根据表中是否已经存在相关的记录。内部调用save实现创建和更新:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`。为实现这个功能必须正确地为表定义一个主键来确定一个记录应该被更新还是创建。

The method Phalcon\\Mvc\\Model::save() allows you to create/update records according to whether they already exist in the table
associated with a model. The save method is called internally by the create and update methods of :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`.
For this to work as expected it is necessary to have properly defined a primary key in the entity to determine whether a record
should be updated or created.

如果虚拟外键和事件在模型中定义，save也验证器相关联的方法:

Also the method executes associated validators, virtual foreign keys and events that are defined in the model:

.. code-block:: php

    <?php

    $robot       = new Robots();
    $robot->type = "mechanical";
    $robot->name = "Astro Boy";
    $robot->year = 1952;
    if ($robot->save() == false) {
        echo "Umh, We can't store robots right now: \n";
        foreach ($robot->getMessages() as $message) {
            echo $message, "\n";
        }
    } else {
        echo "Great, a new robot was saved successfully!";
    }

数组可以直接被传递给save方法，从而避免为每一列单独赋值。 Phalcon\\Mvc\\Model将检查是否有实现setter列，如果有则优先考虑setter,而不使用指定数组的值:	
	
An array could be passed to "save" to avoid assign every column manually. Phalcon\\Mvc\\Model will check if there are setters implemented for
the columns passed in the array giving priority to them instead of assign directly the values of the attributes:

.. code-block:: php

    <?php

    $robot = new Robots();
    $robot->save(array(
        "type" => "mechanical",
        "name" => "Astro Boy",
        "year" => 1952
    ));

数组传递的值会根据属性类型自动转义。所以你可以传递一个非法的数组,而不用担心可能的SQL注入:	
	
Values assigned directly or via the array of attributes are escaped/sanitized according to the related attribute data type. So you can pass
an insecure array without worrying about possible SQL injections:

.. code-block:: php

    <?php

    $robot = new Robots();
    $robot->save($_POST);

.. highlights::

    没有预防措施，攻击者可能会设置数据库中任何列的值。如果你想允许用户插入/更新模型的每一列中,即使这些字段不在提交表单中，可以使用这种方式。

    Without precautions mass assignment could allow attackers to set any database column’s value. Only use this feature
    if you want to permit a user to insert/update every column in the model, even if those fields are not in the submitted
    form.

当有大量参数赋值的时候，可以为save设置一个白名单参数，只有在参数中的值才会被保存:	
	
You can set an additional parameter in 'save' to set a whitelist of fields that only must taken into account when doing
the mass assignment:

.. code-block:: php

    <?php

    $robot = new Robots();
    $robot->save($_POST, array('name', 'type'));

创建与更新结果判断Create/Update with Confidence
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果应用有许多操作，我们希望去创建一条记录而程序实际上却执行了更新，这种情况可能会在使用Phalcon\\Mvc\\Model::save()保存记录的时候产生。如果想彻底避免这种情况发生，可以使用create() 或者 update()替代save()方法：

When an application has a lot of competition, we could be expecting create a record but it is actually updated. This
could happen if we use Phalcon\\Mvc\\Model::save() to persist the records in the database. If we want to be absolutely
sure that a record is created or updated, we can change the save() call with create() or update():

.. code-block:: php

    <?php

    $robot       = new Robots();
    $robot->type = "mechanical";
    $robot->name = "Astro Boy";
    $robot->year = 1952;

    //This record only must be created
    if ($robot->create() == false) {
        echo "Umh, We can't store robots right now: \n";
        foreach ($robot->getMessages() as $message) {
            echo $message, "\n";
        }
    } else {
        echo "Great, a new robot was created successfully!";
    }

create和update方法也接受数组作为参数。	
	
These methods "create" and "update" also accept an array of values as parameter.

自动生成标识列Auto-generated identity columns
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
有些模型有标示列。这些列通常是映射表的主键。:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`在产生SQL INSERT的时候可以自动忽略这些列,数据库系统会自动为其生成一个值。在每次新的记录被创建后，这些标示列会被数据库自动产生的值赋值:

Some models may have identity columns. These columns usually are the primary key of the mapped table. :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`
can recognize the identity column omitting it in the generated SQL INSERT, so the database system can generate an auto-generated value for it.
Always after creating a record, the identity field will be registered with the value generated in the database system for it:

.. code-block:: php

    <?php

    $robot->save();

    echo "The generated id is: ", $robot->id;

:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 能够识别标识列。根据使用的数据库不同，这些列可能是序列列（PostgreSQL）或者是auto_increment列（MySQL）。	
	
:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` is able to recognize the identity column. Depending on the database system, those columns may be
serial columns like in PostgreSQL or auto_increment columns in the case of MySQL.

PostgreSQL使用序列生成auto-numeric值,默认情况下,Phalcon试图获得从“table_field_seq”序列生成的价值, 例如:robots_id_seq,如果序列具有不同的名称,“getSequenceName”方法需要实现:

PostgreSQL uses sequences to generate auto-numeric values, by default, Phalcon tries to obtain the generated value from the sequence "table_field_seq",
for example: robots_id_seq, if that sequence has a different name, the method "getSequenceName" needs to be implemented:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function getSequenceName()
        {
            return "robots_sequence_name";
        }

    }

存储相关记录Storing related records
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
魔法属性可以用来存储记录及其相关属性:

Magic properties can be used to store a records and its related properties:

.. code-block:: php

    <?php

    // Create an artist
    $artist          = new Artists();
    $artist->name    = 'Shinichi Osawa';
    $artist->country = 'Japan';

    // Create an album
    $album          = new Albums();
    $album->name    = 'The One';
    $album->artist  = $artist; //Assign the artist
    $album->year    = 2008;

    //Save both records
    $album->save();

保存记录及其相关has-many记录:	
	
Saving a record and its related records in a has-many relation:

.. code-block:: php

    <?php

    // Get an existing artist
    $artist = Artists::findFirst('name = "Shinichi Osawa"');

    // Create an album
    $album          = new Albums();
    $album->name    = 'The One';
    $album->artist  = $artist;

    $songs = array();

    // Create a first song
    $songs[0]           = new Songs();
    $songs[0]->name     = 'Star Guitar';
    $songs[0]->duration = '5:54';

    // Create a second song
    $songs[1]           = new Songs();
    $songs[1]->name     = 'Last Days';
    $songs[1]->duration = '4:29';

    // Assign the songs array
    $album->songs = $songs;

    // Save the album + its songs
    $album->save();

同时保存album和artist会触发事务操作处理，如果保存相关记录出错，主记录信息也不会被报错。有关任何错误信息都会返回给用户。	
	
Saving the album and the artist at the same time implicitly makes use of a transaction so if anything
goes wrong with saving the related records, the parent will not be saved either. Messages are
passed back to the user for information regarding any errors.

注意:添加相关实体通过重载下面的方法是不可能的（重载下面的方法执行添加相关记录不会成功）:

Note: Adding related entities by overloading the following methods is not possible:

 - Phalcon\Mvc\Model::beforeSave()
 - Phalcon\Mvc\Model::beforeCreate()
 - Phalcon\Mvc\Model::beforeUpdate()

需要重载Phalcon\Mvc\Model::save() 才能保存相关记录。 
 
You need to overload Phalcon\Mvc\Model::save() for this to work from within a model.

验证信息Validation Messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`有个消息子系统，可以灵活的显示或者储存在执行insert/update流程中的验证信息。

:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` has a messaging subsystem that provides a flexible way to output or store the
validation messages generated during the insert/update processes.

每个消息包含一个:doc:`Phalcon\\Mvc\\Model\\Message <../api/Phalcon_Mvc_Model_Message>`类的实例。产生的消息可以通过getMessages()获得。每个消息提供了扩展字段名，消息内容，消息类型:

Each message consists of an instance of the class :doc:`Phalcon\\Mvc\\Model\\Message <../api/Phalcon_Mvc_Model_Message>`. The set of
messages generated can be retrieved with the method getMessages(). Each message provides extended information like the field name that
generated the message or the message type:

.. code-block:: php

    <?php

    if ($robot->save() == false) {
        foreach ($robot->getMessages() as $message) {
            echo "Message: ", $message->getMessage();
            echo "Field: ", $message->getField();
            echo "Type: ", $message->getType();
        }
    }

:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 会产生以下的验证信息：	
	
:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` can generate the following types of validation messages:

+----------------------+------------------------------------------------------------------------------------------------------------------------------------+
| Type                 | Description                                                                                                                        |
+======================+====================================================================================================================================+
| PresenceOf           | Generated when a field with a non-null attribute on the database is trying to insert/update a null value                           |
+----------------------+------------------------------------------------------------------------------------------------------------------------------------+
| ConstraintViolation  | Generated when a field part of a virtual foreign key is trying to insert/update a value that doesn't exist in the referenced model |
+----------------------+------------------------------------------------------------------------------------------------------------------------------------+
| InvalidValue         | Generated when a validator failed because of an invalid value                                                                      |
+----------------------+------------------------------------------------------------------------------------------------------------------------------------+
| InvalidCreateAttempt | Produced when a record is attempted to be created but it already exists                                                            |
+----------------------+------------------------------------------------------------------------------------------------------------------------------------+
| InvalidUpdateAttempt | Produced when a record is attempted to be updated but it doesn't exist                                                             |
+----------------------+------------------------------------------------------------------------------------------------------------------------------------+

可以在模型中getMessages()中替换或者翻译ORM生成的默认信息。

The method getMessages() can be overridden in a model to replace/translate the default messages generated automatically by the ORM:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {
        public function getMessages()
        {
            $messages = array();
            foreach (parent::getMessages() as $message) {
                switch ($message->getType()) {
                    case 'InvalidCreateAttempt':
                        $messages[] = 'The record cannot be created because it already exists';
                        break;
                    case 'InvalidUpdateAttempt':
                        $messages[] = 'The record cannot be updated because it already exists';
                        break;
                    case 'PresenceOf':
                        $messages[] = 'The field ' . $message->getField() . ' is mandatory';
                        break;
                }
            }
            return $messages;
        }
    }

事件和事件管理器Events and Events Manager
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
数据模型允许在执行insert/update/delete触发事件。帮助我们更好的建立业务逻辑。下面是:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`支持的事件和执行顺序：

Models allow you to implement events that will be thrown when performing an insert/update/delete. They help define business rules for a
certain model. The following are the events supported by :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` and their order of execution:

+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Operation          | Name                     | Can stop operation?   | Explanation                                                                                                                       |
+====================+==========================+=======================+===================================================================================================================================+
| Inserting/Updating | beforeValidation         | YES                   | Is executed before the fields are validated for not nulls/empty strings or foreign keys                                           |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Inserting          | beforeValidationOnCreate | YES                   | Is executed before the fields are validated for not nulls/empty strings or foreign keys when an insertion operation is being made |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Updating           | beforeValidationOnUpdate | YES                   | Is executed before the fields are validated for not nulls/empty strings or foreign keys when an updating operation is being made  |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Inserting/Updating | onValidationFails        | YES (already stopped) | Is executed after an integrity validator fails                                                                                    |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Inserting          | afterValidationOnCreate  | YES                   | Is executed after the fields are validated for not nulls/empty strings or foreign keys when an insertion operation is being made  |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Updating           | afterValidationOnUpdate  | YES                   | Is executed after the fields are validated for not nulls/empty strings or foreign keys when an updating operation is being made   |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Inserting/Updating | afterValidation          | YES                   | Is executed after the fields are validated for not nulls/empty strings or foreign keys                                            |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Inserting/Updating | beforeSave               | YES                   | Runs before the required operation over the database system                                                                       |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Updating           | beforeUpdate             | YES                   | Runs before the required operation over the database system only when an updating operation is being made                         |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Inserting          | beforeCreate             | YES                   | Runs before the required operation over the database system only when an inserting operation is being made                        |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Updating           | afterUpdate              | NO                    | Runs after the required operation over the database system only when an updating operation is being made                          |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Inserting          | afterCreate              | NO                    | Runs after the required operation over the database system only when an inserting operation is being made                         |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Inserting/Updating | afterSave                | NO                    | Runs after the required operation over the database system                                                                        |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+

模型中自定义事件Implementing Events in the Model's class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
让一个模型对事件做出反应更简单的方法是在事件模型的类中实现一个具有相同名称的方法:

The easier way to make a model react to events is implement a method with the same name of the event in the model's class:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function beforeValidationOnCreate()
        {
            echo "This is executed before creating a Robot!";
        }

    }

事件在执行之前进行赋值非常有用,例如:	
	
Events can be useful to assign values before performing an operation, for example:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Products extends Model
    {

        public function beforeCreate()
        {
            //Set the creation date
            $this->created_at = date('Y-m-d H:i:s');
        }

        public function beforeUpdate()
        {
            //Set the modification date
            $this->modified_in = date('Y-m-d H:i:s');
        }

    }

使用自定义事件管理器Using a custom Events Manager
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
事件管理器组件被整合到:doc:`Phalcon\\Events\\Manager <../api/Phalcon_Events_Manager>`，我们可以创建事件监听器，并监听事件是否被触发。

Additionally, this component is integrated with :doc:`Phalcon\\Events\\Manager <../api/Phalcon_Events_Manager>`,
this means we can create listeners that run when an event is triggered.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;
    use Phalcon\Events\Manager as EventsManager;

    class Robots extends Model
    {

        public function initialize()
        {

            $eventsManager = new EventsManager();

            //Attach an anonymous function as a listener for "model" events
            $eventsManager->attach('model', function($event, $robot) {
                if ($event->getType() == 'beforeSave') {
                    if ($robot->name == 'Scooby Doo') {
                        echo "Scooby Doo isn't a robot!";
                        return false;
                    }
                }
                return true;
            });

            //Attach the events manager to the event
            $this->setEventsManager($eventsManager);
        }

    }

在上面的例子中,事件管理器只是充当在一个对象和一个侦听器之间的桥梁(匿名函数)。在‘robots’保存的时候事件将被监听者收到:	
	
In the example given above, EventsManager only acts as a bridge between an object and a listener (the anonymous function).
Events will be fired to the listener when 'robots' are saved:

.. code-block:: php

    <?php

    $robot       = new Robots();
    $robot->name = 'Scooby Doo';
    $robot->year = 1969;
    $robot->save();

如果我们想要在我们的应用程序中使用相同的事件管理器,我们需要将它分配给模型管理器：	
	
If we want all objects created in our application use the same EventsManager, then we need to assign it to the Models Manager:

.. code-block:: php

    <?php

    //Registering the modelsManager service
    $di->setShared('modelsManager', function() {

        $eventsManager = new \Phalcon\Events\Manager();

        //Attach an anonymous function as a listener for "model" events
        $eventsManager->attach('model', function($event, $model){

            //Catch events produced by the Robots model
            if (get_class($model) == 'Robots') {

                if ($event->getType() == 'beforeSave') {
                    if ($model->name == 'Scooby Doo') {
                        echo "Scooby Doo isn't a robot!";
                        return false;
                    }
                }

            }
            return true;
        });

        //Setting a default EventsManager
        $modelsManager = new ModelsManager();
        $modelsManager->setEventsManager($eventsManager);
        return $modelsManager;
    });

如果一个侦听器返回false,将停止当前正在执行的操作。	
	
If a listener returns false that will stop the operation that is executing currently.

实现业务逻辑Implementing a Business Rule
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
当执行插入、更新或删除时,该模型验证在表中是否有任何和事件相关的方法。

When an insert, update or delete is executed, the model verifies if there are any methods with the names of
the events listed in the table above.

我们建议验证方法声明为protected,防止业务逻辑实现被公开曝光。

We recommend that validation methods are declared protected to prevent that business logic implementation
from being exposed publicly.

下面的示例实现一个事件来验证年份小于0的不能被更新或插入:

The following example implements an event that validates the year cannot be smaller than 0 on update or insert:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function beforeSave()
        {
            if ($this->year < 0) {
                echo "Year cannot be smaller than zero!";
                return false;
            }
        }

    }

一些事件返回false作为停止当前操作的指示。如果一个事件不返回任何值,:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`假设返回了true。	
	
Some events return false as an indication to stop the current operation. If an event doesn't return anything, :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`
will assume a true value.

验证数据完整性Validating Data Integrity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`提供了一些事件来验证数据和实施业务规则。特殊的"验证"事件允许我们调用内置的验证器验证记录。Phalcon暴露一些内置的验证器,我们可以使用这些验证。

:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` provides several events to validate data and implement business rules. The special "validation"
event allows us to call built-in validators over the record. Phalcon exposes a few built-in validators that can be used at this stage of validation.



The following example shows how to use it:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;
    use Phalcon\Mvc\Model\Validator\Uniqueness;
    use Phalcon\Mvc\Model\Validator\InclusionIn;

    class Robots extends Model
    {

        public function validation()
        {

            $this->validate(new InclusionIn(
                array(
                    "field"  => "type",
                    "domain" => array("Mechanical", "Virtual")
                )
            ));

            $this->validate(new Uniqueness(
                array(
                    "field"   => "name",
                    "message" => "The robot name must be unique"
                )
            ));

            return $this->validationHasFailed() != true;
        }

    }

上面的例子演示了使用内置“InclusionIn”(包含)验证器。它验证了字段“type”是否在domain列表中。如果验证失败就会返回false。内置的验证器如下所示：	
	
The above example performs a validation using the built-in validator "InclusionIn". It checks the value of the field "type" in a domain list. If
the value is not included in the method then the validator will fail and return false. The following built-in validators are available:

+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Name         | Explanation                                                                                                                                                      | Example                                                           |
+==============+==================================================================================================================================================================+===================================================================+
| PresenceOf   | Validates that a field's value isn't null or empty string. This validator is automatically added based on the attributes marked as not null on the mapped table  | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_PresenceOf>`    |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Email        | Validates that field contains a valid email format                                                                                                               | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_Email>`         |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| ExclusionIn  | Validates that a value is not within a list of possible values                                                                                                   | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_Exclusionin>`   |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| InclusionIn  | Validates that a value is within a list of possible values                                                                                                       | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_Inclusionin>`   |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Numericality | Validates that a field has a numeric format                                                                                                                      | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_Numericality>`  |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Regex        | Validates that the value of a field matches a regular expression                                                                                                 | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_Regex>`         |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Uniqueness   | Validates that a field or a combination of a set of fields are not present more than once in the existing records of the related table                           | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_Uniqueness>`    |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| StringLength | Validates the length of a string                                                                                                                                 | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_StringLength>`  |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Url          | Validates that a value has a valid URL format                                                                                                                    | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_Url>`           |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+

除了内置的验证器，我们还可以自定义验证器：

In addition to the built-in validators, you can create your own validators:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\Validator;
    use Phalcon\Mvc\Model\ValidatorInterface;

    class MaxMinValidator extends Validator implements ValidatorInterface
    {

        public function validate($model)
        {
            $field  = $this->getOption('field');

            $min    = $this->getOption('min');
            $max    = $this->getOption('max');

            $value  = $model->$field;

            if ($min <= $value && $value <= $max) {
                $this->appendMessage(
                    "The field doesn't have the right range of values",
                    $field,
                    "MaxMinValidator"
                );
                return false;
            }
            return true;
        }

    }

将验证器加入模型：
	
Adding the validator to a model:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Customers extends Model
    {

        public function validation()
        {
            $this->validate(new MaxMinValidator(
                array(
                    "field" => "price",
                    "min"   => 10,
                    "max"   => 100
                )
            ));
            if ($this->validationHasFailed() == true) {
                return false;
            }
        }

    }

创建验证器的想法是让他们可以在几个模型之间可重用。一个验证器也可以一样简单定义如下:	
	
The idea of creating validators is make them reusable between several models. A validator can also be as simple as:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;
    use Phalcon\Mvc\Model\Message;

    class Robots extends Model
    {

        public function validation()
        {
            if ($this->type == "Old") {
                $message = new Message(
                    "Sorry, old robots are not allowed anymore",
                    "type",
                    "MyType"
                );
                $this->appendMessage($message);
                return false;
            }
            return true;
        }

    }

避免SQL注入Avoiding SQL injections
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
每个值分配给模型属性会根据数据类型被转义。程序员不必手动转义每个值保存到数据库。Phalcon使用内置的`bound parameters <http://php.net/manual/en/pdostatement.bindparam.php>`_ 由PDO提供的功能自动转义存入数据库的值。

Every value assigned to a model attribute is escaped depending of its data type. A developer doesn't need to escape manually
each value before storing it on the database. Phalcon uses internally the `bound parameters <http://php.net/manual/en/pdostatement.bindparam.php>`_
capability provided by PDO to automatically escape every value to be stored in the database.

.. code-block:: bash

    mysql> desc products;
    +------------------+------------------+------+-----+---------+----------------+
    | Field            | Type             | Null | Key | Default | Extra          |
    +------------------+------------------+------+-----+---------+----------------+
    | id               | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
    | product_types_id | int(10) unsigned | NO   | MUL | NULL    |                |
    | name             | varchar(70)      | NO   |     | NULL    |                |
    | price            | decimal(16,2)    | NO   |     | NULL    |                |
    | active           | char(1)          | YES  |     | NULL    |                |
    +------------------+------------------+------+-----+---------+----------------+
    5 rows in set (0.00 sec)

如果我们以一个安全的方式使用PDO存储记录,我们需要编写下面的代码:	
	
If we use just PDO to store a record in a secure way, we need to write the following code:

.. code-block:: php

    <?php

    $name           = 'Artichoke';
    $price          = 10.5;
    $active         = 'Y';
    $productTypesId = 1;

    $sql = 'INSERT INTO products VALUES (null, :productTypesId, :name, :price, :active)';
    $sth = $dbh->prepare($sql);

    $sth->bindParam(':productTypesId', $productTypesId, PDO::PARAM_INT);
    $sth->bindParam(':name', $name, PDO::PARAM_STR, 70);
    $sth->bindParam(':price', doubleval($price));
    $sth->bindParam(':active', $active, PDO::PARAM_STR, 1);

    $sth->execute();

Phalcon已经帮我们自动处理了：	
	
The good news is that Phalcon do this for you automatically:

.. code-block:: php

    <?php

    $product                    = new Products();
    $product->product_types_id  = 1;
    $product->name              = 'Artichoke';
    $product->price             = 10.5;
    $product->active            = 'Y';
    $product->create();

忽略指定列的数据Skipping Columns
---------------------------------------
可以让Phalcon\\Mvc\\Model在创建或者更新记录的时候忽略一些字段，这样可以将数据库赋值委托给一个触发器。

To tell Phalcon\\Mvc\\Model that always omits some fields in the creation and/or update of records in order
to delegate the database system the assignation of the values by a trigger or a default:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function initialize()
        {
            //Skips fields/columns on both INSERT/UPDATE operations
            $this->skipAttributes(array('year', 'price'));

            //Skips only when inserting
            $this->skipAttributesOnCreate(array('created_at'));

            //Skips only when updating
            $this->skipAttributesOnUpdate(array('modified_in'));
        }

    }

上面的操作会在整个应用中INSERT/UPDATE操作时忽略上面的字段。如果想在不同的INSERT/UPDATE操作忽略不同的字段，可以定义第二个参数为true。
	
This will ignore globally these fields on each INSERT/UPDATE operation on the whole application. If you want to ignore different attributes on different INSERT/UPDATE operations, you can specify the second parameter (boolean) - true for replacement.

强制设置默认值可以用下面方法实现：

Forcing a default value can be done in the following way:

.. code-block:: php

    <?php

    use Phalcon\Db\RawValue;

    $robot              = new Robots();
    $robot->name        = 'Bender';
    $robot->year        = 1999;
    $robot->created_at  = new RawValue('default');
    $robot->create();

回调同样可以被用于根据条件设置默认值。	
	
A callback also can be used to create a conditional assignment of automatic default values:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;
    use Phalcon\Db\RawValue;

    class Robots extends Model
    {
        public function beforeCreate()
        {
            if ($this->price > 10000) {
                $this->type = new RawValue('default');
            }
        }
    }

.. highlights::

    永远不要用\\Phalcon\\Db\\RawValue去给用户输入的数据或者变量赋值。这些字段的值在绑定参数到请求的时候会被忽略，可能让应用有SQL注入的风险。

    Never use a \\Phalcon\\Db\\RawValue to assign external data (such as user input)
    or variable data. The value of these fields is ignored when binding parameters to the query.
    So it could be used to attack the application injecting SQL.

动态更新Dynamic Update
^^^^^^^^^^^^^^^^^^^^^^^^^^
默认SQL UPDATE语句更新模型中定义的每一列(完整的所有字段SQL更新)。可以改变特定的模型进行动态更新,在这种情况下只有变更的字段才用于创建最终的SQL语句。

SQL UPDATE statements are by default created with every column defined in the model (full all-field SQL update).
You can change specific models to make dynamic updates, in this case, just the fields that had changed
are used to create the final SQL statement.

在某些情况下这可能会提高性能,减少应用程序和数据库服务器之间的流量,特别是当表中有blob/文本字段:

In some cases this could improve the performance by reducing the traffic between the application and the database server,
this specially helps when the table has blob/text fields:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {
        public function initialize()
        {
            $this->useDynamicUpdate(true);
        }
    }

删除记录Deleting Records
-------------------------------
 Phalcon\\Mvc\\Model::delete()方法被用于删除记录。使用方法如下：

The method Phalcon\\Mvc\\Model::delete() allows to delete a record. You can use it as follows:

.. code-block:: php

    <?php

    $robot = Robots::findFirst(11);
    if ($robot != false) {
        if ($robot->delete() == false) {
            echo "Sorry, we can't delete the robot right now: \n";
            foreach ($robot->getMessages() as $message) {
                echo $message, "\n";
            }
        } else {
            echo "The robot was deleted successfully!";
        }
    }

可以在foreach循环结果集批量删除记录：	
	
You can also delete many records by traversing a resultset with a foreach:

.. code-block:: php

    <?php

    foreach (Robots::find("type='mechanical'") as $robot) {
        if ($robot->delete() == false) {
            echo "Sorry, we can't delete the robot right now: \n";
            foreach ($robot->getMessages() as $message) {
                echo $message, "\n";
            }
        } else {
            echo "The robot was deleted successfully!";
        }
    }

当删除被执行的时候下面的事件可以被用于自定义业务逻辑：	
	
The following events are available to define custom business rules that can be executed when a delete operation is
performed:

+-----------+--------------+---------------------+------------------------------------------+
| Operation | Name         | Can stop operation? | Explanation                              |
+===========+==============+=====================+==========================================+
| Deleting  | beforeDelete | YES                 | Runs before the delete operation is made |
+-----------+--------------+---------------------+------------------------------------------+
| Deleting  | afterDelete  | NO                  | Runs after the delete operation was made |
+-----------+--------------+---------------------+------------------------------------------+

使用上面的事件我们可以在模型中定义业务逻辑：

With the above events can also define business rules in the models:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function beforeDelete()
        {
            if ($this->status == 'A') {
                echo "The robot is active, it can't be deleted";
                return false;
            }
            return true;
        }

    }

		
验证失败事件Validation Failed Events
----------------------------------------
在数据验证过程中发现失败的时候另外一些事件可以被使用。

Another type of events are available when the data validation process finds any inconsistency:

+--------------------------+--------------------+--------------------------------------------------------------------+
| Operation                | Name               | Explanation                                                        |
+==========================+====================+====================================================================+
| Insert or Update         | notSave            | Triggered when the INSERT or UPDATE operation fails for any reason |
+--------------------------+--------------------+--------------------------------------------------------------------+
| Insert, Delete or Update | onValidationFails  | Triggered when any data manipulation operation fails               |
+--------------------------+--------------------+--------------------------------------------------------------------+

行为Behaviors
-------------------
行为是为了重用代码而在几个模块中公用的部分,ORM提供一个API可以用来实现我们自己的模型行为。另外,可以使用事件和回调更自由的去实现行为。

Behaviors are shared conducts that several models may adopt in order to re-use code, the ORM provides an API to implement
behaviors in your models. Also, you can use the events and callbacks as seen before as an alternative to implement Behaviors with more freedom.

行为必须被添加到模型的初始化行为中，一个模型可以用零个或者多个行为：

A behavior must be added in the model initializer, a model can have zero or more behaviors:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;
    use Phalcon\Mvc\Model\Behavior\Timestampable;

    class Users extends Model
    {
        public $id;

        public $name;

        public $created_at;

        public function initialize()
        {
            $this->addBehavior(new Timestampable(
                array(
                    'beforeCreate'  => array(
                        'field'     => 'created_at',
                        'format'    => 'Y-m-d'
                    )
                )
            ));
        }

    }

框架提供了下面内置的行为：	
	
The following built-in behaviors are provided by the framework:

+----------------+-------------------------------------------------------------------------------------------------------------------------------+
| Name           | Description                                                                                                                   |
+================+===============================================================================================================================+
| Timestampable  | Allows to automatically update a model's attribute saving the datetime when a record is created or updated                    |
+----------------+-------------------------------------------------------------------------------------------------------------------------------+
| SoftDelete     | Instead of permanently delete a record it marks the record as deleted changing the value of a flag column                     |
+----------------+-------------------------------------------------------------------------------------------------------------------------------+

生成时间戳Timestampable
^^^^^^^^^^^^^^^^^^^^^^^^^^
这种行为接收一个数组作为参数,第一级键必须是一个事件名称指明了在什么时候列必须被赋值:

This behavior receives an array of options, the first level key must be an event name indicating when the column must be assigned:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\Behavior\Timestampable;

    public function initialize()
    {
        $this->addBehavior(new Timestampable(
            array(
                'beforeCreate'  => array(
                    'field'     => 'created_at',
                    'format'    => 'Y-m-d'
                )
            )
        ));
    }

每个事件可以有自己的选项，‘field’是必须更新的列。如果'format'是字符，将会用PHP内置的函数date进行格式化。format还可以是匿名函数方便我们自己的定义时间戳格式。	
	
Each event can have its own options, 'field' is the name of the column that must be updated, if 'format' is a string it will be used
as format of the PHP's function date_, format can also be an anonymous function providing you the free to generate any kind timestamp:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\Behavior\Timestampable;

    public function initialize()
    {
        $this->addBehavior(new Timestampable(
            array(
                'beforeCreate' => array(
                    'field'  => 'created_at',
                    'format' => function() {
                        $datetime = new Datetime(new DateTimeZone('Europe/Stockholm'));
                        return $datetime->format('Y-m-d H:i:sP');
                    }
                )
            )
        ));
    }

如果format选项被忽略了，将会使用PHP默认的time。	
	
If the option 'format' is omitted a timestamp using the PHP's function time_, will be used.

软删除SoftDelete
^^^^^^^^^^^^^^^^^^^^^
行为可以被用于以下的方式：

This behavior can be used in the following way:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;
    use Phalcon\Mvc\Model\Behavior\SoftDelete;

    class Users extends Model
    {

        const DELETED = 'D';

        const NOT_DELETED = 'N';

        public $id;

        public $name;

        public $status;

        public function initialize()
        {
            $this->addBehavior(new SoftDelete(
                array(
                    'field' => 'status',
                    'value' => Users::DELETED
                )
            ));
        }

    }

行为接受两个参数“field”和“value”,"field"是要更新的字段，‘value’是要被删除的表和状态。	
	
This behavior accepts two options: 'field' and 'value', 'field' determines what field must be updated and 'value' the value to be deleted.
Let's pretend the table 'users' has the following data:

.. code-block:: bash

    mysql> select * from users;
    +----+---------+--------+
    | id | name    | status |
    +----+---------+--------+
    |  1 | Lana    | N      |
    |  2 | Brandon | N      |
    +----+---------+--------+
    2 rows in set (0.00 sec)

如果我们删除任意下面的一条记录，只是更新了他们的状态而不是真正的删除：	
	
If we delete any of the two records the status will be updated instead of delete the record:

.. code-block:: php

    <?php

    Users::findFirst(2)->delete();

操作会是以下的结果：	
	
The operation will result in the following data in the table:

.. code-block:: bash

    mysql> select * from users;
    +----+---------+--------+
    | id | name    | status |
    +----+---------+--------+
    |  1 | Lana    | N      |
    |  2 | Brandon | D      |
    +----+---------+--------+
    2 rows in set (0.01 sec)

注意,我们需在的查询条件中手动指定已经软删除的记录来忽略它们（防止已经软删除的被重复操作）,行为并不支持这样的自动操作。	
	
Note that you need to specify the deleted condition in your queries to effectively ignore them as deleted records, this behavior doesn't support that.

创建行为Creating your own behaviors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ORM提供一个API来创建我们自己的行为。行为必须是:doc:`Phalcon\\Mvc\\Model\\BehaviorInterface <../api/Phalcon_Mvc_Model_BehaviorInterface>`的类是实现。同时Phalcon\\Mvc\\Model\\Behavior提供了实现行为大部分的方法。

The ORM provides an API to create your own behaviors. A behavior must be a class implementing the :doc:`Phalcon\\Mvc\\Model\\BehaviorInterface <../api/Phalcon_Mvc_Model_BehaviorInterface>`
Also, Phalcon\\Mvc\\Model\\Behavior provides most of the methods needed to ease the implementation of behaviors.

以下行为是一个例子,它监控用户对模型所做的行为操作:

The following behavior is an example, it implements the Blamable behavior which helps identify the user
that is performed operations over a model:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\Behavior;
    use Phalcon\Mvc\Model\BehaviorInterface;

    class Blamable extends Behavior implements BehaviorInterface
    {

        public function notify($eventType, $model)
        {
            switch ($eventType) {

                case 'afterCreate':
                case 'afterDelete':
                case 'afterUpdate':


                    $userName = // ... get the current user from session

                    //Store in a log the username - event type and primary key
                    file_put_contents(
                        'logs/blamable-log.txt',
                        $userName . ' ' . $eventType . ' ' . $model->id
                    );

                    break;

                default:
                    /* ignore the rest of events */
            }
        }

    }

上面是一个非常简单的行为,但是它演示了如何创建一个行为,现在让我们将这个行为添加到模型中：
	
The former is a very simple behavior, but it illustrates how to create a behavior, now let's add this behavior to a model:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Profiles extends Model
    {

        public function initialize()
        {
            $this->addBehavior(new Blamable());
        }

    }

一个行为也能在模型中拦截丢失的方法:	
	
A behavior is also capable of intercepting missing methods on your models:

.. code-block:: php

    <?php

    use Phalcon\Tag;
    use Phalcon\Mvc\Model\Behavior;
    use Phalcon\Mvc\Model\BehaviorInterface;

    class Sluggable extends Behavior implements BehaviorInterface
    {

        public function missingMethod($model, $method, $arguments=array())
        {
            // if the method is 'getSlug' convert the title
            if ($method == 'getSlug') {
                return Tag::friendlyTitle($model->title);
            }
        }

    }

调用该模型的方法实现Sluggable并返回一个SEO友好的标题:	
	
Call that method on a model that implements Sluggable returns a SEO friendly title:

.. code-block:: php

    <?php

    $title = $post->getSlug();

使用 Traits 实现行为Using Traits as behaviors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
从php5.4我们可以使用Traits_去更好的实现代码重用功能。这是另一个实现自定义行为的方法。下面的trait实现了另一个版本的Timestampable行为：

Starting from PHP 5.4 you can use Traits_ to re-use code in your classes, this is another way to implement
custom behaviors. The following trait implements a simple version of the Timestampable behavior:

.. code-block:: php

    <?php

    trait MyTimestampable
    {

        public function beforeCreate()
        {
            $this->created_at = date('r');
        }

        public function beforeUpdate()
        {
            $this->updated_at = date('r');
        }

    }

Then you can use it in your model as follows:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Products extends Model
    {
        use MyTimestampable;
    }

事务管理Transactions
------------------------
当一个进程执行多个数据库操作,通常每一步都必须成功完成以保证数据的完整性。事务管理提供确保所有数据库操作都成功执行后再将数据提交到数据库的功能。

When a process performs multiple database operations, it is often that each step is completed successfully so that data integrity can
be maintained. Transactions offer the ability to ensure that all database operations have been executed successfully before the data
are committed to the database.

Phalcon中的事务管理允许您提交所有操作在他们被成功执行后，如果出现任何差错就执行回滚操作。

Transactions in Phalcon allow you to commit all operations if they have been executed successfully or rollback
all operations if something went wrong.

手动事务管理Manual Transactions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果一个应用程序只使用一个连接和事务不是很复杂,一个事务只是将当前连接创建为事务模式,根据操作是否成功做回滚或提交:

If an application only uses one connection and the transactions aren't very complex, a transaction can be
created by just moving the current connection to transaction mode, doing a rollback or commit if the operation
is successfully or not:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class RobotsController extends Controller
    {
        public function saveAction()
        {
            $this->db->begin();

            $robot              = new Robots();
            $robot->name        = "WALL·E";
            $robot->created_at  = date("Y-m-d");

            if ($robot->save() == false) {
                $this->db->rollback();
                return;
            }

            $robotPart            = new RobotParts();
            $robotPart->robots_id = $robot->id;
            $robotPart->type      = "head";

            if ($robotPart->save() == false) {
                $this->db->rollback();
                return;
            }

            $this->db->commit();
        }
    }

隐式事务管理Implicit Transactions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在数据库之前有引用关联操作存储记录的情况下,隐式地创建了一个事务处理,确保数据完整正确存储:

Existing relationships can be used to store records and their related instances, this kind of operation
implicitly creates a transaction to ensure that data are correctly stored:

.. code-block:: php

    <?php

    $robotPart          = new RobotParts();
    $robotPart->type    = "head";

    $robot              = new Robots();
    $robot->name        = "WALL·E";
    $robot->created_at  = date("Y-m-d");
    $robot->robotPart   = $robotPart;

    $robot->save(); //Creates an implicit transaction to store both records

单独事务管理Isolated Transactions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
孤立的事务中执行一个新的连接确保所有生成的SQL,虚拟外键检查和业务规则相对于主连接是孤立的。这种全局性的事务需要一个事务管理器管理每个事务确保他们在请求执行结束前正确提交/回滚:

Isolated transactions are executed in a new connection ensuring that all the generated SQL,
virtual foreign key checks and business rules are isolated from the main connection.
This kind of transaction requires a transaction manager that globally manages each
transaction created ensuring that they are correctly rolled back/committed before ending the request:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\Transaction\Failed as TxFailed;
    use Phalcon\Mvc\Model\Transaction\Manager as TxManager;

    try {

        //Create a transaction manager
        $manager     = new TxManager();

        // Request a transaction
        $transaction = $manager->get();

        $robot              = new Robots();
        $robot->setTransaction($transaction);
        $robot->name        = "WALL·E";
        $robot->created_at  = date("Y-m-d");
        if ($robot->save() == false) {
            $transaction->rollback("Cannot save robot");
        }

        $robotPart              = new RobotParts();
        $robotPart->setTransaction($transaction);
        $robotPart->robots_id   = $robot->id;
        $robotPart->type        = "head";
        if ($robotPart->save() == false) {
            $transaction->rollback("Cannot save robot part");
        }

        //Everything goes fine, let's commit the transaction
        $transaction->commit();

    } catch(TxFailed $e) {
        echo "Failed, reason: ", $e->getMessage();
    }

事务可以用来以一致的方式删除许多记录:	
	
Transactions can be used to delete many records in a consistent way:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\Transaction\Failed as TxFailed;
    use Phalcon\Mvc\Model\Transaction\Manager as TxManager;

    try {

        //Create a transaction manager
        $manager     = new TxManager();

        //Request a transaction
        $transaction = $manager->get();

        //Get the robots will be deleted
        foreach (Robots::find("type = 'mechanical'") as $robot) {
            $robot->setTransaction($transaction);
            if ($robot->delete() == false) {
                //Something goes wrong, we should to rollback the transaction
                foreach ($robot->getMessages() as $message) {
                    $transaction->rollback($message->getMessage());
                }
            }
        }

        //Everything goes fine, let's commit the transaction
        $transaction->commit();

        echo "Robots were deleted successfully!";

    } catch(TxFailed $e) {
        echo "Failed, reason: ", $e->getMessage();
    }

事务管理可以被重用无论事务管理对象在哪里被获取。只有当一个commit()和rollback()被执行后才会生成一个新的事务管理。您可以使用服务容器为整个应用程序创建全局事务管理器:	
	
Transactions are reused no matter where the transaction object is retrieved. A new transaction is generated only when a commit() or rollback()
is performed. You can use the service container to create the global transaction manager for the entire application:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\Transaction\Manager as TransactionManager

    $di->setShared('transactions', function(){
        return new TransactionManager();
    });

然后从一个控制器或视图访问它	
	
Then access it from a controller or view:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class ProductsController extends Controller
    {

        public function saveAction()
        {

            //Obtain the TransactionsManager from the services container
            $manager     = $this->di->getTransactions();

            //Or
            $manager     = $this->transactions;

            //Request a transaction
            $transaction = $manager->get();

            //...
        }

    }

当一个事务管理被激活后,事务管理器将始终返回相同的事务管理在整个应用程序。	
	
While a transaction is active, the transaction manager will always return the same transaction across the application.

独立的列映射Independent Column Mapping
-----------------------------------------
ORM支持一个独立的列映射,它允许开发人员相对于数据中的实际定义在模型中使用不同的列名。Phalcon将识别到新的列名并相应地将重命名来匹配在数据库中相应的列。这是一个非常好的的特性,当需要在数据库重命名字段,而不必担心在代码中所有的查询。改变模型中的列映射会自动完整剩下的操作。例如:

The ORM supports an independent column map, which allows the developer to use different column names in the model to the ones in
the table. Phalcon will recognize the new column names and will rename them accordingly to match the respective columns in the database.
This is a great feature when one needs to rename fields in the database without having to worry about all the queries
in the code. A change in the column map in the model will take care of the rest. For example:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function columnMap()
        {
            //Keys are the real names in the table and键是数据库中实际值
            //the values their names in the application值是应用中映射的值
            return array(
                'id'       => 'code',
                'the_name' => 'theName',
                'the_type' => 'theType',
                'the_year' => 'theYear'
            );
        }

    }

然后可以在代码使用新的名字:	
	
Then you can use the new names naturally in your code:

.. code-block:: php

    <?php

    //Find a robot by its name
    $robot = Robots::findFirst("theName = 'Voltron'");
    echo $robot->theName, "\n";

    //Get robots ordered by type
    $robot = Robots::find(array('order' => 'theType DESC'));
    foreach ($robots as $robot) {
        echo 'Code: ', $robot->code, "\n";
    }

    //Create a robot
    $robot          = new Robots();
    $robot->code    = '10101';
    $robot->theName = 'Bender';
    $robot->theType = 'Industrial';
    $robot->theYear = 2999;
    $robot->save();

如果进行列重命名需要考虑以下几点：	
	
Take into consideration the following the next when renaming your columns:

* 属性的引用关系/验证器必须使用新名称　　
* 引用真实的列名将导致ORM异常

* References to attributes in relationships/validators must use the new names
* Refer the real column names will result in an exception by the ORM

独立的列映射允许我们:

The independent column map allow you to:

* 使用自己的约定编写应用程序　　
* 在代码中消除特定的前缀/后缀　　
* 改变实际列名称不需要去改变应用程序代码

* Write applications using your own conventions
* Eliminate vendor prefixes/suffixes in your code
* Change column names without change your application code

操作结果集Operations over Resultsets
------------------------------------------
如果结果集是完整的对象,通过简单的操作结果集就能操作数据记录:

If a resultset is composed of complete objects, the resultset is in the ability to perform operations on the records obtained in a simple manner:

更新关联表记录Updating related records
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
除了使用下面的操作：

Instead of doing this:

.. code-block:: php

    <?php

    foreach ($robots->getParts() as $part) {
        $part->stock        = 100;
        $part->updated_at   = time();
        if ($part->update() == false) {
            foreach ($part->getMessages() as $message) {
                echo $message;
            }
            break;
        }
    }

我们还可以这样操作：	
	
you can do this:

.. code-block:: php

    <?php

    $robots->getParts()->update(array(
        'stock' => 100,
        'updated_at' => time()
    ));

‘update’接受匿名函数去过滤哪些记录应该被更新：	
	
'update' also accepts an anonymous function to filter what records must be updated:

.. code-block:: php

    <?php

    $data = array(
        'stock'      => 100,
        'updated_at' => time()
    );

    //Update all the parts except these whose type is basic
    $robots->getParts()->update($data, function($part) {
        if ($part->type == Part::TYPE_BASIC) {
            return false;
        }
        return true;
    });
		
删除相关记录Deleting related records
^^^^^^^^^^^^^^^^^^^^^^^^
除了使用下面的操作：

Instead of doing this:

.. code-block:: php

    <?php

    foreach ($robots->getParts() as $part) {
        if ($part->delete() == false) {
            foreach ($part->getMessages() as $message) {
                echo $message;
            }
            break;
        }
    }

我们还可以：	
	
you can do this:

.. code-block:: php

    <?php

    $robots->getParts()->delete();

"delete"接受匿名函数去过滤哪些记录应该被删除：	
	
'delete' also accepts an anonymous function to filter what records must be deleted:

.. code-block:: php

    <?php

    //Delete only whose stock is greater or equal than zero
    $robots->getParts()->delete(function($part) {
        if ($part->stock < 0) {
            return false;
        }
        return true;
    });


记录快照Record Snapshots
-------------------------
特定的模型在查询时可以被设置为保持记录快照。您可以使用此功能来实现审计或者仅仅是想知道哪些字段在查询被修改了:

Specific models could be set to maintain a record snapshot when they’re queried. You can use this feature to implement auditing or just to know what
fields are changed according to the data queried from the persistence:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {
        public function initialize()
        {
            $this->keepSnapshots(true);
        }
    }

当激活这个功能是应用程序将消耗更多的内存来跟踪获得的原始值。在激活了该功能的模型中可以检查哪些值发生了改变:	
	
When activating this feature the application consumes a bit more of memory to keep track of the original values obtained from the persistence.
In models that have this feature activated you can check what fields changed:

.. code-block:: php

    <?php

    //Get a record from the database
    $robot = Robots::findFirst();

    //Change a column
    $robot->name = 'Other name';

    var_dump($robot->getChangedFields()); // ['name']
    var_dump($robot->hasChanged('name')); // true
    var_dump($robot->hasChanged('type')); // false

模型元数据Models Meta-Data
------------------------------
为了加速开发:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 可以帮助我们查询字段及表之前的相关约束信息。 :doc:`Phalcon\\Mvc\\Model\\MetaData <../api/Phalcon_Mvc_Model_MetaData>`使用可以满足需求并且可以缓存表的元数据。

To speed up development :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` helps you to query fields and constraints from tables
related to models. To achieve this, :doc:`Phalcon\\Mvc\\Model\\MetaData <../api/Phalcon_Mvc_Model_MetaData>` is available to manage
and cache table meta-data.

有时获取模型的元数据是必须的，我们可以通过下面方法获取：

Sometimes it is necessary to get those attributes when working with models. You can get a meta-data instance as follows:

.. code-block:: php

    <?php

    $robot      = new Robots();

    // Get Phalcon\Mvc\Model\Metadata instance
    $metaData   = $robot->getModelsMetaData();

    // Get robots fields names
    $attributes = $metaData->getAttributes($robot);
    print_r($attributes);

    // Get robots fields data types
    $dataTypes = $metaData->getDataTypes($robot);
    print_r($dataTypes);

缓存模型元数据Caching Meta-Data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
一旦应用程序在生产阶段,没有必要从数据库表查询的模型元数据，可以使用以下的适配器缓存元数据:

Once the application is in a production stage, it is not necessary to query the meta-data of the table from the database system each
time you use the table. This could be done caching the meta-data using any of the following adapters:

+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+
| Adapter | Description                                                                                                                                                                                                                                                                                                                                   | API                                                                                       |
+=========+===============================================================================================================================================================================================================================================================================================================================================+===========================================================================================+
| Memory  | This adapter is the default. The meta-data is cached only during the request. When the request is completed, the meta-data are released as part of the normal memory of the request. This adapter is perfect when the application is in development so as to refresh the meta-data in each request containing the new and/or modified fields. | :doc:`Phalcon\\Mvc\\Model\\MetaData\\Memory <../api/Phalcon_Mvc_Model_MetaData_Memory>`   |
+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+
| Session | This adapter stores meta-data in the $_SESSION superglobal. This adapter is recommended only when the application is actually using a small number of models. The meta-data are refreshed every time a new session starts. This also requires the use of session_start() to start the session before using any models.                        | :doc:`Phalcon\\Mvc\\Model\\MetaData\\Session <../api/Phalcon_Mvc_Model_MetaData_Session>` |
+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+
| Apc     | This adapter uses the `Alternative PHP Cache (APC)`_ to store the table meta-data. You can specify the lifetime of the meta-data with options. This is the most recommended way to store meta-data when the application is in production stage.                                                                                               | :doc:`Phalcon\\Mvc\\Model\\MetaData\\Apc <../api/Phalcon_Mvc_Model_MetaData_Apc>`         |
+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+
| XCache  | This adapter uses `XCache`_ to store the table meta-data. You can specify the lifetime of the meta-data with options. This is the most recommended way to store meta-data when the application is in production stage.                                                                                                                        | :doc:`Phalcon\\Mvc\\Model\\MetaData\\Xcache <../api/Phalcon_Mvc_Model_MetaData_Xcache>`   |
+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+
| Files   | This adapter uses plain files to store meta-data. By using this adapter the disk-reading is increased but the database access is reduced                                                                                                                                                                                                      | :doc:`Phalcon\\Mvc\\Model\\MetaData\\Files <../api/Phalcon_Mvc_Model_MetaData_Files>`     |
+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+

就像ORM的其他依赖项一样,从服务容器请求元数据管理器:

As other ORM's dependencies, the metadata manager is requested from the services container:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\MetaData\Apc as ApcMetaData;

    $di['modelsMetadata'] = function() {

        // Create a meta-data manager with APC
        $metaData = new ApcMetaData(array(
            "lifetime" => 86400,
            "prefix"   => "my-prefix"
        ));

        return $metaData;
    };

元数据策略Meta-Data Strategies
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
正如上面提到的默认策略是从数据库自身来获取模型的元数据。在下面的代码,信息架构被用来表明表中的字段,主键,可空的字段,数据类型,等等。

As mentioned above the default strategy to obtain the model's meta-data is database introspection. In this strategy, the information
schema is used to know the fields in a table, its primary key, nullable fields, data types, etc.

可以使用下面的代码变更默认的元数据获取方法：

You can change the default meta-data introspection in the following way:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\MetaData\Apc as ApcMetaData;

    $di['modelsMetadata'] = function() {

        // Instantiate a meta-data adapter
        $metaData = new ApcMetaData(array(
            "lifetime" => 86400,
            "prefix"   => "my-prefix"
        ));

        //Set a custom meta-data introspection strategy
        $metaData->setStrategy(new MyInstrospectionStrategy());

        return $metaData;
    };

数据库内部策略Database Introspection Strategy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
这种策略不需要任何定制，隐式地被使用元数据适配器使用。

This strategy doesn't require any customization and is implicitly used by all the meta-data adapters.

注释策略Annotations Strategy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
下面代码使用:doc:`annotations <annotations>`来描述模型中的列:

This strategy makes use of :doc:`annotations <annotations>` to describe the columns in a model:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        /**
         * @Primary
         * @Identity
         * @Column(type="integer", nullable=false)
         */
        public $id;

        /**
         * @Column(type="string", length=70, nullable=false)
         */
        public $name;

        /**
         * @Column(type="string", length=32, nullable=false)
         */
        public $type;

        /**
         * @Column(type="integer", nullable=false)
         */
        public $year;

    }

注释必须放置需要映射的属性列的上方。没有@column属性注释按照简单的类属性处理。	
	
Annotations must be placed in properties that are mapped to columns in the mapped source. Properties without the @Column annotation
are handled as simple class attributes.

支持以下注释:

The following annotations are supported:

+----------+-------------------------------------------------------+
| Name     | Description                                           |
+==========+=======================================================+
| Primary  | Mark the field as part of the table's primary key     |
+----------+-------------------------------------------------------+
| Identity | The field is an auto_increment/serial column          |
+----------+-------------------------------------------------------+
| Column   | This marks an attribute as a mapped column            |
+----------+-------------------------------------------------------+

@Column支持如下参数：

The annotation @Column supports the following parameters:

+----------+-------------------------------------------------------+
| Name     | Description                                           |
+==========+=======================================================+
| type     | The column's type (string, integer, decimal, boolean) |
+----------+-------------------------------------------------------+
| length   | The column's length if any                            |
+----------+-------------------------------------------------------+
| nullable | Set whether the column accepts null values or not     |
+----------+-------------------------------------------------------+

注释策略可以设置如下:

The annotations strategy could be set up this way:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\MetaData\Apc as ApcMetaData;
    use Phalcon\Mvc\Model\MetaData\Strategy\Annotations as StrategyAnnotations;

    $di['modelsMetadata'] = function() {

        // Instantiate a meta-data adapter
        $metaData = new ApcMetaData(array(
            "lifetime" => 86400,
            "prefix"   => "my-prefix"
        ));

        //Set a custom meta-data database introspection
        $metaData->setStrategy(new StrategyAnnotations());

        return $metaData;
    };

自定义元数据Manual Meta-Data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Phalcon可以自动获得每个模型的元数据即使开发人员没有按照上面提到的使用任何策略去手动设置它们。

Phalcon can obtain the metadata for each model automatically without the developer must set them manually
using any of the introspection strategies presented above.

开发人员也可以手动定义元数据。这种策略覆盖任何的元数据管理器中的策略。从映射的表中添加新列/修改/删除/也必须对其他的进行添加/修改/删除来保证一切运转正常。

The developer also has the option of define the metadata manually. This strategy overrides
any strategy set in the  meta-data manager. New columns added/modified/removed to/from the mapped
table must be added/modified/removed also for everything to work properly.

下面的例子显示了如何手动定义元数据:

The following example shows how to define the meta-data manually:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;
    use Phalcon\Db\Column;
    use Phalcon\Mvc\Model\MetaData;

    class Robots extends Model
    {

        public function metaData()
        {
            return array(

                //Every column in the mapped table
                MetaData::MODELS_ATTRIBUTES => array(
                    'id', 'name', 'type', 'year'
                ),

                //Every column part of the primary key
                MetaData::MODELS_PRIMARY_KEY => array(
                    'id'
                ),

                //Every column that isn't part of the primary key
                MetaData::MODELS_NON_PRIMARY_KEY => array(
                    'name', 'type', 'year'
                ),

                //Every column that doesn't allows null values
                MetaData::MODELS_NOT_NULL => array(
                    'id', 'name', 'type', 'year'
                ),

                //Every column and their data types
                MetaData::MODELS_DATA_TYPES => array(
                    'id'   => Column::TYPE_INTEGER,
                    'name' => Column::TYPE_VARCHAR,
                    'type' => Column::TYPE_VARCHAR,
                    'year' => Column::TYPE_INTEGER
                ),

                //The columns that have numeric data types
                MetaData::MODELS_DATA_TYPES_NUMERIC => array(
                    'id'   => true,
                    'year' => true,
                ),

                //The identity column, use boolean false if the model doesn't have
                //an identity column
                MetaData::MODELS_IDENTITY_COLUMN => 'id',

                //How every column must be bound/casted
                MetaData::MODELS_DATA_TYPES_BIND => array(
                    'id'   => Column::BIND_PARAM_INT,
                    'name' => Column::BIND_PARAM_STR,
                    'type' => Column::BIND_PARAM_STR,
                    'year' => Column::BIND_PARAM_INT,
                ),

                //Fields that must be ignored from INSERT SQL statements
                MetaData::MODELS_AUTOMATIC_DEFAULT_INSERT => array(
                    'year' => true
                ),

                //Fields that must be ignored from UPDATE SQL statements
                MetaData::MODELS_AUTOMATIC_DEFAULT_UPDATE => array(
                    'year' => true
                )

            );
        }

    }

设置模式Pointing to a different schema
-----------------------------------------
如果一个模型没有采用默认的设置而是映射到一个不同的模式或数据库中的表,可以使用getSchema方法定义:

If a model is mapped to a table that is in a different schemas/databases than the default. You can use the getSchema method to define that:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function getSchema()
        {
            return "toys";
        }

    }

设置多个数据库Setting multiple databases
-----------------------------------------
在Phalcon,所有模型可以属于同一个数据库连接或者是每个都独立。实际上,当:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 需要连接到数据库时它在应用程序的服务容器请求“db”服务。我们可以在服务设置的初始化中覆盖这个方法:

In Phalcon, all models can belong to the same database connection or have an individual one. Actually, when
:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` needs to connect to the database it requests the "db" service
in the application's services container. You can overwrite this service setting it in the initialize method:

.. code-block:: php

    <?php

    use Phalcon\Db\Adapter\Pdo\Mysql as MysqlPdo;
    use Phalcon\Db\Adapter\Pdo\PostgreSQL as PostgreSQLPdo;

    //This service returns a MySQL database
    $di->set('dbMysql', function() {
         return new MysqlPdo(array(
            "host"     => "localhost",
            "username" => "root",
            "password" => "secret",
            "dbname"   => "invo"
        ));
    });

    //This service returns a PostgreSQL database
    $di->set('dbPostgres', function() {
         return new PostgreSQLPdo(array(
            "host"     => "localhost",
            "username" => "postgres",
            "password" => "",
            "dbname"   => "invo"
        ));
    });

然后,在初始化方法中我们定义模型的连接服务:	
	
Then, in the Initialize method, we define the connection service for the model:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function initialize()
        {
            $this->setConnectionService('dbPostgres');
        }

    }

但Phalcon为您提供更大的灵活性,可以定义的连接必须使用“读”或“写”。这在我们用数据库做主从架构实现负载均衡的时候特别有用:	
	
But Phalcon offers you more flexibility, you can define the connection that must be used to 'read' and for 'write'. This is specially useful
to balance the load to your databases implementing a master-slave architecture:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function initialize()
        {
            $this->setReadConnectionService('dbSlave');
            $this->setWriteConnectionService('dbMaster');
        }

    }

ORM还提供水平分片功能,允许我们根据当前查询条件实现一个“分片”选择:	
	
The ORM also provides Horizontal Sharding facilities, by allowing you to implement a 'shard' selection
according to the current query conditions:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {
        /**
         * Dynamically selects a shard
         *
         * @param array $intermediate
         * @param array $bindParams
         * @param array $bindTypes
         */
        public function selectReadConnection($intermediate, $bindParams, $bindTypes)
        {
            //Check if there is a 'where' clause in the select
            if (isset($intermediate['where'])) {

                $conditions = $intermediate['where'];

                //Choose the possible shard according to the conditions
                if ($conditions['left']['name'] == 'id') {
                    $id = $conditions['right']['value'];
                    if ($id > 0 && $id < 10000) {
                        return $this->getDI()->get('dbShard1');
                    }
                    if ($id > 10000) {
                        return $this->getDI()->get('dbShard2');
                    }
                }
            }

            //Use a default shard
            return $this->getDI()->get('dbShard0');
        }

    }

“selectReadConnection”方法来选择正确的连接,这个方法可以拦截任何新查询的执行:	
	
The method 'selectReadConnection' is called to choose the right connection, this method intercepts any new
query executed:

.. code-block:: php

    <?php

    $robot = Robots::findFirst('id = 101');

记录底层SQL语句Logging Low-Level SQL Statements
-------------------------------------------------------
当使用高级抽象组件(如:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`的访问数据库,我们很难理解哪些语句最终被发送到数据库系统执行。:doc:`Phalcon\\Db <../api/Phalcon_Db>`内置支持:doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` 。所以通过:doc:`Phalcon\\Logger <../api/Phalcon_Logger>`和:doc:`Phalcon\\Db <../api/Phalcon_Db>`的交互为数据库抽象层提供了日志功能,能够让我们记录SQL语句。

When using high-level abstraction components such as :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>` to access a database, it is
difficult to understand which statements are finally sent to the database system. :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`
is supported internally by :doc:`Phalcon\\Db <../api/Phalcon_Db>`. :doc:`Phalcon\\Logger <../api/Phalcon_Logger>` interacts
with :doc:`Phalcon\\Db <../api/Phalcon_Db>`, providing logging capabilities on the database abstraction layer, thus allowing us to log SQL
statements as they happen.

.. code-block:: php

    <?php

    use Phalcon\Logger;
    use Phalcon\Events\Manager;
    use Phalcon\Logger\Adapter\File as FileLogger;
    use Phalcon\Db\Adapter\Pdo\Mysql as Connection;

    $di->set('db', function() {

        $eventsManager = new EventsManager();

        $logger = new FileLogger("app/logs/debug.log");

        //Listen all the database events
        $eventsManager->attach('db', function($event, $connection) use ($logger) {
            if ($event->getType() == 'beforeQuery') {
                $logger->log($connection->getSQLStatement(), Logger::INFO);
            }
        });

        $connection = new Connection(array(
            "host"      => "localhost",
            "username"  => "root",
            "password"  => "secret",
            "dbname"    => "invo"
        ));

        //Assign the eventsManager to the db adapter instance
        $connection->setEventsManager($eventsManager);

        return $connection;
    });

模型访问的默认数据库连接,所有发送到数据库系统的SQL语句将被记录在文件中:	
	
As models access the default database connection, all SQL statements that are sent to the database system will be logged in the file:

.. code-block:: php

    <?php

    $robot              = new Robots();
    $robot->name        = "Robby the Robot";
    $robot->created_at  = "1956-07-21";
    if ($robot->save() == false) {
        echo "Cannot save robot";
    }

上面的语言将会保存在*app/logs/db.log*中：	
	
As above, the file *app/logs/db.log* will contain something like this:

.. code-block:: irc

    [Mon, 30 Apr 12 13:47:18 -0500][DEBUG][Resource Id #77] INSERT INTO robots
    (name, created_at) VALUES ('Robby the Robot', '1956-07-21')

分析SQL语句Profiling SQL Statements
----------------------------------------
使用:doc:`Phalcon\\Db <../api/Phalcon_Db>`,底层组件 :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`可以通过分析由ORM产生的SQL语言来分析数据库操作的性能让我们可以诊断性能问题并发现瓶颈。

Thanks to :doc:`Phalcon\\Db <../api/Phalcon_Db>`, the underlying component of :doc:`Phalcon\\Mvc\\Model <../api/Phalcon_Mvc_Model>`,
it's possible to profile the SQL statements generated by the ORM in order to analyze the performance of database operations. With
this you can diagnose performance problems and to discover bottlenecks.

.. code-block:: php

    <?php

    use Phalcon\Db\Profiler as ProfilerDb;
    use Phalcon\Events\Manager as ManagerEvent;
    use Phalcon\Db\Adapter\Pdo\Mysql as MysqlPdo;

    $di->set('profiler', function(){
        return new ProfilerDb();
    }, true);

    $di->set('db', function() use ($di) {

        $eventsManager = new ManagerEvent();

        //Get a shared instance of the DbProfiler
        $profiler      = $di->getProfiler();

        //Listen all the database events
        $eventsManager->attach('db', function($event, $connection) use ($profiler) {
            if ($event->getType() == 'beforeQuery') {
                $profiler->startProfile($connection->getSQLStatement());
            }
            if ($event->getType() == 'afterQuery') {
                $profiler->stopProfile();
            }
        });

        $connection = new MysqlPdo(array(
            "host"      => "localhost",
            "username"  => "root",
            "password"  => "secret",
            "dbname"    => "invo"
        ));

        //Assign the eventsManager to the db adapter instance
        $connection->setEventsManager($eventsManager);

        return $connection;
    });

分析一些查询:	
	
Profiling some queries:

.. code-block:: php

    <?php

    // Send some SQL statements to the database
    Robots::find();
    Robots::find(array("order" => "name"));
    Robots::find(array("limit" => 30));

    //Get the generated profiles from the profiler
    $profiles = $di->get('profiler')->getProfiles();

    foreach ($profiles as $profile) {
       echo "SQL Statement: ", $profile->getSQLStatement(), "\n";
       echo "Start Time: ", $profile->getInitialTime(), "\n";
       echo "Final Time: ", $profile->getFinalTime(), "\n";
       echo "Total Elapsed Time: ", $profile->getTotalElapsedSeconds(), "\n";
    }

每个生成的分析文件包含每条指令以毫秒为单位的完成时间以及生成的SQL语句。	
	
Each generated profile contains the duration in milliseconds that each instruction takes to complete as well as the generated SQL statement.

服务注入到模型Injecting services into Models
------------------------------------------------
您可能需要在一个模型中访问应用程序的服务,下面的示例说明了如何做到这一点:

You may be required to access the application services within a model, the following example explains how to do that:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Robots extends Model
    {

        public function notSave()
        {
            //Obtain the flash service from the DI container
            $flash = $this->getDI()->getFlash();

            //Show validation messages
            foreach ($this->getMessages() as $message) {
                $flash->error($message);
            }
        }

    }
	
每次“创造”或“更新”action执行失败后“notSave”事件被触发。所以我们提示验证消息通过DI容器“flash”服务的。通过这样做,我们不必每次保存后打印信息。	

The "notSave" event is triggered every time that a "create" or "update" action fails. So we're flashing the validation messages
obtaining the "flash" service from the DI container. By doing this, we don't have to print messages after each save.

禁用或启用特性Disabling/Enabling Features
---------------------------------------------------
在ORM中实现了一个机制允许快速的启用/禁用特定功能或全局选项。根据如何使用ORM可以永久或者是临时禁用不需要的功能。:

In the ORM we have implemented a mechanism that allow you to enable/disable specific features or options globally on the fly.
According to how you use the ORM you can disable that you aren't using. These options can also be temporarily disabled if required:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    Model::setup(array(
        'events' => false,
        'columnRenaming' => false
    ));

可用的选项如下：	
	
The available options are:

+---------------------+----------------------------------------------------------------------------------+---------+
| Option              | Description                                                                      | Default |
+=====================+==================================================================================+=========+
| events              | Enables/Disables callbacks, hooks and event notifications from all the models    | true    |
+---------------------+----------------------------------------------------------------------------------+---------+
| columnRenaming      | Enables/Disables the column renaming                                             | true    |
+---------------------+----------------------------------------------------------------------------------+---------+
| notNullValidations  | The ORM automatically validate the not null columns present in the mapped table  | true    |
+---------------------+----------------------------------------------------------------------------------+---------+
| virtualForeignKeys  | Enables/Disables the virtual foreign keys                                        | true    |
+---------------------+----------------------------------------------------------------------------------+---------+
| phqlLiterals        | Enables/Disables literals in the PHQL parser                                     | true    |
+---------------------+----------------------------------------------------------------------------------+---------+


独立组件Stand-Alone component
----------------------------------
:doc:`Phalcon\\Mvc\\Model <models>`可以如下的代码所示独立使用：

Using :doc:`Phalcon\\Mvc\\Model <models>` in a stand-alone mode can be demonstrated below:

.. code-block:: php

    <?php

    use Phalcon\DI;
    use Phalcon\Mvc\Model;
    use Phalcon\Mvc\Model\Manager as ModelsManager;
    use Phalcon\Db\Adapter\Pdo\Sqlite as Connection;
    use Phalcon\Mvc\Model\Metadata\Memory as MetaData;

    $di = new DI();

    //Setup a connection
    $di->set('db', new Connection(array(
        "dbname" => "sample.db"
    )));

    //Set a models manager
    $di->set('modelsManager', new ModelsManager());

    //Use the memory meta-data adapter or other
    $di->set('modelsMetadata', new MetaData());

    //Create a model
    class Robots extends Model
    {

    }

    //Use the model
    echo Robots::count();

.. _Alternative PHP Cache (APC): http://www.php.net/manual/en/book.apc.php
.. _XCache: http://xcache.lighttpd.net/
.. _PDO: http://www.php.net/manual/en/pdo.prepared-statements.php
.. _date: http://php.net/manual/en/function.date.php
.. _time: http://php.net/manual/en/function.time.php
.. _Traits: http://php.net/manual/en/language.oop5.traits.php
