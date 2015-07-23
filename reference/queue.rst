队列Queueing
===============
执行如处理视频，调整图片或发送电子邮件不适合被实时在线执行，因为它可能慢的页面加载时间,影响用户体验。

Perform activities like process a video, resize images or send emails aren't suitable to be executed
online or in real time because it may slow the loading time of pages, impacting the user experience.

这里的最佳解决方案是实现后台的工作。一个web应用程序必须把这些工作做成一个队列,然后等后台去处理它们。

The best solution here is implementing background jobs. A web application must put the job
into a queue and wait that it will be processed.

可以使用RabbitMQ_通过PHP扩展。Phalcon提供一个 Beanstalk_ 的客户端。一个由 Memcache_ 激发而来的任务消息队列。很简单、轻量级和完全专业的队列。

While you can find more sophisticated PHP extensions to address queueing in your applications like RabbitMQ_;
Phalcon provides a client for Beanstalk_, a job queueing backend inspired by Memcache_.
It’s simple, lightweight, and completely specialized on job queueing.

将任务加入队列Putting Jobs into the Queue
----------------------------------------------
在连接到Bens后可以插入任意多的工作。开发人员可以根据应用程序的需要结构定义消息:

After connecting to Bens can insert as many jobs as required. The developer can define the message
structure according to the needs of the application:

.. code-block:: php

    <?php

    //Connect to the queue
    $queue = new Phalcon\Queue\Beanstalk(array(
        'host' => '192.168.0.21'
    ));

    //Insert the job in the queue
    $queue->put(array('processVideo' => 4871));

可用的连接选项：	
	
Available connection options are:

+----------+----------------------------------------------------------+-----------+
| Option   | Description                                              | Default   |
+==========+==========================================================+===========+
| host     | IP where the beanstalk server is located                 | 127.0.0.1 |
+----------+----------------------------------------------------------+-----------+
| port     | Connection port                                          | 11300     |
+----------+----------------------------------------------------------+-----------+

在上面的例子中我们存储消息,将添加一个后台作业过程去处理一段视频。立即存储在队列的消息,并没有一个特定的时间去完成它。

In the above example we stored a message which will allow a background job to process a video.
The message is stored in the queue immediately and does not have a certain time to life.

Additional options as time to run, priority and delay could be passed as second parameter:

.. code-block:: php

    <?php

    //Insert the job in the queue with options
    $queue->put(
        array('processVideo' => 4871),
        array('priority' => 250, 'delay' => 10, 'ttr' => 3600)
    );

下面的选项可用：	
	
The following options are available:

+----------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Option   | Description                                                                                                                                                                                 |
+==========+=============================================================================================================================================================================================+
| priority | It's an integer < 2**32. Jobs with smaller priority values will be scheduled before jobs with larger priorities. The most urgent priority is 0; the least urgent priority is 4,294,967,295. |
+----------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| delay    | It's an integer number of seconds to wait before putting the job in the ready queue. The job will be in the "delayed" state during this time.                                               |
+----------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ttr      | Time to run -- is an integer number of seconds to allow a worker to run this job. This time is counted from the moment a worker reserves this job.                                          |
+----------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

添加到队列的任务将获得一个队列id，开发者可以跟踪任务情况：

Every job put into the queue returns a "job id" the developer can use to track the status of the job:

.. code-block:: php

    <?php

    $jobId = $queue->put(array('processVideo' => 4871));

获取消息Retrieving Messages
--------------------------------
一旦放入队列中中工作,可以被后台有足够的时间来完成任务并产生消息:

Once a job is placed into the queue, those messages can be consumed by a background job which have enough time to complete
the task:

.. code-block:: php

    <?php

    while (($job = $queue->peekReady()) !== false) {

        $message = $job->getBody();

        var_dump($message);

        $job->delete();
    }

工作完成后必须从队列中删除,以避免双重处理。如果实现了多个后台作业, 任务在处理中就不能再被另一个处理:	
	
Jobs must be removed from the queue to avoid double processing. If multiple background jobs workers are implemented,
jobs must be "reserved" so other workers don't re-process them while other workers have them reserved:

.. code-block:: php

    <?php

    while (($job = $queue->reserve())) {

        $message = $job->getBody();

        var_dump($message);

        $job->delete();
    }

虽然Beanstalkd提供足够多的功能，但是我们也可以去创建自己队列实现。	
	
Our client implement a basic set of the features provided by Beanstalkd but enough to allow you to build applications
implementing queues.

.. _RabbitMQ: http://pecl.php.net/package/amqp
.. _Beanstalk: http://www.igvita.com/2010/05/20/scalable-work-queues-with-beanstalk/
.. _Memcache: http://memcached.org/
