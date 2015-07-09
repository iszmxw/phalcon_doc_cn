事件管理器Events Manager
==========================
此组件的目的是为了通过创建“钩子”拦截框架中大部分的组件操作。 这些钩子允许开发者获得状态信息，操纵数据或者改变某个组件进程中的执行流向。

The purpose of this component is to intercept the execution of most of the components of the framework by creating “hooks point”. These hook
points allow the developer to obtain status information, manipulate data or change the flow of execution during the process of a component.

使用示例Usage Example
---------------------------
以下面示例中，我们使用EventsManager来侦听在 Phalcon\Db 管理下的MySQL连接中产生的事件。 首先，我们需要一个侦听者对象来完成这部分的工作。我们创建了一个类，这个类有我们需要侦听事件所对应的方法：

In the following example, we use the EventsManager to listen for events produced in a MySQL connection managed by :doc:`Phalcon\\Db <../api/Phalcon_Db>`.
First, we need a listener object to do this. We created a class whose methods are the events we want to listen:

.. code-block:: php

    <?php

    class MyDbListener
    {

        public function afterConnect()
        {

        }

        public function beforeQuery()
        {

        }

        public function afterQuery()
        {

        }

    }

这个新的类可能有点啰嗦，但我们需要这样做。 事件管理器在组件和我们的侦听类之间充当着接口角色，并提供了基于在我们侦听类中所定义方法的钩子：	
	
This new class can be as verbose as we need it to. The EventsManager will interface between the component and our listener class,
offering hook points based on the methods we defined in our listener class:

.. code-block:: php

    <?php

    use Phalcon\Events\Manager as EventsManager;
    use Phalcon\Db\Adapter\Pdo\Mysql as DbAdapter;

    $eventsManager = new EventsManager();

    //Create a database listener
    $dbListener    = new MyDbListener();

    //Listen all the database events
    $eventsManager->attach('db', $dbListener);

    $connection    = new DbAdapter(array(
        "host"     => "localhost",
        "username" => "root",
        "password" => "secret",
        "dbname"   => "invo"
    ));

    //Assign the eventsManager to the db adapter instance
    $connection->setEventsManager($eventsManager);

    //Send a SQL command to the database server
    $connection->query("SELECT * FROM products p WHERE p.status = 1");

为了纪录我们应用中全部执行的SQL语句，我们需要使用“afterQuery”事件。 第一个传递给事件侦听者的参数包含了关于正在运行事件的上下文信息，第二个则是连接本身。	
	
In order to log all the SQL statements executed by our application, we need to use the event “afterQuery”. The first parameter passed to
the event listener contains contextual information about the event that is running, the second is the connection itself.

.. code-block:: php

    <?php

    use Phalcon\Logger\Adapter\File as Logger;

    class MyDbListener
    {

        protected $_logger;

        public function __construct()
        {
            $this->_logger = new Logger("../apps/logs/db.log");
        }

        public function afterQuery($event, $connection)
        {
            $this->_logger->log($connection->getSQLStatement(), \Phalcon\Logger::INFO);
        }

    }

作为些示例的一部分，我们同样实现了 Phalcon\\Db\\Profiler 来检测SQL语句是否超出了期望的执行时间：	
	
As part of this example, we will also implement the Phalcon\\Db\\Profiler to detect the SQL statements that are taking longer to execute than expected:

.. code-block:: php

    <?php

    use Phalcon\Db\Profiler;
    use Phalcon\Logger;
    use Phalcon\Logger\Adapter\File;

    class MyDbListener
    {

        protected $_profiler;

        protected $_logger;

        /**
         * Creates the profiler and starts the logging
         */
        public function __construct()
        {
            $this->_profiler = new Profiler();
            $this->_logger   = new Logger("../apps/logs/db.log");
        }

        /**
         * This is executed if the event triggered is 'beforeQuery'
         */
        public function beforeQuery($event, $connection)
        {
            $this->_profiler->startProfile($connection->getSQLStatement());
        }

        /**
         * This is executed if the event triggered is 'afterQuery'
         */
        public function afterQuery($event, $connection)
        {
            $this->_logger->log($connection->getSQLStatement(), Logger::INFO);
            $this->_profiler->stopProfile();
        }

        public function getProfiler()
        {
            return $this->_profiler;
        }

    }

可以从侦听者中获取结果分析数据：	
	
The resulting profile data can be obtained from the listener:

.. code-block:: php

    <?php

    //Send a SQL command to the database server
    $connection->execute("SELECT * FROM products p WHERE p.status = 1");

    foreach ($dbListener->getProfiler()->getProfiles() as $profile) {
        echo "SQL Statement: ", $profile->getSQLStatement(), "\n";
        echo "Start Time: ", $profile->getInitialTime(), "\n";
        echo "Final Time: ", $profile->getFinalTime(), "\n";
        echo "Total Elapsed Time: ", $profile->getTotalElapsedSeconds(), "\n";
    }

类似地，我们可以注册一个匿名函数来执行这些任务，而不是再分离出一个侦听类（如上面看到的）：	
	
In a similar manner we can register an lambda function to perform the task instead of a separate listener class (as seen above):

.. code-block:: php

    <?php

    //Listen all the database events
    $eventManager->attach('db', function($event, $connection) {
        if ($event->getType() == 'afterQuery') {
            echo $connection->getSQLStatement();
        }
    });

创建组件触发事件Creating components that trigger Events
---------------------------------------------------------
你可以在你的应用中为事件管理器的触发事件创建组件。这样的结果是，可以有很多存在的侦听者为这些产生的事件作出响应。 在以下的示例中，我们将会创建一个叫做“MyComponent”组件。这是个意识事件管理器组件； 当它的方法“someTask”被执行时它将触发事件管理器中全部侦听者的两个事件：

You can create components in your application that trigger events to an EventsManager. As a consequence, there may exist listeners
that react to these events when generated. In the following example we're creating a component called "MyComponent".
This component is EventsManager aware; when its method "someTask" is executed it triggers two events to any listener in the EventsManager:

.. code-block:: php

    <?php

    use Phalcon\Events\EventsAwareInterface;

    class MyComponent implements EventsAwareInterface
    {

        protected $_eventsManager;

        public function setEventsManager($eventsManager)
        {
            $this->_eventsManager = $eventsManager;
        }

        public function getEventsManager()
        {
            return $this->_eventsManager;
        }

        public function someTask()
        {
            $this->_eventsManager->fire("my-component:beforeSomeTask", $this);

            // do some task

            $this->_eventsManager->fire("my-component:afterSomeTask", $this);
        }

    }

注意到这个组件产生的事件都以“my-component”为前缀。这是一个唯一的关键词，可以帮助我们区分各个组件产生的事件。 你甚至可以在组件的外面生成相同名字的事件。现在让我们来为这个组件创建一个侦听者：	
	
Note that events produced by this component are prefixed with "my-component". This is a unique word that helps us
identify events that are generated from certain component. You can even generate events outside the component with
the same name. Now let's create a listener to this component:

.. code-block:: php

    <?php

    class SomeListener
    {

        public function beforeSomeTask($event, $myComponent)
        {
            echo "Here, beforeSomeTask\n";
        }

        public function afterSomeTask($event, $myComponent)
        {
            echo "Here, afterSomeTask\n";
        }

    }

侦听者可以是简单的一个实现了全部组件触发事件的类。现在让我们把全部的东西整合起来：	
	
A listener is simply a class that implements any of all the events triggered by the component. Now let's make everything work together:

.. code-block:: php

    <?php

    use Phalcon\Events\Manager as EventsManager;

    //Create an Events Manager
    $eventsManager = new EventsManager();

    //Create the MyComponent instance
    $myComponent   = new MyComponent();

    //Bind the eventsManager to the instance
    $myComponent->setEventsManager($eventsManager);

    //Attach the listener to the EventsManager
    $eventsManager->attach('my-component', new SomeListener());

    //Execute methods in the component
    $myComponent->someTask();

当“someTask”被执行时，在侦听者里面的两个方法将会被执行，并产生以下输出：	
	
As "someTask" is executed, the two methods in the listener will be executed, producing the following output:

.. code-block:: php

    Here, beforeSomeTask
    Here, afterSomeTask

当触发一个事件时也可以使用“fire”中的第三个参数来传递额外的数据：	
	
Additional data may also passed when triggering an event using the third parameter of "fire":

.. code-block:: php

    <?php

    $eventsManager->fire("my-component:afterSomeTask", $this, $extraData);

在一个侦听者里，第三个参数可用于接收此参数：	
	
In a listener the third parameter also receives this data:

.. code-block:: php

    <?php

    //Receiving the data in the third parameter
    $eventManager->attach('my-component', function($event, $component, $data) {
        print_r($data);
    });

    //Receiving the data from the event context
    $eventManager->attach('my-component', function($event, $component) {
        print_r($event->getData());
    });

如果一个侦听者仅是对某个特定类型的事件感兴趣，你要吧直接附上一个侦听者：	
	
If a listener it is only interested in listening a specific type of event you can attach a listener directly:

.. code-block:: php

    <?php

    //The handler will only be executed if the event triggered is "beforeSomeTask"
    $eventManager->attach('my-component:beforeSomeTask', function($event, $component) {
        //...
    });

事件传播与取消Event Propagation/Cancellation
-----------------------------------------------
可能会有多个侦听者添加到同一个事件管理器，这意味着对于相同的事件会通知多个侦听者。 这些侦听者会以它们在事件管理器注册的顺序来通知。有些事件是可以被取消的，暗示着这些事件可以被终止以防其他侦听都再收到事件的通知：

Many listeners may be added to the same event manager, this means that for the same type of event many listeners can be notified.
The listeners are notified in the order they were registered in the EventsManager. Some events are cancelable, indicating that
these may be stopped preventing other listeners are notified about the event:

.. code-block:: php

    <?php

    $eventsManager->attach('db', function($event, $connection){

        //We stop the event if it is cancelable
        if ($event->isCancelable()) {
            //Stop the event, so other listeners will not be notified about this
            $event->stop();
        }

        //...

    });

默认情况下全部的事件都是可以取消的，甚至框架提供的事件也是可以取消的。 你可以通过在fire中的第四个参数中传递false来指明这是一个不可取消的事件：	
	
By default events are cancelable, even most of events produced by the framework are cancelables. You can fire a not-cancelable event
by passing "false" in the fourth parameter of fire:

.. code-block:: php

    <?php

    $eventsManager->fire("my-component:afterSomeTask", $this, $extraData, false);

侦听器优先级Listener Priorities
-----------------------------------
当附上侦听者时，你可以设置一个优先级。使用此特性，你可以指定这些侦听者被调用的固定顺序：

When attaching listeners you can set a specific priority. With this feature you can attach listeners indicating the order
in which they must be called:

.. code-block:: php

    <?php

    $evManager->enablePriorities(true);

    $evManager->attach('db', new DbListener(), 150); //More priority
    $evManager->attach('db', new DbListener(), 100); //Normal priority
    $evManager->attach('db', new DbListener(), 50); //Less priority

收集响应Collecting Responses
-------------------------------
事件管理器可以收集每一个被通知的侦听者返回的响应，以下这个示例解释了它是如何工作的：

The events manager can collect every response returned by every notified listener, this example explains how it works:

.. code-block:: php

    <?php

    use Phalcon\Events\Manager as EventsManager;

    $evManager = new EventsManager();

    //Set up the events manager to collect responses
    $evManager->collectResponses(true);

    //Attach a listener
    $evManager->attach('custom:custom', function() {
        return 'first response';
    });

    //Attach a listener
    $evManager->attach('custom:custom', function() {
        return 'second response';
    });

    //Fire the event
    $evManager->fire('custom:custom', null);

    //Get all the collected responses
    print_r($evManager->getResponses());

上面示例将输出：	
	
The above example produces:

.. code-block:: html

    Array ( [0] => first response [1] => second response )

自定义事件管理器Implementing your own EventsManager
----------------------------------------------------------
如果想要替换Phalcon提供的事件管理器，必须实现  :doc:`Phalcon\\Events\\ManagerInterface <../api/Phalcon_Events_ManagerInterface>` 中的接口。

The :doc:`Phalcon\\Events\\ManagerInterface <../api/Phalcon_Events_ManagerInterface>` interface must be implemented to create your own
EventsManager replacing the one provided by Phalcon.
