# Relationships

![Order UI and Entity Diagram](_media/orm/orm-order-items-ui-prototype.svg)


To set the context let's understand the diagram above. It shows a simplified version of an **order** screen on the left and associated table **entities** on the right. The **order** UI shows information about the associated `order`, `customer`, `order items`, and `payment` details.

## Data Modelling

Although the **order** UI shows all the information together on the screen, when modelling the database schema, you would consider normalized forms with:

* tables named `order`, `customer`, `product`, `order_item`, and `payment`. 
* a foreign key column `customer_id` in the `order` table.
* a foreign key column `order_id` in the `payment` table.
* two foreign key columns `order_id` and `product_id` in the `order_item` table.

> Note: to uniquely identify the order and its related items, we need a table named `order_item` which stores references to the `order_id` and `product_id`. Such a table is often called a **pivot** or **junction** or **bridge** table.

## Entity Associations

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


## Association Types

### 1. One to One (1:1)
A **One to One** relationship means that each record in `Table A` is linked to one and only one record in `Table B`, and vice versa. 

Think of it as a **passport** and a **person**: each person has one unique passport, and each passport belongs to one person.

**Order Example:**
- Each `order` has one `payment` record.
- Each `payment` belongs to one `order`.

**In Schema:**
- `payment` table has a unique `order_id` column (foreign key).

**ORM Code:**
```php
// In Order model
public function payment() {
    return $this->hasOne(Payment::class);
}
// In Payment model
public function order() {
    return $this->belongsTo(Order::class);
}
```

---

### 2. One to Many (1:N)
A **One to Many** relationship means that a single record in `Table A` can be related to many records in `Table B`, but each record in `Table B` relates back to only one record in `Table A`. 

Imagine a customer placing multiple orders: one customer, many orders.

**Order Example:**
- A `customer` can have many `orders`.
- Each `order` belongs to one `customer`.

**Schema:**
- `order` table has a `customer_id` column (foreign key).

**ORM Code:**
```php
// In Customer model
public function orders() {
    return $this->hasMany(Order::class);
}
// In Order model
public function customer() {
    return $this->belongsTo(Customer::class);
}
```

---

### 3. Many to One (N:1)
A **Many to One** relationship is simply the inverse of **One to Many**. Many records in `Table A` relate to a single record in `Table B`. 

For example, many `order_item` records point to one `order`.

**Order Example:**
- Many `order_item` rows belong to one `order`.

**Schema:**
- `order_item` table has an `order_id` column (foreign key).

**ORM Code:**
```php
// In OrderItem model
public function order() {
    return $this->belongsTo(Order::class);
}
// In Order model
public function items() {
    return $this->hasMany(OrderItem::class);
}
```

---

### 4. Many to Many (N:M)
A **Many to Many** relationship means that multiple records in `Table A` can relate to multiple records in `Table B`. This is typically implemented using a **junction** (**pivot**) table. 

Think of products and orders: an order can have many products, and a product can appear in many orders.

**Order Example:**
- An `order` can have many `products` (through `order_item`).
- A `product` can belong to many `orders` (through `order_item`).

**Schema:**
- `order_item` table has both `order_id` and `product_id` columns (foreign keys).

**ORM Code:**
```php
// In Order model
public function products() {
    return $this->pivot(Product::class, 'order_item');
}
// In Product model
public function orders() {
    return $this->pivot(Order::class, 'order_item');
}
```

> **Summary:**
> - **One to One:** Each order has one payment.
> - **One to Many:** One customer, many orders.
> - **Many to One:** Many order items, one order.
> - **Many to Many:** Many products in many orders, connected by order items.