日志记录Logging
=====================
Phalcon提供了一个日志记录组件即 :doc:`Phalcon\\Logger <../api/Phalcon_Logger>`。 我们可以使用此组件输出日志到不同的流中，如文件，系统日志等。 这个组件还提供了其它的功能如日志事务（类似于数据库的事务）， 配置选项， 还可以输出不同的格式，另外还支持多种过滤器。 :doc:`Phalcon\\Logger <../api/Phalcon_Logger>`提供了多种日志记录方式，从调试程序到跟踪应用的执行以满足应用的需求。

:doc:`Phalcon\\Logger <../api/Phalcon_Logger>` is a component whose purpose is to provide logging services for applications. It offers logging to different backends using different adapters. It also offers transaction logging, configuration options, different formats and filters. You can use the :doc:`Phalcon\\Logger <../api/Phalcon_Logger>` for every logging need your application has, from debugging processes to tracing application flow.

适配器Adapters
----------------------
此组件使用不同的流适配器来保存日信息。 我们可以按需使用适配器。支持的适配器如下：

This component makes use of adapters to store the logged messages. The use of adapters allows for a common interface for logging
while switching backends if necessary. The adapters supported are:

+---------+---------------------------+----------------------------------------------------------------------------------+
| Adapter | Description               | API                                                                              |
+=========+===========================+==================================================================================+
| File    | Logs to a plain text file | :doc:`Phalcon\\Logger\\Adapter\\File <../api/Phalcon_Logger_Adapter_File>`       |
+---------+---------------------------+----------------------------------------------------------------------------------+
| Stream  | Logs to a PHP Streams     | :doc:`Phalcon\\Logger\\Adapter\\Stream <../api/Phalcon_Logger_Adapter_Stream>`   |
+---------+---------------------------+----------------------------------------------------------------------------------+
| Syslog  | Logs to the system logger | :doc:`Phalcon\\Logger\\Adapter\\Syslog <../api/Phalcon_Logger_Adapter_Syslog>`   |
+---------+---------------------------+----------------------------------------------------------------------------------+
| Firephp | Logs to the FirePHP       | :doc:`Phalcon\\Logger\\Adapter\\FirePHP <../api/Phalcon_Logger_Adapter_Firephp>` |
+---------+---------------------------+----------------------------------------------------------------------------------+

创建日志Creating a Log
--------------------------
下面的例子展示了如何创建日志对象及如何添加日志信息：

The example below shows how to create a log and add messages to it:

.. code-block:: php

    <?php

    use Phalcon\Logger\Adapter\File as FileAdapter;

    $logger = new FileAdapter("app/logs/test.log");
    $logger->log("This is a message");
    $logger->log("This is an error", \Phalcon\Logger::ERROR);
    $logger->error("This is another error");

产生的日志信息如下：	
	
The log generated is below:

.. code-block:: php

    [Tue, 17 Apr 12 22:09:02 -0500][DEBUG] This is a message
    [Tue, 17 Apr 12 22:09:02 -0500][ERROR] This is an error
    [Tue, 17 Apr 12 22:09:02 -0500][ERROR] This is another error

事务Transactions
---------------------
保存日志到适配器如文件(文件系统)是非常消耗系统资源的。 为了减少应用性能上的开销，我们可以使用日志事务。 事务会把日志记录临时的保存到内存中然后再 写入到适配中（此例子中为文件），（这个操作是个原子操作）

Logging data to an adapter i.e. File (file system) is always an expensive operation in terms of performance. To combat that, you
can take advantage of logging transactions. Transactions store log data temporarily in memory and later on write the data to the
relevant adapter (File in this case) in a single atomic operation.

.. code-block:: php

    <?php

    use Phalcon\Logger\Adapter\File as FileAdapter;

    // Create the logger
    $logger = new FileAdapter("app/logs/test.log");

    // Start a transaction
    $logger->begin();

    // Add messages
    $logger->alert("This is an alert");
    $logger->error("This is another error");

    // Commit messages to file
    $logger->commit();

使用多个处理程序进行日志记录Logging to Multiple Handlers
-----------------------------------------------------------
:doc:`Phalcon\\Logger <../api/Phalcon_Logger>`也可以同时保存日志信息到多个适配器中：

:doc:`Phalcon\\Logger <../api/Phalcon_Logger>` can send messages to multiple handlers with a just single call:

.. code-block:: php

    <?php

    use Phalcon\Logger,
        Phalcon\Logger\Multiple as MultipleStream,
        Phalcon\Logger\Adapter\File as FileAdapter,
        Phalcon\Logger\Adapter\Stream as StreamAdapter;

    $logger = new MultipleStream();

    $logger->push(new FileAdapter('test.log'));
    $logger->push(new StreamAdapter('php://stdout'));

    $logger->log("This is a message");
    $logger->log("This is an error", Logger::ERROR);
    $logger->error("This is another error");

信息发送的顺序和处理器（适配器）注册的顺序相同。	
	
The messages are sent to the handlers in the order they were registered.

信息格式Message Formatting
--------------------------------
此组件使用 formatters 在信息发送前格式化日志信息。 支持下而后格式：

This component makes use of 'formatters' to format messages before sending them to the backend. The formatters available are:

+---------+-----------------------------------------------+------------------------------------------------------------------------------------+
| Adapter | Description                                   | API                                                                                |
+=========+===============================================+====================================================================================+
| Line    | Formats the messages using a one-line string  | :doc:`Phalcon\\Logger\\Formatter\\Line <../api/Phalcon_Logger_Formatter_Line>`     |
+---------+-----------------------------------------------+------------------------------------------------------------------------------------+
| Json    | Prepares a message to be encoded with JSON    | :doc:`Phalcon\\Logger\\Formatter\\Json <../api/Phalcon_Logger_Formatter_Json>`     |
+---------+-----------------------------------------------+------------------------------------------------------------------------------------+
| Syslog  | Prepares a message to be sent to syslog       | :doc:`Phalcon\\Logger\\Formatter\\Syslog <../api/Phalcon_Logger_Formatter_Syslog>` |
+---------+-----------------------------------------------+------------------------------------------------------------------------------------+

行格式化处理Line Formatter
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
使用单行格式格式化信息。 默认的格式如下：

Formats the messages using a one-line string. The default logging format is:

[%date%][%type%] %message%

我们可以使用setFormat()来设置自定义格式。 下面是格式变量：

You can change the default format using setFormat(), this allows you to change the format of the logged
messages by defining your own. The log format variables allowed are:

+-----------+------------------------------------------+
| Variable  | Description                              |
+===========+==========================================+
| %message% | The message itself expected to be logged |
+-----------+------------------------------------------+
| %date%    | Date the message was added               |
+-----------+------------------------------------------+
| %type%    | Uppercase string with message type       |
+-----------+------------------------------------------+

下面的例子中展示了如何修改日志格式：

The example below shows how to change the log format:

.. code-block:: php

    <?php

    use Phalcon\Logger\Formatter\Line as LineFormatter;

    //Changing the logger format
    $formatter = new LineFormatter("%date% - %message%");
    $logger->setFormatter($formatter);

自定义格式处理Implementing your own formatters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
若要实现自定义的格式则要实现 :doc:`Phalcon\\Logger\\FormatterInterface <../api/Phalcon_Logger_FormatterInterface>`接口， 这样才能扩展已有的格式或创建自定义的格式

The :doc:`Phalcon\\Logger\\FormatterInterface <../api/Phalcon_Logger_FormatterInterface>` interface must be implemented in order to
create your own logger formatter or extend the existing ones.

适配器Adapters
--------------------
下面的例子中展示了每种适配器的简单用法：

The following examples show the basic use of each adapter:

数据流日志记录器Stream Logger
^^^^^^^^^^^^^
系统日志保存消息到一个已注册的有效的PHP流中。 这里列出了可用的流： `here <http://php.net/manual/en/wrappers.php>`_:

The stream logger writes messages to a valid registered stream in PHP. A list of streams is available `here <http://php.net/manual/en/wrappers.php>`_:

.. code-block:: php

    <?php

    use Phalcon\Logger\Adapter\Stream as StreamAdapter;

    // Opens a stream using zlib compression
    $logger = new StreamAdapter("compress.zlib://week.log.gz");

    // Writes the logs to stderr
    $logger = new StreamAdapter("php://stderr");

文件日志记录器File Logger
^^^^^^^^^^^^^^^^^^^^^^^^^^^
文件适配器保存所有的日志信息到普通的文件中。 默认情况下日志文件使用添加模式打开，打开文件后文件的指针会指向文件的尾端。 如果文件不存在，则会尝试创建。 我们可以通过传递附加参数的形式来修改打开的模式：

This logger uses plain files to log any kind of data. By default all logger files are opened using
append mode which opens the files for writing only; placing the file pointer at the end of the file.
If the file does not exist, an attempt will be made to create it. You can change this mode by passing additional options to the constructor:

.. code-block:: php

    <?php

    use Phalcon\Logger\Adapter\File as FileAdapter;

    // Create the file logger in 'w' mode
    $logger = new FileAdapter("app/logs/test.log", array(
        'mode' => 'w'
    ));

Syslog 日志记录器Syslog Logger
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
使用系统日志适配器。 由于操作系统的不同得到的日志也不尽相同：

This logger sends messages to the system logger. The syslog behavior may vary from one operating system to another.

.. code-block:: php

    <?php
    use Phalcon\Logger\Adapter\Syslog as SyslogAdapter;

    // Basic Usage
    $logger = new SyslogAdapter(null);

    // Setting ident/mode/facility
    $logger = new SyslogAdapter("ident-name", array(
        'option' => LOG_NDELAY,
        'facility' => LOG_MAIL
    ));    
    
    
FirePHP 日志记录器FirePHP Logger
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
发送消息到FirePHP:

This logger sends messages in HTTP response headers that are displayed by `FirePHP <http://www.firephp.org/>`_,
a `Firebug <http://getfirebug.com/>`_ extension for Firefox.

.. code-block:: php

    <?php

    use Phalcon\Logger\Adapter\Firephp as Firephp;

    $logger = new Firephp("");
 	$logger->log("This is a message");
 	$logger->log("This is an error", \Phalcon\Logger::ERROR);
 	$logger->error("This is another error");

自定义适配器Implementing your own adapters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果开发者想自定义新的日志组件则需实现此接口： :doc:`Phalcon\\Logger\\AdapterInterface <../api/Phalcon_Logger_AdapterInterface>`。

The :doc:`Phalcon\\Logger\\AdapterInterface <../api/Phalcon_Logger_AdapterInterface>` interface must be implemented in order to
create your own logger adapters or extend the existing ones.
