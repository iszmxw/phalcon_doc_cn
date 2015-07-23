提高性能：下一步该做什么？Increasing Performance: What's next?
====================================================================
要开发出高性能的应用程序需要考虑多方面的因素： 服务器， 客户端， 网络， 数据库， web服务器，静态资源等。 本章中我集中分析在如何提升系统的性能及 如何检测应用的瓶颈。

Get faster applications requires refine many aspects: server, client, network, database, web server, static sources, etc.
In this chapter we highlight scenarios where you can improve performance and how detect what is really slow in
your application.

关于服务端Profile on the Server
--------------------------------------
每种应用都有不同， 持久的性能分析对找出系统瓶颈是非常必要的。 性能分析可以让我们更直观的看出何处为性能瓶颈，何处不是。 性能分析在一个请示中和另一请求中可能表现不一，所以要做出足够的分析及权衡方可给出结论。

Each application is different, the permanent profiling is important to understand where performance can be increased.
Profiling gives us a real picture on what is really slow and what does not. Profiles can vary between a request and another,
so it is important to make enough measurements to make conclusions.

使用XDebug性能分析 Profiling with XDebug
^^^^^^^^^^^^^^^^^^^^^
Xdebug_ 提供了简易的性能测试的方式， 安装后可在php.ini中 进行如下配置：

Xdebug_ provides an easier way to profile PHP applications, just install the extension and enable profiling in the php.ini:

.. code-block:: ini

    xdebug.profiler_enable = On

使用 Webgrid_ 可以分析出哪些函数或方法比其它的要慢：	
	
Using a tool like Webgrind_ you can see which functions/methods are slower than others:

.. figure:: ../_static/img/webgrind.jpg
    :align: center

使用Xhprof性能分析 Profiling with Xhprof
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Xhprof_ 也是一个非常有意思的扩展。 开发者可以添加如下的代码到启动文件中：

Xhprof_ is another interesting extension to profile PHP applications. Add the following line to the start of the bootstrap file:

.. code-block:: php

    <?php

    xhprof_enable(XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY);

然后在启动文件的结尾保存性能分析数据：	
	
Then at the end of the file save the profiled data:

.. code-block:: php

    <?php

    $xhprof_data = xhprof_disable('/tmp');

    $XHPROF_ROOT = "/var/www/xhprof/";
    include_once $XHPROF_ROOT . "/xhprof_lib/utils/xhprof_lib.php";
    include_once $XHPROF_ROOT . "/xhprof_lib/utils/xhprof_runs.php";

    $xhprof_runs = new XHProfRuns_Default();
    $run_id = $xhprof_runs->save_run($xhprof_data, "xhprof_testing");

    echo "http://localhost/xhprof/xhprof_html/index.php?run={$run_id}&source=xhprof_testing\n";

Xhprof 提供了一个内置的html视图用来对性能分析的数据进行展示：	
	
Xhprof provides a built-in html viewer to analize the profiled data:

.. figure:: ../_static/img/xhprof-2.jpg
    :align: center

.. figure:: ../_static/img/xhprof-1.jpg
    :align: center

SQL语句性能分析 Profiling SQL Statements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Most database systems provide tools to identify slow SQL statements. Detecting and fixing slow queries is very important in order to increase performance
in the server side. In the Mysql case, you can use the slow query log to know what SQL queries are taking more time than expected:

.. code-block:: ini

    log-slow-queries = /var/log/slow-queries.log
    long_query_time = 1.5

客户端性能分析Profile on the Client
------------------------------------------
有时开发者需要提升静态资源加载的速度， 比如图片， javascript, css等。 下面的工具可以让开发者从客户端检测静态资源加载的瓶颈：

Sometimes we may need to improve the loading of static elements such as images, javascript and css to improve performance.
The following tools are useful to detect common bottlenecks in the client side:

使用Chrome/Firefox进行性能分析Profile with Chrome/Firefox
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
几乎所有的现代浏览器都有相应的工具来检测页面加载时间。 Chrome中开发者可使用web探察器来获取一个页面的所有资源加载所需的时间：

Most modern browsers have tools to profile the page loading time. In Chrome you can use the web inspector to know how much time is taking the
loading of the different resources required by a single page:

.. figure:: ../_static/img/chrome-1.jpg
    :align: center

Firebug_ provides a similar functionality:

.. figure:: ../_static/img/firefox-1.jpg
    :align: center

Yahoo! YSlow
---------------
开发者可以使用 YSlow_ 对网页进行分析， YSlow给出基于 rules for high performance web pages （高性能网页规)的建议：

YSlow_ analyzes web pages and suggests ways to improve their performance based on a set of `rules for high performance web pages`_

.. figure:: ../_static/img/yslow-1.jpg
    :align: center

使用Speed Trace进行性能分析Profile with Speed Tracer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
`Speed Tracer`_ 这个工具可以帮助开发者找出web应用性能方面的问题。 这个工个从浏览器的底层分析出web应用的性能。 Speed Tracer 这个插可以安装 在Ｗindows或Linux版本的Chrome上。

`Speed Tracer`_ is a tool to help you identify and fix performance problems in your web applications. It visualizes metrics that are taken
from low level instrumentation points inside of the browser and analyzes them as your application runs. Speed Tracer is available as a
Chrome extension and works on all platforms where extensions are currently supported (Windows and Linux).

.. figure:: ../_static/img/speed-tracer.jpg
    :align: center

这是一个非常有用的工具，它可以为我们显示出html页面渲染的时间， Javascript及css执行(渲染)的时间等。	
	
This tool is very useful because it help you to get the real time used to render the whole page including HTML parsing,
Javascript evaluation and CSS styling.

使用最新的 PHP 版本Use a recent PHP version
-------------------------------------------------
PHP本身的执行速度已经越来越快了， 使用最新版本的php及phalcon可以更高的提升web应用的执行速度。

PHP is faster every day, using the latest version improves the performance of your applications and also of Phalcon.

使用 PHP 字节码缓存Use a PHP Bytecode Cache
-----------------------------------------------
APC 像其它的字节码缓存工具一样可以帮助web应用程序减少读取及解析php文件解析所花的时间。 安装完apc之后在php.ini中添加如何配置：

APC_ as many other bytecode caches help an application to reduce the overhead of read, tokenize and parse PHP files
in each request. Once the extension is installed use the following setting to enable APC:

.. code-block:: ini

    apc.enabled = On

PHP5.5中包含了一个内置的字节码缓存器，即 ZendOptimizer+, 这个扩展在5.3及5.4版本的php中也存在，只不过不是内置的而是用扩展的形式存在的。	
	
PHP 5.5 includes a built-in bytecode cache called ZendOptimizer+, this extension is also available for 5.3 and 5.4.

将可能发生阻塞的操作放到后台运行Do blocking work in the background
-----------------------------------------------------------------------
处理视频， 发送e-mail, 压缩文件和图片等是非常耗时的， 这些最好放在后台执行。 开发者可以使用队列及消息系统以提高web应用的性能，可使用如下组件：

Process a video, send e-mails, compress a file or an image, etc., are slow tasks that must be processed in background jobs.
There are a variety of tools that provide queuing or messaging systems that work well with PHP:

* `Beanstalkd <http://kr.github.io/beanstalkd/>`_
* `Redis <http://redis.io/>`_
* `RabbitMQ <http://www.rabbitmq.com/>`_
* `Resque <https://github.com/chrisboulton/php-resque>`_
* `Gearman <http://gearman.org/>`_
* `ZeroMQ <http://www.zeromq.org/>`_

Google Page SpeedGoogle Page Speed
---------------------------------------
mod_pagespeed_ 可以加速网站的运行速度及减少网站的加载时间。 这个开源的apache web服务器模块（nginx下为ngx_pagespeed）会 自动对网页，静态资源（CSS, JavaScript, images）等进行性能相关的优化，而无需开发者修改已存在的代码，内容，及工作流等。

mod_pagespeed_ speeds up your site and reduces page load time. This open-source Apache HTTP server module (also available
for nginx as ngx_pagespeed) automatically applies web performance best practices to pages, and associated assets
(CSS, JavaScript, images) without requiring that you modify your existing content or workflow.

注： 更多的性能相关的配置或建议可以查看具体的web服务器, 如apache中提供了mod_cache, mod_disk_cache等.

.. _firebug: http://getfirebug.com/
.. _YSlow: http://developer.yahoo.com/yslow/
.. _rules for high performance web pages: http://developer.yahoo.com/performance/rules.html
.. _XDebug: http://xdebug.org/docs
.. _Xhprof: https://github.com/facebook/xhprof
.. _Speed Tracer: https://developers.google.com/web-toolkit/speedtracer/
.. _Webgrind: http://github.com/jokkedk/webgrind/
.. _APC: http://php.net/manual/en/book.apc.php
.. _mod_pagespeed: https://developers.google.com/speed/pagespeed/mod
.. _ngx_pagespeed: https://developers.google.com/speed/pagespeed/ngx
