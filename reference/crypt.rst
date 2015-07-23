加密与解密Encryption/Decryption
========================================
Phalcon通过  :doc:`Phalcon\\Crypt <../api/Phalcon_Crypt>`  组件提供了加密和解密工具。这个类提供了对php mcrypt 的封装。默认情况下这个组件使用AES-256 (rijndael-256-cbc)。

Phalcon provides encryption facilities via the :doc:`Phalcon\\Crypt <../api/Phalcon_Crypt>` component.
This class offers simple object-oriented wrappers to the mcrypt_ php's encryption library.

By default, this component provides secure encryption using AES-256 (rijndael-256-cbc).

基本使用Basic Usage
-------------------------
这个组件极易使用：

This component is designed to provide a very simple usage:

.. code-block:: php

    <?php

    use Phalcon\Crypt;

    //Create an instance
    $crypt     = new Crypt();

    $key       = 'le password';
    $text      = 'This is a secret text';

    $encrypted = $crypt->encrypt($text, $key);

    echo $crypt->decrypt($encrypted, $key);

也可以使用同一实例加密多次：	
	
You can use the same instance to encrypt/decrypt several times:

.. code-block:: php

    <?php

    use Phalcon\Crypt;

    //Create an instance
    $crypt = new Crypt();

    $texts = array(
        'my-key'    => 'This is a secret text',
        'other-key' => 'This is a very secret'
    );

    foreach ($texts as $key => $text) {

        //Perform the encryption
        $encrypted = $crypt->encrypt($text, $key);

        //Now decrypt
        echo $crypt->decrypt($encrypted, $key);
    }

加密选项Encryption Options
----------------------------------
下面的选项可以改变加密的行为：

The following options are available to change the encryption behavior:

+------------+---------------------------------------------------------------------------------------------------+
| Name       | Description                                                                                       |
+============+===================================================================================================+
| Cipher     | The cipher is one of the encryption algorithms supported by libmcrypt. You can see a list here_   |
+------------+---------------------------------------------------------------------------------------------------+
| Mode       | One of the encryption modes supported by libmcrypt (ecb, cbc, cfb, ofb)                           |
+------------+---------------------------------------------------------------------------------------------------+

例子:

Example:

.. code-block:: php

    <?php

    use Phalcon\Crypt;

    //Create an instance
    $crypt = new Crypt();

    //Use blowfish
    $crypt->setCipher('blowfish');

    $key   = 'le password';
    $text  = 'This is a secret text';

    echo $crypt->encrypt($text, $key);

支持Base64 Base64 Support
-----------------------------
为了方便传输或显示我们可以对加密后的数据进行 base64_ 转码：

In order that encryption is properly transmitted (emails) or displayed (browsers) base64_ encoding is usually applied to encrypted texts:

.. code-block:: php

    <?php

    use Phalcon\Crypt;

    //Create an instance
    $crypt   = new Crypt();

    $key     = 'le password';
    $text    = 'This is a secret text';

    $encrypt = $crypt->encryptBase64($text, $key);

    echo $crypt->decryptBase64($text, $key);

配置加密服务Setting up an Encryption service
-----------------------------------------------
你也可以把加密组件放入服务容器中这样我们可以在应用中的任何一个地方访问这个组件：

You can set up the encryption component in the services container in order to use it from any part of the application:

.. code-block:: php

    <?php

    use Phalcon\Crypt;

    $di->set('crypt', function() {

        $crypt = new Crypt();

        //Set a global encryption key
        $crypt->setKey('%31.1e$i86e$f!8jz');

        return $crypt;
    }, true);

然后，例如，我们可以在控制器中使用它了：	
	
Then, for example, in a controller you can use it as follows:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class SecretsController extends Controller
    {

        public function saveAction()
        {
            $secret = new Secrets();

            $text = $this->request->getPost('text');

            $secret->content = $this->crypt->encrypt($text);

            if ($secret->save()) {
                $this->flash->success('Secret was successfully created!');
            }
        }
    }

.. _mcrypt: http://www.php.net/manual/en/book.mcrypt.php
.. _here: http://www.php.net/manual/en/mcrypt.ciphers.php
.. _base64: http://www.php.net/manual/en/function.base64-encode.php
