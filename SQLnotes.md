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



