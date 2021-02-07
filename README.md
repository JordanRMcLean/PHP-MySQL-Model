# MySQL Model Class

PHP Model class for simplifying common MySQL actions.
Intended to be extended for various tables - Users.php as example
Automatically connects with PDO object using persistent connection for re-use, eliminating the need for a DB connection step.
--------------------

**Set database credentials in the code line 16 - line 20**

------------------
Example usage (Users model example included in Users.php)
```php
$posts = new \Models\Model('posts');
$users = new \Models\Users();

$posts->set_where('id', 5)->delete();

$users->set_where('id', 4)->update([
	'posts'		=> 3,
	'active'	=> true
]);
```

## Constructing & Set Table.
Construct Model object with a table name or use set_table method at a later point.

```php
$model = new \Models\Model();

$user1 = $model->set_table('users')->get_first();
$post1 = $model->set_table('posts')->get_first();
```

## Setting Query Options
Before executing you can set up the various parameters of the SQL Query; where clauses, joins, limit, grouping etc.
All query option set functions return the model object so they are chain-able.


### Set fields
Set which fields to retrieve from the table using the set_fields method.

`$model->set_fields(['id', 'name', 'password']);`

### Set Where Clauses
You can set several WHERE clauses for the SQL query.

The simplest method is using `set_where` and providing a field and a value, which creates `WHERE field = value`
-------------------------
Multiple WHERE clauses are joined with 'AND' unless specified as an 'OR' WHERE using `set_or_where` :

`$model->set_or_where('inactive', 1);`
-------------------------
The operator assumed between field and value is '=' however an operator can be specified as a third parameter:

`$model->set_where('posts', 5, '>');` SQL: `AND posts > 5`
------------------------
You can use the IN() function of SQL, with `set_in_where` method.

`$model->set_in_where('id', ['2','3','4']);` SQL: `AND id IN(2, 3, 4)`
-----------------------
You can use a shortcut function `set_search` to set a common where clause in SQL.

`$model->set_search('name', 'jordan');` SQL: `AND name LIKE '%jordan%'`
----------------------
You have complete freedom with a custom where clause using the `set_custom_where` method.
If there is no AND or OR specified at the start of the provided WHERE clause then AND will be prepended.

`$model->set_custom_where('OR name != john')`

### Set Limit
Set the SQL query limit.

Specify 2 digits for the start point and second digit being the amount of results.
Or just a single digit would be just the amount of results.

`$model->set_limit(0, 5);`

### Set Order
Set the ordering of the SQL results. By default, order direction is ascending. Specify 'd', 'D', 'DESC' or 'desc' for descending

`$model->set_order('name', 'desc');` SQL: `ORDER BY name DESC`

### Set Grouping
Set the grouping of the SQL query.

`$model->set_group('role')` SQL: `GROUP BY role`

### Set Joins.
To set a joined table or multiple, there are 2 steps; defining the table and fields to join and then defining the join condition.
---------------------
Set the join table, fields and type of join using the `set_join_table` method. Default is a LEFT JOIN.

`$model->set_join_table('roles', ['role_id', 'role_name']);`
---------------------
Set the join condition, aka how it will be joined to the left table. The same table must be specified.
Then the left table value and the right table value.

`$model->set_join_condition($table, $left_table_value, $right_table_value, $operator = '=', $and_or = 'AND')`

Example:
```php
$users = new \Models\Users();

//join the roles table to get the role name.
$users->set_join_table('roles', ['role_name']);
$users->set_join_condition('roles', 'role_id', 'role_id');
```
SQL: `LEFT JOIN roles ON users.role_id = roles.role_id`

## Executing Queries
After setting the various parameters and criteria for the query, you can then get the results or update or delete.

### Preserve Settings
After a query executing function, the criteria of the query is cleared, so all WHERE clauses, limits etc will be reset. Use the preserve_settings method to change this and preserve all criteria until turned back off.

Example:
```php
//get the user with ID of 5, but preserve the setting of 'ID = 5'
$user = $users->set_where('id', 5)->preserve_settings()->get_first();

if( $user['inactive'] ) {
	//we can call delete here, and the WHERE clause of ID = 5 is still set.
	//SQL: DELETE users WHERE id = 5
	$user->delete();
}
```

**You will need to use the 'clear' method to reset all the criteria:** `$user->clear();`

### Select Query and get results.
Getting the results will always result in records being associative arrays with field names for keys, with the exception of `get_count`.

- `$model->get_results()` returns the result set that matches the rest criteria.
- `$model->get()` same as get_results
- `$model->get_first()` returns only the first matching result.
- `$model->get_count()` returns an integer, the number of results matching the criteria.

### Updating the matching records.
The update method allows you to update all matching rows by using an associative array of all fields with their new values.

Setting the value to '++' or '--' allows you to increment or decrement a rows field.

```php
$users->set_where('posts', 20, '>')->update([
	'Title'		=> 'Regular Poster'
]);

$users->set_where('id', 5)->update([
	'posts' 	=> '++'
]);
```

### Deleting matching records
Use the delete method to delete all matching records. This can't be undone and is assumed to already be confirmed.

`$users->set_where('id', 5)->delete();`

### Inserting Records
Insert a record using the insert method, supplying an associative array of fields and values. Inserting multiple records isn't supported yet but using a loop or calling the insert method multiple times won't be too harsh with a persistent connection.

Insert method ignores all criteria set as it doesn't apply.

```php
$users->insert([
	'name'		=> 'Jordan',
	'email' 	=> 'foo@bar.com',
	'password'	=> $password
]);
```

**Use the 'replace' method to perform an insert or replace SQL query.**

### Execute a query as-is
Use the raw_query method to execute your own SQL query, optionally with the provided variables (to avoid sql injection)
```php
$model->raw_query('SELECT * FROM users WHERE posts > :min AND posts < :max', [
	':min'	=> 5,
	':max'  => 10
]);
```


## Debug Mode
Statically set debug mode on for more detailed output of queries gone wrong; `\Models\Model::$debug = true;`

Access more info with:
- Last query: `$model->get_last_query();`
- Last PDOstatement object: `$model->get_last_statement();`
- The last result set: `$model->get_last_results();`
- The count of the last result set: `$model->get_last_result_count();`

## SQL Queries
Access how many queries have been executed throughout the script with `\Models\Model::$sql_queries`
