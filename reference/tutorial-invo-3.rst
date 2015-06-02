教程4:使用 CRUDs
=======================
后端通常为用户提供表单让用户去操作数据。我们将继续之前的INVO的讲解，下面我们讲解CRUDs的使用，使用phalcon这个最基础的功能，我们讲能更加熟练的应用表单，验证，分页等功能。

Backends usually provides forms to allow users to manipulate data. Continuing the explanation of
INVO, we now address the creation of CRUDs, a very common task that Phalcon will facilitate you
using forms, validations, paginators and more.

使用 CRUD
---------------------
在INVO（公司，产品，产品类型模块）中我们使用基础普通的 CRUD_ (创建，读取，更新和删除)操作。每个CRUD包含下面的文件：

Most options that manipulate data in INVO (companies, products and types of products), were developed
using a basic and common CRUD_ (Create, Read, Update and Delete). Each CRUD contains the following files:

.. code-block:: bash

    invo/
        app/
            controllers/
                ProductsController.php
            models/
                Products.php
            forms/
                ProductsForm.php
            views/
                products/
                    edit.volt
                    index.volt
                    new.volt
                    search.volt

每个控制器包含下面的动作方法：
					
Each controller has the following actions:

.. code-block:: php

    <?php

    class ProductsController extends ControllerBase
    {

        /**
		 *动作开始显示“搜索”视图
         * The start action, it shows the "search" view
         */
        public function indexAction()
        {
            //...
        }

        /**
		 * 从搜索页接受参数执行搜索并返回分页结果
         * Execute the "search" based on the criteria sent from the "index"
         * Returning a paginator for the results
         */
        public function searchAction()
        {
            //...
        }

        /**
		 * 显示创建新商品视图
         * Shows the view to create a "new" product
         */
        public function newAction()
        {
            //...
        }

        /**
		 * 显示编辑商品视图
         * Shows the view to "edit" an existing product
         */
        public function editAction()
        {
            //...
        }

        /**
		 * 从新建商品接受参数并创建商品
         * Creates a product based on the data entered in the "new" action
         */
        public function createAction()
        {
            //...
        }

        /**
		 * 根据在编辑视图输入的数据更新产品信息
         * Updates a product based on the data entered in the "edit" action
         */
        public function saveAction()
        {
            //...
        }

        /**
		 * 删除一个现有的商品
         * Deletes an existing product
         */
        public function deleteAction($id)
        {
            //...
        }

    }

搜索表单
^^^^^^^^^^^
每一个CRUD始于一个搜索表单。这张表单显示了(产品)表中的每个字段,允许用户从任何字段创建一个搜索条件。“products”表和“products_types”表有关联。在这种情况下，为了方便搜索的字段，我们先查询表“products_types”中的记录:

Every CRUD starts with a search form. This form shows each field that has the table (products), allowing the user
to create a search criteria from any field. Table "products" has a relationship to the table "products_types".
In this case, we previously queried the records in this table in order to facilitate the search by that field:

.. code-block:: php

    <?php

    /**
     * The start action, it shows the "search" view
     */
    public function indexAction()
    {
        $this->persistent->searchParams = null;
        $this->view->form = new ProductsForm;
    }

ProductsForm (app/forms/ProductsForm.php)实例被传递给视图。这个表单定义了用户可见的字段。	
	
An instance of the form ProductsForm (app/forms/ProductsForm.php) is passed to the view.
This form defines the fields that are visible to the user:

.. code-block:: php

    <?php

    use Phalcon\Forms\Form;
    use Phalcon\Forms\Element\Text;
    use Phalcon\Forms\Element\Hidden;
    use Phalcon\Forms\Element\Select;
    use Phalcon\Validation\Validator\Email;
    use Phalcon\Validation\Validator\PresenceOf;
    use Phalcon\Validation\Validator\Numericality;

    class ProductsForm extends Form
    {

        /**
         * Initialize the products form
         */
        public function initialize($entity = null, $options = array())
        {

            if (!isset($options['edit'])) {
                $element = new Text("id");
                $this->add($element->setLabel("Id"));
            } else {
                $this->add(new Hidden("id"));
            }

            $name = new Text("name");
            $name->setLabel("Name");
            $name->setFilters(array('striptags', 'string'));
            $name->addValidators(array(
                new PresenceOf(array(
                    'message' => 'Name is required'
                ))
            ));
            $this->add($name);

            $type = new Select('profilesId', ProductTypes::find(), array(
                'using'      => array('id', 'name'),
                'useEmpty'   => true,
                'emptyText'  => '...',
                'emptyValue' => ''
            ));
            $this->add($type);

            $price = new Text("price");
            $price->setLabel("Price");
            $price->setFilters(array('float'));
            $price->addValidators(array(
                new PresenceOf(array(
                    'message' => 'Price is required'
                )),
                new Numericality(array(
                    'message' => 'Price is required'
                ))
            ));
            $this->add($price);
        }
    }

表单中的元素基于 :doc:`forms <forms>` 组件并以面向对象方式声明，几乎每个元素遵循相同的结构:	
	
The form is declared using an object-oriented scheme based on the elements provided by the :doc:`forms <forms>` component.
Every element follows almost the same structure:

.. code-block:: php

    <?php

    // Create the element
    $name = new Text("name");

    // Set its label
    $name->setLabel("Name");

    // Before validating the element apply these filters
    $name->setFilters(array('striptags', 'string'));

    // Apply this validators
    $name->addValidators(array(
        new PresenceOf(array(
            'message' => 'Name is required'
        ))
    ));

    // Add the element to the form
    $this->add($name);

其他元素也使用这种形式:	
	
Other elements are also used in this form:

.. code-block:: php

    <?php

    // Add a hidden input to the form
    $this->add(new Hidden("id"));

    // ...

    // Add a HTML Select (list) to the form
    // and fill it with data from "product_types"
    $type = new Select('profilesId', ProductTypes::find(), array(
        'using'      => array('id', 'name'),
        'useEmpty'   => true,
        'emptyText'  => '...',
        'emptyValue' => ''
    ));

注意ProductTypes:find()包含必要的数据来填充 Phalcon\\Tag::select 中的SELECT标记。当表单传递到视图,它可以被渲染后并呈现给用户:	
	
Note that ProductTypes::find() contains the data necessary to fill the SELECT tag using Phalcon\\Tag::select.
Once the form is passed to the view, it can be rendered and presented to the user:

.. code-block:: html+jinja

    {{ form("products/search") }}

    <h2>Search products</h2>

    <fieldset>

        {% for element in form %}
            <div class="control-group">
                {{ element.label(['class': 'control-label']) }}
                <div class="controls">{{ element }}</div>
            </div>
        {% endfor %}

        <div class="control-group">
            {{ submit_button("Search", "class": "btn btn-primary") }}
        </div>

    </fieldset>

这生成了以下HTML:	
	
This produces the following HTML:

.. code-block:: html

    <form action="/invo/products/search" method="post">

    <h2>Search products</h2>

    <fieldset>

        <div class="control-group">
            <label for="id" class="control-label">Id</label>
            <div class="controls"><input type="text" id="id" name="id" /></div>
        </div>

        <div class="control-group">
            <label for="name" class="control-label">Name</label>
            <div class="controls">
                <input type="text" id="name" name="name" />
            </div>
        </div>

        <div class="control-group">
            <label for="profilesId" class="control-label">profilesId</label>
            <div class="controls">
                <select id="profilesId" name="profilesId">
                    <option value="">...</option>
                    <option value="1">Vegetables</option>
                    <option value="2">Fruits</option>
                </select>
            </div>
        </div>

        <div class="control-group">
            <label for="price" class="control-label">Price</label>
            <div class="controls"><input type="text" id="price" name="price" /></div>
        </div>

        <div class="control-group">
            <input type="submit" value="Search" class="btn btn-primary" />
        </div>

    </fieldset>

当提交表单时,控制器中“搜索”方法根据用户输入的数据执行搜索。	
	
When the form is submitted, the action "search" is executed in the controller performing the search
based on the data entered by the user.

执行搜索
^^^^^^^^^^
“search”方法具有双重的行为。当通过POST访问时,它执行一个基于接受数据的搜索。但是当通过GET访问的时候，它移动分页器paginator中当前的页面。为了区分请求的HTTP方法，我们使用 :doc:`Request <request>` 组件来进行检查:

The action "search" has a dual behavior. When accessed via POST, it performs a search based on the data sent from the
form. But when accessed via GET it moves the current page in the paginator. To differentiate one from another HTTP method,
we check it using the :doc:`Request <request>` component:

.. code-block:: php

    <?php

    /**
     * Execute the "search" based on the criteria sent from the "index"
     * Returning a paginator for the results
     */
    public function searchAction()
    {

        if ($this->request->isPost()) {
            //create the query conditions
        } else {
            //paginate using the existing conditions
        }

        //...

    }

使用 :doc:`Phalcon\\Mvc\\Model\\Criteria <../api/Phalcon_Mvc_Model_Criteria>` 我们可以基于从表单发送的数据类型和值智能的创建搜索条件:	
	
With the help of :doc:`Phalcon\\Mvc\\Model\\Criteria <../api/Phalcon_Mvc_Model_Criteria>`, we can create the search
conditions intelligently based on the data types and values sent from the form:

.. code-block:: php

    <?php

    $query = Criteria::fromInput($this->di, "Products", $this->request->getPost());

这个方法验证不是""(空字符串)和null的值，并由这些值创建搜索条件:	
	
This method verifies which values are different from "" (empty string) and null and takes them into account to create
the search criteria:

* 如果字段类型是文本或者是相似类型(char, varchar, text, etc.)。使用SQL "like"操作去过滤结果。
* 如果数据类型不是文本或者相关的，就使用"="操作。

* If the field data type is text or similar (char, varchar, text, etc.) It uses an SQL "like" operator to filter the results.
* If the data type is not text or similar, it'll use the operator "=".

此外,“Criteria”忽略了$_POST中所有不匹配表中的任何字段的变量。值使用“约束参数值”自动进行了转义。

Additionally, "Criteria" ignores all the $_POST variables that do not match any field in the table.
Values are automatically escaped using "bound parameters".

现在我们将在控制器的会话袋中存储产生的参数:

Now, we store the produced parameters in the controller's session bag:

.. code-block:: php

    <?php

    $this->persistent->searchParams = $query->getParams();

会话袋是在请求之间持续使用会话服务的控制器中的一个特殊的属性。访问时,:doc:`Phalcon\\Session\\Bag <../api/Phalcon_Session_Bag>` 实例注入到该属性 ，并在每个控制器中独立。	
	
A session bag, is a special attribute in a controller that persists between requests using the session service.
When accessed, this attribute injects a :doc:`Phalcon\\Session\\Bag <../api/Phalcon_Session_Bag>` instance
that is independent in each controller.

然后,基于构建的参数我们执行查询:

Then, based on the built params we perform the query:

.. code-block:: php

    <?php

    $products = Products::find($parameters);
    if (count($products) == 0) {
        $this->flash->notice("The search did not found any products");
        return $this->forward("products/index");
    }

如果搜索不到任何产品,我们引导用户返回首页。假设搜索返回了结果,然后我们创建一个分页器paginator来轻松导航:	
	
If the search doesn't return any product, we forward the user to the index action again. Let's pretend the
search returned results, then we create a paginator to navigate easily through them:

.. code-block:: php

    <?php

    use Phalcon\Paginator\Adapter\Model as Paginator;

    // ...

    $paginator = new Paginator(array(
        "data"  => $products,    // Data to paginate
        "limit" => 5,            // Rows per page
        "page"  => $numberPage   // Active page
    ));

    // Get active page in the paginator
    $page = $paginator->getPaginate();

最后,我们将返回页面输出到视图:	
	
Finally we pass the returned page to view:

.. code-block:: php

    <?php

    $this->view->page = $page;

(app/views/products/search.phtml)在视图中，我们遍历对应于当前页面的每一条结果显示给当前用户:	
	
In the view (app/views/products/search.phtml), we traverse the results corresponding to the current page,
showing every row in the current page to the user:

.. code-block:: html+jinja

    {% for product in page.items %}
      {% if loop.first %}
        <table>
          <thead>
            <tr>
              <th>Id</th>
              <th>Product Type</th>
              <th>Name</th>
              <th>Price</th>
              <th>Active</th>
            </tr>
          </thead>
        <tbody>
      {% endif %}
      <tr>
        <td>{{ product.id }}</td>
        <td>{{ product.getProductTypes().name }}</td>
        <td>{{ product.name }}</td>
        <td>{{ "%.2f"|format(product.price) }}</td>
        <td>{{ product.getActiveDetail() }}</td>
        <td width="7%">{{ link_to("products/edit/" ~ product.id, 'Edit') }}</td>
        <td width="7%">{{ link_to("products/delete/" ~ product.id, 'Delete') }}</td>
      </tr>
      {% if loop.last %}
      </tbody>
        <tbody>
          <tr>
            <td colspan="7">
              <div>
                {{ link_to("products/search", 'First') }}
                {{ link_to("products/search?page=" ~ page.before, 'Previous') }}
                {{ link_to("products/search?page=" ~ page.next, 'Next') }}
                {{ link_to("products/search?page=" ~ page.last, 'Last') }}
                <span class="help-inline">{{ page.current }} of {{ page.total_pages }}</span>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
      {% endif %}
    {% else %}
      No products are recorded
    {% endfor %}

在上面的例子中有很多东西值得详细深入。首先,在当前页面遍历项目使用Volt的“for”。Volt提供了一个简单的语法实现一个PHP foreach。	
	
There are many things in the above example that worth detailing. First of all, active items
in the current page are traversed using a Volt's 'for'. Volt provides a simpler syntax for a PHP 'foreach'.

.. code-block:: html+jinja

    {% for product in page.items %}

在PHP是一样的:	
	
Which in PHP is the same as:

.. code-block:: php

    <?php foreach ($page->items as $product) { ?>

整个的“for”块如下所示：	
	
The whole 'for' block provides the following:

    {% for product in page.items %}
      {% if loop.first %}
        Executed before the first product in the loop
      {% endif %}
        Executed for every product of page.items
      {% if loop.last %}
        Executed after the last product is loop
      {% endif %}
    {% else %}
      Executed if page.items does not have any products
    {% endfor %}

现在我们可以返回到视图和找出每一块代码做什么。每一个在“产品”中的字段打印相应信息:	
	
Now you can go back to the view and find out what every block is doing. Every field
in "product" is printed accordingly:

.. code-block:: html+jinja

    <tr>
        <td>{{ product.id }}</td>
        <td>{{ product.productTypes.name }}</td>
        <td>{{ product.name }}</td>
        <td>{{ "%.2f"|format(product.price) }}</td>
        <td>{{ product.getActiveDetail() }}</td>
        <td width="7%">{{ link_to("products/edit/" ~ product.id, 'Edit') }}</td>
        <td width="7%">{{ link_to("products/delete/" ~ product.id, 'Delete') }}</td>
      </tr>

正如我们之前看到的一样，使用product.id和在PHP中使用:$product->id是一样的，我们使用product.name是一样的。在其他地方渲染是不相同的,例如,product.productTypes.name。要理解这部分,我们必须看下products模型(app/models/Products.php):	  
	  
As we seen before using product.id is the same as in PHP as doing: $product->id,
we made the same with product.name and so on. Other fields are rendered differently,
for instance, let's focus in product.productTypes.name. To understand this part,
we have to check the model Products (app/models/Products.php):

.. code-block:: php

    <?php

    use Phalcon\Mvc\Model;

    /**
     * Products
     */
    class Products extends Model
    {
        // ...

        /**
         * Products initializer
         */
        public function initialize()
        {
            $this->belongsTo('product_types_id', 'ProductTypes', 'id', array(
                'reusable' => true
            ));
        }

        // ...
    }

一个模型,可以有一个名为“initialize”的方法,这种方法每次请求都会被调用一次来初始化一个ORM数据模型。在这种情况下,“Products”是通过初始化定义,模型有一对多关系到另一个“ProductTypes”模型。	
	
A model, can have a method called "initialize", this method is called once per request and it serves
the ORM to initialize a model. In this case, "Products" is initialized by defining that this model
has a one-to-many relationship to another model called "ProductTypes".

.. code-block:: php

    <?php

    $this->belongsTo('product_types_id', 'ProductTypes', 'id', array(
        'reusable' => true
    ));

这意味着,Products中的属性“product_types_id”有一个一对多的关系到"ProductTypes"模型中的"id"属性。通过定义这种关系我们可以访问产品的类型名称使用如下方式:	
	
Which means, the local attribute "product_types_id" in "Products" has an one-to-many relation to
the model "ProductTypes" in its attribute "id". By defining this relation we can access the name of
the product type by using:

.. code-block:: html+jinja

    <td>{{ product.productTypes.name }}</td>

“price”输出使用Volt过滤器格式:	
	
The field "price" is printed by its formatted using a Volt filter:

.. code-block:: html+jinja

    <td>{{ "%.2f"|format(product.price) }}</td>

在PHP实现是:	
	
What in PHP would be:

.. code-block:: php

    <?php echo sprintf("%.2f", $product->price) ?>

是否输出产品信息使用的是模型的辅助实现:	
	
Printing whether the product is active or not uses a helper implemented in the model:

.. code-block:: php

    <td>{{ product.getActiveDetail() }}</td>

该方法在模型中的定义:	
	
This method is defined in the model:



创建和更新记录
^^^^^^^^^^^^^^^^^^
现在让我们看看CRUD是如何创建和更新记录。由用户从“new”和“edit”视图输入的数据,被发送到动作方法“create”和“save”,执行“creating”和“updating”操作产品。

Now let's see how the CRUD creates and updates records. From the "new" and "edit" views the data entered by the user
are sent to the actions "create" and "save" that perform actions of "creating" and "updating" products respectively.

在创建的情况下,我们拿到提交的数据,并将它们分配给一个新的“products”实例:

In the creation case, we recover the data submitted and assign them to a new "products" instance:

.. code-block:: php

    <?php

    /**
     * Creates a new product
     */
    public function createAction()
    {
        if (!$this->request->isPost()) {
            return $this->forward("products/index");
        }

        $form    = new ProductsForm;
        $product = new Products();

        // ...
    }

还记得我们在产品表单中定义的过滤器吗?数据在被分配给$product对象之前进行了过滤。这种过滤是可选的,ORM也转义了输入的数据并根据列类型执行了额外的数据转换:	
	
Remember the filters we defined in the Products form? Data is filtered before being assigned to the object $product.
This filtering is optional, also the ORM escapes the input data and performs additional casting according to the column types:

.. code-block:: php

    <?php

    // ...

    $name = new Text("name");
    $name->setLabel("Name");

    // Filters for name
    $name->setFilters(array('striptags', 'string'));

    // Validators for name
    $name->addValidators(array(
            new PresenceOf(array(
                'message' => 'Name is required'
            ))
    ));

    $this->add($name);

当我们保存数据的时候我们会知道数据是否符合业务规则和在ProductsForm (app/forms/ProductsForm.php)中实现的验证规则:	
	
When saving we'll know whether the data conforms to the business rules and validations implemented
in the form ProductsForm (app/forms/ProductsForm.php):

.. code-block:: php

    <?php

    // ...

    $form    = new ProductsForm;
    $product = new Products();

    // Validate the input
    $data = $this->request->getPost();
    if (!$form->isValid($data, $product)) {
        foreach ($form->getMessages() as $message) {
            $this->flash->error($message);
        }
        return $this->forward('products/new');
    }

最后,如果表单不返回任何验证信息我们可以保存产品实例:	
	
Finally, if the form does not return any validation message we can save the product instance:

.. code-block:: php

    <?php

    // ...

    if ($product->save() == false) {
        foreach ($product->getMessages() as $message) {
            $this->flash->error($message);
        }
        return $this->forward('products/new');
    }

    $form->clear();

    $this->flash->success("Product was created successfully");
    return $this->forward("products/index");

在更新产品的时候,首先我们必须向用户展示当前正在编辑的数据记录:	
	
Now, in the case of product updating, first we must present to the user the data that is currently in the edited record:

.. code-block:: php

    <?php

    /**
     * Edits a product based on its id
     */
    public function editAction($id)
    {

        if (!$this->request->isPost()) {

            $product = Products::findFirstById($id);
            if (!$product) {
                $this->flash->error("Product was not found");
                return $this->forward("products/index");
            }

            $this->view->form = new ProductsForm($product, array('edit' => true));
        }
    }

绑定到表单的数据被为第一个参数传递给模型。由于这一点,用户可以改变任何值,然后通过“save”保存到数据库:	
	
The data found is bound to the form passing the model as first parameter. Thanks to this,
the user can change any value and then sent it back to the database through to the "save" action:

.. code-block:: php

    <?php

    /**
     * Saves current product in screen
     *
     * @param string $id
     */
    public function saveAction()
    {
        if (!$this->request->isPost()) {
            return $this->forward("products/index");
        }

        $id = $this->request->getPost("id", "int");
        $product = Products::findFirstById($id);
        if (!$product) {
            $this->flash->error("Product does not exist");
            return $this->forward("products/index");
        }

        $form = new ProductsForm;

        $data = $this->request->getPost();
        if (!$form->isValid($data, $product)) {
            foreach ($form->getMessages() as $message) {
                $this->flash->error($message);
            }
            return $this->forward('products/new');
        }

        if ($product->save() == false) {
            foreach ($product->getMessages() as $message) {
                $this->flash->error($message);
            }
            return $this->forward('products/new');
        }

        $form->clear();

        $this->flash->success("Product was updated successfully");
        return $this->forward("products/index");
    }
	
我们已经看到使用Phalcon如何创建表单并从数据库中以结构化的方式绑定数据。在下一章,我们将看到如何添加自定义的HTML元素，比如，菜单。
	
We have seen how Phalcon lets you create forms and bind data from a database in a structured way.
In next chapter, we will see how to add custom HTML elements like a menu.

.. _Jinja: http://jinja.pocoo.org/
.. _CRUD: http://en.wikipedia.org/wiki/Create,_read,_update_and_delete
