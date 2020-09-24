# Tutorial: Python + Postgres = <3

## Drivers

We saw that PostgreSQL, like many databases, operates in a client-server architecture.

Often, the client is created and run inside a piece of software, such as your program written in Python.

To do that, you need Python **drivers** for your database. These are packages that act as a client for the database, connecting, sending, and retrieving data, and controlled entirely by Python code!


## Pyscopg2

The most popular **driver** for PostgreSQL in python is a package called `psycopg2`.

As always, be sure to open the documentation and refer to it with any problems:

https://www.psycopg.org/docs/

To connect to your database, you will most likely need to install this package. It can be installed with either the `psycopg2` package or the `psycopg2-binary` package (the former expects local C libraries to connect to Postgres, the later comes with all the necessary libraries already - so use the later if in doubt).

Lets install it with the binary:

``` shell
pip install psycopg2-binary
```

And make sure we can import it into a python file by writing the following in a python file:

``` python
import psycopg2
```

And running the file.

All good? Now we just need to connect.

## Psycopg2 - connect

To connect to the database we use the connect function! This returns an instance of the `connection` class. The instance is commonly called `conn`:

``` python
import os
import psycopg2

password = os.getenv('MY_PASSWORD')
conn = psycopg2.connect(f'postgresql://postgres:{password}@0.0.0.0:5432/foo')
```

So far this looks pretty similar to what we did before! Note a couple of things about dealing with passwords:

1. You should _never_ have your password in your code. The `os.getenv` command gets a variable from the environment, which is where passwords should live. See [secrets.md](secrets.md) for more information on working with passwords and environment variables.

2. As before, if your database doesn't have a password, you just leave off the password: `'postgresql://postgres@0.0.0.0:5432/foo'`


## Psycopg2 - cursor

The `cursor` class in psycopg2 performs the actual work of executing commands. We create a cursor from a connection:

``` python
cur = conn.cursor()
```

Then we execute SQL queries with the `execute` method on the `cursor` class:

``` python
cur.execute('SELECT * FROM bar')
```

After we execute a `SELECT` statement, the cursor will be an iterable that we can iterate over:

``` python
for record in cur:
    print(record)
```

Where each `record` will be a tuple containing the data for one row.

There are a couple of gotchas here:

1. Cursors should be closed after using! Because of that, we will want to use a `with` context manager with them.

2. Any operation that writes or mutates our data is wrapped in a "transaction" (remember those? the building blocks of ACID). To finish a transaction, and make sure the changes persists, we need to "commit" the changes. Commiting is done via the connection instance, but we don't have to do it manually, if we use a `with` context manager with the connection instanct, it will commit for us!

This leads us to usually write in the follow pattern:


```python
with conn:
    with conn.cursor() as cur:

        cur.execute("INSERT INTO test (num, data) VALUES (%s, %s)",
            (100, "abcdef"))

        cur.execute("SELECT * FROM test;")

        for record in cur:
            print(record)
```

Note how we insert data:

``` python
cur.execute("INSERT INTO test (num, data) VALUES (%s, %s)", (100, "abcdef"))
```

This lets psycopg2 create the final SQL query that will be sent to the database. Notice that we are NOT using basic python string formating to turn the data into a string query that can be sent.

**Never insert data into your own SQL query**, always let a driver, like psycopg2, do the inserting for you. This is because it makes sure the data is SAFE and that the strict "abcdef" doesn't secretly contain dangerous commands like "drop database foo" inside of it that will accidentally get executed.

Go ahead and read more about passing values to SQL queries here:

https://www.psycopg.org/docs/usage.html#query-parameters


Now try out the code (make sure to either change the code to work with tables in your database, or create a table that works with the code!)

Be sure to play around with this a bit, you've got all the tools now, and you can try to insert data to your database!

## DB-API

The makers of Python knew that connecting to databases was a thing that would be needed.

So they created a uniform API that all drivers could implement.

This has allowed many other packages to be written that work with a variety of SQL databases (not just PostgreSQL). Personally, I really like the `psycopg2` package, but there are many other packages one can use!

For example, an even simpler package we could use if we want to do simple things:

https://dataset.readthedocs.io/en/latest/

## Using Dataset


```python
import dataset

db = dataset.connect('postgres://postgres@0.0.0.0:5432/foo')
test_table = db['test']

db.insert({'num': 10, 'data': 'foo'})

recs = db.query("SELECT * FROM test;")

for rec in recs:
    print(rec)
```

## Bulk Inserting

We're going to be in the business of moving data around, so often we want to insert a lot of data into our database at once. While we could do this with the previous methods, these libraries provide some more performant options than inserting data one row at a time.


With psycopg2:


```python
from psycopg2.extras import execute_batch

dat = [(2, 'foo'), (5, 'bar')]

with psycopg2.connect('postgres://postgres@0.0.0.0:5432/foo') as conn:
    with conn.cursor() as cur:
        query = "INSERT INTO test(num, data) VALUES (%s, %s)"
        execute_batch(cur, dat)

```

With dataset:


```python
import dataset

db = dataset.connect('postgres://postgres@0.0.0.0:5432/foo')

test_table = db['test']

data = [{'num': 10, 'data': 'foo'}, {'num': 5, 'data': 'bar'}]

test_table.insert_many(data)
```
