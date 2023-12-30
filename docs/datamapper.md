# Data Mapper
- - -

!!! info "NOTE"

    These components have been heavily influenced by [Aura PHP][auraphp] and [Atlas PHP][atlasphp] 

!!! warning "NOTE"

    The full implementation of a DataMapper is not yet complete. There are however a few components that can be used in any project, such as the `Connection` and `Query/Select` 

## Overview

The Data Mapper pattern as described by [Martin Fowler][datamapper] in [Patterns of Enterprise Application Architecture][eaa] is:

!!! info "NOTE"

    A layer of Mappers that moves data between objects and a database while keeping them independent of each other and the mapper itself.

The `Phalcon\DataMapper` namespace contains components to help with accessing your data source, with the [Data Mapper][datamapper].

## PDO

### Connection

One of the components required by this implementation is a PDO connector. The [Phalcon\DataMapper\Pdo\Connection][datamapper-pdo-connection] offers a wrapper to PHP's PDO implementation, making it easier to maintain connections.

**Connecting to a source**

Connecting to a database requires the DSN string as well as the username and the password of the account with permission to access the database we need to connect to.

The DSN is as follows:

|   Engine    | DSN                                                                      |
|:-----------:|--------------------------------------------------------------------------|
|    Mysql    | `mysql:host=<host>;dbname=<database name>;charset=<charset>;port=<port>` |
| Postgresql  | `pgsql:host=<host>;dbname=<database name>`                               |
|   Sqlite    | `sqlite:<file>`                                                          |

You will only need to substitute the values in `<>` with the respective values for your environment. The `charset` and `port` are optional for `Mysql`. For `Sqlite` you can use `memory` as the `<file>` but the database will not persist. A file name in an appropriate location will create the necessary storage file for `Sqlite`.

```php
<?php

use Phalcon\DataMapper\Connection;

$host     = '127.0.0.1';
$database = 'phalon_test';
$charset  = 'utf8mb4';
$port     = 3306;
$username = 'phalcon';
$password = 'secret';

$dsn = sprintf(
    "mysql:host=%s;dbname=%s;charset=%s;port=%s",
    $host,
    $database,
    $charset,
    $port
);

$connection = new Connection($dsn, $username, $password);

$sql = '
    SELECT 
        inv_id, 
        inv_title 
    FROM 
        co_invoices 
    WHERE 
        inv_cst_id = :cst_id
';

$bind = [
    'cst_id' => 1
];

$result = $connection->fetchAll($statement, $bind);
```

#### Methods

```php
public function __construct(
    string $dsn,
    string $username = null,
    string $password = null,
    array $options = [],
    array $queries = [],
    ProfilerInterface $profiler = null
)
```
Constructs the object. The `$dsn`, `$username` and `$password` are used to connect to the source. The `$options` allows for additional `PDO` options to be specified. The `$queries` array contains a list of queries that will be executed when the connection is established. The `$profiler` is an optional object implementing the `ProfilerInterface` interface, used to profile the connection.

```php
public function __debugInfo():  array
```
The purpose of this method is to hide sensitive data from stack traces (such as usernames, passwords).

```php
public function beginTransaction(): bool
```
Begins a transaction. If the profiler is enabled, the operation will be recorded.

```php
public function commit(): bool
```
Commits the existing transaction. If the profiler is enabled, the operation will be recorded.

```php
abstract public function connect(): void;
```
Connects to the database.

```php
abstract public function disconnect(): void;
```
Disconnects from the database.

```php
public function errorCode(): string | null
```
Gets the most recent error code.

```php
public function errorInfo(): array
```
Gets the most recent error info.

```php
public function exec(string $statement): int
```
Executes an SQL statement and returns the number of affected rows. If the profiler is enabled, the operation will be recorded.

```php
public function fetchAffected(string $statement, array $values = []): int
```
Performs a statement and returns the number of affected rows.

```php
public function fetchAll(string $statement, array $values = []): array
```
Fetches a sequential array of rows from the database; the rows are returned as associative arrays.

```php
public function fetchAssoc(string $statement, array $values = []): array
```
Fetches an associative array of rows from the database; the rows are returned as associative arrays, and the array of rows is keyed on the first column of each row.

If multiple rows have the same first column value, the last row with that value will overwrite earlier rows. This method is more resource intensive and should be avoided if possible.

```php
public function fetchColumn(
    string $statement,
    array $values = [],
    int $column = 0
): array
```
Fetches a column of rows as a sequential array (default first one).

```php
public function fetchGroup(
    string $statement,
    array $values = [],
    int $flags = \PDO::FETCH_ASSOC
): array 
```
Fetches multiple from the database as an associative array. The first column will be the index key. The default flags are `PDO::FETCH_ASSOC` | `PDO::FETCH_GROUP`

```php
public function fetchObject(
    string $statement,
    array $values = [],
    string $className = "stdClass",
    array $arguments = []
): object 
```
Fetches one row from the database as an object where the column values are mapped to object properties.

Since PDO injects property values before invoking the constructor, any initializations for defaults that you potentially have in your object's constructor, will override the values that have been injected by `fetchObject`. The default object returned is `\stdClass`

```php
public function fetchObjects(
    string $statement,
    array $values = [],
    string $className = "stdClass",
    array $arguments = []
): array {
```
Fetches a sequential array of rows from the database; the rows are returned as objects where the column values are mapped to object properties.

Since PDO injects property values before invoking the constructor, any initializations for defaults that you potentially have in your object's constructor, will override the values that have been injected by `fetchObject`. The default object returned is `\stdClass`

```php
public function fetchOne(string $statement, array $values = []): array
```
Fetches one row from the database as an associative array.

```php
public function fetchPairs(string $statement, array $values = []): array
```
Fetches an associative array of rows as key-value pairs (first column is the key, second column is the value).

```php
public function fetchValue(string $statement, array $values = [])
```
Fetches the very first value (i.e., first column of the first row).

```php
public function getAdapter(): \PDO
```
Return the inner PDO (if any)

```php
public function getAttribute(int $attribute): var
```
Retrieve a database connection attribute

```php
public static function getAvailableDrivers(): array
```
Return an array of available PDO drivers (empty array if none available)

```php
public function getDriverName(): string
```
Return the driver name

```php
public function getProfiler(): <ProfilerInterface>
```
Returns the Profiler instance.

```php
public function getQuoteNames(string $driver = ""): array
```
Gets the quote parameters based on the driver

```php
public function inTransaction(): bool
```
Is a transaction currently active? If the profiler is enabled, the operation will be recorded. If the profiler is enabled, the operation will be recorded.

```php
public function isConnected(): bool
```
Is the PDO connection active?

```php
public function lastInsertId(string $name = null): string
```
Returns the last inserted autoincrement sequence value. If the profiler is enabled, the operation will be recorded.

```php
public function perform(
    string $statement,
    array $values = []
): \PDOStatement
```
Performs a query with bound values and returns the resulting PDOStatement; array $values will be passed through `quote()` and their respective placeholders will be replaced in the query string. If the profiler is enabled, the operation will be recorded.

```php
public function prepare(
    string $statement,
    array $options = []
): \PDOStatement
```
Prepares an SQL statement for execution.

```php
public function query(string $statement, ...$fetch): <\PDOStatement> | bool
```
Queries the database and returns a PDOStatement. If the profiler is enabled, the operation will be recorded.

```php
public function quote(mixed $value, int $type = \PDO::PARAM_STR): string
```
Quotes a value for use in an SQL statement. This differs from `PDO::quote()` in that it will convert an array into a string of comma-separated quoted values. The default type is `PDO::PARAM_STR`

```php
public function rollBack(): bool
```
Rolls back the current transaction, and restores autocommit mode. If the profiler is enabled, the operation will be recorded.

```php
public function setAttribute(int $attribute, mixed $value): bool
```
Set a database connection attribute

```php
public function setProfiler(ProfilerInterface $profiler)
```
Sets the Profiler instance.

```php
protected function fetchData(
    string $method,
    array $arguments,
    string $statement,
    array $values = []
): array
```
Helper method to get data from PDO based on the method passed

```php
protected function performBind(
    \PDOStatement $statement,
    mixed $name,
    mixed $arguments
): void
```
Bind a value using the proper `PDO::PARAM_*` type.

### Connection - Decorated

### ConnectionLocator
Applications with high traffic may utilize multiple database servers. For instance, one could employ a high-powered database server for writes, while smaller ones with memory based tables for reads. 

The [Phalcon\DataMapper\ConnectionLocator][datamapper-pdo-connectionlocator] allows you to define multiple [Phalcon\DataMapper\Pdo\Connection][datamapper-pdo-connection] objects for reading and writing. All these objects are lazy-loaded, instantiated only when necessary. 

#### Instantiation
The easiest way to create a [Phalcon\DataMapper\ConnectionLocator][datamapper-pdo-connectionlocator] to instantiate it and pass a [Phalcon\DataMapper\Pdo\Connection][datamapper-pdo-connection] object to it. Additionally, the constructor can optionally receive two arrays, one for the write connections and one for the read connections. The first connection is always the `master` one.

```php

$host     = '127.0.0.1';
$database = 'phalon_test';
$charset  = 'utf8mb4';
$port     = 3306;
$username = 'phalcon';
$password = 'secret';

$dsn = sprintf(
    "mysql:host=%s;dbname=%s;charset=%s;port=%s",
    $host,
    $database,
    $charset,
    $port
);

$connection = new Connection($dsn, $username, $password);

$locator = new ConnectionLocator($connection);

```

#### Methods

```php
public function __construct(
    ConnectionInterface $master,
    array $read = [],
    array $write = []
)
```
Constructor.

```php
public function getMaster():  ConnectionInterface
```
Returns the default connection object.

```php
public function getRead(string $name = ""):  ConnectionInterface
```
Returns a read connection by name; if no name is given, picks a random connection; if no read connections are present, returns the default connection.

```php
public function getWrite(string $name = ""):  ConnectionInterface
```
Returns a write connection by name; if no name is given, picks a random connection; if no write connections are present, returns the default connection.

```php
public function setMaster(ConnectionInterface $callableObject):  ConnectionLocatorInterface
```
Sets the default connection factory.

```php
public function setRead(
    string $name,
    callable $callableObject
):  ConnectionLocatorInterface
```
Sets a read connection factory by name.

```php
public function setWrite(
    string $name,
    callable $callableObject
): ConnectionLocatorInterface
```
Sets a write connection factory by name.

```php
protected function getConnection(
    string $type,
    string $name = ""
):  ConnectionInterface
```
Returns a connection by name.

#### Configuration
Once the [Phalcon\DataMapper\ConnectionLocator][datamapper-pdo-connectionlocator] is created, you can add as many additional read or write connections as required. You can do so either during the construction of the locator or at runtime.

##### Runtime
First, you create the [Phalcon\DataMapper\ConnectionLocator][datamapper-pdo-connectionlocator] object with the master connection. The master connection is the connection that will be used when read or write connections are not defined.

```php
<?php

$locator = new ConnectionLocator(
    function () use ($options) {
        return new Connection(
            'mysql:host=10.4.6.1;dbname=phalcon_db;charset=utf8mb4;port=3306',
            'username', 
            'password'
        );
    }
);
```

Now you can add as many read and write servers as required
```php
<?php

// Write: master
$locator->addRead(
    'master',
    function () {
        return new Connection(
            'mysql:host=10.4.4.1;dbname=phalcon_db;charset=utf8mb4;port=3306',
            'username', 
            'password'
        );
    }
);

// Read: slave01
$locator->addRead(
    'slave01',
    function () {
        return new Connection(
            'mysql:host=10.4.8.1;dbname=phalcon_db;charset=utf8mb4;port=3306',
            'username', 
            'password'
        );
    }
);

// Read: slave02
$locator->addRead(
    'slave02',
    function () {
        return new Connection(
            'mysql:host=10.4.8.2;dbname=phalcon_db;charset=utf8mb4;port=3306',
            'username', 
            'password'
        );
    }
);

// Read: slave03
$locator->addRead(
    'slave03',
    function () {
        return new Connection(
            'mysql:host=10.4.8.3;dbname=phalcon_db;charset=utf8mb4;port=3306',
            'username', 
            'password'
        );
    }
);
```

##### On construction
You can also set everything up when the locator is being constructed. This is particularly useful when setting up the locator as a service in a DI container.

```php
<?php

// Set up write connections
$write = [
    'master' => function () {
        return new Connection(
            'mysql:host=10.4.4.1;dbname=phalcon_db;charset=utf8mb4;port=3306',
            'username', 
            'password'
        );
    }
];

// Set up read connections
$read = [
    'slave01' => function () {
        return new Connection(
            'mysql:host=10.4.8.1;dbname=phalcon_db;charset=utf8mb4;port=3306',
            'username', 
            'password'
        );
    },
    'slave02' => function () {
        return new Connection(
            'mysql:host=10.4.8.2;dbname=phalcon_db;charset=utf8mb4;port=3306',
            'username', 
            'password'
        );
    },
    'slave03' => function () {
        return new Connection(
            'mysql:host=10.4.8.3;dbname=phalcon_db;charset=utf8mb4;port=3306',
            'username', 
            'password'
        );
    }
];

$locator = new ConnectionLocator(
    function () use ($options) {
        return new Connection(
            'mysql:host=10.4.6.1;dbname=phalcon_db;charset=utf8mb4;port=3306',
            'username', 
            'password'
        );
    },
    $read,
    $write
);
```


#### Getting Connections
Getting a connection from the locator will instantiate the object if it is not instantiated yet and then return it.

- `getMaster()` will return the master/default [Phalcon\DataMapper\Pdo\Connection][datamapper-pdo-connection].
- `getRead()` will return a random read [Phalcon\DataMapper\Pdo\Connection][datamapper-pdo-connection]; after the first call, `getRead()` will always return the same [Phalcon\DataMapper\Pdo\Connection][datamapper-pdo-connection]. (If no read Connections are defined, it will return the default connection.)
- `getWrite()` will return a random write [Phalcon\DataMapper\Pdo\Connection][datamapper-pdo-connection]; after the first call, `getWrite()` will always return the same [Phalcon\DataMapper\Pdo\Connection][datamapper-pdo-connection]. (If no write Connections are defined, it will return the default connection.)

You can retrieve a specific read or write connection by passing its name (as it was registered), to the `getRead()` or `getWrite()` methods.

### Profiler

The [Phalcon\DataMapper\Profiler\Profiler][datamapper-pdo-profiler-profiler] is a component that allows you to profile database connections. That entails logging which queries have been executed and where they came from in the codebase, as well as what their execution time is. The [Phalcon\DataMapper\Profiler\Profiler][datamapper-pdo-profiler-profiler] accepts a [Phalcon\Logger\Logger][logger] object to log all the information collected to a file. By default, the [Phalcon\DataMapper\Profiler\MemoryLogger][datamapper-pdo-profiler-memorylogger] is used.

The [Phalcon\DataMapper\Profiler\Profiler][datamapper-pdo-profiler-profiler] can be activated by calling the `setActive()` method. The method accepts a boolean flag, which serves also as the deactivation method. Data is only logged when the profiler is active.

```php
<?php

use Phalcon\DataMapper\Connection;
use Phalcon\DataMapper\Profiler\MemoryLogger;
use Phalcon\DataMapper\Profiler\Profiler;

$host     = '127.0.0.1';
$database = 'phalon_test';
$charset  = 'utf8mb4';
$port     = 3306;
$username = 'phalcon';
$password = 'secret';

$dsn = sprintf(
    "mysql:host=%s;dbname=%s;charset=%s;port=%s",
    $host,
    $database,
    $charset,
    $port
);

$profiler   = new Profiler(new MemoryLogger());
$connection = new Connection(
    $dsn, 
    $username, 
    $password,
    [
        PDO::ATTR_EMULATE_PREPARES => true, // PDO options
    ],
    [
        'SET NAMES utf8mb4', // startup queries
    ],
    $profiler
);

// Same profiler as the one we created above
$profiler = $connection->getProfiler();
$profiler->setActive(true)
```
and to retrieve the data stored:

```php
<?php

$data = $connection->getProfiler()->getLogger()->getMessages();

var_dump($messages);
```

The messages are logged by default according to this pattern:

```text
"{method} ({duration}s): {statement} {backtrace}"
```

You can customize the message format using the `setLogFormat()` on the profiler

```php
<?php

$connection
    ->getProfiler()
    ->setLogFormat("{duration}: {method} {statement}{values}")
```

The parameters available are:

|   Parameter   | Description                                   |
|:-------------:|-----------------------------------------------|
| `{backtrace}` | The backtrace of where the query was executed |
| `{duration}`  | The execution duration for the query          |
|  `{finish}`   | The microtime when the profile finished       |
|  `{method}`   | The method that was called the connection     |
|   `{start}`   | The microtime when the profile began          |
| `{statement}` | The query executed                            |
|  `{values}`   | Any values passed to the query                |

!!! info "NOTE"

    The parameters must be enclosed in curly brackets `{}`

## Query
### Factory
### Delete
### Insert
### Select

#### Activation
To instantiate a `Phalcon\DataMapper\Query\Select` builder, you can use the `Phalcon\DataMapper\Query\QueryFactory` with a `Phalcon\DataMapper\Connection`.

```php
<?php

use Phalcon\DataMapper\Connection;
use Phalcon\DataMapper\Query\QueryFactory;

$host     = '127.0.0.1';
$database = 'phalon_test';
$charset  = 'utf8mb4';
$port     = 3306;
$username = 'phalcon';
$password = 'secret';

$dsn = sprintf(
    "mysql:host=%s;dbname=%s;charset=%s;port=%s",
    $host,
    $database,
    $charset,
    $port
);

$connection = new Connection($dsn, $username, $password);
$factory    = new QueryFactory();
$select     = $factory->newSelect($connection);
```

#### Build

##### Columns

To add columns to the Select, use the `columns()` method and pass the columns as an array.  If a key is defined as a string, it will be used as an alias for the column.

**Column Names**

```php
<?php

$columns = [
    'inv_id', 
    'inv_cst_id', 
    'inv_status_flag', 
    'inv_title', 
    'inv_total', 
    'inv_created_at',
];

$select->columns($columns);

// SELECT
//      inv_id,
//      inv_cst_id,
//      inv_status_flag,
//      inv_title,
//      inv_total,
//      inv_created_at
```

**Aliases**

```php
<?php

$columns = [
    'id'         => 'inv_id', 
    'customerId' => 'inv_cst_id', 
    'status'     => 'inv_status_flag', 
    'title'      => 'inv_title', 
    'total'      => 'inv_total', 
    'createdAt'  => 'inv_created_at',
];

$select->columns($columns);

// SELECT 
//      id, 
//      customerId, 
//      status, 
//      title, 
//      total, 
//      createdAt
```

**Count**

```php
<?php

$columns = [
    'customerId' => 'inv_cst_id', 
    'totalCount' => 'COUNT(inv_total)'
];

$select->columns($columns);

// SELECT 
//      customerId, 
//      COUNT(inv_total) AS totalCount
```

##### `FROM`

To add a FROM clause, use the `from()` method:

**Direct**

```php
<?php

$select
    ->from('co_invoices')
;

// SELECT * FROM co_invoices
```

**Alias**

```php
<?php

$select
    ->from('co_invoices AS i')
;

// SELECT * FROM co_invoices i
```

##### `JOIN`
To add a JOIN clause, use the join() method:

**LEFT**

```php
<?php

$select
    ->from('co_invoices')
    ->join($select::JOIN_LEFT, 'co_customers', 'inv_cst_id = cst_id')
;

// SELECT * FROM co_invoices 
//  LEFT JOIN co_customers ON inv_cst_id = cst_id
```

**RIGHT**

```php
<?php

$select
    ->from('co_invoices')
    ->join($select::JOIN_RIGHT, 'co_customers', 'inv_cst_id = cst_id')
;

// SELECT * FROM co_invoices 
//  RIGHT JOIN co_customers ON inv_cst_id = cst_id
```

**INNER**

```php
<?php

$select
    ->from('co_invoices')
    ->join($select::JOIN_INNER, 'co_customers', 'inv_cst_id = cst_id')
;

// SELECT * FROM co_invoices 
//  INNER JOIN co_customers ON inv_cst_id = cst_id
```

**NATURAL**

```php
<?php

$select
    ->from('co_invoices AS i')
    ->join($select::JOIN_NATURAL, 'co_customers', 'inv_cst_id = cst_id')
;

// SELECT * FROM co_invoices 
//  NATURAL JOIN co_customers ON inv_cst_id = cst_id
```

**With Bind**

```php
<?php

$status = 1;
$select
    ->from('co_invoices')
    ->join(
        $select::JOIN_LEFT, 
        'co_customers', 
        'inv_cst_id = cst_id AND cst_status_flag = ',
        $status
    )
    ->appendJoin(' AND cst_name LIKE ', '%john%')
;

// SELECT * FROM co_invoices 
//  LEFT JOIN co_customers ON inv_cst_id = cst_id 
//      AND cst_status_flag = :__1__
//      AND cst_name LIKE :__2__
```

##### WHERE

To add WHERE conditions, use the where() method. Additional calls to `where()` will implicitly `AND` the subsequent condition.

**Single**

```php
<?php

$invoiceId = 1;
$select
    ->from('co_invoices')
    ->where('inv_id > ', $invoiceId)
;

// SELECT * FROM co_invoices 
//  WHERE inv_id > :__1__
```

**`andWhere`**

```php
<?php

$customerIds = [1, 2, 3];
$status      = 1;
$totalValue  = 100;
$select
    ->from('co_invoices')
    ->where('inv_id > 1')
    ->andWhere('inv_total > :total')
    ->andWhere('inv_cst_id IN ', $customerIds)
    ->appendWhere(' AND inv_status_flag = ' . $select->bindInline($status))
    ->bindValue('total', $totalValue)
;

// SELECT * FROM co_invoices 
//  WHERE inv_id > 1
//      AND inv_total > :total 
//      AND inv_cst_id IN (:__1__, :__2__, :__3__) 
//      AND inv_status_flag = :__4__
```

**`orWhere`**

```php
<?php

$status      = 1;
$totalValue  = 100;
$select
    ->from('co_invoices')
    ->appendWhere('inv_total > ', $totalValue)
    ->orWhere("inv_status_flag = :status")
    ->bindValue('status', $status)
;

// SELECT * FROM co_invoices 
//  WHERE inv_total > :__1__ "
//      OR inv_status_flag = :status
```

**`whereEquals`**

There is an additional `whereEquals()` convenience method that adds a series of `AND` equality conditions for you based on an array of key-value pairs:

- Given an array value, the condition will be `IN ()`.
- Given an empty array, the condition will be `FALSE` (which means the query will return no results).
- Given a `null` value, the condition will be `IS NULL`.
- For all other values, the condition will be `=`.
- If you pass a key without a value, that key will be used as a raw unescaped condition.

```php
<?php

$invoiceIds = [1, 2, 3];
$select
    ->from('co_invoices')
    ->whereEquals(
        [
            'inv_id'     => $invoiceIds,
            'inv_cst_id' => null,
            'inv_title'  => 'ACME',
            'inv_created_at = NOW()',
        ]
    )
;

// SELECT * FROM co_invoices 
//  WHERE inv_id IN (:__1__, :__2__, :__3__)
//      AND inv_cst_id IS NULL 
//      AND inv_title = :__4__ 
//      AND inv_created_at = NOW()
```

##### `GROUP BY`
To add `GROUP BY` expressions, use the `groupBy()` method and pass each expression as a variadic argument.

```php
<?php

$select
    ->from('co_invoices')
    ->groupBy('inv_cst_id')
    ->groupBy('inv_status_flag')
;

// SELECT * FROM co_invoices 
//  GROUP BY inv_cst_id, inv_status_flag
```

##### `HAVING`

The `HAVING` methods work just like their equivalent `WHERE` methods:

- `having()` and `andHaving()` `AND` a `HAVING` condition
- `orHaving()` `OR`s a `HAVING` condition
- `appendHaving()` concatenates onto the end of the most recent `HAVING` condition

##### `ORDER BY`

To add `ORDER BY` expressions, use the `orderBy()` method and pass each expression an element of an array.

```php
<?php

$select
    ->from('co_invoices')
    ->orderBy(
        [
            'inv_cst_id',
            'UPPER(inv_title) DESC',
        ]
    )
;

// SELECT * FROM co_invoices 
//  ORDER BY inv_cst_id, UPPER(inv_title) DESC
```

##### `LIMIT`, `OFFSET`, Pagination

To set a `LIMIT` and `OFFSET`, use the `limit()` and `offset()` methods.

```php
<?php

$select
    ->from('co_invoices')
    ->limit(10)
;

// SELECT * FROM co_invoices 
//  LIMIT 10

$select
    ->from('co_invoices')
    ->limit(10)
    ->offset(50)
;

// SELECT * FROM co_invoices 
//  LIMIT 10 OFFSET 50
```

**Pagination**

Alternatively, you can limit by "pages" using the `page()` and `perPage()` methods:

```php
<?php

$select
    ->from('co_invoices')
    ->page(5)
    ->perPage(10)
;

// SELECT * FROM co_invoices 
//  LIMIT 10 OFFSET 5
```

##### `DISTINCT`
You can set the `DISTINCT` clause as follows:

```php
<?php

$select
    ->distinct()
    ->from('co_invoices')
    ->columns(
        [
            'inv_id', 
            'inc_cst_id'
        ]
    )
;

// SELECT DISTINCT inv_id, inc_cst_id
// FROM co_invoices
```
!!! info "NOTE"

    The method accepts an optional boolean parameter to enable (`true`) or disable (`false`) the flag.

##### `FOR UPDATE`
You can set the `FOR UPDATE` clause as follows:

```php
<?php

$select
    ->from('co_invoices')
    ->forUpdate()
;

// SELECT * FROM co_invoices FOR UPDATE

$select
    ->from('co_invoices')
    ->forUpdate()
    ->forUpdate(false)
;

// SELECT * FROM co_invoices
```

!!! info "NOTE"

    The method accepts an optional boolean parameter to enable (`true`) or disable (`false`) the flag.

##### Additional Flags

You can set flags recognized by your database server using the `setFlag()` method. For example, you can set a MySQL `HIGH_PRIORITY` flag like so:

```php
<?php

$select
    ->from('co_invoices')
    ->setFlag('HIGH_PRIORITY')
;

// SELECT HIGH_PRIORITY * FROM co_invoices
```

##### `UNION`
To `UNION` or `UNION ALL` the current `Select` with a followup statement, call one the `union*()` methods:

```php
<?php

$select
    ->from('co_invoices')
    ->where('inv_id = 1')
    ->union()
    ->from('co_invoices')
    ->where('inv_id = 2')
    ->union()
    ->from('co_invoices')
    ->where('inv_id = 3')
;

// SELECT * FROM co_invoices WHERE inv_id = 1 UNION 
// SELECT * FROM co_invoices WHERE inv_id = 2 UNION
// SELECT * FROM co_invoices WHERE inv_id = 3

$select
    ->from('co_invoices')
    ->where('inv_id = 1')
    ->unionAll()
    ->from('co_invoices')
    ->where('inv_id = 2')
;

// SELECT * FROM co_invoices WHERE inv_id = 1 UNION ALL 
// SELECT * FROM co_invoices WHERE inv_id = 2 
```

#### Reset
The `Select` class exposes the `reset()` method, that allows you to reset the object to its original state and reuse it (e.g., to re-issue a statement to get a `COUNT(*)` without a `LIMIT`, to find the total number of rows to be paginated over).


#### Subselect Objects
If you want to create a subselect, call the `subSelect()` method. When you are done building the subselect, give it an alias using the `asAlias()` method; the object itself can be used in the desired condition or expression.

```php
<?php

$select
    ->from(
        $select
            ->subSelect()
            ->columns("inv_id")
            ->from('co_invoices')
            ->asAlias('inv')
            ->getStatement()
    )
;

// SELECT *
// FROM (SELECT inv_id FROM co_invoices) AS inv 
```

### Update

[auraphp]: https://github.com/auraphp
[atlasphp]: https://github.com/atlasphp
[datamapper]: https://martinfowler.com/eaaCatalog/dataMapper.html
[datamapper-pdo-connection]: api/phalcon_datamapper.md#datamapper-pdo-connection
[datamapper-pdo-connection-abstractconnection]: api/phalcon_datamapper.md#datamapper-pdo-connection-abstractconnection
[datamapper-pdo-connection-connectioninterface]: api/phalcon_datamapper.md#datamapper-pdo-connection-connectioninterface
[datamapper-pdo-connection-decorated]: api/phalcon_datamapper.md#datamapper-pdo-connection-decorated
[datamapper-pdo-connection-pdointerface]: api/phalcon_datamapper.md#datamapper-pdo-connection-pdointerface
[datamapper-pdo-connectionlocator]: api/phalcon_datamapper.md#datamapper-pdo-connectionlocator
[datamapper-pdo-connectionlocatorinterface]: api/phalcon_datamapper.md#datamapper-pdo-connectionlocatorinterface
[datamapper-pdo-exception-cannotdisconnect]: api/phalcon_datamapper.md#datamapper-pdo-exception-cannotdisconnect
[datamapper-pdo-exception-connectionnotfound]: api/phalcon_datamapper.md#datamapper-pdo-exception-connectionnotfound
[datamapper-pdo-exception-exception]: api/phalcon_datamapper.md#datamapper-pdo-exception-exception
[datamapper-pdo-profiler-memorylogger]: api/phalcon_datamapper.md#datamapper-pdo-profiler-memorylogger
[datamapper-pdo-profiler-profiler]: api/phalcon_datamapper.md#datamapper-pdo-profiler-profiler
[datamapper-pdo-profiler-profilerinterface]: api/phalcon_datamapper.md#datamapper-pdo-profiler-profilerinterface
[datamapper-query-abstractconditions]: api/phalcon_datamapper.md#datamapper-query-abstractconditions
[datamapper-query-abstractquery]: api/phalcon_datamapper.md#datamapper-query-abstractquery
[datamapper-query-bind]: api/phalcon_datamapper.md#datamapper-query-bind
[datamapper-query-delete]: api/phalcon_datamapper.md#datamapper-query-delete
[datamapper-query-insert]: api/phalcon_datamapper.md#datamapper-query-insert
[datamapper-query-queryfactory]: api/phalcon_datamapper.md#datamapper-query-queryfactory
[datamapper-query-select]: api/phalcon_datamapper.md#datamapper-query-select
[datamapper-query-update]: api/phalcon_datamapper.md#datamapper-query-update
[eaa]: https://martinfowler.com/books/eaa.html
[logger]: logger.md