多语言支持Multi-lingual Support
=================================
:doc:`Phalcon\\Translate <../api/Phalcon_Translate>`提供辅助去创建多语言应用。使用它可以在应用中基于用户的选择显示不同的语言。

The component :doc:`Phalcon\\Translate <../api/Phalcon_Translate>` aids in creating multilingual applications. Applications using this component,
display content in different languages, based on the user's chosen language supported by the application.

适配器Adapters
----------------
该组件使用适配器以统一的方式读取翻译来自不同来源的信息。

This component makes use of adapters to read translation messages from different sources in a unified way.

+-------------+-----------------------------------------------------------------------------------------+
| Adapter     | Description                                                                             |
+=============+=========================================================================================+
| NativeArray | Uses PHP arrays to store the messages. This is the best option in terms of performance. |
+-------------+-----------------------------------------------------------------------------------------+

组件的使用Component Usage
----------------------------
翻译的字符串存储在文件中。这些文件的结构可以根据使用的适配器。可以自由组织翻译字符串。一个简单的结构:

Translation strings are stored in files. The structure of these files could vary depending of the adapter used. Phalcon gives you the freedom
to organize your translation strings. A simple structure could be:

.. code-block:: bash

    app/messages/en.php
    app/messages/es.php
    app/messages/fr.php
    app/messages/zh.php

每个文件包含一个数组的键/值的方式翻译。对于每一个翻译文件,keys是独一无二的。在不同的文件中使用相同的数组,keys是相同的，值是根据每种语言翻译后的字符串。	
	
Each file contains an array of the translations in a key/value manner. For each translation file, keys are unique. The same array is used in
different files, where keys remain the same and values contain the translated strings depending on each language.

.. code-block:: php

    <?php

    //app/messages/es.php
    $messages = array(
        "hi"      => "Hello",
        "bye"     => "Good Bye",
        "hi-name" => "Hello %name%",
        "song"    => "This song is %song%"
    );

.. code-block:: php

    <?php

    //app/messages/fr.php
    $messages = array(
        "hi"      => "Bonjour",
        "bye"     => "Au revoir",
        "hi-name" => "Bonjour %name%",
        "song"    => "La chanson est %song%"
    );

在应用程序中实现翻译机制是微不足道但取决于如何去实现它可以从用户的浏览器使用自动检测语言或您可以提供一个设置页面,用户可以选择他们的语言。	
	
Implementing the translation mechanism in your application is trivial but depends on how you wish to implement it. You can use an
automatic detection of the language from the user's browser or you can provide a settings page where the user can select their language.

A simple way of detecting the user's language is to parse the $_SERVER['HTTP_ACCEPT_LANGUAGE'] contents, or if you wish, access it
directly by calling $this->request->getBestLanguage() from an action/controller:

.. code-block:: php

    <?php

    class UserController extends \Phalcon\Mvc\Controller
    {

      protected function _getTranslation()
      {

        //Ask browser what is the best language
        $language = $this->request->getBestLanguage();

        //Check if we have a translation file for that lang
        if (file_exists("app/messages/".$language.".php")) {
           require "app/messages/".$language.".php";
        } else {
           // fallback to some default
           require "app/messages/en.php";
        }

        //Return a translation object
        return new \Phalcon\Translate\Adapter\NativeArray(array(
           "content" => $messages
        ));

      }

      public function indexAction()
      {
        $this->view->setVar("name", "Mike");
        $this->view->setVar("t", $this->_getTranslation());
      }

    }

_getTranslation在需要翻译的每个actions中都有效。$t被传递到视图。使用它我们可以完成翻译。	
	
The _getTranslation method is available for all actions that require translations. The $t variable is passed to the views, and with it,
we can translate strings in that layer:

.. code-block:: html+php

    <!-- welcome -->
    <!-- String: hi => 'Hello' -->
    <p><?php echo $t->_("hi"), " ", $name; ?></p>

"_"函数根据参数的参数返回翻译后的字符串。一些字符串需要合并的占位符计算数据。例如Hello %name%。占位符可以被"_"函数中的参数替换。传递参数也是键值对。键名对应占位符，键名对应实际的字符串。
	
The "_" function is returning the translated string based on the index passed. Some strings need to incorporate placeholders for
calculated data i.e. Hello %name%. These placeholders can be replaced with passed parameters in the "_ function. The passed parameters
are in the form of a key/value array, where the key matches the placeholder name and the value is the actual data to be replaced:

.. code-block:: html+php

    <!-- welcome -->
    <!-- String: hi-name => 'Hello %name%' -->
    <p><?php echo $t->_("hi-name", array("name" => $name)); ?></p>

一些应用程序实现多语种如URL	http://www.mozilla.org/**es-ES**/firefox/. Phalcon可以通过:doc:`Router <routing>`实现。
	
Some applications implement multilingual on the URL such as http://www.mozilla.org/**es-ES**/firefox/. Phalcon can implement
this by using a :doc:`Router <routing>`.

自定义适配器Implementing your own adapters
---------------------------------------------
:doc:`Phalcon\\Translate\\AdapterInterface <../api/Phalcon_Translate_AdapterInterface>`一定要被继承，如果想要创建自己的语言适配器。

The :doc:`Phalcon\\Translate\\AdapterInterface <../api/Phalcon_Translate_AdapterInterface>` interface must be implemented in order to create your own translate adapters or extend the existing ones:

.. code-block:: php

    <?php

    class MyTranslateAdapter implements Phalcon\Translate\AdapterInterface
    {

        /**
         * Adapter constructor
         *
         * @param array $data
         */
        public function __construct($options);

        /**
         * Returns the translation string of the given key
         *
         * @param   string $translateKey
         * @param   array $placeholders
         * @return  string
         */
        public function _($translateKey, $placeholders=null);

        /**
         * Returns the translation related to the given key
         *
         * @param   string $index
         * @param   array $placeholders
         * @return  string
         */
        public function query($index, $placeholders=null);

        /**
         * Check whether is defined a translation key in the internal array
         *
         * @param   string $index
         * @return  bool
         */
        public function exists($index);

    }

更多适配器可以参看`Phalcon Incubator <https://github.com/phalcon/incubator/tree/master/Library/Phalcon/Translate/Adapter>`_	
	
There are more adapters available for this components in the `Phalcon Incubator <https://github.com/phalcon/incubator/tree/master/Library/Phalcon/Translate/Adapter>`_
