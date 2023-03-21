Defining Schema in SQLAlchemy ORM
The following listing defines a Post model which can be used to store authors books info.   

    ``` python
    from sqlalchemy import create_engine, MetaData, Table, Integer, String, \
    Column, DateTime, ForeignKey, Numeric, SmallInteger

    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy.orm import relationship

    from datetime import datetime

    engine = create_engine("postgres+psycopg2://postgres:pass@localhost/sqlalchemy_tuts")

    Base = declarative_base()
    #We can peek at the Table instance associated with the model using the __table__ attribute. (>>>Customer.__table__)
    class Customer(Base):
    __tablename__ = 'customers'
    id = Column(Integer(), primary_key=True)
    first_name = Column(String(100), nullable=False)
    last_name = Column(String(100), nullable=False)
    username = Column(String(50), nullable=False)
    email = Column(String(200), nullable=False)
    created_on = Column(DateTime(), default=datetime.now)
    updated_on = Column(DateTime(), default=datetime.now, onupdate=datetime.now)
    orders = relationship("Order", backref='customer')


    class Item(Base):
    __tablename__ = 'items'
    id = Column(Integer(), primary_key=True)
    name = Column(String(200), nullable=False)
    cost_price =  Column(Numeric(10, 2), nullable=False)
    selling_price = Column(Numeric(10, 2),  nullable=False)
    #     orders = relationship("Order", backref='customer')


    class Order(Base):
    __tablename__ = 'orders'
    id = Column(Integer(), primary_key=True)
    customer_id = Column(Integer(), ForeignKey('customers.id'))
    date_placed = Column(DateTime(), default=datetime.now)
    line_items = relationship("OrderLine", secondary="order_lines", backref='order')


    class OrderLine(Base):
    __tablename__ = 'order_lines'
    id =  Column(Integer(), primary_key=True)
    order_id = Column(Integer(), ForeignKey('orders.id'))
    item_id = Column(Integer(), ForeignKey('items.id'))
    quantity = Column(SmallInteger())
    item = relationship("Item")


    Base.metadata.create_all(engine)
```

For the class to be a valid model, it must do the following:  

Inherit from a declarative base class created by calling declarative_base() function.   
define table name via __tablename__ attribute.    
define at-least one column which must be a part of the primary key.    
The last two points are self-explanatory but the first one deserves a bit of explanation.   

The base class maintains a catalog of classes and tables. In other words, the declarative base class wraps the mapper and the MetaData. The mapper maps the subclass to the table and MetaData holds all the information about the database and the tables it contains. Just as in Core, in ORM we use create_all() and drop_all() methods of the MetaData object to create and drop tables.     

## Creating Session
When using SQLAlchemy ORM, we interact with the database using the Session object. The Session object also wraps the database connection and transaction. The transaction implicitly starts as soon as the Session starts communicating with the database and will remain open until the Session is committed, rolled back or closed.    

One way to create a Session object is to use the Session class from the sqlalchemy.orm package.    
``` python
    from sqlalchemy.orm import create_engine, Session
    engine = create_engine("postgres+psycopg2://postgres:pass@localhost/mydb")
    session = Session(bind=engine)
```

You will have to create the `Session` object everytime you want to communicate with the database.  

The `Session` constructor accepts a number of argument to customize its working. If we choose to create `Session` using this method, we would have to call the `Session` constructor with the same set of parameters over and over again throughout the application.   

To make things easier, SQLAlchemy provides `sessionmaker` class which creates `Session` class with default arguments set for its constructor.   

``` python
    from sqlalchemy.orm import sessionmaker, Session
    Session = sessionmaker(bind=engine)
```

You should call `sessionmaker` once in your application at the global scope   

You should call call `sessionmaker` once in your application at the global scope.   

Once we have access to the custom `Session` class you can instantiate it as many time as you need without passing any arguments to it.   

``` python
session = Session()
```
Note that instantiating Session object doesn't instantly establish connection to the database. The connection is established only when you start sending queries to the database.   

## Inserting Data
To create a new record using SQLAlchemy ORM, we follow these steps:   

* Create an object.
* Add the object to the session.
* Commit the session.

Let's create two new Customer objects as follows:

``` python
c1 = Customer(first_name = 'Toby', 
            last_name = 'Miller', 
            username = 'tmiller', 
            email = 'tmiller@example.com', 
            address = '1662 Kinney Street',
            town = 'Wolfden'
            )

c2 = Customer(first_name = 'Scott', 
            last_name = 'Harvey', 
            username = 'scottharvey', 
            email = 'scottharvey@example.com', 
            address = '424 Patterson Street',
            town = 'Beckinsdale'
            )
c1, c2
```
### Expected Output

``` 
(<Customer:None-johngreen>, <Customer:None-katwilson>)
```
Here we have created two Customer objects. We can access the attributes of an object using the dot(.) operator as follows:   

``` python
c1.first_name, c1.last_name
c2.first_name, c2.last_name
```

Expected Output:  
```
('John', 'Green')
('Katherine', 'Wilson')
```

Instead of adding one object to the session at a time, we can use add_all() method. The add_all() method accepts a list of objects to be added to the session.  

``` python
session.add_all([c1, c2])
```
Adding an object to the session multiple times doesn't throw any errors. At any time, you can view the objects added to the session using `session.new`.

```
session.new
```
## Expected Output:

```
IdentitySet([<Customer:None-johngreen>, <Customer:None-katwilson>])
```
Finally, to save the objects to the database call `commit()` method as follows:   

```
session.commit()
```
Once you commit the transaction, the connection resources referenced by the `Session` object is returned to the connection pool. Subsequent operations will occur in a new transaction.   

Accessing the `id` attribute of the Customer object will now return the primary key instead of `None`.

```
c1.id, c2.id
```
## Expected Output:
```
(1, 2)
```

Let's add some more customers to the customers table:

``` python
c3 = Customer(
            first_name = "John", 
            last_name = "Lara", 
            username = "johnlara", 
            email = "johnlara@mail.com", 
            address = "3073 Derek Drive",
            town = "Norfolk"
)

c4 = Customer(          
            first_name = "Sarah", 
            last_name = "Tomlin", 
            username = "sarahtomlin", 
            email = "sarahtomlin@mail.com",
            address = "3572 Poplar Avenue",
            town = "Norfolk"        
)

c5 = Customer(first_name = 'Toby', 
              last_name = 'Miller', 
              username = 'tmiller', 
              email = 'tmiller@example.com', 
              address = '1662 Kinney Street',
              town = 'Wolfden'
             )

c6 = Customer(first_name = 'Scott', 
              last_name = 'Harvey', 
              username = 'scottharvey', 
              email = 'scottharvey@example.com', 
              address = '424 Patterson Street',
              town = 'Beckinsdale'
             )

session.add_all([c3, c4, c5, c6])
session.commit()
```
Before we can take orders, let's add some products to the items table.   

``` python
i1 = Item(name = 'Chair', cost_price = 9.21, selling_price = 10.81, quantity = 5)
i2 = Item(name = 'Pen', cost_price = 3.45, selling_price = 4.51, quantity = 3)
i3 = Item(name = 'Headphone', cost_price = 15.52, selling_price = 16.81, quantity = 50)
i4 = Item(name = 'Travel Bag', cost_price = 20.1, selling_price = 24.21, quantity = 50)
i5 = Item(name = 'Keyboard', cost_price = 20.1, selling_price = 22.11, quantity = 50)
i6 = Item(name = 'Monitor', cost_price = 200.14, selling_price = 212.89, quantity = 50)
i7 = Item(name = 'Watch', cost_price = 100.58, selling_price = 104.41, quantity = 50)
i8 = Item(name = 'Water Bottle', cost_price = 20.89, selling_price = 25, quantity = 50)

session.add_all([i1, i2, i3, i4, i5, i6, i7, i8])
session.commit()
```
Create some orders now:  

``` python
o1 = Order(customer = c1)
o2 = Order(customer = c1)

line_item1 = OrderLine(order = o1, item = i1, quantity =  3)
line_item2 = OrderLine(order = o1, item = i2, quantity =  2)
line_item3 = OrderLine(order = o2, item = i1, quantity =  1)
line_item3 = OrderLine(order = o2, item = i2, quantity =  4)

session.add_all([o1, o2])

session.new
session.commit()
```

# Querying Data
To query database we use the `query()` method of the session object. The `query()` method returns an object of type `sqlalchemy.orm.query.Query`, simply called `Query`. The `Query` object represents the `SELECT` statement that will be used to query the database. The following table lists some common methods of the `Query` class.

-------------------------
|Method   | Description |
-------------------------
|`all()` | returns the result of the query (represented by Query) as a list.|
--------------------------
|`count()` |  returns the total number of records in the query.|
--------------------------
|`first()`  | returns the first result of the query or `None`, if there are no rows in the result.|
---------------------------
|scalar()	|returns the first column of the first row or `None` if the result set is empty. If multiple rows are encountered it throws `MultipleResultsFound` exception.|
---------------------------
|`one`	|returns exactly only row. If it encounters multiple rows it throws `MultipleResultsFound` exception. If the result set is empty it throws `NoResultFound` exception.|
---------------------------------
|`get(pk)`	| returns an object that matches the given primary key (pk), or None, if no such object is found. |
-----------------------------------
|`filter(*criterion)`	| returns a new Query instance after applying the WHERE clause to the query.|
------------------------------
|`limit(limit)`	| return a new Query instance after applying the LIMIT clause to the query.|
----------------------------
|`offset(offset)`	| return a new Query instance after applying the OFFSET clause to the query. |
----------------------------------
|`order_by(*criterion)`	| return a new Query instance after applying ORDER BY clause to the query.|
---------------------------
|`join(*props, **kwargs)`	| return a new Query instance after creating SQL INNER JOIN on the query.|
-------------------------------
|`outerjoin(*props, **kwargs)`	| return a new Query instance after creating SQL LEFT OUTER JOIN on the query.|
------------------------------
|`group_by(*criterion)`	|return a new Query instance after adding GROUP BY clause to the query.|
--------------------------------
|`having(criterion)`	| return a new Query instance after adding HAVING clause to the query.|
----------------------------------

# all() methond
```
session.query(Customer).all()
```

Calling all() method on a large result set is inefficient instead we can use a for loop to iterate over the Query object as follows::

``` python
q = session.query(Customer)

for c in q:
    print(c.id, c.first_name)
```

The preceding queries have returned data from all columns of the table. We can prevent this by passing the column names explicitly to the query() method as follows:   
``` python
session.query(Customer.id, Customer.first_name).all()
```
## Expected Output:
```
[(1, 'John'),
 (2, 'Katherine'),
 (3, 'John'),
 (4, 'Sarah'),
 (5, 'Toby'),
 (6, 'Scott')]
```
Notice that now each item in the list is a tuple instead of a model instance.