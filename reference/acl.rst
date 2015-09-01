访问控制列表Access Control Lists ACL
=============================================
Phalcon在权限方面通过 :doc:`Phalcon\\Acl <../api/Phalcon_Acl>` 提供了一个轻量级的 ACL(访问控制列表). `Access Control Lists`_  (ACL) 允许系统对用户的访问权限进行控制，比如允许访问某些资源而不允许访问其它资源等。 这里我们建议开发者了解一些关于ACL的技术。

:doc:`Phalcon\\Acl <../api/Phalcon_Acl>` provides an easy and lightweight management of ACLs as well as the permissions
attached to them. `Access Control Lists`_ (ACL) allow an application to control access to its areas and the underlying
objects from requests. You are encouraged to read more about the ACL methodology so as to be familiar with its concepts.

ACL有两部分组成即角色和资源。 资源即是ACL定义的权限所依附的对象。 角色即是ACL所字义的请求者的身份，ACL决定了角色对资源的访问权限，允许访问或拒绝访问。

In summary, ACLs have roles and resources. Resources are objects which abide by the permissions defined to them by
the ACLs. Roles are objects that request access to resources and can be allowed or denied access by the ACL mechanism.

创建 ACLCreating an ACL
----------------------------
这个组件起先是设计工作在内存中的， 这样做提供了更高的访问速度。 :doc:`Phalcon\\Acl <../api/Phalcon_Acl>` 构造器的第一个参数用于设置取得ACL的方式。 下面是使用内存适配器的例子：

This component is designed to initially work in memory. This provides ease of use and speed in accessing every aspect of the list. The :doc:`Phalcon\\Acl <../api/Phalcon_Acl>` constructor takes as its first parameter an adapter used to retriever the information related to the control list. An example using the memory adapter is below:

.. code-block:: php

    <?php

    use Phalcon\Acl\Adapter\Memory as AcList;

    $acl = new AclList();

默认情况下 :doc:`Phalcon\\Acl <../api/Phalcon_Acl>` 允许我们访问未定义的资源中的action，为了提高安全性， 我们设置默认访问级别为‘拒绝’。	
	
By default :doc:`Phalcon\\Acl <../api/Phalcon_Acl>` allows access to action on resources that have not been yet defined. To increase the security level of the access list we can define a "deny" level as a default access level.

.. code-block:: php

    <?php

    // Default action is deny access
    $acl->setDefaultAction(Phalcon\Acl::DENY);

添加角色Adding Roles to the ACL
-------------------------------------
角色即是权限的集合体，其中定义了我们对资源的访问权限。 例如， 我们会把一个组织内的不同的人定义为不同的角色。 :doc:`Phalcon\\Acl\\Role <../api/Phalcon_Acl_Role>` 类使用一种更有组织的方式来定义角色。 这里我们创建一些角色：

A role is an object that can or cannot access certain resources in the access list. As an example, we will define roles as groups of people in an organization. The :doc:`Phalcon\\Acl\\Role <../api/Phalcon_Acl_Role>` class is available to create roles in a more structured way. Let's add some roles to our recently created list:

.. code-block:: php

    <?php

    use Phalcon\Acl\Role;

    // Create some roles
    $roleAdmins = new Role("Administrators", "Super-User role");
    $roleGuests = new Role("Guests");

    // Add "Guests" role to acl
    $acl->addRole($roleGuests);

    // Add "Designers" role to acl without a Phalcon\Acl\Role
    $acl->addRole("Designers");

上面我们看到，我们可以直接使用字符串来定义角色。	
	
As you can see, roles are defined directly without using an instance.

添加资源Adding Resources
-----------------------------
资源即是访问控制要控制的对象之一。 正常情况下在mvc中资源一般是控制器。 Phalcon中我们使用 :doc:`Phalcon\\Acl\\Resource <../api/Phalcon_Acl_Resource>`  来定义资源。 非常重要的一点即是我们把相关的action或操作添加到资源中这样ACL才知道控制什么资源。

Resources are objects where access is controlled. Normally in MVC applications resources refer to controllers. Although this is not mandatory, the :doc:`Phalcon\\Acl\\Resource <../api/Phalcon_Acl_Resource>` class can be used in defining resources. It's important to add related actions or operations to a resource so that the ACL can understand what it should to control.

.. code-block:: php

    <?php

    use Phalcon\Acl\Resource;

    // Define the "Customers" resource
    $customersResource = new Resource("Customers");

    // Add "customers" resource with a couple of operations
    $acl->addResource($customersResource, "search");
    $acl->addResource($customersResource, array("create", "update"));

定义访问控制Defining Access Controls
-------------------------------------
至此我们定义了角色及资源， 现在是定义ACL的时候了，即是定义角色对资源的访问。 这个部分是极其重要的，特别是在我们设定了默认的访问级别后。

Now we've roles and resources. It's time to define the ACL i.e. which roles can access which resources. This part is very important especially taking in consideration your default access level "allow" or "deny".

.. code-block:: php

    <?php

    // Set access level for roles into resources
    $acl->allow("Guests", "Customers", "search");
    $acl->allow("Guests", "Customers", "create");
    $acl->deny("Guests", "Customers", "update");

allow()方法指定了允许角色对资源的访问， deny()方法则反之。	
	
The allow method designates that a particular role has granted access to a particular resource. The deny method does the opposite.

查询 ACLQuerying an ACL
--------------------------
一旦访问控制表定义之后， 我们就可以通过它来检查角色是否有访问权限了。

Once the list has been completely defined. We can query it to check if a role has a given permission or not.

.. code-block:: php

    <?php

    // Check whether role has access to the operations
    $acl->isAllowed("Guests", "Customers", "edit");   //Returns 0
    $acl->isAllowed("Guests", "Customers", "search"); //Returns 1
    $acl->isAllowed("Guests", "Customers", "create"); //Returns 1

角色继承Roles Inheritance
------------------------------
我们可以使用 :doc:`Phalcon\\Acl\\Role <../api/Phalcon_Acl_Role>` 提供的继承机制来构造更复杂的角色。 Phalcon中的角色可以继承来自其它角色的 权限, 这样就可以实现更巧妙的资源访问控制。 如果要继承权限用户， 我们需要在添加角色函数的第二个参数中写上要继承的那个角色实例。

You can build complex role structures using the inheritance that :doc:`Phalcon\\Acl\\Role <../api/Phalcon_Acl_Role>` provides. Roles can inherit from other roles, thus allowing access to supersets or subsets of resources. To use role inheritance, you need to pass the inherited role as the second parameter of the function call, when adding that role in the list.

.. code-block:: php

    <?php

    use Phalcon\Acl\Role;

    // ...

    // Create some roles
    $roleAdmins = new Role("Administrators", "Super-User role");
    $roleGuests = new Role("Guests");

    // Add "Guests" role to acl
    $acl->addRole($roleGuests);

    // Add "Administrators" role inheriting from "Guests" its accesses
    $acl->addRole($roleAdmins, $roleGuests);

序列化 ACL 列表Serializing ACL lists
----------------------------------------
为了提高性能， :doc:`Phalcon\\Acl <../api/Phalcon_Acl>` 的实例可以被实例化到APC, session， 文本或数据库中， 这样开发者就不需要重复的 定义acl了。 下面展示了如何去做：

To improve performance :doc:`Phalcon\\Acl <../api/Phalcon_Acl>` instances can be serialized and stored in APC, session, text files or a database table
so that they can be loaded at will without having to redefine the whole list. You can do that as follows:

.. code-block:: php

    <?php

    use Phalcon\Acl\Adapter\Memory as AclList;

    // ...

    // Check whether acl data already exist
    if (!is_file("app/security/acl.data")) {

        $acl = new AclList();

        //... Define roles, resources, access, etc

        // Store serialized list into plain file
        file_put_contents("app/security/acl.data", serialize($acl));
    } else {

         // Restore acl object from serialized file
         $acl = unserialize(file_get_contents("app/security/acl.data"));
    }

    // Use acl list as needed
    if ($acl->isAllowed("Guests", "Customers", "edit")) {
        echo "Access granted!";
    } else {
        echo "Access denied :(";
    }

ACL 事件Acl Events
---------------------
如果需要的话 :doc:`Phalcon\\Acl <../api/Phalcon_Acl>` 可以发送事件到 :doc:`EventsManager <events>` 。 这里我们为acl绑定事件。 其中一些事件的处理结果如果返回了false则表示正在处理的操作会被中止。 支持如下的事件：

:doc:`Phalcon\\Acl <../api/Phalcon_Acl>` is able to send events to a :doc:`EventsManager <events>` if it's present. Events
are triggered using the type "acl". Some events when returning boolean false could stop the active operation. The following events are supported:

+----------------------+------------------------------------------------------------+---------------------+
| Event Name           | Triggered                                                  | Can stop operation? |
+======================+============================================================+=====================+
| beforeCheckAccess    | Triggered before checking if a role/resource has access    | Yes                 |
+----------------------+------------------------------------------------------------+---------------------+
| afterCheckAccess     | Triggered after checking if a role/resource has access     | No                  |
+----------------------+------------------------------------------------------------+---------------------+

下面的例子中展示了如何绑定事件到此组件：

The following example demonstrates how to attach listeners to this component:

.. code-block:: php

    <?php

    use Phalcon\Acl\Adapter\Memory as AclList;
    use Phalcon\Events\Manager as EventsManager;

    // ...

    // Create an event manager
    $eventsManager = new EventsManager();

    // Attach a listener for type "acl"
    $eventsManager->attach("acl", function($event, $acl) {
        if ($event->getType() == "beforeCheckAccess") {
             echo   $acl->getActiveRole(),
                    $acl->getActiveResource(),
                    $acl->getActiveAccess();
        }
    });

    $acl = new AclList();

    //Setup the $acl
    //...

    //Bind the eventsManager to the acl component
    $acl->setEventsManager($eventManagers);

自定义适配器Implementing your own adapters
--------------------------------------------
开发者要创建自己的扩展或已存在适配器则需要实现此 :doc:`Phalcon\\Acl\\AdapterInterface <../api/Phalcon_Acl_AdapterInterface>` 接口。

The :doc:`Phalcon\\Acl\\AdapterInterface <../api/Phalcon_Acl_AdapterInterface>` interface must be implemented in order
to create your own ACL adapters or extend the existing ones.

.. _Access Control Lists: http://en.wikipedia.org/wiki/Access_control_list
