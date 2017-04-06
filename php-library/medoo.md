# Medoo
The lightest PHP database framework to accelerate development

## Get Started

### Requirement
- PHP 5.4+ with PDO support.
- Installed SQL database like MySQL, MSSQL, SQLite or others.
- Make sure php_pdo_xxx extension is correctly installed and enabled.
- A little bit SQL knowledge.

### Configuration
    // Using Medoo namespace
    use Medoo\Medoo;

    $database = new Medoo([
	  // required
	  'database_type' => 'mysql',
	  'database_name' => 'name',
	  'server' => 'localhost',
	  'username' => 'your_username',
	  'password' => 'your_password',
	  'charset' => 'utf8',

	  // [optional]
	  'port' => 3306,

	  // [optional] Table prefix
	  'prefix' => 'PREFIX_',

	  // [optional] MySQL socket (shouldn't be used with server and port)
	  'socket' => '/tmp/mysql.sock',

	  // [optional] driver_option for connection, read more from http://www.php.net/manual/en/pdo.setattribute.php
	  'option' => [
		PDO::ATTR_CASE => PDO::CASE_NATURAL
	  ],

	  // [optional] Medoo will execute those commands after connected to the database for initialization
	  'command' => [
		'SET SQL_MODE=ANSI_QUOTES'
	  ]
    ]);

## WHERE Syntax
Some of Medoo functions are required $where argument to filter record like SQL WHERE clause which is powerful but with a lot of complex syntax, logical relativity, and potential security problem about SQL injection. But Medoo provided a powerful and extremely easy way to build WHERE query clause and prevent injection.

### Basic Condition
The basic condition is simple enough to understand. You can use additional symbol to get advanced filter range for number.

	$database->select("account", "user_name", [
		"email" => "foo@bar.com"
	]);
	// WHERE email = 'foo@bar.com'
	 
	$database->select("account", "user_name", [
		"user_id" => 200
	]);
	// WHERE user_id = 200
	 
	$database->select("account", "user_name", [
		"user_id[>]" => 200
	]);
	// WHERE user_id > 200
	 
	$database->select("account", "user_name", [
		"user_id[>=]" => 200
	]);
	// WHERE user_id >= 200
	 
	$database->select("account", "user_name", [
		"user_id[!]" => 200
	]);
	// WHERE user_id != 200
	 
	$database->select("account", "user_name", [
		"age[<>]" => [200, 500]
	]);
	// WHERE age BETWEEN 200 AND 500
	 
	$database->select("account", "user_name", [
		"age[><]" => [200, 500]
	]);
	// WHERE age NOT BETWEEN 200 AND 500
	 
	// [><] and [<>] is also available for datetime
	$database->select("account", "user_name", [
		"birthday[><]" => [date("Y-m-d", mktime(0, 0, 0, 1, 1, 2015)), date("Y-m-d")]
	]);
	//WHERE "create_date" BETWEEN '2015-01-01' AND '2015-05-01' (now)
	 
	// You can use not only single string or number value, but also array
	$database->select("account", "user_name", [
		"OR" => [
			"user_id" => [2, 123, 234, 54],
			"email" => ["foo@bar.com", "cat@dog.com", "admin@medoo.in"]
		]
	]);
	// WHERE
	// user_id IN (2,123,234,54) OR
	// email IN ('foo@bar.com','cat@dog.com','admin@medoo.in')
	 
	// [Negative condition]
	$database->select("account", "user_name", [
		"AND" => [
			"user_name[!]" => "foo",
			"user_id[!]" => 1024,
			"email[!]" => ["foo@bar.com", "cat@dog.com", "admin@medoo.in"],
			"city[!]" => null,
			"promoted[!]" => true
		]
	]);
	// WHERE
	// `user_name` != 'foo' AND
	// `user_id` != 1024 AND
	// `email` NOT IN ('foo@bar.com','cat@dog.com','admin@medoo.in') AND
	// `city` IS NOT NULL
	// `promoted` != 1
	 
	// Or fetched from select() or get() function
	$database->select("account", "user_name", [
		"user_id" => $database->select("post", "user_id", ["comments[>]" => 40])
	]);
	// WHERE user_id IN (2, 51, 321, 3431)

### Relativity Condition
The relativity condition can describe complex relationship between data and data. You can use AND and OR to build complex relativity condition query.

**Basic**

	$database->select("account", "user_name", [
		"AND" => [
			"user_id[>]" => 200,
			"age[<>]" => [18, 25],
			"gender" => "female"
		]
	]);
	 
	// Medoo will connect relativity condition with AND by default. The following usage is the same like above.
	$database->select("account", "user_name", [
		"user_id[>]" => 200,
		"age[<>]" => [18, 25],
		"gender" => "female"
	]);
	 
	// WHERE user_id > 200 AND age BETWEEN 18 AND 25 AND gender = 'female'
	 
	$database->select("account", "user_name", [
		"OR" => [
			"user_id[>]" => 200,
			"age[<>]" => [18, 25],
			"gender" => "female"
		]
	]);
	// WHERE user_id > 200 OR age BETWEEN 18 AND 25 OR gender = 'female'

**Compound**

	$database->has("account", [
		"AND" => [
			"OR" => [
				"user_name" => "foo",
				"email" => "foo@bar.com"
			],
			"password" => "12345"
		]
	]);
	// WHERE (user_name = 'foo' OR email = 'foo@bar.com') AND password = '12345'
	 
	// [IMPORTANT]
	// Because Medoo is using array data construction to describe relativity condition,
	// array with duplicated key will be overwritten.
	//
	// This will be error:
	$database->select("account", '*', [
		"AND" => [
			"OR" => [
				"user_name" => "foo",
				"email" => "foo@bar.com"
			],
			"OR" => [
				"user_name" => "bar",
				"email" => "bar@foo.com"
			]
		]
	]);
	// [X] SELECT * FROM "account" WHERE ("user_name" = 'bar' OR "email" = 'bar@foo.com')
	 
	// To correct that, just assign a comment for each AND and OR key name. The comment content can be everything.
	$database->select("account", '*', [
		"AND #Actually, this comment feature can be used on every AND and OR relativity condition" => [
			"OR #the first condition" => [
				"user_name" => "foo",
				"email" => "foo@bar.com"
			],
			"OR #the second condition" => [
				"user_name" => "bar",
				"email" => "bar@foo.com"
			]
		]
	]);
	// SELECT * FROM "account"
	// WHERE (
	// 	(
	// 		"user_name" = 'foo' OR "email" = 'foo@bar.com'
	// 	)
	// 	AND
	// 	(
	// 		"user_name" = 'bar' OR "email" = 'bar@foo.com'
	// 	)
	// )

**Columns Relationship**

	$database->select("post", [
			"[>]account" => ["author_id" => "user_id"],
		], [
			"post.id",
			"post.content"
		], [
			"AND" => [
				// Connect two column with condition sign like [=], [>], [<], [!=] as one of array value
				"post.restrict[<]account.age",
				
				"account.user_name" => "foo",
				"account.email" => "foo@bar.com",
			]
		]
	);
	 
	// WHERE "post"."restrict" < "account"."age" AND "account"."user_name" = 'foo' AND "account"."email" = 'foo@bar.com'

### LIKE Condition
LIKE condition can be use it like basic condition or relativity condition with just adding `[~]` syntax now.

	// By default, the keyword will be quoted with % front and end to match the whole word.
	$database->select("person", "id", [
		"city[~]" => "lon"
	]);
	 
	WHERE "city" LIKE '%lon%'
	 
	// Array support
	$database->select("person", "id", [
		"city[~]" => ["lon", "foo", "bar"]
	]);
	 
	WHERE "city" LIKE '%lon%' OR "city" LIKE '%foo%' OR "city" LIKE '%bar%'
	 
	// Negative condition [!~]
	$database->select("person", "id", [
		"city[!~]" => "lon"
	]);
	 
	WHERE "city" NOT LIKE '%lon%'

**SQL Wildcard**

	// You can use SQL wildcard to match more complex situation
	$database->select("person", "id", [
		"city[~]" => "%stan" // Kazakhstan,  Uzbekistan, Türkmenistan
	]);
	 
	$database->select("person", "id", [
		"city[~]" => "Londo_" // London, Londox, Londos...
	]);
	 
	$database->select("person", "id", [
		"name[~]" => "[BCR]at" // Bat, Cat, Rat
	]);
	 
	$database->select("person", "id", [
		"name[~]" => "[!BCR]at" // Eat, Fat, Hat...
	]);

**Compound**

	// You can use SQL wildcard to match more complex situation
	$database->select("person", "id", [
		"content[~]" => ["AND" => ["lon", "on"]]
	]);
	 
	// WHERE ("content" LIKE '%lon%' AND "content" LIKE '%on%')
	 
	$database->select("person", "id", [
		"content[~]" => ["OR" => ["lon", "on"]]
	]);
	 
	// WHERE ("content" LIKE '%lon%' OR "content" LIKE '%on%')

### Order Condition

	$database->select("account", "user_id", [
	 
		// Single condition
		"ORDER" => "user_id",
	 
		// Multiple condition
		"ORDER" => [
			// Order by column with sorting by customized order.
			"user_id" => [43, 12, 57, 98, 144, 1],
	 
			// Order by column
			"register_date",
	 
			// Order by column with descending sorting
			"profile_id" => "DESC",
	 
			// Order by column with ascending sorting
			"date" => "ASC"
		]
	]);

### Full Text Searching

	// [MATCH]
	$database->select("post_table", "post_id", [
		"MATCH" => [
			"columns" => ["content", "title"],
			"keyword" => "foo"
		]
	]);
	// WHERE MATCH (content, title) AGAINST ('foo')

### Using SQL Functions
In some of special case, you may need to use SQL functions to process data. Just assign # in front of the column name, then the value will not be quoted.

	$data = $database->select('account', [
		'user_id',
		'user_name'
	], [
		'#datetime' => 'NOW()'
	]);
	 
	// SELECT "user_id","user_name"
	// FROM "account"
	// WHERE "datetime" = NOW()
	 
	// [IMPORTANT] Keep in mind that, the value will not be quoted should be matched as XXX() uppercase.
	// The following sample will be failed.
	$database->select('account', [
		'user_id',
		'user_name'
	], [
	    '#datetime2' => 'now()',
	 
	    'datetime3' => 'NOW()',
	 
	    '#datetime4' => 'NOW'
	]);

### Additional Condition

	$database->select("account", "user_id", [
		"GROUP" => "type",
	 
		// Must have to use it with GROUP together
		"HAVING" => [
			"user_id[>]" => 500
		],
	 
		// LIMIT => 20
		"LIMIT" => [20, 100]
	]);
	//	SELECT user_id FROM account
	//	GROUP BY type
	//	HAVING user_id > 500
	//	LIMIT 20,100

## select
Select data from database

> select($table, $columns, $where)

- `table [string]` The table name.
- `columns [string/array]` The target columns of data will be fetched.
- `where (optional) [array]` The WHERE clause to filter records.

> select($table, $join, $columns, $where)

- `table [string]` The table name.
- `join [array]` Table relativity for table joining. Ignore it if no table joining required.
- `columns [string/array]` The target columns of data will be fetched.
- `where (optional) [array]` The WHERE clause to filter records.

**Return: [array]**

You can use "*" as columns parameter to fetch all columns, but to increase the performance, providing the target columns is much better.

	$datas = $database->select("account", [
		"user_name",
		"email"
	], [
		"user_id[>]" => 100
	]);
	 
	// $datas = array(
	// 	[0] => array(
	// 		"user_name" => "foo",
	// 		"email" => "foo@bar.com"
	// 	),
	// 	[1] => array(
	// 		"user_name" => "cat",
	// 		"email" => "cat@dog.com"
	// 	)
	// )
	 
	foreach($datas as $data)
	{
		echo "user_name:" . $data["user_name"] . " - email:" . $data["email"] . "<br/>";
	}
	 
	// Select all columns
	$datas = $database->select("account", "*");
	 
	// Select a column
	$datas = $database->select("account", "user_name");
	 
	// $datas = array(
	// 	[0] => "foo",
	// 	[1] => "cat"
	// )

### Table Joining
SQL JOIN clause can combine rows together between two table. Medoo provide simple syntax for JOIN clause.

	// [>] == LEFT JOIN
	// [<] == RIGH JOIN
	// [<>] == FULL JOIN
	// [><] == INNER JOIN
	 
	$database->select("post", [
		// Here is the table relativity argument that tells the relativity between the table you want to join.
	 
		// The row author_id from table post is equal the row user_id from table account
		"[>]account" => ["author_id" => "user_id"],
	 
		// The row user_id from table post is equal the row user_id from table album.
		// This is a shortcut to declare the relativity if the row name are the same in both table.
		"[>]album" => "user_id",
	 
		// [post.user_id is equal photo.user_id and post.avatar_id is equal photo.avatar_id]
		// Like above, there are two row or more are the same in both table.
		"[>]photo" => ["user_id", "avatar_id"],
	 
		// If you want to join the same table with different value,
		// you have to assign the table with alias.
		"[>]account (replyer)" => ["replyer_id" => "user_id"],
	 
		// You can refer the previous joined table by adding the table name before the column.
		"[>]account" => ["author_id" => "user_id"],
		"[>]album" => ["account.user_id" => "user_id"],
	 
		// Multiple condition
		"[>]account" => [
			"author_id" => "user_id",
			"album.user_id" => "user_id"
		]
	], [
		"post.post_id",
		"post.title",
		"account.user_id",
		"account.city",
		"replyer.user_id",
		"replyer.city"
	], [
		"post.user_id" => 100,
		"ORDER" => ["post.post_id" => "DESC"],
		"LIMIT" => 50
	]);
	 
	// SELECT
	// 	`post`.`post_id`,
	// 	`post`.`title`,
	// 	`account`.`city`
	// FROM `post`
	// LEFT JOIN `account` ON `post`.`author_id` = `account`.`user_id`
	// LEFT JOIN `album` USING (`user_id`)
	// LEFT JOIN `photo` USING (`user_id`, `avatar_id`)
	// WHERE
	// 	`post`.`user_id` = 100
	// ORDER BY `post`.`post_id` DESC
	// LIMIT 50

### Data Mapping
Customize output data construction - The key name for wrapping data has no relation with columns itself and it is multidimensional.

	$data = $database->select("post", [
		"[>]account" => ["user_id"]
	], [
		"post.post_id",
		"post.content",
	 
		"userData" => [
			"account.user_id",
			"account.email",
	 
			"meta" => [
				"account.location",
				"account.gender"
			]
		]
	], [
		"LIMIT" => [0, 2]
	]);
	 
	echo json_encode($data);
	 
	// Outputed data
	[
		{
			post_id: "1",
			content: "Hello world!",
			userData: {
				user_id: "1",
				email: "foo@example.com",
				meta: {
					location: "New York",
					gender: "male"
				}
			}
		},
		{
			post_id: "2",
			content: "Hey everyone",
			userData: {
				user_id: "2",
				email: "bar@example.com",
				meta: {
					location: "London",
					gender: "female"
				}
			}
		}
	]

### Alias
You can use alias as a new column or table name instead of original one. This is useful for table joining to prevent name conflict.

	$data = $database->select("account", [
		"user_id",
		"nickname(my_nickname)"
	], [
		"LIMIT" => 20
	]);
	 
	// $data = array(
	// 	[0] => array(
	// 		"user_id" => "1",
	// 		"my_nickname" => "foo"
	// 	),
	// 	[1] => array(
	// 		"user_id" => "2",
	// 		"my_nickname" => "bar"
	// 	)
	// )
	 
	$data = $database->select("post (content)", [
		"[>]account (user)" => "user_id",
	], [
		"content.user_id (author_id)",
		"user.user_id"
	], [
		"LIMIT" => 20
	]);
	 
	// SELECT
	// 	"content"."user_id" AS author_id,
	// 	"user"."user_id"
	// FROM
	// 	"post" AS "content"
	// LEFT JOIN "account" AS "user" USING ("user_id")
	// LIMIT 2
	 
	// $data = array(
	// 	[0] => array(
	// 		"author_id" => "1",
	// 		"user_id" => "321"
	// 	),
	// 	[1] => array(
	// 		"author_id" => "2",
	// 		"user_id" => "322"
	// 	)
	// )

## insert
Insert new records in table

> insert($table, $data)

- `table [string]` The table name.
- `data [array]` The data that will be inserted into table.

**Return: [number]** The number of rows affected.

	$database->insert("account", [
		"user_name" => "foo",
		"email" => "foo@bar.com",
		"age" => 25
	]);

### Last Insert ID
If you want the row ID after the insertion, you need to call the lastInsertId() alone and get it.

	$database->insert("account", [
		"user_name" => "foo",
		"email" => "foo@bar.com",
		"age" => 25
	]);
	 
	$account_id = $database->id();

### Array Serialization
By default, the array data will be serialized by `serialize()` before insertion, but you can assign it as JSON to be serialized by `json_encode()`.

	$database->insert("account", [
		"user_name" => "foo",
		"email" => "foo@bar.com",
		"age" => 25,
		"lang" => ["en", "fr", "jp", "cn"] // => 'a:4:{i:0;s:2:"en";i:1;s:2:"fr";i:2;s:2:"jp";i:3;s:2:"cn";}'
	]);
	 
	$database->insert("account", [
		"user_name" => "foo",
		"email" => "foo@bar.com",
		"age" => 25,
		"(JSON) lang" => ["en", "fr", "jp", "cn"] // => '["en","fr","jp","cn"]'
	]);

### Multi-Insertion
You can also insert data multiply.

	$database->insert("account", [
		[
			"user_name" => "foo",
			"email" => "foo@bar.com",
			"age" => 25,
			"city" => "New York",
			"(JSON) lang" => ["en", "fr", "jp", "cn"]
		],
		[
			"user_name" => "bar",
			"email" => "bar@foo.com",
			"age" => 14,
			"city" => "Hong Kong",
			"(JSON) lang" => ["en", "jp", "cn"]
		]
	]);

### Using SQL Functions
In some of special case, you may need to use SQL functions to process data. Just assign `#` in front of the column name, then the value will not be quoted.

	$database->insert("account", [
		"user_name" => "bar",
		"#uid" => "UUID()"
	]);

## update
Modify data in table

> update($table, $data, $where)

- `table [string]` The table name.
- `data [array]` The data that will be modified.
- `where (optional) [array]` The WHERE clause to filter records.

**Return: [number]** The number of rows affected.

Like insert(), you can modify array data without serialization, and you can use [+], [-], [*] and [/] for mathematical operation.

	$database->update("account", [
		"type" => "user",
	 
		// All age plus one
		"age[+]" => 1,
	 
		// All level subtract 5
		"level[-]" => 5,
	 
		// All score multiplied by 2
		"score[*]" => 2,
	 
		// Like insert, you can assign the serialization
		"lang" => ["en", "fr", "jp", "cn", "de"],
	 
		"(JSON) fav_lang" => ["en", "fr", "jp", "cn", "de"],
	 
		// You can also assign # for using SQL functions
		"#uid" => "UUID()"
	], [
		"user_id[<]" => 1000
	]);

## delete
Delete data from table. Most **dangerous** function, consider deeply before query.

> delete($table, $where)

- `table [string]` The table name.
- `where [array]` The WHERE clause to filter records.

**Return: [number]** The number of rows affected.

	$database->delete("account", [
		"AND" => [
			"type" => "business",
			"age[<]" => 18
		]
	]);

## replace
Replace old data into new one

> replace($table, $column, $search, $replace, $where)

- `table [string]` The table name.
- `columns [string/array]` The target columns of data will be replaced.
- `search [string]` The value being searched for.
- `replace [string]` The replacement value that replaces found search values.
- `where (optional)` [array] The WHERE clause to filter records.

> replace($table, $column, $replacement, $where)

- `table [String]` The table name.
- `columns [string/array]` The target columns of data will be replaced.
- `replacement [array]` The replacement array value that contained search value as key and replace string as value.
- `where (optional) [array]` The WHERE clause to filter records.

> replace($table, $columns, $where)

- `table [string]` The table name.
- `columns [array]` The target columns of data will be replaced.
- `where (optional) [array]` The WHERE clause to filter records.

**Return: [number]** The number of rows affected.

	$database->replace("account", "type", "user", "new_user", [
		"user_id[>]" => 1000
	]);
	 
	$database->replace("account", "type", [
		"user" => "new_user",
		"business" => "new_business"
	], [
		"user_id[>]" => 1000
	]);
	 
	$database->replace("account", [
		"type" => [
			"user" => "new_user",
			"business" => "new_business"
		],
		"group" => [
			"groupA" => "groupB"
		]
	], [
		"user_id[>]" => 1000
	]);

## get
Get only one record from table

> get($table, $columns, $where)

- `table [string]` The table name.
- `columns [string/array]` The target columns of data will be fetch.
- `where (optional) [array]` The WHERE clause to filter records.

**Return: [string/array]** Return the data of the column.

	$email = $database->get("account", "email", [
		"user_id" => 1234
	]);
	 
	// $email = "foo@bar.com"
	 
	$profile = $database->get("account", [
		"email",
		"gender",
		"location"
	], [
		"user_id" => 1234
	]);
	 
	// $profile = array(
	// 	"email" => "foo@bar.com",
	// 	"gender" => "female",
	// 	"location" => "earth"
	// )

## has
Determine whether the target data existed

> has($table, $where)

- `table [string]` The table name.
- `where [array]` The WHERE clause to filter records.

> has($table, $join, $where)

- `table [string]` The table name.
- `join [array]` Table relativity for table joining.
- `where [array]` The WHERE clause to filter records.

**Return: [boolean]** True of False if the target data has been founded.

The Where argument is required. This function is the best to verify data of password.

	if ($database->has("account", [
		"AND" => [
			"OR" => [
				"user_name" => "foo",
				"email" => "foo"
			],
			"password" => "12345"
		]
	]))
	{
		echo "Password is correct.";
	}
	else
	{
		echo "Password error.";
	}

## count
Counts the number of rows

> count($table, $where)

- `table [string]` The table name.
- `where (optional) [array]` The WHERE clause to filter records.

> count($table, $join, $column, $where)

- `table [string]` The table name.
- `join [array]` Table relativity for table joining.
- `column [string]` The target column will be counted.
- `where (optional) [array]` The WHERE clause to filter records.

**Return: [number]** The number of rows.

	$count = $database->count("account", [
		"gender" => "female"
	]);
	 
	echo "We have " . $count . " female users.";

## max
Get the maximum value for the column

> max($table, $column, $where)

- `table [string]` The table name.
- `column [string]` The target column will be calculated.
- `where (optional) [array]` The WHERE clause to filter records.

> max($table, $join, $column, $where)

- `table [string]` The table name.
- `join [array]` Table relativity for table joining.
- `column [string]` The target column will be calculated.
- `where (optional) [array]` The WHERE clause to filter records.

**Return: [number]** The maximum number of the column.

	$max = $database->max("account", "age", [
		"gender" => "female"
	]);

echo "The age of oldest female user is " . $max;

## min
Get the minimum value for the column

> min($table, $column, $where)

- `table [string]` The table name.
- `column [string]` The target column will be calculated.
- `where (optional) [array]` The WHERE clause to filter records.

> min($table, $join, $column, $where)

- `table [string]` The table name.
- `join [array]` Table relativity for table joining.
- `column [string]` The target column will be calculated.
- `where (optional) [array]` The WHERE clause to filter records.

**Return: [number]** The minimum number of the column.

	$min = $database->min("account", "age", [
		"gender" => "male"
	]);
	 
	echo "The age of youngest male user is " . $min;

## avg
Get the average value for the column

> avg($table, $column, $where)

- `table [string]` The table name.
- `column [string]` The target column will be calculated.
- `where (optional) [array]` The WHERE clause to filter records.

> avg($table, $join, $column, $where)

- `table [string]` The table name.
- `join [array]` Table relativity for table joining.
- `column [string]` The target column will be calculated.
- `where (optional) [array]` The WHERE clause to filter records.

**Return: [number]** The average number of the column.

	$average = $database->avg("account", "age", [
		"gender" => "male"
	]);
	 
	echo "The average age of male user is " . $average;

## sum
Get the total value for the column

> sum($table, $column, $where)

- `table [string]` The table name.
- `column [string]` The target column will be calculated.
- `where (optional)` [array] The WHERE clause to filter records.

> sum($table, $join, $column, $where)

- `table [string]` The table name.
- `join [array]` Table relativity for table joining.
- `column [string]` The target column will be calculated.
- `where (optional) [array]` The WHERE clause to filter records.

**Return: [number]** The total number of the column.

	$total = $database->sum("account", "money");
	 
	echo "We have $" . $total;

## id
Returns the ID of the last inserted row

> id()

**Return: [number]** The last inserted row ID.

You can use this function to check out the performed SQL queries for debug.

	$database->insert("account", [
		"user_name" => "foo",
		"email" => "foo@bar.com",
		"age" => 25
	]);
	 
	$account_id = $database->id();

## action
Start a transaction

> action( $callback )

- `$callback [function]` The transaction wrap for executing queries.

**Return: void**

Not every database or database engine supports transactions. You have to check before using it. All queries will be automatically committed inside the transaction wrap. You can also return false value to rollback the transactions.

	$database->action(function($database) {
		$database->insert("account", [
			"name" => "foo",
			"email" => "bar@abc.com"
		]);
	 
		$database->delete("account", [
			"user_id" => 2312
		]);
	 
		// If you want to  find something wrong, just return false to rollback the whole transaction.
		if ($database->has("post", ["user_id" => 2312]))
		{
			return false;
		}
	});

## query
Insert new records in a table

> query($query)

- query [string] The SQL query.

**Return: [object]** The PDOStatement object.

This function is for special and customized SQL query that used for complex query. With each data that will be inserted, please use quote function to prevent SQL injection.

	$database->query("CREATE TABLE table (
		c1 INT STORAGE DISK,
		c2 INT STORAGE MEMORY
	) ENGINE NDB;");
	 
	$data = $database->query("SELECT email FROM account")->fetchAll();
	print_r($data);

## quote
Quotes the string for query

> quote($string)

- `$string [string]` The target string.

**Return: [string]**

Quote() places quotes around the input string and escapes special characters within the input string.

	$data = "Medoo";
	 
	echo "We love " . $data; // We love Medoo
	 
	echo "We love " . $database->quote($data); // We love 'Medoo'

## pdo
Medoo is based on PDO object. You can access the PDO object directly via using $database->pdo, so that you can use all PDO functions if you needed, like prepare, transaction, rollBack or more. 

For more information about PDO class, read more about from: http://php.net/manual/en/class.pdo.php.

### Transaction

	$database->pdo->beginTransaction();
	 
	$database->insert("account", [
		"user_name" => "foo",
		"email" => "foo@bar.com",
		"age" => 25
	]);
	 
	/* Commit the changes */
	$database->pdo->commit();
	 
	/* Recognize mistake and roll back changes */
	$database->pdo->rollBack();

### Prepare
Sometime, if Medoo cannot process complex SQL query, you can use it like a PDO wrapper with PDO internal functions to process the query in case of SQL injection.

	$calories = 150;
	$colour = 'red';
	 
	$sth = $database->pdo->prepare('SELECT name, colour, calories
		FROM fruit
		WHERE calories < :calories AND colour = :colour');
	 
	$sth->bindParam(':calories', $calories, PDO::PARAM_INT);
	$sth->bindParam(':colour', $colour, PDO::PARAM_STR, 12);
	 
	$sth->execute();

## debug
Output the generated SQL without execute it.

> debug()

**Return: [object]** Medoo object with debug mode enabled

This feature will output the generated SQL query automatically without using echo or other function. Please keep removed it when you finish debugging. And check the following code will be executed normally if this query do not be executed.

	$database->debug()->select("bccount", [
		"user_name",
		"email"
	], [
		"user_id[<]" => 20
	]);
	 
	// Will output:
	// SELECT "user_name","email" FROM "bccount" WHERE "user_id" < 20
	 
	// [Multiple situation]
	// Output nothing
	$database->insert("account", [
		"user_name" => "foo",
		"email" => "foo@bar.com"
	]);
	 
	// Will output the generated query
	$post_id = $database->debug()->get("post", "post_id", ["user_name" => "foo"]);
	 
	// Be careful, this query will be executed.
	$database->update("account", [
		"level[+]" => 5,
		"post" => $post_id
	], [
		"user_name" => "foo"
	]);

## error
Return error information associated with the last operation.

> error()

**Return: [array]** an array of error information about the last operation performed

	$database->select("bccount", [
		"user_name",
		"email"
	], [
		"user_id[<]" => 20
	]);
	 
	var_dump($database->error());
	 
	// array(3) { [0]=> string(5) "42S02" [1]=> int(1146) [2]=> string(36) "Table 'my_database.bccount' doesn't exist" }

## log
Return the all executed queries.

> log()

**Return: [array]** an array with all executed queries

	$database->select("account", [
		"user_name",
		"email"
	], [
		"user_id[<]" => 20
	]);
	 
	$database->insert("account", [
		"user_name" => "foo",
		"email" => "foo@bar.com"
	]);
	 
	var_dump( $database->log() );
	// array(2) {
	//	[0]=> string(62) "SELECT "user_name","email" FROM "account" WHERE "user_id" < 20"
	//	[1]=> string(74) "INSERT INTO "account" ("user_name", "email") VALUES ('foo', 'foo@bar.com')"
	// }

## last
Return the last query performed. Like log(), but just return the last query.

> last()

**Return: [string]**

	$database->select("account", [
		"user_name",
		"email"
	], [
		"user_id[<]" => 20
	]);
	 
	$database->insert("account", [
		"user_name" => "foo",
		"email" => "foo@bar.com"
	]);
	 
	echo $database->last();
	// INSERT INTO "account" ("user_name", "email") VALUES ('foo', 'foo@bar.com')

## info
Get the information of database.

> info()

**Return: [string]**

	print_r($database->info());
	
	/*
	Array
	(
		[server] => Uptime: 5074  Threads: 1  Questions: 15  Slow queries: 0  Opens: 67  Flush tables: 1
			Open tables: 60  Queries per second avg: 0.002
		[client] => mysqlnd 5.0.10 - 20111026 - $Id: e707c415db32080b3752b232487a435ee0372157 $
		[driver] => mysql
		[version] => 5.6.10
		[connection] => localhost via TCP/IP
	)
	*/
