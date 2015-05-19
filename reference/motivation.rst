框架的动机（Our motivation）
===========================

现在市面上有不计其数的框架，但是没有一个能和Phalcon相提并论的（真的，请相信Phalcon）。

There are many PHP frameworks nowadays, but none of them is like Phalcon (Really, trust us on this one).

几乎所有的程序员都喜欢用框架，因为框架为我们提供了很多经过测试的便捷的函数方法，让我们不用再去重复造轮子。但是在每次请求的时候框架要去加载和解析成千上万行的代码和文件，越是面向对象封装越完善的框架加载越慢，正是因为这样应用会变的越来越慢从而影响了最终的用户体验。

Almost all programmers prefer to use a framework. This is primarily because it provides a lot of functionality
that is already tested and ready to use, therefore keeping code DRY (Don't Repeat Yourself). However, the framework
itself demands a lot of file inclusions and hundreds of lines of code to be interpreted and executed on each request
from the actual application. Object-Oriented frameworks also add a lot of overhead to execution making complex
application slow. All these operations slows the application down and subsequently impacts the end user experience.

问题（The Question）
---------------------

为什么我们不能拥有一个非常稳定健壮并包含全部的优点的完美无缺的框架呢？即使有缺点也就那么一点点也可以啊！

Why can't we have a robust framework with all of its advantages but with none or very few disadvantages?

这就是为什么Phalcon被创造出来了！

This is why Phalcon was born!

在这之前Phalcon团队仔细的研究了php的执行流程和内部机制，考虑了大大小小所有可以优化的地方，将底层的方案进行了多次的改进迭代从而达到让Phalcon的性能最大化。

During the last few months, we have extensively researched PHP's behavior, investigating areas for significant optimizations
(big or small). Through this understanding, we managed to remove unnecessary validations, compacted code, performed optimizations
and generated low-level solutions so as to achieve maximum performance from Phalcon.

为什么开发Phalcon？（Why?）
------------------------------

* 使用框架来做php开发越来越普遍，而且几乎都成为一种标准。
* The use of frameworks has become mandatory in professional development with PHP
* 框架让我们更加合理的去组织维护项目，让我们写的更少做的更多。
* Frameworks offer a structured philosophy to easily maintain projects writing less code and making work more fun
* Phalcon团队喜欢php并且认为php应该能用来开发更庞大的项目。
* We love PHP and we think it can be used to create larger and more ambitious projects

PHP 内部是如何运作的？（Inner workings of PHP?）
----------------------
* PHP变量是动态弱类型的，每次进行二元计算时（比如2 + "2"）PHP都会去检查操作数类型并进行隐式类型转换。
* PHP has dynamic and weak variable types. Every time a binary operation is made (ex. 2 + "2"), PHP checks the operand types to perform potential conversions
* PHP 是解释型语言而不是编译型的，这个最大的确定就是性能低。
* PHP is interpreted and not compiled. The major disadvantage is performance loss
* 每个PHP脚本被执行前都需要被解析一次。
* Every time a script is requested it must be first interpreted
* 即使安装了bytecode缓存（像是APC），在每次请求的时候每个文件都还是会进行语法检查。
* If a bytecode cache (like APC) isn't used, syntax checking is performed every time for every file in the request

传统的 PHP 框架如何工作？（How do traditional PHP frameworks work?）
---------------------------------------------------------------------

* 在每次请求的时候包含类和函数的大量的php文件会被加载，硬盘大量的读操作影响性能，特别是在文件目录层次多的时候更加明显。
* Many files with classes and functions are read on every request made. Disk reading is expensive in terms of performance, especially when the file structure includes deep folders
* 现在的框架都用了延迟加载（autoload）去增加性能（只加载执行需要的代码）。
* Modern frameworks use lazy loading (autoload) to increase performance (for load and execute only the code needed)
* 有些类包含的函数并不是在每次请求的时候都会用到，但是他们都会被加载造成了内存消耗。
* Some of these classes contain methods that aren't used in every request but they're loaded always consuming memory
* 重复不断的加载解析严重影响了性能。
* Continuous loading or interpreting is expensive and impacts performance
* 框架的代码是不会经常更新修改的，但是每次请求的时候框架代码都要被加载一遍。
* The framework code does not change very often, and yet an application needs to load and interpret it every time a request is made

PHP C扩展如何工作？（How does a PHP C-extension work?）
-------------------------------------------------------
* PHP 的c扩展在web服务进程启动的时候和php一块一次性被加载到内存中了。
* C extensions are loaded together with PHP one time on the web server's daemon start process
* 扩展编写的框架提供的类和函数可以被多个web应用随时调用。
* Classes and functions provided by the extension are ready to use for any application
* 扩展编写的框架不需要被解析，因为它们已经为针对特定的平台（win mac linux）和处理器（intel amd）编译了。
* The code isn't interpreted because is already compiled to a specific platform and processor

Phalcon 如何工作？（How does Phalcon work?）
----------------------------------------------
* Phalcon框架的组件是松耦合的，没有任何强制的捆绑，我们可以自由的去使用全部的框架组件，或者只使用其中的一部分作为中间耦合件。
* Components are loosely coupled. With Phalcon, nothing is imposed on you: you're free to use the full framework, or just some parts of it as a glue components.
* Phalcon框架底层的优化为基于MVC的应用带来了最低的系统开销。
* Low-level optimizations provides the lowest overhead for MVC-based applications
* Phalcon中C语言编写的php ORM 为数据库操作提供了最大性能的支持。
* Interact with databases with maximum performance by using a C-language ORM for PHP
* Phalcon直接访问PHP内部结构从而优化执行过程。
* Phalcon directly accesses internal PHP structures optimizing execution in that way as well

为什么需要 Phalcon？（Why do I need Phalcon?）
----------------------------------------------

每个web应用的需求和目的各不相同。一些应用只是用于接受用户请求展示内容，并且很少有更新，这样的应用可以用任意的语言或者是框架来实现，只要使用了前端缓存技术，即使再差的架构设计，使用依然很流畅。

Each application requirements and tasks are different than another's. Some for instance are designed to do a set
of tasks and generate content that rarely changes. These applications can be created with any programming language or
framework. Using a front-end cache usually makes such an application, no matter how poorly designed or slow it might be,
perform very fast.

一些应用每次请求返回的数据都不相同，PHP需要去接受请求重新生成内容，类似的应用比如API接口，大量用户的论坛，有大量评论和贡献的博客，统计应用，管理员控制面板，ERP系统，处理实时数据商业智能分析系统等等。


Other applications generate content almost immediately that changes from request to request. In this case, PHP is used
to address all requests and generate the content. These applications can be APIs, discussion forums with high traffic loads,
blogs with a high number of comments and contributors, statistic applications, admin dashboards, enterprise resource
planners (ERP), business-intelligence software dealing with real time data and more.

应用的性能由最慢的那个流程环节决定（木桶原理），Phalcon提供了一个非常快速并且完善的框架，让应用运行速度更快。通过合理的代码编写，Phalcon可以让我们使用更少的内存和资源达到最大的性能。

An application will be as slow as its slowest component/process. Phalcon offers a very fast yet feature rich framework
that allows developers to concentrate on making their applications/code faster. Following proper coding processes,
Phalcon can deliver a lot more functionality/requests with less memory consumption and processing cycles.

结束语（Conclusion）
-------------------
Phalcon旨在打造最快的PHP框架。我们现在有了最快最简单的方式去实践“性能真的很重要”这条真理了！开始享受吧！

Phalcon is an effort to build the fastest framework for PHP. You now have an even easier and robust way
to develop applications with a framework implemented with the philosophy "Performance Really Matters"! Enjoy!
