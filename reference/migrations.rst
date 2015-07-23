数据库迁移Database Migrations
====================================
迁移是一个方便的结构化的和有组织的方式来改变数据库。

Migrations are a convenient way for you to alter your database in a structured and organized manner.

.. highlights::
    **重要：** 迁移在:doc:`Phalcon Developer Tools <tools>` 中可用。需要使用最新的框架,php框架要求PHP 5.4及以上。

    **Important:** Migrations are available on :doc:`Phalcon Developer Tools <tools>` You need at least Phalcon Framework version 0.5.0 to use developer tools. Also is recommended to have PHP 5.4 or greater installed.

通常在开发中我们需要更新生产环境。这些变化可能是数据库修改包括字段,新建表,删除索引,等。	
	
Often in development we need to update changes in production environments. Some of these changes could be database modifications like new fields, new tables, removing indexes, etc.

迁移时生成的一组类来描述如何创建数据库结构。这些类可用于将我们数据库的变化同步到远程生产服务器上。使用纯PHP描述这些迁移转换。

When a migration is generated a set of classes are created to describe how your database is structured at that moment. These classes can be used to synchronize the schema structure on remote databases setting your database ready to work with the new changes that your application implements. Migrations describe these transformations using plain PHP.

.. raw:: html

    <div align="center">
        <iframe src="http://player.vimeo.com/video/41381817" width="500" height="281" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe>
    </div>

数据定义导出Schema Dumping
------------------------------------
 :doc:`Phalcon Developer Tools <tools>`提供了脚本去管理迁移(生成, 运行和回滚).

The :doc:`Phalcon Developer Tools <tools>` provides scripts to manage migrations (generation, running and rollback).

可用生成迁移的选项:

The available options for generating migrations are:

.. figure:: ../_static/img/migrations-1.png
   :align: center

如果不使用参数运行。将会导出所有对象（表和视图）到迁移类文件。   
   
Running this script without any parameters will simply dump every object (tables and views) from your database in migration classes.

每个迁移都有一个版本符号。版本数字用于鉴定迁移比当前数据库的版本是旧还是新。版本号还会告诉Phalcon执行迁移的顺序。

Each migration has a version identifier associated to it. The version number allows us to identify if the migration is newer or older than the current 'version' of our database. Versions also inform Phalcon of the running order when executing a migration.

.. figure:: ../_static/img/migrations-2.png
   :align: center

生成的迁移时,指令显示在控制台来描述不同的迁移步骤,这些语句的执行时间。最后生成迁移版本。   
   
When a migration is generated, instructions are displayed on the console to describe the different steps of the migration and the execution time of those statements. At the end, a migration version is generated.

默认:doc:`Phalcon Developer Tools <tools>`使用*app/migrations*作为迁移文件导出目录。可以通过设置参数改变导出的路径。数据库中的每个表都会导出一个单独的文件在版本号目录中。

By default :doc:`Phalcon Developer Tools <tools>` use the *app/migrations* directory to dump the migration files. You can change the location by setting one of the parameters on the generation script. Each table in the database has its respective class generated in a separated file under a directory referring its version:

.. figure:: ../_static/img/migrations-3.png
   :align: center

迁移类剖析Migration Class Anatomy
---------------------------------------
每个文件包含一个唯一的类继承自Phalcon\\Mvc\\Model\\Migration。类有两个方法up() 和 down()。Up() 执行迁移, down() 提供回滚。

Each file contains a unique class that extends the Phalcon\\Mvc\\Model\\Migration These classes normally have two methods: up() and down(). Up() performs the migration, while down() rolls it back.

Up()包含了一个魔术方法morphTable()。 神奇的是它根据描述去同步改变实际数据库的表。

Up() also contains the *magic* method morphTable(). The magic comes when it recognizes the changes needed to synchronize the actual table in the database to the description given.

.. code-block:: php

    <?php

    use Phalcon\Db\Column as Column;
    use Phalcon\Db\Index as Index;
    use Phalcon\Db\Reference as Reference;

    class ProductsMigration_100 extends \Phalcon\Mvc\Model\Migration
    {

        public function up()
        {
            $this->morphTable(
                "products",
                array(
                    "columns" => array(
                        new Column(
                            "id",
                            array(
                                "type"          => Column::TYPE_INTEGER,
                                "size"          => 10,
                                "unsigned"      => true,
                                "notNull"       => true,
                                "autoIncrement" => true,
                                "first"         => true,
                            )
                        ),
                        new Column(
                            "product_types_id",
                            array(
                                "type"     => Column::TYPE_INTEGER,
                                "size"     => 10,
                                "unsigned" => true,
                                "notNull"  => true,
                                "after"    => "id",
                            )
                        ),
                        new Column(
                            "name",
                            array(
                                "type"    => Column::TYPE_VARCHAR,
                                "size"    => 70,
                                "notNull" => true,
                                "after"   => "product_types_id",
                            )
                        ),
                        new Column(
                            "price",
                            array(
                                "type"    => Column::TYPE_DECIMAL,
                                "size"    => 16,
                                "scale"   => 2,
                                "notNull" => true,
                                "after"   => "name",
                            )
                        ),
                    ),
                    "indexes" => array(
                        new Index(
                            "PRIMARY",
                            array("id")
                        ),
                        new Index(
                            "product_types_id",
                            array("product_types_id")
                        )
                    ),
                    "references" => array(
                        new Reference(
                            "products_ibfk_1",
                            array(
                                "referencedSchema"  => "invo",
                                "referencedTable"   => "product_types",
                                "columns"           => array("product_types_id"),
                                "referencedColumns" => array("id"),
                            )
                        )
                    ),
                    "options" => array(
                        "TABLE_TYPE"      => "BASE TABLE",
                        "ENGINE"          => "InnoDB",
                        "TABLE_COLLATION" => "utf8_general_ci",
                    )
                )
            );
        }

    }

类名叫做"ProductsMigration_100"。100尾缀表示版本号为1.0.0。	morphTable() 接受包含四个板块的数组。
	
The class is called "ProductsMigration_100". Suffix 100 refers to the version 1.0.0. morphTable() receives an associative array with 4 possible sections:

+--------------+---------------------------------------------------------------------------------------------------------------------------------------------+----------+
| Index        | Description                                                                                                                                 | Optional |
+==============+=============================================================================================================================================+==========+
| "columns"    | An array with a set of table columns                                                                                                        | No       |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "indexes"    | An array with a set of table indexes.                                                                                                       | Yes      |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "references" | An array with a set of table references (foreign keys).                                                                                     | Yes      |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "options"    | An array with a set of table creation options. These options are often related to the database system in which the migration was generated. | Yes      |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------+----------+

定义列Defining Columns
^^^^^^^^^^^^^^^^^^^^^^^^^^^
:doc:`Phalcon\\Db\\Column <../api/Phalcon_Db_Column>`被用于定义表的列。它封装了列的一系列的功能。接受第一个参数作为列名。一个数组作为列的描述。下面是列描述的参数。

:doc:`Phalcon\\Db\\Column <../api/Phalcon_Db_Column>` is used to define table columns. It encapsulates a wide variety of column related features. Its constructor receives as first parameter the column name and an array describing the column. The following options are available when describing columns:

+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| Option          | Description                                                                                                                                | Optional |
+=================+============================================================================================================================================+==========+
| "type"          | Column type. Must be a :doc:`Phalcon_Db_Column <../api/Phalcon_Db_Column>` constant (see below)                                            | No       |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "size"          | Some type of columns like VARCHAR or INTEGER may have a specific size                                                                      | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "scale"         | DECIMAL or NUMBER columns may be have a scale to specify how much decimals it must store                                                   | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "unsigned"      | INTEGER columns may be signed or unsigned. This option does not apply to other types of columns                                            | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "notNull"       | Column can store null values?                                                                                                              | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "autoIncrement" | With this attribute column will filled automatically with an auto-increment integer. Only one column in the table can have this attribute. | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "first"         | Column must be placed at first position in the column order                                                                                | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+
| "after"         | Column must be placed after indicated column                                                                                               | Yes      |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------+----------+

数据库迁移支持以下数据列类型：

Database migrations support the following database column types:

* Phalcon\\Db\\Column::TYPE_INTEGER
* Phalcon\\Db\\Column::TYPE_DATE
* Phalcon\\Db\\Column::TYPE_VARCHAR
* Phalcon\\Db\\Column::TYPE_DECIMAL
* Phalcon\\Db\\Column::TYPE_DATETIME
* Phalcon\\Db\\Column::TYPE_CHAR
* Phalcon\\Db\\Column::TYPE_TEXT

定义索引Defining Indexes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
:doc:`Phalcon\\Db\\Index <../api/Phalcon_Db_Index>`定义数据库表的索引。索引只需要你为它定义一个名称和一个列表的列。注意,如果任何索引的名称为PRIMARY，Phalcon将创建一个表的主键索引。

:doc:`Phalcon\\Db\\Index <../api/Phalcon_Db_Index>` defines table indexes. An index only requires that you define a name for it and a list of its columns. Note that if any index has the name PRIMARY, Phalcon will create a primary key index in that table.

定义关系Defining References
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
:doc:`Phalcon\\Db\\Reference <../api/Phalcon_Db_Reference>`定义表引用(也称为外键)。以下选项可用于定义一个引用:

:doc:`Phalcon\\Db\\Reference <../api/Phalcon_Db_Reference>` defines table references (also called foreign keys). The following options can be used to define a reference:

+---------------------+-----------------------------------------------------------------------------------------------------+----------+
| Index               | Description                                                                                         | Optional |
+=====================+=====================================================================================================+==========+
| "referencedTable"   | It's auto-descriptive. It refers to the name of the referenced table.                               | No       |
+---------------------+-----------------------------------------------------------------------------------------------------+----------+
| "columns"           | An array with the name of the columns at the table that have the reference                          | No       |
+---------------------+-----------------------------------------------------------------------------------------------------+----------+
| "referencedColumns" | An array with the name of the columns at the referenced table                                       | No       |
+---------------------+-----------------------------------------------------------------------------------------------------+----------+
| "referencedTable"   | The referenced table maybe is on another schema or database. This option allows you to define that. | Yes      |
+---------------------+-----------------------------------------------------------------------------------------------------+----------+

创建迁移类Writing Migrations
-----------------------------------
迁移并不是只为了“变形”表。迁移只是一个常规PHP类所以你不限于这些函数。例如添加一列之后您可以编写代码来为现有的记录设置列的值。更多的细节和例子查看 :doc:`database component <db>`。

Migrations aren't only designed to "morph" table. A migration is just a regular PHP class so you're not limited to these functions. For example after adding a column you could write code to set the value of that column for existing records. For more details and examples of individual methods, check the :doc:`database component <db>`.

.. code-block:: php

    <?php

    class ProductsMigration_100 extends \Phalcon\Mvc\Model\Migration
    {

        public function up()
        {
            //...
            self::$_connection->insert(
                "products",
                array("Malabar spinach", 14.50),
                array("name", "price")
            );
        }

    }

执行迁移Running Migrations
--------------------------------
生成的迁移上传目标服务器,可以很容易地运行它们，如下面例子所示:

Once the generated migrations are uploaded on the target server, you can easily run them as shown in the following example:

.. figure:: ../_static/img/migrations-4.png
   :align: center

.. figure:: ../_static/img/migrations-5.png
   :align: center

取决于数据库是否过时来执行迁移,在同一个迁移过程Phalcon可能运行多个迁移版本。如果你指定一个目标版本,Phalcon将运行所需的迁移直到到达指定的版本。   
   
Depending on how outdated is the database with respect to migrations, Phalcon may run multiple migration versions in the same migration process. If you specify a target version, Phalcon will run the required migrations until it reaches the specified version.

