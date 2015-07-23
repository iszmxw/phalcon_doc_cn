注解解析器Annotations Parser
=================================
这是第一个为PHP用C语言写的注解解析器。  Phalcon\\Annotations是一个通用组件，为应用中的PHP类提供易于解析和缓存注解的功能。

It is the first time that an annotations parser component is written in C for the PHP world. Phalcon\\Annotations is
a general purpose component that provides ease of parsing and caching annotations in PHP classes to be used in applications.

注解内容是读自类，方法和属性的注解区域。一个注解单元可以放在注解区域的任何位置。

Annotations are read from docblocks in classes, methods and properties. An annotation can be placed at any position in the docblock:

.. code-block:: php

    <?php

    /**
     * This is the class description
     *
     * @AmazingClass(true)
     */
    class Example
    {

        /**
         * This a property with a special feature
         *
         * @SpecialFeature
         */
        protected $someProperty;

        /**
         * This is a method
         *
         * @SpecialFeature
         */
        public function someMethod()
        {
            // ...
        }

    }

在上面的例子中，我们发现注解块中除了注解单元，还可以有注解内容，一个注解单元语法如下：	
	
In the above example we find some annotations in the comments, an annotation has the following syntax:

@注解名称[(参数1, 参数2, ...)]

@Annotation-Name[(param1, param2, ...)]

当然，一个注解单元可以放在注解内容里的任意位置：

Also, an annotation could be placed at any part of a docblock:

.. code-block:: php

    <?php

    /**
     * This a property with a special feature
     *
     * @SpecialFeature
     *
     * More comments
     *
     * @AnotherSpecialFeature(true)
     */

这个解析器是高度灵活的，下面这样的注解单元是合法可解析的：	 
	 
The parser is highly flexible, the following docblock is valid:

.. code-block:: php

    <?php

    /**
     * This a property with a special feature @SpecialFeature({
    someParameter="the value", false

     })  More comments @AnotherSpecialFeature(true) @MoreAnnotations
     **/

然而，为了使代码更容易维护和理解，我们推荐把注解单元放在注解块的最后：	 
	 
However, to make the code more maintainable and understandable it is recommended to place annotations at the end of the docblock:

.. code-block:: php

    <?php

    /**
     * This a property with a special feature
     * More comments
     *
     * @SpecialFeature({someParameter="the value", false})
     * @AnotherSpecialFeature(true)
     */

读取注解Reading Annotations
-------------------------------
实现反射器（Reflector）可以轻松获取被定义在类中的注解，使用一个面向对象的接口即可：

A reflector is implemented to easily get the annotations defined on a class using an object-oriented interface:

.. code-block:: php

    <?php

    $reader = new \Phalcon\Annotations\Adapter\Memory();

    //Reflect the annotations in the class Example
    $reflector = $reader->get('Example');

    //Read the annotations in the class' docblock
    $annotations = $reflector->getClassAnnotations();

    //Traverse the annotations
    foreach ($annotations as $annotation) {

        //Print the annotation name
        echo $annotation->getName(), PHP_EOL;

        //Print the number of arguments
        echo $annotation->numberArguments(), PHP_EOL;

        //Print the arguments
        print_r($annotation->getArguments());
    }

虽然这个注解的读取过程是非常快速的，然而，出于性能原因，我们建议使用一个适配器储存解析后的注解内容。 适配器把处理后的注解内容缓存起来，避免每次读取都需要解析一遍注解。	
	
The annotation reading process is very fast, however, for performance reasons it is recommended to store the parsed annotations using an adapter.
Adapters cache the processed annotations avoiding the need of parse the annotations again and again.

:doc:`Phalcon\\Annotations\\Adapter\\Memory <../api/Phalcon_Annotations_Adapter_Memory>`  被用在上面的例子中。这个适配器只在请求过程中缓存注解（译者注：请求完成后缓存将被清空），因为这个原因，这个适配器非常适合用于开发环境中。当应用跑在生产环境中还有其他适配器可以替换。

:doc:`Phalcon\\Annotations\\Adapter\\Memory <../api/Phalcon_Annotations_Adapter_Memory>` was used in the above example. This adapter
only caches the annotations while the request is running, for this reason th adapter is more suitable for development. There are
other adapters to swap out when the application is in production stage.

注解类型Types of Annotations
--------------------------------
注解单元可以有参数也可以没有。参数可以为简单的文字(strings, number, boolean, null)，数组，哈希列表或者其他注解单元：

Annotations may have parameters or not. A parameter could be a simple literal (strings, number, boolean, null), an array, a hashed list or other annotation:

.. code-block:: php

    <?php

    /**
     * Simple Annotation
     *
     * @SomeAnnotation
     */

    /**
     * Annotation with parameters
     *
     * @SomeAnnotation("hello", "world", 1, 2, 3, false, true)
     */

    /**
     * Annotation with named parameters
     *
     * @SomeAnnotation(first="hello", second="world", third=1)
     * @SomeAnnotation(first: "hello", second: "world", third: 1)
     */

    /**
     * Passing an array
     *
     * @SomeAnnotation([1, 2, 3, 4])
     * @SomeAnnotation({1, 2, 3, 4})
     */

    /**
     * Passing a hash as parameter
     *
     * @SomeAnnotation({first=1, second=2, third=3})
     * @SomeAnnotation({'first'=1, 'second'=2, 'third'=3})
     * @SomeAnnotation({'first': 1, 'second': 2, 'third': 3})
     * @SomeAnnotation(['first': 1, 'second': 2, 'third': 3])
     */

    /**
     * Nested arrays/hashes
     *
     * @SomeAnnotation({"name"="SomeName", "other"={
     *      "foo1": "bar1", "foo2": "bar2", {1, 2, 3},
     * }})
     */

    /**
     * Nested Annotations
     *
     * @SomeAnnotation(first=@AnotherAnnotation(1, 2, 3))
     */

实际使用Practical Usage
-----------------------------
接下来我们将解释PHP应用程序中的注解的一些实际的例子：

Next we will explain some practical examples of annotations in PHP applications:

注解开启缓存Cache Enabler with Annotations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
我们假设一下，假设我们接下来的控制器和开发者想要建一个插件，如果被执行的方法被标记为可缓存的话，这个插件可以自动开启缓存。首先，我们先注册这个插件到Dispatcher服务中，这样这个插件将被通知当控制器的路由被执行的时候：

Let's pretend we've the following controller and the developer wants to create a plugin that automatically start the
cache if the latest action executed is marked as cacheable. First off all we register a plugin in the Dispatcher service
to be notified when a route is executed:

.. code-block:: php

    <?php

    $di['dispatcher'] = function() {

        $eventsManager = new \Phalcon\Events\Manager();

        //Attach the plugin to 'dispatch' events
        $eventsManager->attach('dispatch', new CacheEnablerPlugin());

        $dispatcher = new \Phalcon\Mvc\Dispatcher();
        $dispatcher->setEventsManager($eventsManager);
        return $dispatcher;
    };

CacheEnablerPlugin 这个插件拦截每一个被dispatcher执行的action，检查如果需要则启动缓存：	
	
CacheEnablerPlugin is a plugin that intercept every action executed in the dispatcher enabling the cache if needed:

.. code-block:: php

    <?php

    /**
     * Enables the cache for a view if the latest
     * executed action has the annotation @Cache
     */
    class CacheEnablerPlugin extends \Phalcon\Mvc\User\Plugin
    {

        /**
         * This event is executed before every route is executed in the dispatcher
         *
         */
        public function beforeExecuteRoute($event, $dispatcher)
        {

            //Parse the annotations in the method currently executed
            $annotations = $this->annotations->getMethod(
                $dispatcher->getActiveController(),
                $dispatcher->getActiveMethod()
            );

            //Check if the method has an annotation 'Cache'
            if ($annotations->has('Cache')) {

                //The method has the annotation 'Cache'
                $annotation = $annotations->get('Cache');

                //Get the lifetime
                $lifetime = $annotation->getNamedParameter('lifetime');

                $options = array('lifetime' => $lifetime);

                //Check if there is an user defined cache key
                if ($annotation->hasNamedParameter('key')) {
                    $options['key'] = $annotation->getNamedParameter('key');
                }

                //Enable the cache for the current method
                $this->view->cache($options);
            }

        }

    }

现在，我们可以使用注解单元在控制器中：	
	
Now, we can use the annotation in a controller:

.. code-block:: php

    <?php

    class NewsController extends \Phalcon\Mvc\Controller
    {

        public function indexAction()
        {

        }

        /**
         * This is a comment
         *
         * @Cache(lifetime=86400)
         */
        public function showAllAction()
        {
            $this->view->article = Articles::find();
        }

        /**
         * This is a comment
         *
         * @Cache(key="my-key", lifetime=86400)
         */
        public function showAction($slug)
        {
            $this->view->article = Articles::findFirstByTitle($slug);
        }

    }

选择渲染模版Choose template to render
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在这个例子中，当方法被执行的时候，我们将使用注解单元去告诉:doc:Phalcon\Mvc\View\Simple <views>，哪一个模板文件需要渲染：

In this example we're going to use annotations to tell :doc:`Phalcon\\Mvc\\View\\Simple <views>` what template must me rendered
once the action has been executed:


注解适配器Annotations Adapters
----------------------------------
这些组件利用了适配器去缓存或者不缓存已经解析和处理过的注解内容，从而提升了性能或者为开发环境提供了开发/测试的适配器：

This component makes use of adapters to cache or no cache the parsed and processed annotations thus improving the performance or providing facilities to development/testing:

+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------+
| Name       | Description                                                                                                                                                                                                                          | API                                                                                      |
+============+======================================================================================================================================================================================================================================+==========================================================================================+
| Memory     | The annotations are cached only in memory. When the request ends the cache is cleaned reloading the annotations in each request. This adapter is suitable for a development stage                                                    | :doc:`Phalcon\\Annotations\\Adapter\\Memory <../api/Phalcon_Annotations_Adapter_Memory>` |
+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------+
| Files      | Parsed and processed annotations are stored permanently in PHP files improving performance. This adapter must be used together with a bytecode cache.                                                                                | :doc:`Phalcon\\Annotations\\Adapter\\Files <../api/Phalcon_Annotations_Adapter_Files>`   |
+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------+
| APC        | Parsed and processed annotations are stored permanently in the APC cache improving performance. This is the faster adapter                                                                                                           | :doc:`Phalcon\\Annotations\\Adapter\\Apc <../api/Phalcon_Annotations_Adapter_Apc>`       |
+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------+
| XCache     | Parsed and processed annotations are stored permanently in the XCache cache improving performance. This is a fast adapter too                                                                                                        | :doc:`Phalcon\\Annotations\\Adapter\\Xcache <../api/Phalcon_Annotations_Adapter_Xcache>` |
+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------+

自定义适配器Implementing your own adapters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
为了建立自己的注解适配器或者继承一个已存在的适配器，这个  :doc:`Phalcon\\Annotations\\AdapterInterface <../api/Phalcon_Annotations_AdapterInterface>` 接口都必须实现。

The :doc:`Phalcon\\Annotations\\AdapterInterface <../api/Phalcon_Annotations_AdapterInterface>` interface must be implemented in order to create your own
annotations adapters or extend the existing ones.

外部资源External Resources
-------------------------------
* `Tutorial: Creating a custom model’s initializer with Annotations <http://blog.phalconphp.com/post/47471246411/tutorial-creating-a-custom-models-initializer-with>`_
