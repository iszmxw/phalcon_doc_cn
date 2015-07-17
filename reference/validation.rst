验证Validation
======================
Phalcon\\Validation对Phalcon来说是一个相对独立的组件，它可以对任意的数据进行验证。 当然也可以用来对非模型内的数据进行验证。 下面的例子展示了一些基本的使用方法：

Phalcon\\Validation is an independent validation component that validates an arbitrary set of data.
This component can be used to implement validation rules on data objects that do not belong to a model or collection.

The following example shows its basic usage:

.. code-block:: php

    <?php

    use Phalcon\Validation;
    use Phalcon\Validation\Validator\Email;
    use Phalcon\Validation\Validator\PresenceOf;

    $validation = new Validation();

    $validation->add('name', new PresenceOf(array(
        'message' => 'The name is required'
    )));

    $validation->add('email', new PresenceOf(array(
        'message' => 'The e-mail is required'
    )));

    $validation->add('email', new Email(array(
        'message' => 'The e-mail is not valid'
    )));

    $messages = $validation->validate($_POST);
    if (count($messages)) {
        foreach ($messages as $message) {
            echo $message, '<br>';
        }
    }

由于此模型是松耦合设计的，故此我们也可以使用自己书写的验证工具：	
	
The loosely-coupled design of this component allows you to create your own validators along with the ones provided by the framework.

初始化验证Initializing Validation
------------------------------------
我们可以直接在Phalcon\\Validation初始化时添加验证链。我们可以把验证器放在一个单独的文件中以提高代码的重用率及可组织性：

Validation chains can be initialized in a direct manner by just adding validators to the Phalcon\\Validation object.
You can put your validations in a separate file for better re-use code and organization:

.. code-block:: php

    <?php

    use Phalcon\Validation;
    use Phalcon\Validation\Validator\Email;
    use Phalcon\Validation\Validator\PresenceOf;

    class MyValidation extends Validation
    {
        public function initialize()
        {
            $this->add('name', new PresenceOf(array(
                'message' => 'The name is required'
            )));

            $this->add('email', new PresenceOf(array(
                'message' => 'The e-mail is required'
            )));

            $this->add('email', new Email(array(
                'message' => 'The e-mail is not valid'
            )));
        }
    }

然后初始化并使用自己的验证器：	
	
Then initialize and use your own validator:

.. code-block:: php

    <?php

    $validation = new MyValidation();

    $messages = $validation->validate($_POST);
    if (count($messages)) {
        foreach ($messages as $message) {
            echo $message, '<br>';
        }
    }

验证器Validators
-------------------
Phalcon的验证组件中内置了一些验证器：

Phalcon exposes a set of built-in validators for this component:

+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Name         | Explanation                                                                                                                                                      | Example                                                           |
+==============+==================================================================================================================================================================+===================================================================+
| PresenceOf   | Validates that a field's value is not null or empty string.                                                                                                      | :doc:`Example <../api/Phalcon_Validation_Validator_PresenceOf>`   |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Identical    | Validates that a field's value is the same as a specified value                                                                                                  | :doc:`Example <../api/Phalcon_Validation_Validator_Identical>`    |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Email        | Validates that field contains a valid email format                                                                                                               | :doc:`Example <../api/Phalcon_Validation_Validator_Email>`        |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| ExclusionIn  | Validates that a value is not within a list of possible values                                                                                                   | :doc:`Example <../api/Phalcon_Validation_Validator_ExclusionIn>`  |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| InclusionIn  | Validates that a value is within a list of possible values                                                                                                       | :doc:`Example <../api/Phalcon_Validation_Validator_InclusionIn>`  |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Regex        | Validates that the value of a field matches a regular expression                                                                                                 | :doc:`Example <../api/Phalcon_Validation_Validator_Regex>`        |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| StringLength | Validates the length of a string                                                                                                                                 | :doc:`Example <../api/Phalcon_Validation_Validator_StringLength>` |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Between      | Validates that a value is between two values                                                                                                                     | :doc:`Example <../api/Phalcon_Validation_Validator_Between>`      |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Confirmation | Validates that a value is the same as another present in the data                                                                                                | :doc:`Example <../api/Phalcon_Validation_Validator_Confirmation>` |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Url          | Validates that field contains a valid URL                                                                                                                        | :doc:`Example <../api/Phalcon_Validation_Validator_Url>`          |
+--------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+

下面代码演示了如何为这个组件创建额外的验证器：

The following example explains how to create additional validators for this component:

.. code-block:: php

    <?php

    use Phalcon\Validation\Message;
    use Phalcon\Validation\Validator;
    use Phalcon\Validation\ValidatorInterface;

    class IpValidator extends Validator implements ValidatorInterface
    {

        /**
         * Executes the validation
         *
         * @param Phalcon\Validation $validator
         * @param string $attribute
         * @return boolean
         */
        public function validate($validator, $attribute)
        {
            $value = $validator->getValue($attribute);

            if (!filter_var($value, FILTER_VALIDATE_IP, FILTER_FLAG_IPV4 | FILTER_FLAG_IPV6)) {

                $message = $this->getOption('message');
                if (!$message) {
                    $message = 'The IP is not valid';
                }

                $validator->appendMessage(new Message($message, $attribute, 'Ip'));

                return false;
            }

            return true;
        }

    }

最重要的一点即是难证器要返回一个布尔值以标识验证是否成功：	
	
It is important that validators return a valid boolean value indicating if the validation was successful or not.

验证信息Validation Messages
-----------------------------
:doc:`Phalcon\\Validation <../api/Phalcon_Validation>`  内置了一个消息子系统，这提供了一个非常好的验证消息回传机制，以便在验证结束后取得验证信息，比如失败原因等。每个消息由一个 :doc:`Phalcon\\Validation\\Message <../api/Phalcon_Mvc_Model_Message>`类的实例构成。验证过程产生的消息可以使用getMessages()方法取得。 每条消息都有一些扩展的信息组成比如产生错误的属性或消息的类型等：

:doc:`Phalcon\\Validation <../api/Phalcon_Validation>` has a messaging subsystem that provides a flexible way to output or store the
validation messages generated during the validation processes.

Each message consists of an instance of the class :doc:`Phalcon\\Validation\\Message <../api/Phalcon_Mvc_Model_Message>`. The set of
messages generated can be retrieved with the getMessages() method. Each message provides extended information like the attribute that
generated the message or the message type:

.. code-block:: php

    <?php

    $messages = $validation->validate();
    if (count($messages)) {
        foreach ($validation->getMessages() as $message) {
            echo "Message: ", $message->getMessage(), "\n";
            echo "Field: ", $message->getField(), "\n";
            echo "Type: ", $message->getType(), "\n";
        }
    }

当然这里我们也可以对getMessages()方法进行重写， 以取得我们想要的信息：	
	
The getMessages() method can be overridden in a validation class to replace/translate the default messages generated by the validators:

.. code-block:: php

    <?php

    use Phalcon\Validation;

    class MyValidation extends Validation
    {

        public function initialize()
        {
            // ...
        }

        public function getMessages()
        {
            $messages = array();
            foreach (parent::getMessages() as $message) {
                switch ($message->getType()) {
                    case 'PresenceOf':
                        $messages[] = 'The field ' . $message->getField() . ' is mandatory';
                        break;
                }
            }
            return $messages;
        }
    }

或我们也可以传送一个message参数以覆盖验证器中默认的信息：	
	
Or you can pass a 'message' parameter to change the default message in each validator:

.. code-block:: php

    <?php

    use Phalcon\Validation\Validator\Email;

    $validation->add('email', new Email(array(
        'message' => 'The e-mail is not valid'
    )));

默认，getMessages()方法会返回在验证过程中所产生的信息。 我们可以使用filter()方法来过滤我们感兴趣的消息：	
	
By default, 'getMessages' returns all the messages generated during validation. You can filter messages
for a specific field using the 'filter' method:

.. code-block:: php

    <?php

    $messages = $validation->validate();
    if (count($messages)) {
        //Filter only the messages generated for the field 'name'
        foreach ($validation->getMessages()->filter('name') as $message) {
            echo $message;
        }
    }

过滤数据Filtering of Data
----------------------------
我们可以在数据被验证之前对其先进行过滤，以确保那些恶意的或不正确的数据不被验证。

Data can be filtered prior to the validation ensuring that malicious or incorrect data is not validated.

.. code-block:: php

    <?php

    use Phalcon\Validation;

    $validation = new Validation();

    $validation
        ->add('name', new PresenceOf(array(
            'message' => 'The name is required'
        )))
        ->add('email', new PresenceOf(array(
            'message' => 'The email is required'
        )));

    //Filter any extra space
    $validation->setFilters('name', 'trim');
    $validation->setFilters('email', 'trim');

这里我们使用  :doc:`filter <filter>`: 组件进行过滤。 我们还可以使用自定义的或内置的过滤器。	
	
Filtering and sanitizing is performed using the :doc:`filter <filter>`: component. You can add more filters to this
component or use the built-in ones.

验证事件Validation Events
-----------------------------
当在类中执行验证时， 我们可以在beforeValidation或afterValidation方法（事件）中执行额外的检查，过滤，清理等工作。 如果beforeValidation方法返回了false 则验证会被中止：

When validations are organized in classes, you can implement the 'beforeValidation' and 'afterValidation' methods to
perform additional checks, filters, clean-up, etc. If 'beforeValidation' method returns false the validation is automatically
cancelled:

.. code-block:: php

    <?php

    use Phalcon\Validation;

    class LoginValidation extends Validation
    {

        public function initialize()
        {
            // ...
        }

        /**
         * Executed before validation
         *
         * @param array $data
         * @param object $entity
         * @param Phalcon\Validation\Message\Group $messages
         * @return bool
         */
        public function beforeValidation($data, $entity, $messages)
        {
            if ($this->request->getHttpHost() != 'admin.mydomain.com') {
                $messages->appendMessage(new Message('Only users can log on in the administration domain'));
                return false;
            }
            return true;
        }

        /**
         * Executed after validation
         *
         * @param array $data
         * @param object $entity
         * @param Phalcon\Validation\Message\Group $messages
         */
        public function afterValidation($data, $entity, $messages)
        {
            //... add additional messages or perform more validations
        }

    }

取消验证Cancelling Validations
================================
默认所有的验证器都会被执行，不管验证成功与否。 我们可以通过设置 cancelOnFail 参数为 true 来指定某个验证器验证失败时中止以后的所有验证：

By default all validators assigned to a field are tested regardless if one of them have failed or not. You can change
this behavior by telling the validation component which validator may stop the validation:

.. code-block:: php

    <?php

    use Phalcon\Validation;
    use Phalcon\Validation\Validator\Regex;
    use Phalcon\Validation\Validator\PresenceOf;

    $validation = new Validation();

    $validation
        ->add('telephone', new PresenceOf(array(
            'message' => 'The telephone is required',
            'cancelOnFail' => true
        )))
        ->add('telephone', new Regex(array(
            'message' => 'The telephone is required',
            'pattern' => '/\+44 [0-9]+/'
        )))
        ->add('telephone', new StringLength(array(
            'messageMinimum' => 'The telephone is too short',
            'min' => 2
        )));

第一个验证器中 cancelOnFail 参数设置为 true 则表示如果此验证器验证失败则验证链中接下的验证不会被执行。		
		
The first validator has the option 'cancelOnFail' with a value of true, therefore if that validator fails the remaining
validators in the chain are not executed.

我们可以在自定义的验证器中设置 cancelOnFail 为 true 来停止验证链：

If you are creating custom validators you can dynamically stop the validation chain by setting the 'cancelOnFail' option:

.. code-block:: php

    <?php

    use Phalcon\Validation\Message;
    use Phalcon\Validation\Validator;
    use Phalcon\Validation\ValidatorInterface;


    class MyValidator extends Validator implements ValidatorInterface
    {

        /**
         * Executes the validation
         *
         * @param Phalcon\Validation $validator
         * @param string $attribute
         * @return boolean
         */
        public function validate($validator, $attribute)
        {
            // If the attribute value is name we must stop the chain
            if ($attribute == 'name') {
                $validator->setOption('cancelOnFail', true);
            }

            //...
        }

    }
