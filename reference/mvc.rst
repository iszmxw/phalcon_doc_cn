MVC 架构The MVC Architecture
==============================
Phalcon 提供了面向对象的类，在应用程序中必须实现模型（Model）、视图（View）、控制器（Controller）架构（通常我们称之为 MVC）。这种设计模式被广泛的应用到其他 web 框架以及桌面应用程序中。

Phalcon offers the object-oriented classes, necessary to implement the Model, View, Controller architecture
(often referred to as MVC_) in your application. This design pattern is widely used by other web frameworks
and desktop applications.

MVC 优点

MVC benefits include:

* 隔离业务逻辑、用户界面和数据库层
* 不同类型的代码之间更加明确易于维护。

* Isolation of business logic from the user interface and the database layer
* Making it clear where different types of code belong for easier maintenance

如果你决定使用MVC架构来开发你的程序，那么应用程序的每个请求都将采用 MVC 架构的方式来管理。 Phalcon 的类是使用 C 语言编写而成， 这是为这种模式开发的 PHP 应用程序提供高性能的方法。

If you decide to use MVC, every request to your application resources will be managed by the MVC_ architecture.
Phalcon classes are written in C language, offering a high performance approach of this pattern in a PHP based application.

模型Models
-------------
模型代表了应用程序中的信息（数据）和处理数据的规则。模型主要用于管理与相应数据库表进行交互的规则。 大多数情况中，在应用程序中，数据库中每个表将对应一个模型。 应用程序中的大部分业务逻辑都将集中在模型里。 :doc:`了解更多 <models>`

A model represents the information (data) of the application and the rules to manipulate that data. Models are primarily used for
managing the rules of interaction with a corresponding database table. In most cases, each table in your database will correspond
to one model in your application. The bulk of your application's business logic will be concentrated in the models. :doc:`Learn more <models>`

视图Views
-----------
视图代表了应用程序中的用户界面. 视图通常是在 HTML 文件里嵌入 PHP 代码，这些代码仅仅是用来展示数据。 视图的任务是当应用程序发生请求时，提供数据给 web 浏览器或者其他工具。 :doc:`了解更多 <views>`

Views represent the user interface of your application. Views are often HTML files with embedded PHP code that perform tasks
related solely to the presentation of the data. Views handle the job of providing data to the web browser or other tool that
is used to make requests from your application. :doc:`Learn more <views>`

控制器Controllers
------------------
控制器用于控制应用程序的流程，调用模型和视图。负责处理来自 web 浏览器的请求，从模型中获取数据，然后将数据传递给视图完成展示。

The controllers provide the "flow" between models and views. Controllers are responsible for processing the incoming requests
from the web browser, interrogating the models for data, and passing that data on to the views for presentation. :doc:`Learn more <controllers>`


.. _MVC: http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
