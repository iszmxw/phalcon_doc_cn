国际化Internationalization
=============================
Phalcon是用C编写的PHP扩展。有一个PECL_扩展PHP应用程序称为intl_提供国际化功能。从PHP 5.4/5.5这个扩展与PHP绑定。它的文档的页面可以找到官方 `PHP manual`_。

Phalcon is written in C as an extension for PHP. There is a PECL_ extension that offers internationalization functions to PHP applications called intl_.
Starting from PHP 5.4/5.5 this extension is bundled with PHP. Its documentation can be found in the pages of the official `PHP manual`_.

Phalcon没有提供这个功能,因为创建这样一个组件将复制现有的代码。

Phalcon does not offer this functionality, since creating such a component would be replicating existing code.

在下面的例子中,我们将向您展示如何实现intl_扩展的功能到Phalcon驱动的应用程序中。

In the examples below, we will show you how to implement the intl_ extension's functionality into Phalcon powered applications.

.. highlights::
   本指南并不打算成为一个完整的文档的intl_扩展。请访问其documentation_扩展的参考。

   This guide is not intended to be a complete documentation of the intl_ extension. Please visit its the documentation_ of the extension for a reference.

匹配最佳的区域设置Find out best available Locale
--------------------------------------------------
有几种方法可以使用intl_找出最有效的语言环境。其中之一是检查HTTP“Accept-Language”的头:

There are several ways to find out the best available locale using intl_. One of them is to check the HTTP "Accept-Language" header:

.. code-block:: php

    <?php

    $locale = Locale::acceptFromHttp($_SERVER["HTTP_ACCEPT_LANGUAGE"]);

    // Locale could be something like "en_GB" or "en"
    echo $locale;

下面的方法返回一个语言环境。它是用来获得语言、文化、地区API或当地特有的行为。　　标识符的例子包括:	
	
Below method returns a locale identified. It is used to get language, culture, or regionally-specific behavior from the Locale API.
Examples of identifiers include:

* en-US (English, United States)
* zh-Hant-TW (Chinese, Traditional Script, Taiwan)
* fr-CA, fr-FR (French for Canada and France respectively)

基于区域设置格式化信息Formatting messages based on Locale
----------------------------------------------------------------
创建一个本地化的应用程序的一部分是产生连接,与语言无关的消息。MessageFormatter_ 允许产生这些消息。

Part of creating a localized application is to produce concatenated, language-neutral messages. The MessageFormatter_ allows for the
production of those messages.

打印格式化数字基于一些语言环境:

Printing numbers formatted based on some locale:

.. code-block:: php

    <?php

    // Prints € 4 560
    $formatter = new MessageFormatter("fr_FR", "€ {0, number, integer}");
    echo $formatter->format(array(4560));

    // Prints USD$ 4,560.5
    $formatter = new MessageFormatter("en_US", "USD$ {0, number}");
    echo $formatter->format(array(4560.50));

    // Prints ARS$ 1.250,25
    $formatter = new MessageFormatter("es_AR", "ARS$ {0, number}");
    echo $formatter->format(array(1250.25));

消息格式使用日期和时间模式:	
	
Message formatting using time and date patterns:

.. code-block:: php

    <?php

    //Setting parameters
    $time   = time();
    $values = array(7, $time, $time);

    // Prints "At 3:50:31 PM on Apr 19, 2012, there was a disturbance on planet 7."
    $pattern   = "At {1, time} on {1, date}, there was a disturbance on planet {0, number}.";
    $formatter = new MessageFormatter("en_US", $pattern);
    echo $formatter->format($values);

    // Prints "À 15:53:01 le 19 avr. 2012, il y avait une perturbation sur la planète 7."
    $pattern   = "À {1, time} le {1, date}, il y avait une perturbation sur la planète {0, number}.";
    $formatter = new MessageFormatter("fr_FR", $pattern);
    echo $formatter->format($values);

特定区域设置的字符串比较Locale-Sensitive comparison
-----------------------------------------------------------
Collator_类提供字符串比较功能根据匹配的语言环境的排序方法。查看下面的例子使用这个类:

The Collator_ class provides string comparison capability with support for appropriate locale-sensitive sort orderings. Check the
examples below on the usage of this class:

.. code-block:: php

    <?php

    // Create a collator using Spanish locale
    $collator = new Collator("es");

    // Returns that the strings are equal, in spite of the emphasis on the "o"
    $collator->setStrength(Collator::PRIMARY);
    var_dump($collator->compare("una canción", "una cancion"));

    // Returns that the strings are not equal
    $collator->setStrength(Collator::DEFAULT_VALUE);
    var_dump($collator->compare("una canción", "una cancion"));

音译Transliteration
-------------------------
Transliterator_ 提供音译的字符串:

Transliterator_ provides transliteration of strings:

.. code-block:: php

    <?php

    $id = "Any-Latin; NFD; [:Nonspacing Mark:] Remove; NFC; [:Punctuation:] Remove; Lower();";
    $transliterator = Transliterator::create($id);

    $string = "garçon-étudiant-où-L'école";
    echo $transliterator->transliterate($string); // garconetudiantoulecole

.. _PECL: http://pecl.php.net/package/intl
.. _intl: http://pecl.php.net/package/intl
.. _PHP manual: http://www.php.net/manual/en/intro.intl.php
.. _documentation: http://www.php.net/manual/en/book.intl.php
.. _MessageFormatter: http://www.php.net/manual/en/class.messageformatter.php
.. _Collator: http://www.php.net/manual/en/class.collator.php
.. _Transliterator: http://www.php.net/manual/en/class.transliterator.php
