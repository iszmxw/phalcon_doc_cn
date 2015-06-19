对象文档映射 ODMODM (Object-Document Mapper)
============================================
除了可以 :doc:`map tables <models>` 映射关系数据库的表之外，Phalcon还可以使用NoSQL数据库如MongoDB等。Phalcon中的ODM具有可以非常容易的实现如下功能：CRUD,事件，验证等。

In addition to its ability to :doc:`map tables <models>` in relational databases, Phalcon can map documents from NoSQL databases.
The ODM offers a CRUD functionality, events, validations among other services.

因为NoSQL数据库中无sql查询及计划等操作故可以提高数据操作的性能。再者，由于无SQL语句创建的操作故可以减少SQL注入的危险。

Due to the absence of SQL queries and planners, NoSQL databases can see real improvements in performance using the Phalcon approach.
Additionally, there are no SQL building reducing the possibility of SQL injections.

当前Phalcon中支持的NosSQL数据库如下：

The following NoSQL databases are supported:

+------------+----------------------------------------------------------------------+
| Name       | Description                                                          |
+============+======================================================================+
| MongoDB_   | MongoDB is a scalable, high-performance, open source NoSQL database. |
+------------+----------------------------------------------------------------------+

创建模型Creating Models
------------------------------
NoSQL中的模型类扩展自 :doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>`.模型必须要放入模型文件夹中而且每个模型文件必须只能有一个模型类； 模型类名应该为小驼峰法书写：

A model is a class that extends from :doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>`. It must be placed in the models directory. A model
file must contain a single class; its class name should be in camel case notation:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Collection;

    class Robots extends Collection
    {

    }

.. highlights::
	
	如果PHP版本为5.4/5.5或更高版本，为了提高性能节省内存开销，最好在模型类文件中定义每个字段。
	
    If you're using PHP 5.4/5.5 is recommended declare each column that makes part of the model in order to save
    memory and reduce the memory allocation.

模型Robots默认和数据库中的robots表格映射。如果想使用别的名字映射数据库中的表格则只需要重写getSource()方法即可：	
	
By default model "Robots" will refer to the collection "robots". If you want to manually specify another name for the mapping collection,
you can use the getSource() method:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Collection;

    class Robots extends Collection
    {
        public function getSource()
        {
            return "the_robots";
        }
    }

理解文档对象Understanding Documents To Objects
-------------------------------------------------
每个模型的实例和数据库表中的一个文档（记录）相对应。我们可以非常容易的通过读取对象属性来访问表格的数据。例如访问robots表格：

Every instance of a model represents a document in the collection. You can easily access collection data by reading object properties. For example,
for a collection "robots" with the documents:

.. code-block:: bash

    $ mongo test
    MongoDB shell version: 1.8.2
    connecting to: test
    > db.robots.find()
    { "_id" : ObjectId("508735512d42b8c3d15ec4e1"), "name" : "Astro Boy", "year" : 1952,
        "type" : "mechanical" }
    { "_id" : ObjectId("5087358f2d42b8c3d15ec4e2"), "name" : "Bender", "year" : 1999,
        "type" : "mechanical" }
    { "_id" : ObjectId("508735d32d42b8c3d15ec4e3"), "name" : "Wall-E", "year" : 2008 }
    >

模型中使用命名空间Models in Namespaces
-------------------------------------------------
我们在这里可以使用命名空间来避免类名冲突。这个例子中我们使用getSource方法来标明要使用的数据库表：

Namespaces can be used to avoid class name collision. In this case it is necessary to indicate the name of the related collection using getSource:

.. code-block:: php

    <?php

    namespace Store\Toys;

    use Phalcon\Mvc\Collection;

    class Robots extends Collection
    {

        public function getSource()
        {
            return "robots";
        }

    }

我们可以通过对象的ID查找到对象然后打印出其名字：	
	
You could find a certain document by its id and then print its name:

.. code-block:: php

    <?php

    // Find record with _id = "5087358f2d42b8c3d15ec4e2"
    $robot = Robots::findById("5087358f2d42b8c3d15ec4e2");

    // Prints "Bender"
    echo $robot->name;

一旦记录被加载到内存中，我们就可以对这些数据进行修改了，修改之后还可以保存：	
	
Once the record is in memory, you can make modifications to its data and then save changes:

.. code-block:: php

    <?php

    $robot       = Robots::findFirst(array(
        array('name' => 'Astroy Boy')
    ));
    $robot->name = "Voltron";
    $robot->save();

设置连接 Setting a Connection
---------------------------------
这里的MongoDB服务是从服务容器中取得的。默认，Phalcon会使mongo作服务名：

Connections are retrieved from the services container. By default, Phalcon tries to find the connection in a service called "mongo":

.. code-block:: php

    <?php

    // Simple database connection to localhost
    $di->set('mongo', function() {
        $mongo = new MongoClient();
        return $mongo->selectDB("store");
    }, true);

    // Connecting to a domain socket, falling back to localhost connection
    $di->set('mongo', function() {
        $mongo = new MongoClient("mongodb:///tmp/mongodb-27017.sock,localhost:27017");
        return $mongo->selectDB("store");
    }, true);

查找文档Finding Documents
------------------------------
 :doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>`依赖mongo的PHP扩展，我们有同样的功能区查询文档并在模型中进行转换：

As :doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>` relies on the Mongo PHP extension you have the same facilities
to query documents and convert them transparently to model instances:

.. code-block:: php

    <?php

    // How many robots are there?
    $robots = Robots::find();
    echo "There are ", count($robots), "\n";

    // How many mechanical robots are there?
    $robots = Robots::find(array(
        array("type" => "mechanical")
    ));
    echo "There are ", count($robots), "\n";

    // Get and print mechanical robots ordered by name upward
    $robots = Robots::find(array(
        array("type" => "mechanical"),
        "sort" => array("name" => 1)
    ));

    foreach ($robots as $robot) {
        echo $robot->name, "\n";
    }

    // Get first 100 mechanical robots ordered by name
    $robots = Robots::find(array(
        array("type" => "mechanical"),
        "sort"  => array("name" => 1),
        "limit" => 100
    ));

    foreach ($robots as $robot) {
       echo $robot->name, "\n";
    }

这里我们可以使用findFirst()来取得配置查询的第一条记录：	
	
You could also use the findFirst() method to get only the first record matching the given criteria:

.. code-block:: php

    <?php

    // What's the first robot in robots collection?
    $robot = Robots::findFirst();
    echo "The robot name is ", $robot->name, "\n";

    // What's the first mechanical robot in robots collection?
    $robot = Robots::findFirst(array(
        array("type" => "mechanical")
    ));
    echo "The first mechanical robot name is ", $robot->name, "\n";

find()和findFirst方法都接收一个关联数据组为查询的条件：	
	
Both find() and findFirst() methods accept an associative array specifying the search criteria:

.. code-block:: php

    <?php

    // First robot where type = "mechanical" and year = "1999"
    $robot = Robots::findFirst(array(
        "conditions" => array(
            "type" => "mechanical",
            "year" => "1999"
        )
    ));

    // All virtual robots ordered by name downward
    $robots = Robots::find(array(
        "conditions" => array("type" => "virtual"),
        "sort"       => array("name" => -1)
    ));

可用的查询选项：

The available query options are:

+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| Parameter   | Description                                                                                                                                                                                  | Example                                                                 |
+=============+==============================================================================================================================================================================================+=========================================================================+
| conditions  | Search conditions for the find operation. Is used to extract only those records that fulfill a specified criterion. By default Phalcon_model assumes the first parameter are the conditions. | "conditions" => array('$gt' => 1990)                                    |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| fields      | Returns specific columns instead of the full fields in the collection. When using this option an incomplete object is returned                                                               | "fields" => array('name' => true)                                       |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| sort        | It's used to sort the resultset. Use one or more fields as each element in the array, 1 means ordering upwards, -1 downward                                                                  | "sort" => array("name" => -1, "status" => 1)                            |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| limit       | Limit the results of the query to results to certain range                                                                                                                                   | "limit" => 10                                                           |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
| skip        | Skips a number of results                                                                                                                                                                    | "skip" => 50                                                            |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------+

如果你有使用sql(关系)数据库的经验，你也许想查看二者的映射表格 `SQL to Mongo Mapping Chart`_ . 

If you have experience with SQL databases, you may want to check the `SQL to Mongo Mapping Chart`_.

聚合Aggregations
-----------------------
我们可以使用Mongo提供的`aggregation framework`_方法使用Mongo模型返回聚合结果。聚合结果不是使用MapReduce来计算的。基于此，我们可以非常容易的取得聚合值，比如总计或平均值等:

A model can return calculations using `aggregation framework`_ provided by Mongo. The aggregated values are calculate without having to use MapReduce.
With this option is easy perform tasks such as totaling or averaging field values:

.. code-block:: php

    <?php

    $data = Article::aggregate(array(
        array(
            '$project' => array('category' => 1)
        ),
        array(
            '$group' => array(
                '_id' => array('category' => '$category'),
                'id'  => array('$max' => '$_id')
            )
        )
    ));

创建和更新记录Creating Updating/Records
--------------------------------------------
Phalcon\\Mvc\\Collection::save()方法可以用来保存数据，Phalcon会根据当前数据库中的数据来对比以确定是新加一条数据还是更新数据。在Phalcon内部会直接使用 :doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>` 的save或update方法来进行操作。 当然这个方法内部也会调用我们在模型中定义的验证方法或事件等：

The method Phalcon\\Mvc\\Collection::save() allows you to create/update documents according to whether they already exist in the collection
associated with a model. The 'save' method is called internally by the create and update methods of :doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>`.

Also the method executes associated validators and events that are defined in the model:

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

“_id”属性会被Mongo驱动自动的随 MongId_ 而更新。	
	
The "_id" property is automatically updated with the MongoId_ object created by the driver:

.. code-block:: php

    <?php

    $robot->save();
    echo "The generated id is: ", $robot->getId();

验证信息Validation Messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
:doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>` 提供了一个信息子系统，使用此系统开发者可以非常容易的实现在数据处理中的验证信息的显示及保存。 每条信息即是一个:doc:`Phalcon\\Mvc\\Model\\Message <../api/Phalcon_Mvc_Model_Message>`类的对象实例。我们使用getMessages来取得此信息。每条信息中包含了如哪个字段产生的消息，或是消息类型等信息：

:doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>` has a messaging subsystem that provides a flexible way to output or store the
validation messages generated during the insert/update processes.

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

验证事件和事件管理 Validation Events and Events Manager
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在模型类的数据操作过程中可以产生一些事件。我们可以在这些事件中定义一些业务规则。下面是  :doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>` 所支持的事件及其执行顺序：

Models allow you to implement events that will be thrown when performing an insert or update. They help define business rules for a
certain model. The following are the events supported by :doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>` and their order of execution:

+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Operation          | Name                     | Can stop operation?   | Explanation                                                                                                         |
+====================+==========================+=======================+=====================================================================================================================+
| Inserting/Updating | beforeValidation         | YES                   | Is executed before the validation process and the final insert/update to the database                               |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Inserting          | beforeValidationOnCreate | YES                   | Is executed before the validation process only when an insertion operation is being made                            |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Updating           | beforeValidationOnUpdate | YES                   | Is executed before the fields are validated for not nulls or foreign keys when an updating operation is being made  |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Inserting/Updating | onValidationFails        | YES (already stopped) | Is executed before the validation process only when an insertion operation is being made                            |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Inserting          | afterValidationOnCreate  | YES                   | Is executed after the validation process when an insertion operation is being made                                  |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Updating           | afterValidationOnUpdate  | YES                   | Is executed after the validation process when an updating operation is being made                                   |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Inserting/Updating | afterValidation          | YES                   | Is executed after the validation process                                                                            |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Inserting/Updating | beforeSave               | YES                   | Runs before the required operation over the database system                                                         |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Updating           | beforeUpdate             | YES                   | Runs before the required operation over the database system only when an updating operation is being made           |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Inserting          | beforeCreate             | YES                   | Runs before the required operation over the database system only when an inserting operation is being made          |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Updating           | afterUpdate              | NO                    | Runs after the required operation over the database system only when an updating operation is being made            |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Inserting          | afterCreate              | NO                    | Runs after the required operation over the database system only when an inserting operation is being made           |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+
| Inserting/Updating | afterSave                | NO                    | Runs after the required operation over the database system                                                          |
+--------------------+--------------------------+-----------------------+---------------------------------------------------------------------------------------------------------------------+

为了响应一个事件，我们需在模型中实现同名方法：

To make a model to react to an event, we must to implement a method with the same name of the event:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Collection;

    class Robots extends Collection
    {

        public function beforeValidationOnCreate()
        {
            echo "This is executed before creating a Robot!";
        }

    }

在执行操作之前先在指定的事件中设置值有时是非常有用的：	
	
Events can be useful to assign values before performing an operation, for example:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Collection;

    class Products extends Collection
    {

        public function beforeCreate()
        {
            // Set the creation date
            $this->created_at = date('Y-m-d H:i:s');
        }

        public function beforeUpdate()
        {
            // Set the modification date
            $this->modified_in = date('Y-m-d H:i:s');
        }

    }

另外，这个组件也可以和 :doc:`Phalcon\\Events\\Manager <events>` 进行集成，这就意味着我们在事件触发创建监听器	
	
Additionally, this component is integrated with :doc:`Phalcon\\Events\\Manager <events>`, this means we can create
listeners that run when an event is triggered.

.. code-block:: php

    <?php

    use Phalcon\Events\Manager as EventsManager;

    $eventsManager = new EventsManager();

    //Attach an anonymous function as a listener for "model" events
    $eventsManager->attach('collection', function($event, $robot) {
        if ($event->getType() == 'beforeSave') {
            if ($robot->name == 'Scooby Doo') {
                echo "Scooby Doo isn't a robot!";
                return false;
            }
        }
        return true;
    });

    $robot       = new Robots();
    $robot->setEventsManager($eventsManager);
    $robot->name = 'Scooby Doo';
    $robot->year = 1969;
    $robot->save();


上面的例子中EventsManager仅在对象和监听器（匿名函数）之间扮演了一个桥接器的角色。如果我们想在创建应用时使用同一个EventsManager,我们需要把这个EventsManager对象设置到
collectionManager服务中：	
	
In the example given above the EventsManager only acted as a bridge between an object and a listener (the anonymous function). If we want all
objects created in our application use the same EventsManager, then we need to assign this to the Models Manager:

.. code-block:: php

    <?php

    use Phalcon\Events\Manager as EventsManager;
    use Phalcon\Mvc\Collection\Manager as CollectionManager;

    //Registering the collectionManager service
    $di->set('collectionManager', function() {

        $eventsManager = new EventsManager();

        // Attach an anonymous function as a listener for "model" events
        $eventsManager->attach('collection', function($event, $model) {
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

        // Setting a default EventsManager
        $modelsManager = new CollectionManager();
        $modelsManager->setEventsManager($eventsManager);
        return $modelsManager;

    }, true);

实现业务规则Implementing a Business Rule
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
当插入或更新删除等执行时，模型会检查上面表格中列出的方法是否存在。我们建议定义模型里的验证方法以避免业务逻辑暴露出来。下面的例子中实现了在保存或更新时对年份的验证，年份不能小于0年：

When an insert, update or delete is executed, the model verifies if there are any methods with the names of the events listed in the table above.

We recommend that validation methods are declared protected to prevent that business logic implementation from being exposed publicly.

The following example implements an event that validates the year cannot be smaller than 0 on update or insert:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Collection;

    class Robots extends Collection
    {

        public function beforeSave()
        {
            if ($this->year < 0) {
                echo "Year cannot be smaller than zero!";
                return false;
            }
        }

    }

在响应某些事件时返回了false则会停止当前的操作。 如果事实响应未返回任何值， :doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>`会假定返回了true值。	
	
Some events return false as an indication to stop the current operation. If an event doesn't return anything,
:doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>` will assume a true value.

验证数据完整性Validating Data Integrity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
:doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>` 提供了若干个事件用于验证数据和实现业务逻辑。特定的事件中我们可以调用内建的验证器 Phalcon提供了一些验证器可以用在此阶段的验证上。

:doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>` provides several events to validate data and implement business rules. The special "validation"
event allows us to call built-in validators over the record. Phalcon exposes a few built-in validators that can be used at this stage of validation.

下面的例子中展示了如何使用：

The following example shows how to use it:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Collection;
    use Phalcon\Mvc\Model\Validator\InclusionIn;
    use Phalcon\Mvc\Model\Validator\Numericality;

    class Robots extends Collection
    {

        public function validation()
        {

            $this->validate(new InclusionIn(
                array(
                    "field"   => "type",
                    "message" => "Type must be: mechanical or virtual",
                    "domain"  => array("Mechanical", "Virtual")
                )
            ));

            $this->validate(new Numericality(
                array(
                    "field"   => "price",
                    "message" => "Price must be numeric"
                )
            ));

            return $this->validationHasFailed() != true;
        }

    }

上面的例子使用了内建的”InclusionIn”验证器。这个验证器检查了字段的类型是否在指定的范围内。如果值不在范围内即验证失败会返回false. 下面支持的内验证器：

The example given above performs a validation using the built-in validator "InclusionIn". It checks the value of the field "type" in a domain list. If
the value is not included in the method, then the validator will fail and return false. The following built-in validators are available:

+--------------+----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Name         | Explanation                                                                                                                            | Example                                                           |
+==============+========================================================================================================================================+===================================================================+
| Email        | Validates that field contains a valid email format                                                                                     | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_Email>`         |
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| ExclusionIn  | Validates that a value is not within a list of possible values                                                                         | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_Exclusionin>`   |
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| InclusionIn  | Validates that a value is within a list of possible values                                                                             | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_Inclusionin>`   |
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Numericality | Validates that a field has a numeric format                                                                                            | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_Numericality>`  |
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Regex        | Validates that the value of a field matches a regular expression                                                                       | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_Regex>`         |
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| StringLength | Validates the length of a string                                                                                                       | :doc:`Example <../api/Phalcon_Mvc_Model_Validator_StringLength>`  |
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+


除了内建的验证器外，我们还可以创建自己的验证器：	

In addition to the built-in validators, you can create your own validators:

.. code-block:: php

    <?php

    use \Phalcon\Mvc\Model\Validator as CollectionValidator;

    class UrlValidator extends CollectionValidator
    {

        public function validate($model)
        {
            $field = $this->getOption('field');

            $value    = $model->$field;
            $filtered = filter_var($value, FILTER_VALIDATE_URL);
            if (!$filtered) {
                $this->appendMessage("The URL is invalid", $field, "UrlValidator");
                return false;
            }
            return true;
        }

    }

添加验证器到模型：	
	
Adding the validator to a model:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Collection;

    class Customers extends Collection
    {

        public function validation()
        {
            $this->validate(new UrlValidator(array(
                "field"  => "url",
            )));
            if ($this->validationHasFailed() == true) {
                return false;
            }
        }

    }

创建验证器的目的即是使之在多个模型间重复利用以实现代码重用。验证器可简单如下：	
	
The idea of creating validators is make them reusable across several models. A validator can also be as simple as:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Collection;
    use Phalcon\Mvc\Model\Message as ModelMessage;

    class Robots extends Collection
    {

        public function validation()
        {
            if ($this->type == "Old") {
                $message = new ModelMessage(
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

删除记录Deleting Records
-------------------------------
Phalcon\\Mvc\\Collection::delete()方法用来删除记录条目。我们可以如下使用：

The method Phalcon\\Mvc\\Collection::delete() allows to delete a document. You can use it as follows:

.. code-block:: php

    <?php

    $robot = Robots::findFirst();
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

也可以使用遍历的方式删除多个条目的数据：	
	
You can also delete many documents by traversing a resultset with a foreach:

.. code-block:: php

    <?php

    $robots = Robots::find(array(
        array("type" => "mechanical")
    ));
    foreach ($robots as $robot) {
        if ($robot->delete() == false) {
            echo "Sorry, we can't delete the robot right now: \n";
            foreach ($robot->getMessages() as $message) {
                echo $message, "\n";
            }
        } else {
            echo "The robot was deleted successfully!";
        }
    }

当删除操作执行时我们可以执行如下事件，以实现定制业务逻辑的目的：	
	
The following events are available to define custom business rules that can be executed when a delete operation is performed:

+-----------+--------------+---------------------+------------------------------------------+
| Operation | Name         | Can stop operation? | Explanation                              |
+===========+==============+=====================+==========================================+
| Deleting  | beforeDelete | YES                 | Runs before the delete operation is made |
+-----------+--------------+---------------------+------------------------------------------+
| Deleting  | afterDelete  | NO                  | Runs after the delete operation was made |
+-----------+--------------+---------------------+------------------------------------------+

验证失败事件Validation Failed Events
----------------------------------------
验证失败时依据不同的情形下列事件会触发：


Another type of events is available when the data validation process finds any inconsistency:

+--------------------------+--------------------+--------------------------------------------------------------------+
| Operation                | Name               | Explanation                                                        |
+==========================+====================+====================================================================+
| Insert or Update         | notSave            | Triggered when the insert/update operation fails for any reason    |
+--------------------------+--------------------+--------------------------------------------------------------------+
| Insert, Delete or Update | onValidationFails  | Triggered when any data manipulation operation fails               |
+--------------------------+--------------------+--------------------------------------------------------------------+

固有 Id 和 用户主键Implicit Ids vs. User Primary Keys
----------------------------------------------------------------
默认Phalcon\Mvc\Collection会使用MongoIds_来产生_id.如果用户想自定义主键也可以只需：

By default Phalcon\\Mvc\\Collection assumes that the _id attribute is automatically generated using MongoIds_.
If a model uses custom primary keys this behavior can be overridden:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Collection;

    class Robots extends Collection
    {
        public function initialize()
        {
            $this->useImplicitObjectIds(false);
        }
    }

设置多个数据库Setting multiple databases
------------------------------------------------
Phalcon中，所有的模可以只属于一个数据库或是每个模型有一个数据。事实上当 :doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>` 试图连接数据库时 Phalcon会从DI中取名为mongo的服务。当然我们可在模型的initialize方法中进行连接设置：

In Phalcon, all models can belong to the same database connection or have an individual one. Actually, when
:doc:`Phalcon\\Mvc\\Collection <../api/Phalcon_Mvc_Collection>` needs to connect to the database it requests the "mongo" service
in the application's services container. You can overwrite this service setting it in the initialize method:

.. code-block:: php

    <?php

    // This service returns a mongo database at 192.168.1.100
    $di->set('mongo1', function() {
        $mongo = new MongoClient("mongodb://scott:nekhen@192.168.1.100");
        return $mongo->selectDB("management");
    }, true);

    // This service returns a mongo database at localhost
    $di->set('mongo2', function() {
        $mongo = new MongoClient("mongodb://localhost");
        return $mongo->selectDB("invoicing");
    }, true);

然后在初始化方法，我们定义了模型的连接：	
	
Then, in the Initialize method, we define the connection service for the model:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Collection;

    class Robots extends Collection
    {
        public function initialize()
        {
            $this->setConnectionService('mongo1');
        }

    }

注入服务到模型Injecting services into Models
------------------------------------------------
我们可能需要在模型内使用应用的服务，下面的例子中展示了如何去做：

You may be required to access the application services within a model, the following example explains how to do that:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Collection;

    class Robots extends Collection
    {

        public function notSave()
        {
            // Obtain the flash service from the DI container
            $flash = $this->getDI()->getShared('flash');

            // Show validation messages
            foreach ($this->getMessages() as $message){
                $flash->error((string) $message);
            }
        }

    }

notSave事件在创建和更新失败时触发。我们使用flash服务来处理验证信息。如此做我们无需在每次保存后打印消息出来。	
	
The "notSave" event is triggered whenever a "creating" or "updating" action fails. We're flashing the validation messages
obtaining the "flash" service from the DI container. By doing this, we don't have to print messages after each saving.

.. _MongoDB: http://www.mongodb.org/
.. _MongoId: http://www.php.net/manual/en/class.mongoid.php
.. _MongoIds: http://www.php.net/manual/en/class.mongoid.php
.. _`SQL to Mongo Mapping Chart`: http://www.php.net/manual/en/mongo.sqltomongo.php
.. _`aggregation framework`: http://docs.mongodb.org/manual/applications/aggregation/
