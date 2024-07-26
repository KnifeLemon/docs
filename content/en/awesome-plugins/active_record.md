# Flight Active Record 

An active record is mapping a database entity to a PHP object. Spoken plainly, if you have a users table in your database, you can "translate" a row in that table to a `User` class and a `$user` object in your codebase. See [basic example](#basic-example).

Click [here](https://github.com/flightphp/active-record) for the repository in GitHub.

## Basic Example

Let's assume you have the following table:

```sql
CREATE TABLE users (
	id INTEGER PRIMARY KEY, 
	name TEXT, 
	password TEXT 
);
```

Now you can setup a new class to represent this table:

```php
/**
 * An ActiveRecord class is usually singular
 * 
 * It's highly recommended to add the properties of the table as comments here
 * 
 * @property int    $id
 * @property string $name
 * @property string $password
 */ 
class User extends flight\ActiveRecord {
	public function __construct($database_connection)
	{
		// you can set it this way
		parent::__construct($database_connection, 'users');
		// or this way
		parent::__construct($database_connection, null, [ 'table' => 'users']);
	}
}
```

Now watch the magic happen!

```php
// for sqlite
$database_connection = new PDO('sqlite:test.db'); // this is just for example, you'd probably use a real database connection

// for mysql
$database_connection = new PDO('mysql:host=localhost;dbname=test_db&charset=utf8bm4', 'username', 'password');

// or mysqli
$database_connection = new mysqli('localhost', 'username', 'password', 'test_db');
// or mysqli with non-object based creation
$database_connection = mysqli_connect('localhost', 'username', 'password', 'test_db');

$user = new User($database_connection);
$user->name = 'Bobby Tables';
$user->password = password_hash('some cool password');
$user->insert();
// or $user->save();

echo $user->id; // 1

$user->name = 'Joseph Mamma';
$user->password = password_hash('some cool password again!!!');
$user->insert();
// can't use $user->save() here or it will think it's an update!

echo $user->id; // 2
```

And it was just that easy to add a new user! Now that there is a user row in the database, how do you pull it out?

```php
$user->find(1); // find id = 1 in the database and return it.
echo $user->name; // 'Bobby Tables'
```

And what if you want to find all the users?

```php
$users = $user->findAll();
```

What about with a certain condition?

```php
$users = $user->like('name', '%mamma%')->findAll();
```

See how much fun this is? Let's install it and get started!

## Installation

Simply install with Composer

```php
composer require flightphp/active-record 
```

## Usage

This can be used as a standalone library or with the Flight PHP Framework. Completely up to you.

### Standalone
Just makes sure you pass a PDO connection to the constructor.

```php
$pdo_connection = new PDO('sqlite:test.db'); // this is just for example, you'd probably use a real database connection

$User = new User($pdo_connection);
```

> Don't want to always set your database connection in the constructor? See [Database Connection Management](#database-connection-management) for other ideas!

### Flight PHP Framework
If you are using the Flight PHP Framework, you can register the ActiveRecord class as a service (but you honestly don't have to).

```php
Flight::register('user', 'User', [ $pdo_connection ]);

// then you can use it like this in a controller, a function, etc.

Flight::user()->find(1);
```

## CRUD functions

#### `find($id = null) : boolean|ActiveRecord`

Find one record and assign in to current object. If you pass an `$id` of some kind it will perform a lookup on the primary key with that value. If nothing is passed, it will just find the first record in table.

Additionally you can pass it other helper methods to query your table.

```php
// find a record with some conditions before hand
$user->notNull('password')->orderBy('id DESC')->find();

// find a record by a specific id
$id = 123;
$user->find($id);
```

#### `findAll(): array<int,ActiveRecord>`

Finds all records in the table that you specify.

```php
$user->findAll();
```

#### `isHydrated(): boolean` (v0.4.0)

Returns `true` if the current record has been hydrated (fetched from the database).

```php
$user->find(1);
// if a record is found with data...
$user->isHydrated(); // true
```

#### `insert(): boolean|ActiveRecord`

Inserts the current record into database.

```php
$user = new User($pdo_connection);
$user->name = 'demo';
$user->password = md5('demo');
$user->insert();
```

##### Text Based Primary Keys

If you have a text based primary key (such as a UUID), you can set the primary key value before inserting in one of two ways.

```php
$user = new User($pdo_connection, [ 'primaryKey' => 'uuid' ]);
$user->uuid = 'some-uuid';
$user->name = 'demo';
$user->password = md5('demo');
$user->insert(); // or $user->save();
```

or you can have the primary key automatically generated for you through events.

```php
class User extends flight\ActiveRecord {
	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'users', [ 'primaryKey' => 'uuid' ]);
		// you can also set the primaryKey this way instead of the array above.
		$this->primaryKey = 'uuid';
	}

	protected function beforeInsert(self $self) {
		$self->uuid = uniqid(); // or however you need to generated your unique ids
	}
}
```

If you don't set the primary key before inserting, it will be set to the `rowid` and the 
database will generate it for you, but it won't persist because that field may not exist
in your table. This is why it's recommended to use the event to automatically handle this 
for you.

#### `update(): boolean|ActiveRecord`

Updates the current record into the database.

```php
$user->greaterThan('id', 0)->orderBy('id desc')->find();
$user->email = 'test@example.com';
$user->update();
```

#### `save(): boolean|ActiveRecord`

Inserts or updates the current record into the database. If the record has an id, it will update, otherwise it will insert.

```php
$user = new User($pdo_connection);
$user->name = 'demo';
$user->password = md5('demo');
$user->save();
```

**Note:** If you have relationships defined in the class, it will recursively save those relations as well if they have been defined, instantiated and have dirty data to update. (v0.4.0 and above)

#### `delete(): boolean`

Deletes the current record from the database.

```php
$user->gt('id', 0)->orderBy('id desc')->find();
$user->delete();
```

You can also delete multiple records executing a search before hand.

```php
$user->like('name', 'Bob%')->delete();
```

#### `dirty(array  $dirty = []): ActiveRecord`

Dirty data refers to the data that has been changed in a record.

```php
$user->greaterThan('id', 0)->orderBy('id desc')->find();

// nothing is "dirty" as of this point.

$user->email = 'test@example.com'; // now email is considered "dirty" since it's changed.
$user->update();
// now there is no data that is dirty because it's been updated and persisted in the database

$user->password = password_hash()'newpassword'); // now this is dirty
$user->dirty(); // passing nothing will clear all the dirty entries.
$user->update(); // nothing will update cause nothing was captured as dirty.

$user->dirty([ 'name' => 'something', 'password' => password_hash('a different password') ]);
$user->update(); // both name and password are updated.
```

#### `copyFrom(array $data): ActiveRecord` (v0.4.0)

This is an alias for the `dirty()` method. It's a little more clear what you are doing.

```php
$user->copyFrom([ 'name' => 'something', 'password' => password_hash('a different password') ]);
$user->update(); // both name and password are updated.
```

#### `isDirty(): boolean` (v0.4.0)

Returns `true` if the current record has been changed.

```php
$user->greaterThan('id', 0)->orderBy('id desc')->find();
$user->email = 'test@email.com';
$user->isDirty(); // true
```

#### `reset(bool $include_query_data = true): ActiveRecord`

Resets the current record to it's initial state. This is really good to use in loop type behaviors.
If you pass `true` it will also reset the query data that was used to find the current object (default behavior).

```php
$users = $user->greaterThan('id', 0)->orderBy('id desc')->find();
$user_company = new UserCompany($pdo_connection);

foreach($users as $user) {
	$user_company->reset(); // start with a clean slate
	$user_company->user_id = $user->id;
	$user_company->company_id = $some_company_id;
	$user_company->insert();
}
```

#### `getBuiltSql(): string` (v0.4.1)

After you run a `find()`, `findAll()`, `insert()`, `update()`, or `save()` method you can get the SQL that was built and use it for debugging purposes.

## SQL Query Methods
#### `select(string $field1 [, string $field2 ... ])`

You can select only a few of the columns in a table if you'd like (it is more performant on really wide tables with many columns)

```php
$user->select('id', 'name')->find();
```

#### `from(string $table)`

You can technically choose another table too! Why the heck not?!

```php
$user->select('id', 'name')->from('user')->find();
```

#### `join(string $table_name, string $join_condition)`

You can even join to another table in the database.

```php
$user->join('contacts', 'contacts.user_id = users.id')->find();
```

#### `where(string $where_conditions)`

You can set some custom where arguments (you cannot set params in this where statement)

```php
$user->where('id=1 AND name="demo"')->find();
```

**Security Note** - You might be tempted to do something like `$user->where("id = '{$id}' AND name = '{$name}'")->find();`. Please DO NOT DO THIS!!! This is susceptible to what is knows as SQL Injection attacks. There are lots of articles online, please Google "sql injection attacks php" and you'll find a lot of articles on this subject. The proper way to handle this with this library is instead of this `where()` method, you would do something more like `$user->eq('id', $id)->eq('name', $name)->find();` If you absolutely have to do this, the `PDO` library has `$pdo->quote($var)` to escape it for you. Only after you use `quote()` can you use it in a `where()` statement.

#### `group(string $group_by_statement)/groupBy(string $group_by_statement)`

Group your results by a particular condition.

```php
$user->select('COUNT(*) as count')->groupBy('name')->findAll();
```

#### `order(string $order_by_statement)/orderBy(string $order_by_statement)`

Sort the returned query a certain way.

```php
$user->orderBy('name DESC')->find();
```

#### `limit(string $limit)/limit(int $offset, int $limit)`

Limit the amount of records returned. If a second int is given, it will be offset, limit just like in SQL.

```php
$user->orderby('name DESC')->limit(0, 10)->findAll();
```

## WHERE conditions
#### `equal(string $field, mixed $value) / eq(string $field, mixed $value)`

Where `field = $value`

```php
$user->eq('id', 1)->find();
```

#### `notEqual(string $field, mixed $value) / ne(string $field, mixed $value)`

Where `field <> $value`

```php
$user->ne('id', 1)->find();
```

#### `isNull(string $field)`

Where `field IS NULL`

```php
$user->isNull('id')->find();
```
#### `isNotNull(string $field) / notNull(string $field)`

Where `field IS NOT NULL`

```php
$user->isNotNull('id')->find();
```

#### `greaterThan(string $field, mixed $value) / gt(string $field, mixed $value)`

Where `field > $value`

```php
$user->gt('id', 1)->find();
```

#### `lessThan(string $field, mixed $value) / lt(string $field, mixed $value)`

Where `field < $value`

```php
$user->lt('id', 1)->find();
```
#### `greaterThanOrEqual(string $field, mixed $value) / ge(string $field, mixed $value) / gte(string $field, mixed $value)`

Where `field >= $value`

```php
$user->ge('id', 1)->find();
```
#### `lessThanOrEqual(string $field, mixed $value) / le(string $field, mixed $value) / lte(string $field, mixed $value)`

Where `field <= $value`

```php
$user->le('id', 1)->find();
```

#### `like(string $field, mixed $value) / notLike(string $field, mixed $value)`

Where `field LIKE $value` or `field NOT LIKE $value`

```php
$user->like('name', 'de')->find();
```

#### `in(string $field, array $values) / notIn(string $field, array $values)`

Where `field IN($value)` or `field NOT IN($value)`

```php
$user->in('id', [1, 2])->find();
```

#### `between(string $field, array $values)`

Where `field BETWEEN $value AND $value1`

```php
$user->between('id', [1, 2])->find();
```

## Relationships
You can set several kinds of relationships using this library. You can set one->many and one->one relationships between tables. This requires a little extra setup in the class beforehand.

Setting the `$relations` array is not hard, but guessing the correct syntax can be confusing.

```php
protected array $relations = [
	// you can name the key anything you'd like. The name of the ActiveRecord is probably good. Ex: user, contact, client
	'user' => [
		// required
		// self::HAS_MANY, self::HAS_ONE, self::BELONGS_TO
		self::HAS_ONE, // this is the type of relationship

		// required
		'Some_Class', // this is the "other" ActiveRecord class this will reference

		// required
		// depending on the relationship type
		// self::HAS_ONE = the foreign key that references the join
		// self::HAS_MANY = the foreign key that references the join
		// self::BELONGS_TO = the local key that references the join
		'local_or_foreign_key',
		// just FYI, this also only joins to the primary key of the "other" model

		// optional
		[ 'eq' => [ 'client_id', 5 ], 'select' => 'COUNT(*) as count', 'limit' 5 ], // additional conditions you want when joining the relation
		// $record->eq('client_id', 5)->select('COUNT(*) as count')->limit(5))

		// optional
		'back_reference_name' // this is if you want to back reference this relationship back to itself Ex: $user->contact->user;
	];
]
```

```php
class User extends ActiveRecord{
	protected array $relations = [
		'contacts' => [ self::HAS_MANY, Contact::class, 'user_id' ],
		'contact' => [ self::HAS_ONE, Contact::class, 'user_id' ],
	];

	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'users');
	}
}

class Contact extends ActiveRecord{
	protected array $relations = [
		'user' => [ self::BELONGS_TO, User::class, 'user_id' ],
		'user_with_backref' => [ self::BELONGS_TO, User::class, 'user_id', [], 'contact' ],
	];
	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'contacts');
	}
}
```

Now we have the references setup so we can use them very easily!

```php
$user = new User($pdo_connection);

// find the most recent user.
$user->notNull('id')->orderBy('id desc')->find();

// get contacts by using relation:
foreach($user->contacts as $contact) {
	echo $contact->id;
}

// or we can go the other way.
$contact = new Contact();

// find one contact
$contact->find();

// get user by using relation:
echo $contact->user->name; // this is the user name
```

Pretty cool eh?

## Setting Custom Data
Sometimes you may need to attach something unique to your ActiveRecord such as a custom calculation that might be easier to just attach to the object that would then be passed to say a template.

#### `setCustomData(string $field, mixed $value)`
You attach the custom data with the `setCustomData()` method.
```php
$user->setCustomData('page_view_count', $page_view_count);
```

And then you simply reference it like a normal object property.

```php
echo $user->page_view_count;
```

## Events

One more super awesome feature about this library is about events. Events are triggered at certain times based on certain methods you call. They are very very helpful in setting up data for you automatically.

#### `onConstruct(ActiveRecord $ActiveRecord, array &config)`

This is really helpful if you need to set a default connection or something like that.

```php
// index.php or bootstrap.php
Flight::register('db', 'PDO', [ 'sqlite:test.db' ]);

//
//
//

// User.php
class User extends flight\ActiveRecord {

	protected function onConstruct(self $self, array &$config) { // don't forget the & reference
		// you could do this to automatically set the connection
		$config['connection'] = Flight::db();
		// or this
		$self->transformAndPersistConnection(Flight::db());
		
		// You can also set the table name this way.
		$config['table'] = 'users';
	} 
}
```

#### `beforeFind(ActiveRecord $ActiveRecord)`

This is likely only useful if you need a query manipulation each time.

```php
class User extends flight\ActiveRecord {
	
	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'users');
	}

	protected function beforeFind(self $self) {
		// always run id >= 0 if that's your jam
		$self->gte('id', 0); 
	} 
}
```

#### `afterFind(ActiveRecord $ActiveRecord)`

This one is likely more useful if you always need to run some logic every time this record is fetched. Do you need to decrypt something? Do you need to run a custom count query each time (not performant but whatevs)?

```php
class User extends flight\ActiveRecord {
	
	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'users');
	}

	protected function afterFind(self $self) {
		// decrypting something
		$self->secret = yourDecryptFunction($self->secret, $some_key);

		// maybe storing something custom like a query???
		$self->setCustomData('view_count', $self->select('COUNT(*) count')->from('user_views')->eq('user_id', $self->id)['count']; 
	} 
}
```

#### `beforeFindAll(ActiveRecord $ActiveRecord)`

This is likely only useful if you need a query manipulation each time.

```php
class User extends flight\ActiveRecord {
	
	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'users');
	}

	protected function beforeFindAll(self $self) {
		// always run id >= 0 if that's your jam
		$self->gte('id', 0); 
	} 
}
```

#### `afterFindAll(array<int,ActiveRecord> $results)`

Similar to `afterFind()` but you get to do it to all the records instead!

```php
class User extends flight\ActiveRecord {
	
	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'users');
	}

	protected function afterFindAll(array $results) {

		foreach($results as $self) {
			// do something cool like afterFind()
		}
	} 
}
```

#### `beforeInsert(ActiveRecord $ActiveRecord)`

Really helpful if you need some default values set each time.

```php
class User extends flight\ActiveRecord {
	
	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'users');
	}

	protected function beforeInsert(self $self) {
		// set some sound defaults
		if(!$self->created_date) {
			$self->created_date = gmdate('Y-m-d');
		}

		if(!$self->password) {
			$self->password = password_hash((string) microtime(true));
		}
	} 
}
```

#### `afterInsert(ActiveRecord $ActiveRecord)`

Maybe you have a user case for changing data after it's inserted?

```php
class User extends flight\ActiveRecord {
	
	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'users');
	}

	protected function afterInsert(self $self) {
		// you do you
		Flight::cache()->set('most_recent_insert_id', $self->id);
		// or whatever....
	} 
}
```

#### `beforeUpdate(ActiveRecord $ActiveRecord)`

Really helpful if you need some default values set each time on an update.

```php
class User extends flight\ActiveRecord {
	
	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'users');
	}

	protected function beforeInsert(self $self) {
		// set some sound defaults
		if(!$self->updated_date) {
			$self->updated_date = gmdate('Y-m-d');
		}
	} 
}
```

#### `afterUpdate(ActiveRecord $ActiveRecord)`

Maybe you have a user case for changing data after it's updated?

```php
class User extends flight\ActiveRecord {
	
	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'users');
	}

	protected function afterInsert(self $self) {
		// you do you
		Flight::cache()->set('most_recently_updated_user_id', $self->id);
		// or whatever....
	} 
}
```

#### `beforeSave(ActiveRecord $ActiveRecord)/afterSave(ActiveRecord $ActiveRecord)`

This is useful if you want events to happen both when inserts or updates happen. I'll spare you the long explanation, but I'm sure you can guess what it is.

```php
class User extends flight\ActiveRecord {
	
	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'users');
	}

	protected function beforeSave(self $self) {
		$self->last_updated = gmdate('Y-m-d H:i:s');
	} 
}
```

#### `beforeDelete(ActiveRecord $ActiveRecord)/afterDelete(ActiveRecord $ActiveRecord)`

Not sure what you'd want to do here, but no judgments here! Go for it!

```php
class User extends flight\ActiveRecord {
	
	public function __construct($database_connection)
	{
		parent::__construct($database_connection, 'users');
	}

	protected function beforeDelete(self $self) {
		echo 'He was a brave soldier... :cry-face:';
	} 
}
```

## Database Connection Management

When you are using this library, you can set the database connection in a few different ways. You can set the connection in the constructor, you can set it via a config variable `$config['connection']` or you can set it via `setDatabaseConnection()` (v0.4.1). 

```php
$pdo_connection = new PDO('sqlite:test.db'); // for example
$user = new User($pdo_connection);
// or
$user = new User(null, [ 'connection' => $pdo_connection ]);
// or
$user = new User();
$user->setDatabaseConnection($pdo_connection);
```

If you want to avoid always setting a `$database_connection` every time you call an active record, there are ways around that!

```php
// index.php or bootstrap.php
// Set this as a registered class in Flight
Flight::register('db', 'PDO', [ 'sqlite:test.db' ]);

// User.php
class User extends flight\ActiveRecord {
	
	public function __construct(array $config = [])
	{
		$database_connection = $config['connection'] ?? Flight::db();
		parent::__construct($database_connection, 'users', $config);
	}
}

// And now, no args required!
$user = new User();
```

> **Note:** If you are planning on unit testing, doing it this way can add some challenges to unit testing, but overall because you can inject your 
connection with `setDatabaseConnection()` or `$config['connection']` it's not too bad.

If you need to refresh the database connection, for instance if you are running a long running CLI script and need to refresh the connection every so often, you can re-set the connection with `$your_record->setDatabaseConnection($pdo_connection)`.

## Contributing

Please do. :D

### Setup

When you contribute, make sure you run `composer test-coverage` to maintain 100% test coverage (this isn't true unit test coverage, more like integration testing).

Also make sure you run `composer beautify` and `composer phpcs` to fix any linting errors.

## License

MIT