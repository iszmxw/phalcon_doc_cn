Phalcon查询语言 Phalcon Query Language (PHQL)
===============================================
Phalcon查询语言,PhalconQL或者加PHQL是一个高层次、面向对象的SQL语言,允许使用标准化的sql的语言编写查询。PHQL是一个SQL解析器(用C编写),把查询语句翻译目标RDBMS的语法。

Phalcon Query Language, PhalconQL or simply PHQL is a high-level, object-oriented SQL dialect that allows to write queries using a
standardized SQL-like language. PHQL is implemented as a parser (written in C) that translates syntax in that of the target RDBMS.

为了达到最高的性能,Phalcon提供了一个和 SQLite_ 使用相同的技术解析器。这种技术是一个非常小的常驻内存解析器占用非常低的内存,同时也是线程安全的。

To achieve the highest performance possible, Phalcon provides a parser that uses the same technology as SQLite_. This technology
provides a small in-memory parser with a very low memory footprint that is also thread-safe.

解析器首先检查传递过来的PHQL语句的语法,然后构建一个语句的中间表示，最后将其转换为相应的的目标RDBMS的SQL查询语句。

The parser first checks the syntax of the pass PHQL statement, then builds an intermediate representation of the statement and
finally it converts it to the respective SQL dialect of the target RDBMS.

在PHQL,我们实现一组使你对数据库的访问更安全的特性:

In PHQL, we've implemented a set of features to make your access to databases more secure:

* 绑定参数是PHQL语言的内置功能帮助确保代码安全　　
* PHQL只允许每次请求执行一个SQL语句预防SQL注入　　
* PHQL忽略所有通常用于SQL注入SQL的注释　　
* PHQL只允许数据操作语句,避免错误或者是没有授权的去修改或删除表/数据库
* PHQL实现高层抽象允许将表作为模型，将字段作为类属性来处理。

* Bound parameters are part of the PHQL language helping you to secure your code
* PHQL only allows one SQL statement to be executed per call preventing injections
* PHQL ignores all SQL comments which are often used in SQL injections
* PHQL only allows data manipulation statements, avoiding altering or dropping tables/databases by mistake or externally without authorization
* PHQL implements a high-level abstraction allowing you to handle tables as models and fields as class attributes

使用方法 Usage Example
------------------------
更好地解释PHQL工作参考下面的例子。我们有两个模型“Cars”和“Brands”:

To better explain how PHQL works consider the following example. We have two models “Cars” and “Brands”:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Cars extends Model
    {
        public $id;

        public $name;

        public $brand_id;

        public $price;

        public $year;

        public $style;

        /**
         * This model is mapped to the table sample_cars
         */
        public function getSource()
        {
            return 'sample_cars';
        }

        /**
         * A car only has a Brand, but a Brand have many Cars
         */
        public function initialize()
        {
            $this->belongsTo('brand_id', 'Brands', 'id');
        }
    }

每个车都品牌，所有一个品牌下有多辆车：	
	
And every Car has a Brand, so a Brand has many Cars:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    class Brands extends Model
    {

        public $id;

        public $name;

        /**
         * The model Brands is mapped to the "sample_brands" table
         */
        public function getSource()
        {
            return 'sample_brands';
        }

        /**
         * A Brand can have many Cars
         */
        public function initialize()
        {
            $this->hasMany('id', 'Cars', 'brand_id');
        }
    }

创建PHQL查询 Creating PHQL Queries
-------------------------------------
通过:doc:`Phalcon\\Mvc\\Model\\Query <../api/Phalcon_Mvc_Model_Query>`的实例可以创建PHQL查询。

PHQL queries can be created just by instantiating the class :doc:`Phalcon\\Mvc\\Model\\Query <../api/Phalcon_Mvc_Model_Query>`:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model\Query;

    // Instantiate the Query
    $query = new Query("SELECT * FROM Cars", $this->getDI());

    // Execute the query returning a result if any
    $cars = $query->execute();

从一个控制器或者是视图使用:doc:`models manager <../api/Phalcon_Mvc_Model_Manager>`可以非常简单的创建或者是执行PHQL:	
	
From a controller or a view, it's easy to create/execute them using an injected :doc:`models manager <../api/Phalcon_Mvc_Model_Manager>`:

.. code-block:: php

    <?php

    //Executing a simple query
    $query  = $this->modelsManager->createQuery("SELECT * FROM Cars");
    $cars   = $query->execute();

    //With bound parameters
    $query  = $this->modelsManager->createQuery("SELECT * FROM Cars WHERE name = :name:");
    $cars   = $query->execute(array(
        'name' => 'Audi'
    ));

或简单的执行：	
	
Or simply execute it:

.. code-block:: php

    <?php

    //Executing a simple query
    $cars = $this->modelsManager->executeQuery("SELECT * FROM Cars");

    //Executing with bound parameters
    $cars = $this->modelsManager->executeQuery("SELECT * FROM Cars WHERE name = :name:", array(
        'name' => 'Audi'
    ));

查询记录 Selecting Records
-----------------------------
就像普通的SQL语句一样，PHQL允许使用SELECT去查询记录，除非在一个特定的数据表中，否则我们使用模型类：

As the familiar SQL, PHQL allows querying of records using the SELECT statement we know, except that instead of specifying tables, we use the models classes:

.. code-block:: php

    <?php

    $query = $manager->createQuery("SELECT * FROM Cars ORDER BY Cars.name");
    $query = $manager->createQuery("SELECT Cars.name FROM Cars ORDER BY Cars.name");

命名空间中的类也是可以的：	
	
Classes in namespaces are also allowed:

.. code-block:: php

    <?php

    $phql   = "SELECT * FROM Formula\Cars ORDER BY Formula\Cars.name";
    $query  = $manager->createQuery($phql);

    $phql   = "SELECT Formula\Cars.name FROM Formula\Cars ORDER BY Formula\Cars.name";
    $query  = $manager->createQuery($phql);

    $phql   = "SELECT c.name FROM Formula\Cars c ORDER BY c.name";
    $query  = $manager->createQuery($phql);

PHQL支持大多数SQL标准语句，甚至是非标准的LIMIT也支持：	
	
Most of the SQL standard is supported by PHQL, even nonstandard directives such as LIMIT:

.. code-block:: php

    <?php

    $phql   = "SELECT c.name FROM Cars AS c "
       . "WHERE c.brand_id = 21 ORDER BY c.name LIMIT 100";
    $query = $manager->createQuery($phql);

结果类型Result Types
^^^^^^^^^^^^^^^^^^^^^^
根据查询的不同，返回数据的类型也是多种多样的。如果检索一个整体对象，返回的类型就为:doc:`Phalcon\\Mvc\\Model\\Resultset\\Simple <../api/Phalcon_Mvc_Model_Resultset_Simple>`。这样的结果集是一组完整的模型对象。

Depending on the type of columns we query, the result type will vary. If you retrieve a single whole object, then the object returned is
a :doc:`Phalcon\\Mvc\\Model\\Resultset\\Simple <../api/Phalcon_Mvc_Model_Resultset_Simple>`. This kind of resultset is a set of complete model objects:

.. code-block:: php

    <?php

    $phql = "SELECT c.* FROM Cars AS c ORDER BY c.name";
    $cars = $manager->executeQuery($phql);
    foreach ($cars as $car) {
        echo "Name: ", $car->name, "\n";
    }

使用下面的结果是一样的：	
	
This is exactly the same as:

.. code-block:: php

    <?php

    $cars = Cars::find(array("order" => "name"));
    foreach ($cars as $car) {
        echo "Name: ", $car->name, "\n";
    }

完成对象可以修改并重新保存在数据库中,因为他们代表一个完整的表相关联的记录。也有查询不返回完整的对象,例如:	
	
Complete objects can be modified and re-saved in the database because they represent a complete record of the associated table. There are
other types of queries that do not return complete objects, for example:

.. code-block:: php

    <?php

    $phql = "SELECT c.id, c.name FROM Cars AS c ORDER BY c.name";
    $cars = $manager->executeQuery($phql);
    foreach ($cars as $car) {
        echo "Name: ", $car->name, "\n";
    }

我们只是请求一些表中字段,因此不能被视为一个完整的对象,所以返回的对象还是resulset :doc:`Phalcon\\Mvc\\Model\\Resultset\\Simple <../api/Phalcon_Mvc_Model_Resultset_Simple>`。然而,每个元素是一个标准只包含请求的两列的对象。	
	
We are only requesting some fields in the table, therefore those cannot be considered an entire object, so the returned object is
still a resulset of type :doc:`Phalcon\\Mvc\\Model\\Resultset\\Simple <../api/Phalcon_Mvc_Model_Resultset_Simple>`. However, each element is a standard
object that only contain the two columns that were requested.

这些并不代表完整的对象的值我们称为标量scalars。PHQL允许您查询所有的标量类型:字段,函数,字符,表达式,等:

These values that don't represent complete objects are what we call scalars. PHQL allows you to query all types of scalars: fields, functions, literals, expressions, etc..:

.. code-block:: php

    <?php

    $phql = "SELECT CONCAT(c.id, ' ', c.name) AS id_name FROM Cars AS c ORDER BY c.name";
    $cars = $manager->executeQuery($phql);
    foreach ($cars as $car) {
        echo $car->id_name, "\n";
    }

我们可以查询完整的对象或标量,同时我们也可以查询两者:	
	
As we can query complete objects or scalars, we can also query both at once:

.. code-block:: php

    <?php

    $phql   = "SELECT c.price*0.16 AS taxes, c.* FROM Cars AS c ORDER BY c.name";
    $result = $manager->executeQuery($phql);

这种情况下结果集是:doc:`Phalcon\\Mvc\\Model\\Resultset\\Complex <../api/Phalcon_Mvc_Model_Resultset_Complex>`类型的。这种允许我们同时访问完整的对象或者是标量：	
	
The result in this case is an object :doc:`Phalcon\\Mvc\\Model\\Resultset\\Complex <../api/Phalcon_Mvc_Model_Resultset_Complex>`.
This allows access to both complete objects and scalars at once:

.. code-block:: php

    <?php

    foreach ($result as $row) {
        echo "Name: ", $row->cars->name, "\n";
        echo "Price: ", $row->cars->price, "\n";
        echo "Taxes: ", $row->taxes, "\n";
    }

标量映射作为“行”的属性,而完整的对象映射作为相关模型的属性名称。	
	
Scalars are mapped as properties of each "row", while complete objects are mapped as properties with the name of its related model.

Joins
^^^^^
使用PHQL很容易从多个模型请求记录。支持大多数类型的连接。当我们定义模型关系,PHQL自动添加这些条件:

It's easy to request records from multiple models using PHQL. Most kinds of Joins are supported. As we defined
relationships in the models, PHQL adds these conditions automatically:

.. code-block:: php

    <?php

    $phql  = "SELECT Cars.name AS car_name, Brands.name AS brand_name FROM Cars JOIN Brands";
    $rows  = $manager->executeQuery($phql);
    foreach ($rows as $row) {
        echo $row->car_name, "\n";
        echo $row->brand_name, "\n";
    }

默认使用INNER JOIN ，我们可以在请求中定义join类型：	
	
By default, an INNER JOIN is assumed. You can specify the type of JOIN in the query:

.. code-block:: php

    <?php

    $phql = "SELECT Cars.*, Brands.* FROM Cars INNER JOIN Brands";
    $rows = $manager->executeQuery($phql);

    $phql = "SELECT Cars.*, Brands.* FROM Cars LEFT JOIN Brands";
    $rows = $manager->executeQuery($phql);

    $phql = "SELECT Cars.*, Brands.* FROM Cars LEFT OUTER JOIN Brands";
    $rows = $manager->executeQuery($phql);

    $phql = "SELECT Cars.*, Brands.* FROM Cars CROSS JOIN Brands";
    $rows = $manager->executeQuery($phql);

同样可以手动设置join条件：	
	
Also is possible set manually the conditions of the JOIN:

.. code-block:: php

    <?php

    $phql = "SELECT Cars.*, Brands.* FROM Cars INNER JOIN Brands ON Brands.id = Cars.brands_id";
    $rows = $manager->executeQuery($phql);

同样可以使用FROM join多个表：	
	
Also, the joins can be created using multiple tables in the FROM clause:

.. code-block:: php

    <?php

    $phql = "SELECT Cars.*, Brands.* FROM Cars, Brands WHERE Brands.id = Cars.brands_id";
    $rows = $manager->executeQuery($phql);
    foreach ($rows as $row) {
        echo "Car: ", $row->cars->name, "\n";
        echo "Brand: ", $row->brands->name, "\n";
    }

如果使用别名来重命名模型查询中,这些将被用来命名属性的每一行的结果:	
	
If an alias is used to rename the models in the query, those will be used to name the attributes in the every row of the result:

.. code-block:: php

    <?php

    $phql = "SELECT c.*, b.* FROM Cars c, Brands b WHERE b.id = c.brands_id";
    $rows = $manager->executeQuery($phql);
    foreach ($rows as $row) {
        echo "Car: ", $row->c->name, "\n";
        echo "Brand: ", $row->b->name, "\n";
    }

当别连接的模型有多对多的关系时，中间模型隐式添加到生成的查询语句中:	
	
When the joined model has a many-to-many relation to the 'from' model, the intermediate model is implicitly added to the generated query:

.. code-block:: php

    <?php

    $phql = 'SELECT Brands.name, Songs.name FROM Artists ' .
            'JOIN Songs WHERE Artists.genre = "Trip-Hop"';
    $result = $this->modelsManager->query($phql);

上面代码会生成如下查询语句：	
	
This code produces the following SQL in MySQL:

.. code-block:: sql

    SELECT `brands`.`name`, `songs`.`name` FROM `artists`
    INNER JOIN `albums` ON `albums`.`artists_id` = `artists`.`id`
    INNER JOIN `songs` ON `albums`.`songs_id` = `songs`.`id`
    WHERE `artists`.`genre` = 'Trip-Hop'

聚合Aggregations
^^^^^^^^^^^^^^^^^^^^^
下面的例子展示如何使用PHQL聚合:

The following examples show how to use aggregations in PHQL:

.. code-block:: php

    <?php

    // How much are the prices of all the cars?
    $phql = "SELECT SUM(price) AS summatory FROM Cars";
    $row  = $manager->executeQuery($phql)->getFirst();
    echo $row['summatory'];

    // How many cars are by each brand?
    $phql = "SELECT Cars.brand_id, COUNT(*) FROM Cars GROUP BY Cars.brand_id";
    $rows = $manager->executeQuery($phql);
    foreach ($rows as $row) {
        echo $row->brand_id, ' ', $row["1"], "\n";
    }

    // How many cars are by each brand?
    $phql = "SELECT Brands.name, COUNT(*) FROM Cars JOIN Brands GROUP BY 1";
    $rows = $manager->executeQuery($phql);
    foreach ($rows as $row) {
        echo $row->name, ' ', $row["1"], "\n";
    }

    $phql = "SELECT MAX(price) AS maximum, MIN(price) AS minimum FROM Cars";
    $rows = $manager->executeQuery($phql);
    foreach ($rows as $row) {
        echo $row["maximum"], ' ', $row["minimum"], "\n";
    }

    // Count distinct used brands
    $phql = "SELECT COUNT(DISTINCT brand_id) AS brandId FROM Cars";
    $rows = $manager->executeQuery($phql);
    foreach ($rows as $row) {
        echo $row->brandId, "\n";
    }

条件Conditions
^^^^^^^^^^^^^^^^^
条件查询允许我们过滤得到我们想要的记录。WHERE子句允许这样做:

Conditions allow us to filter the set of records we want to query. The WHERE clause allows to do that:

.. code-block:: php

    <?php

    // Simple conditions
    $phql = "SELECT * FROM Cars WHERE Cars.name = 'Lamborghini Espada'";
    $cars = $manager->executeQuery($phql);

    $phql = "SELECT * FROM Cars WHERE Cars.price > 10000";
    $cars = $manager->executeQuery($phql);

    $phql = "SELECT * FROM Cars WHERE TRIM(Cars.name) = 'Audi R8'";
    $cars = $manager->executeQuery($phql);

    $phql = "SELECT * FROM Cars WHERE Cars.name LIKE 'Ferrari%'";
    $cars = $manager->executeQuery($phql);

    $phql = "SELECT * FROM Cars WHERE Cars.name NOT LIKE 'Ferrari%'";
    $cars = $manager->executeQuery($phql);

    $phql = "SELECT * FROM Cars WHERE Cars.price IS NULL";
    $cars = $manager->executeQuery($phql);

    $phql = "SELECT * FROM Cars WHERE Cars.id IN (120, 121, 122)";
    $cars = $manager->executeQuery($phql);

    $phql = "SELECT * FROM Cars WHERE Cars.id NOT IN (430, 431)";
    $cars = $manager->executeQuery($phql);

    $phql = "SELECT * FROM Cars WHERE Cars.id BETWEEN 1 AND 100";
    $cars = $manager->executeQuery($phql);

准备参数作为PHQL功能的一部分会自动转义输入数据,让数据库操作更安全:	
	
Also, as part of PHQL, prepared parameters automatically escape the input data, introducing more security:

.. code-block:: php

    <?php

    $phql = "SELECT * FROM Cars WHERE Cars.name = :name:";
    $cars = $manager->executeQuery($phql, array("name" => 'Lamborghini Espada'));

    $phql = "SELECT * FROM Cars WHERE Cars.name = ?0";
    $cars = $manager->executeQuery($phql, array(0 => 'Lamborghini Espada'));


插入数据Inserting Data
--------------------------
在PHQL里面可以使用熟悉的INSERT语句插入数据：

With PHQL it's possible to insert data using the familiar INSERT statement:

.. code-block:: php

    <?php

    // Inserting without columns
    $phql = "INSERT INTO Cars VALUES (NULL, 'Lamborghini Espada', "
          . "7, 10000.00, 1969, 'Grand Tourer')";
    $manager->executeQuery($phql);

    // Specifying columns to insert
    $phql = "INSERT INTO Cars (name, brand_id, year, style) "
          . "VALUES ('Lamborghini Espada', 7, 1969, 'Grand Tourer')";
    $manager->executeQuery($phql);

    // Inserting using placeholders
    $phql = "INSERT INTO Cars (name, brand_id, year, style) "
          . "VALUES (:name:, :brand_id:, :year:, :style)";
    $manager->executeQuery($sql,
        array(
            'name'     => 'Lamborghini Espada',
            'brand_id' => 7,
            'year'     => 1969,
            'style'    => 'Grand Tourer',
        )
    );

Phalcon不仅可以将PHQL语句转换成SQL。所有在模型中事件和业务规则定义将会被执行就好像我们手动创建了单个对象。让我们在汽车模型中添加一条业务规则。一辆车价格不能低于10000美元。	
	
Phalcon doesn't only transform the PHQL statements into SQL. All events and business rules defined
in the model are executed as if we created individual objects manually. Let's add a business rule
on the model cars. A car cannot cost less than $ 10,000:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;
    use Phalcon\Mvc\Model\Message;

    class Cars extends Model
    {

        public function beforeCreate()
        {
            if ($this->price < 10000)
            {
                $this->appendMessage(new Message("A car cannot cost less than $ 10,000"));
                return false;
            }
        }

    }

如果我们在汽车模型执行如下插入操作将不会成功，因为价格不符合我们定义的业务规则,:	
	
If we made the following INSERT in the models Cars, the operation will not be successful
because the price does not meet the business rule that we implemented:

.. code-block:: php

    <?php

    $phql   = "INSERT INTO Cars VALUES (NULL, 'Nissan Versa', 7, 9999.00, 2012, 'Sedan')";
    $result = $manager->executeQuery($phql);
    if ($result->success() == false)
    {
        foreach ($result->getMessages() as $message)
        {
            echo $message->getMessage();
        }
    }

更新数据Updating Data
-------------------------
更新数据和插入非常相似。正如我们知道的一样更新用UPDATE语句。当一个记录被更新时和更新操作相关的事件将会每一行记录被执行一次。

Updating rows is very similar than inserting rows. As you may know, the instruction to
update records is UPDATE. When a record is updated the events related to the update operation
will be executed for each row.

.. code-block:: php

    <?php

    // Updating a single column
    $phql = "UPDATE Cars SET price = 15000.00 WHERE id = 101";
    $manager->executeQuery($phql);

    // Updating multiples columns
    $phql = "UPDATE Cars SET price = 15000.00, type = 'Sedan' WHERE id = 101";
    $manager->executeQuery($phql);

    // Updating multiples rows
    $phql = "UPDATE Cars SET price = 7000.00, type = 'Sedan' WHERE brands_id > 5";
    $manager->executeQuery($phql);

    // Using placeholders
    $phql = "UPDATE Cars SET price = ?0, type = ?1 WHERE brands_id > ?2";
    $manager->executeQuery($phql, array(
        0 => 7000.00,
        1 => 'Sedan',
        2 => 5
    ));

UPDATE语句通过以下两个阶段执行：	
	
An UPDATE statement performs the update in two phases:

* 首先,如果更新语句包含WHERE则检索符合这些条件的所有对象
* 第二,将查询到的对象属性进行更新或修改，并存储到关系数据库中

* First, if the UPDATE has a WHERE clause it retrieves all the objects that match these criteria,
* Second, based on the queried objects it updates/changes the requested attributes storing them to the relational database

这种更新方式允许事件,虚拟外键和验证。如下代码:

This way of operation allows that events, virtual foreign keys and validations take part of the updating process.
In summary, the following code:

.. code-block:: php

    <?php

    $phql    = "UPDATE Cars SET price = 15000.00 WHERE id > 101";
    $success = $manager->executeQuery($phql);

相当于：	
	
is somewhat equivalent to:

.. code-block:: php

    <?php

    $messages = null;

    $process  = function() use (&$messages) {
        foreach (Cars::find("id > 101") as $car) {
            $car->price = 15000;
            if ($car->save() == false) {
                $messages = $car->getMessages();
                return false;
            }
        }
        return true;
    };

    $success = $process();

删除数据Deleting Data
----------------------------
当一条记录删除时相关的事件将会每一行执行一次:

When a record is deleted the events related to the delete operation will be executed for each row:

.. code-block:: php

    <?php

    // Deleting a single row
    $phql = "DELETE FROM Cars WHERE id = 101";
    $manager->executeQuery($phql);

    // Deleting multiple rows
    $phql = "DELETE FROM Cars WHERE id > 100";
    $manager->executeQuery($phql);

    // Using placeholders
    $phql = "DELETE FROM Cars WHERE id BETWEEN :initial: AND :final:";
    $manager->executeQuery(
        $phql,
        array(
            'initial' => 1,
            'final'   => 100
        )
    );

删除操作也像UPDATE一样分两个阶段执行。	
	
DELETE operations are also executed in two phases like UPDATEs.

使用Query Builder创建查询Creating queries using the Query Builder
-------------------------------------------------------------------
查询构建器可以创建PHQL查询,而不需要编写PHQL语句,也提供IDE整合:

A builder is available to create PHQL queries without the need to write PHQL statements, also providing IDE facilities:

.. code-block:: php

    <?php

    //Getting a whole set
    $robots = $this->modelsManager->createBuilder()
        ->from('Robots')
        ->join('RobotsParts')
        ->orderBy('Robots.name')
        ->getQuery()
        ->execute();

    //Getting the first row
    $robots = $this->modelsManager->createBuilder()
        ->from('Robots')
        ->join('RobotsParts')
        ->orderBy('Robots.name')
        ->getQuery()
        ->getSingleResult();

和下面代码一样：		
		
That is the same as:

.. code-block:: php

    <?php

    $phql   = "SELECT Robots.*
        FROM Robots JOIN RobotsParts p
        ORDER BY Robots.name LIMIT 20";
    $result = $manager->executeQuery($phql);

关于构建器更多的例子：	
	
More examples of the builder:

.. code-block:: php

    <?php

    // 'SELECT Robots.* FROM Robots';
    $builder->from('Robots');

    // 'SELECT Robots.*, RobotsParts.* FROM Robots, RobotsParts';
    $builder->from(array('Robots', 'RobotsParts'));

    // 'SELECT * FROM Robots';
    $phql = $builder->columns('*')
                    ->from('Robots');

    // 'SELECT id FROM Robots';
    $builder->columns('id')
            ->from('Robots');

    // 'SELECT id, name FROM Robots';
    $builder->columns(array('id', 'name'))
            ->from('Robots');

    // 'SELECT Robots.* FROM Robots WHERE Robots.name = "Voltron"';
    $builder->from('Robots')
            ->where('Robots.name = "Voltron"');

    // 'SELECT Robots.* FROM Robots WHERE Robots.id = 100';
    $builder->from('Robots')
            ->where(100);

    // 'SELECT Robots.* FROM Robots WHERE Robots.type = "virtual" AND Robots.id > 50';
    $builder->from('Robots')
            ->where('type = "virtual"')
            ->andWhere('id > 50');

    // 'SELECT Robots.* FROM Robots WHERE Robots.type = "virtual" OR Robots.id > 50';
    $builder->from('Robots')
            ->where('type = "virtual"')
            ->orWhere('id > 50');

    // 'SELECT Robots.* FROM Robots GROUP BY Robots.name';
    $builder->from('Robots')
            ->groupBy('Robots.name');

    // 'SELECT Robots.* FROM Robots GROUP BY Robots.name, Robots.id';
    $builder->from('Robots')
            ->groupBy(array('Robots.name', 'Robots.id'));

    // 'SELECT Robots.name, SUM(Robots.price) FROM Robots GROUP BY Robots.name';
    $builder->columns(array('Robots.name', 'SUM(Robots.price)'))
        ->from('Robots')
        ->groupBy('Robots.name');

    // 'SELECT Robots.name, SUM(Robots.price) FROM Robots GROUP BY Robots.name HAVING SUM(Robots.price) > 1000';
    $builder->columns(array('Robots.name', 'SUM(Robots.price)'))
        ->from('Robots')
        ->groupBy('Robots.name')
        ->having('SUM(Robots.price) > 1000');

    // 'SELECT Robots.* FROM Robots JOIN RobotsParts';
    $builder->from('Robots')
        ->join('RobotsParts');

    // 'SELECT Robots.* FROM Robots JOIN RobotsParts AS p';
    $builder->from('Robots')
        ->join('RobotsParts', null, 'p');

    // 'SELECT Robots.* FROM Robots JOIN RobotsParts ON Robots.id = RobotsParts.robots_id AS p';
    $builder->from('Robots')
        ->join('RobotsParts', 'Robots.id = RobotsParts.robots_id', 'p');

    // 'SELECT Robots.* FROM Robots ;
    // JOIN RobotsParts ON Robots.id = RobotsParts.robots_id AS p ;
    // JOIN Parts ON Parts.id = RobotsParts.parts_id AS t';
    $builder->from('Robots')
        ->join('RobotsParts', 'Robots.id = RobotsParts.robots_id', 'p')
        ->join('Parts', 'Parts.id = RobotsParts.parts_id', 't');

    // 'SELECT r.* FROM Robots AS r';
    $builder->addFrom('Robots', 'r');

    // 'SELECT Robots.*, p.* FROM Robots, Parts AS p';
    $builder->from('Robots')
        ->addFrom('Parts', 'p');

    // 'SELECT r.*, p.* FROM Robots AS r, Parts AS p';
    $builder->from(array('r' => 'Robots'))
            ->addFrom('Parts', 'p');

    // 'SELECT r.*, p.* FROM Robots AS r, Parts AS p';
    $builder->from(array('r' => 'Robots', 'p' => 'Parts'));

    // 'SELECT Robots.* FROM Robots LIMIT 10';
    $builder->from('Robots')
        ->limit(10);

    // 'SELECT Robots.* FROM Robots LIMIT 10 OFFSET 5';
    $builder->from('Robots')
            ->limit(10, 5);

    // 'SELECT Robots.* FROM Robots WHERE id BETWEEN 1 AND 100';
    $builder->from('Robots')
            ->betweenWhere('id', 1, 100);

    // 'SELECT Robots.* FROM Robots WHERE id IN (1, 2, 3)';
    $builder->from('Robots')
            ->inWhere('id', array(1, 2, 3));

    // 'SELECT Robots.* FROM Robots WHERE id NOT IN (1, 2, 3)';
    $builder->from('Robots')
            ->notInWhere('id', array(1, 2, 3));

    // 'SELECT Robots.* FROM Robots WHERE name LIKE '%Art%';
    $builder->from('Robots')
            ->where('name LIKE :name:', array('name' => '%' . $name . '%'));

    // 'SELECT r.* FROM Store\Robots WHERE r.name LIKE '%Art%';
    $builder->from(['r' => 'Store\Robots'])
            ->where('r.name LIKE :name:', array('name' => '%' . $name . '%'));

绑定参数Bound Parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
可以在查询构建的时候也可以在查询执行的时候进行参数绑定设置:

Bound parameters in the query builder can be set as the query is constructed or past all at once when executing:

.. code-block:: php

    <?php

    //Passing parameters in the query construction
    $robots = $this->modelsManager->createBuilder()
        ->from('Robots')
        ->where('name = :name:', array('name' => $name))
        ->andWhere('type = :type:', array('type' => $type))
        ->getQuery()
        ->execute();

    //Passing parameters in query execution
    $robots = $this->modelsManager->createBuilder()
        ->from('Robots')
        ->where('name = :name:')
        ->andWhere('type = :type:')
        ->getQuery()
        ->execute(array('name' => $name, 'type' => $type));

禁止使用字面值Disallow literals in PHQL
------------------------------------------------
在PHQL中字面值被禁用,这意味着在PHQL字符串中不允许直接使用字符串、数字和布尔值。如果PHQL语句中支持创建嵌入外部数据,这将让应用有潜在的SQL注入的风险:

Literals can be disabled in PHQL, this means that directly using strings, numbers and boolean values in PHQL strings
will be disallowed. If PHQL statements are created embedding external data on them, this could open the application
to potential SQL injections:

.. code-block:: php

    <?php

    $login  = 'voltron';
    $phql   = "SELECT * FROM Models\Users WHERE login = '$login'";
    $result = $manager->executeQuery($phql);

如果$login被替换为 ' OR '' = '  生成的PHQL语句如下：	
	
If $login is changed to ' OR '' = ', the produced PHQL is:

.. code-block:: php

    <?php

    "SELECT * FROM Models\Users WHERE login = '' OR '' = ''"

无论login值是什么，条件永远成立。	
	
Which is always true no matter what the login stored in the database is.

如果字面值在PHQL中禁止了,则会抛出异常迫使开发人员使用绑定参数。上面的例子可以用更安全的方式实现如下:

If literals are disallowed strings can be used as part of a PHQL statement, thus an exception
will be thrown forcing the developer to use bound parameters. The same query can be written in a
secure way like this:

.. code-block:: php

    <?php

    $phql   = "SELECT Robots.* FROM Robots WHERE Robots.name = :name:";
    $result = $manager->executeQuery($phql, array('name' => $name));

可以用以下方式禁止字面常量：	
	
You can disallow literals in the following way:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    Model::setup(array('phqlLiterals' => false));

如果字面值是否被禁止，我们都可以使用参数绑定。禁止他们提示开发者在开发过程中应更加注意安全性问题。	
	
Bound parameters can be used even if literals are allowed or not. Disallowing them is just
another security decision a developer could take in web applications.

转义保留字Escaping Reserved Words
---------------------------------------
PHQL有一些保留关键词字,如果想要使用其中的任何一个来作为属性或模型名称,需要使用'[' 和 ']'来进行转义:

PHQL has a few reserved words, if you want to use any of them as attributes or models names, you need to escape those
words using the cross-database escaping delimiters '[' and ']':

.. code-block:: php

    <?php

    $phql   = "SELECT * FROM [Update]";
    $result = $manager->executeQuery($phql);

    $phql   = "SELECT id, [Like] FROM Posts";
    $result = $manager->executeQuery($phql);

分隔符将会动态转换成有效的分隔符根据当前应用程序运行的数据库系统。	
	
The delimiters are dynamically translated to valid delimiters depending on the database system where the application is currently running on.

PHQL 生命周期PHQL Lifecycle
---------------------------------
作为一个高级的语言,PHQL使开发人员能够个性化和定制不同的功能,以满足他们的需求。下面是每个PHQL语句执行的生命周期:

Being a high-level language, PHQL gives developers the ability to personalize and customize different aspects in order to suit their needs.
The following is the life cycle of each PHQL statement executed:

* PHQL解析和转换成一个中间表示(IR)是独立于SQL数据库系统实现的　　
* 根据红外转换为有效的SQL数据库系统模型相关　　
* PHQL语句解析一次,在内存中缓存。进一步执行相同的语句导致更快的执行

* The PHQL is parsed and converted into an Intermediate Representation (IR) which is independent of the SQL implemented by database system
* The IR is converted to valid SQL according to the database system associated to the model
* PHQL statements are parsed once and cached in memory. Further executions of the same statement result in a slightly faster execution

使用原生 SQLUsing Raw SQL
----------------------------
数据库系统可以提供特定的而PHQL不支持的SQL扩展,在这种情况下,可以使用原生的SQL:

A database system could offer specific SQL extensions that aren't supported by PHQL, in this case, a raw SQL can be appropriate:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;
    use Phalcon\Mvc\Model\Resultset\Simple as Resultset;

    class Robots extends Model
    {
        public static function findByCreateInterval()
        {
            // A raw SQL statement
            $sql   = "SELECT * FROM robots WHERE id > 0";

            // Base model
            $robot = new Robots();

            // Execute the query
            return new Resultset(null, $robot, $robot->getReadConnection()->query($sql));
        }
    }

如果原生查询经常被使用，可以将它们加入到模型中：	
	
If Raw SQL queries are common in your application a generic method could be added to your model:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;
    use Phalcon\Mvc\Model\Resultset\Simple as Resultset;

    class Robots extends Model
    {
        public static function findByRawSql($conditions, $params=null)
        {
            // A raw SQL statement
            $sql   = "SELECT * FROM robots WHERE $conditions";

            // Base model
            $robot = new Robots();

            // Execute the query
            return new Resultset(null, $robot, $robot->getReadConnection()->query($sql, $params));
        }
    }

findByRawSql可以使用如下调用方式：	
	
The above findByRawSql could be used as follows:

.. code-block:: php

    <?php

    $robots = Robots::findByRawSql('id > ?', array(10));

注意事项Troubleshooting
--------------------------
当使用PHQL时需要注意以下几点：

Some things to keep in mind when using PHQL:

* 类是区分大小写的,如果定义和调用的大小写不一致,这可能会导致意外的行为在区分大小写的文件系统中,如Linux操作系统。　　
* 当正确的设置字符集后绑定参数功能才能正常使用
* 命名空间类没有完全替换别名类，因为只是在PHP中生效的而并不是在字符串中替换
* 如果启用了列重命名应避免使用和列别名相同的名称来命名列,这可能让查询解析器混淆

* Classes are case-sensitive, if a class is not defined with the same name as it was created this could lead to an unexpected behavior in operating systems with case-sensitive file systems such as Linux.
* Correct charset must be defined in the connection to bind parameters with success
* Aliased classes aren't replaced by full namespaced classes since this only occurs in PHP code and not inside strings
* If column renaming is enabled avoid using column aliases with the same name as columns to be renamed, this may confuse the query resolver

.. _SQLite: http://en.wikipedia.org/wiki/Lemon_Parser_Generator
