# ORM Relationships

## Introduction

![Order UI and Entity Diagram](_media/orm/orm-order-items-ui-prototype.svg)


To set the context let's understand the diagram above. It shows a simplified version of an **order** screen on the left and associated table **entities** on the right. The **order** UI shows information about the associated `order`, `customer`, `order items`, and `payment` details.

### Data Modelling

Although the **order** UI shows all the information together on the screen, when modelling the database schema, you would consider normalized forms with:

* tables named `order`, `customer`, `product`, `order_item`, and `payment`. 
* a foreign key column `customer_id` in the `order` table.
* a foreign key column `order_id` in the `payment` table.
* two foreign key columns `order_id` and `product_id` in the `order_item` table.

> Note: to uniquely identify the order and its related items, we need a table named `order_item` which stores references to the `order_id` and `product_id`. Such a table is often called a **pivot** or **junction** or **bridge** table.

### Entity Associations

When relating data models (**entities**) together, we may think in terms of **associations** or **relationships**. For example:

* A `customer` has many `order`.
* An `order` belongs to one `customer`.
* An `order` has one `payment`.
* A `payment` belongs to one `order`.
* A `product` has many `order`.
* An `order` has many `product`. 

In **Relation Databases**, we introduce 4 main types of associations:

* One to One
* One to Many
* Many to One
* Many to Many

![Association Types between Entities](_media/orm/rdbms-association-types.svg)


### Association Types

#### One to One (1:1)
A **One to One** relationship means that each record in **Table A** is linked to one and only one record in **Table B**, and vice versa. 

Think of it as a **passport** and a **person**: each person has one unique passport, and each passport belongs to one person.

**Order Example:**
- Each `order` has one `payment` record.
- Each `payment` belongs to one `order`.

**In Schema:**
- `payment` table has a unique `order_id` column (foreign key).

#### One to Many (1:N)
A **One to Many** relationship means that a single record in **Table A** can be related to many records in **Table B**, but each record in **Table B** relates back to only one record in **Table A**. 

Imagine a customer placing multiple orders: one customer, many orders.

**Order Example:**
- A `customer` can have many `orders`.
- Each `order` belongs to one `customer`.

**Schema:**
- `order` table has a `customer_id` column (foreign key).

#### Many to One (N:1)
A **Many to One** relationship is simply the inverse of **One to Many**. Many records in **Table A** relate to a single record in **Table B**. 

Think of students and schools: Many students can attend the same school, but each student is enrolled in only one school.

**Order Example:**
- Many `order_item` rows belong to one `order`.

**Schema:**
- `order_item` table has an `order_id` column (foreign key).

#### Many to Many (N:M)
A **Many to Many** relationship means that multiple records in **Table A** can relate to multiple records in **Table B**. This is typically implemented using a **junction** (**pivot**) table. 

Think of products and orders: an order can have many products, and a product can appear in many orders.

**Order Example:**
- An `order` can have many `products` (through `order_item`).
- A `product` can belong to many `orders` (through `order_item`).

**Schema:**
- `order_item` table has both `order_id` and `product_id` columns (foreign keys).

> **Summary:**
> - **One to One:** Each order has one payment.
> - **One to Many:** One customer, many orders.
> - **Many to One:** Many order items, one order.
> - **Many to Many:** Many products in many orders, connected by order items.

## ORM Mapping

Now that you’ve seen how relationships are structured at the database level, let’s translate these concepts into the world of ORMs. While the database focuses on tables, foreign keys, and junction tables, an ORM lets you work with your data as rich, interconnected objects—making your code more expressive, maintainable, and closer to how you think about your domain.

### Relationship Methods

In an ORM, each type of database relationship is represented by a specific method or association on your model classes. Instead of writing SQL joins, you define these relationships once, and then access related data as if you were simply navigating object properties.

#### One to One

Use the relationship method `hasOne()` to define **one to one** relationhsip between **order** and **payment** entity.

```php
class Order extends Model
{
    public function payment()
    {
        return $this->hasOne(Payment::class, 'order_id');
    }
}
```

Now to get the associated payment for an order, simply use the name of the **payment()** method on **Order** instance.

```php
/**
 * Find order with id: 23
 */
$order = new Order(23);

/**
 * Get the associated payment
 */
$order->payment;
```

Behind the scenes, the ORM intercepts the call to `$order->payment` and resolves the associated **Payment** instance.

##### Inverse of hasOne()

Use the relationship method **belongsTo()** to define the inverse of the **hasOne()** relationship.

```php
class Payment extends Model
{
    public function order()
    {
        return $this->belongsTo(Order::class, 'order_id');
    }
}
```

Now this makes it possible to fetch the `Order` instance that the `Payment` belongs to.

```php
/**
 * Find the payment with id: 101
 */
$payment = new Payment(101);

/**
* Get the associated order
*/
$payment->order;
```

#### One to Many
**Database:** One record in Table A links to many in Table B (e.g., `customer` and `orders`).

**ORM Mapping:**

**Customer model (has many orders):**
```php
class Customer extends Model
{
    public function orders()
    {
        return $this->hasMany(Order::class, 'customer_id');
    }
}
```

**Order model (inverse):**
```php
class Order extends Model
{
    public function customer()
    {
        return $this->belongsTo(Customer::class, 'customer_id');
    }
}
```

**Parameter Explanation:**
- `TargetModel::class`: The related model's class name.
- `'customer_id'`: The foreign key column in the related table (`order`).
- `'id'`: The local/owner key (primary key) in the current table (`customer`).

**Usage Example:**
```php
$customer = Customer::find(1);
foreach ($customer->orders as $order) {
    // Work with each order for this customer
}
$order = Order::find(1);
$customer = $order->customer; // Get the customer for this order
```

#### Many to One
**Database:** Many records in Table A link to one in Table B (e.g., `order_item` and `order`).

**ORM Mapping:**

**OrderItem model (belongs to order):**
```php
class OrderItem extends Model
{
    public function order()
    {
        return $this->belongsTo(Order::class, 'order_id', 'id');
    }
}
```

**Order model (inverse):**
```php
class Order extends Model
{
    public function items()
    {
        return $this->hasMany(OrderItem::class, 'order_id', 'id');
    }
}
```

**Parameter Explanation:**
- `TargetModel::class`: The related model's class name.
- `'order_id'`: The foreign key column in the related table (`order_item`).
- `'id'`: The local/owner key (primary key) in the current table (`order`).

**Usage Example:**
```php
$item = OrderItem::find(1);
$order = $item->order; // Get the order this item belongs to
$order = Order::find(1);
foreach ($order->items as $item) {
    // Each item in this order
}
```

#### Many to Many

**Database:** Many records in Table A relate to many in Table B, typically via a pivot/junction table (e.g., `orders` and `products` via `order_item`).

**Order model (many products):**

```php
class Order extends Model
{
    public function products()
    {
        return $this->pivot(Product::class, 'order_item', 'order_id', 'product_id');
    }
}
```

**Product model (inverse):**

```php
class Product extends Model
{
    public function orders()
    {
        return $this->pivot(Order::class, 'order_item', 'product_id', 'order_id');
    }
}
```

**Parameter Explanation:**
- `TargetModel::class`: The related model's class name.
- `'order_item'`: The name of the pivot (junction) table.
- `'order_id'` / `'product_id'`: Foreign key columns in the pivot table.
- `'id'`: The primary key in the parent and related tables.

**Usage Example:**
```php
$order = Order::find(1);
foreach ($order->products as $product) {
    // Each product in this order
}

$product = Product::find(1);
foreach ($product->orders as $order) {
    // Each order that includes this product
}
```

> Many ORMs allow you to omit some parameters if you follow naming conventions, but specifying them explicitly is best for clarity and future-proofing.

---

### ORM Terminology Quick Reference

| Database Concept      | ORM Concept/Method    | Example in Code                |
|----------------------|----------------------|-------------------------------|
| Foreign Key          | `belongsTo`          | `$order->customer`            |
| Primary Key          | `hasOne`, `hasMany`  | `$customer->orders`           |
| Junction Table       | `pivot`/`belongsToMany` | `$order->products`         |
| Related Record       | Property/Collection  | `$order->payment`, `$order->items` |

---

### Why This Matters

- **Productivity:** You can fetch, update, and traverse related data with simple, readable code—no need for manual joins.
- **Clarity:** Your code mirrors your business logic. Relationships are explicit in your models, making the structure of your app easy to understand.
- **Consistency:** ORMs handle the heavy lifting of relationship management, so you don’t have to repeat yourself or risk subtle bugs.

---

In summary, ORMs turn the abstract relationships of your database into natural, intuitive code. By mapping database associations to model methods, you unlock the full power of relational data—without the pain of raw SQL.