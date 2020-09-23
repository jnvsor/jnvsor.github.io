---
title: Migrating queries in old PHP applications
---

With PHP 8 around the corner, it's probably a bit late to write about migrating the `mysql_*` functions that were last seen in PHP 5, but we all know how long old code sticks around.

The migration has to be gradual because management never has time for a full rewrite, or they say they do then axe the project half way through when the next shiny comes along. More importantly it has to work without having a separate connection for PDO, since doubling the amount of database connections can be a problem.

Most importantly, the new code has to be significantly easier to use than the old code, or developers will keep using `mysql_real_escape_string` for as long as possible.

# The target code

Let's execute a query in `mysql_`:

```php
$last_logged_in = new DateTime('yesterday');
$list_of_user_ids = [1, 2, 3];
$user_input = $_POST['search'];

$sql = 'SELECT u.* FROM users u WHERE last_logged_in > ';
$sql .= '"'.$last_logged_in->format('Y-m-d H:i:s').'" ';

$param = [];
foreach ($list_of_user_ids as $id) {
    $param[] = '"'.((int) $id).'"';
}
$sql .= ' AND u.id IN('.implode(',', $param).') ';

$sql .= ' AND u.name LIKE "'.mysql_real_escape_string($user_input).'"';

$res = mysql_query($sql);
$result = [];
while ($row = mysql_fetch_assoc($res)) {
    $result[] = $row;
}
```

What an unholy mess! This is the kind of stuff that made people hate PHP back in the day.

Let's rewrite that in PDO and see what it looks like:

```php
$last_logged_in = new DateTime('yesterday');
$list_of_user_ids = [1, 2, 3];
$user_input = $_POST['search'];

$params = [$last_logged_in->format('Y-m-d H:i:s')];
$params = array_merge($params, $list_of_user_ids);
$params[] = $user_input;

$sql = '
    SELECT u.* FROM users u
    WHERE last_logged_in > ?
    AND u.id IN('.implode(',', array_fill(0, count($list_of_user_ids), '?')).')
    AND u.name LIKE ?';

$stmt = $db->prepare($sql);
$stmt->execute($params);
$result = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

Much better. It's more legible and less likely to be injected. But it's not exactly pleasant to work with. This is what we're going to aim for:

```php
$last_logged_in = new DateTime('yesterday');
$list_of_user_ids = [1, 2, 3];
$user_input = $_POST['search'];

$results = $db->fetchAll('
        SELECT u.* FROM users u
        WHERE last_logged_in > ?
        AND u.id IN(?)
        AND u.name LIKE ?
    ',
    [
        $last_logged_in,
        $list_of_user_ids,
        $db->unlike($user_input),
    ]
);
```

Now that's a clean query execution!

# To Mysqli

First off we need our old code to not explode on PHP 7, otherwise there's not much point in any of this.

Essentially most `mysql_` functions are exactly the same in mysqli, except you add the `i` and the connection object as the first parameter.

This allows you to have multiple connections running at the same time without having to call `mysql_select_db` to change the global database.

While you could find-replace through your whole codebase using `$GLOBALS[...]` as your connection parameter, you could just `composer require dshafik/php7-mysql-shim` and it will handle all of this for you.

Don't tell management it will work on PHP 7 until you've finished the job though!

# To PDO

In order to go to PDO we need some sort of a wrapper that will use the mysqli connection but has the same API as PDO. This way we can rewrite old queries in new code without doubling the amount of connections we make and without breaking transactions.

Luckily, such a wrapper already exists.

`doctrine/dbal` is the abstraction layer underpinning the ORM they're more well known for. It's a drop-in replacement for PDO, but it supports mysqli, MSSQL, Oracle, everything that PDO supports, as well as some other ones you've never heard of.

For some absurd reason it *doesn't* have a driver for `pg_` but it shouldn't be too hard to write one if you only have to support your own application.

You can either create the connection in DBAL and set the mysqli object in the shim, or create the connection in the shim and set the mysqli object in the DBAL connection.

For the former you'll need to put the connection into `Dshafik\MySQL::$last_connection` and replicate the `sha1` connection hash (`sha1($hostname.$username.$flags);`) as the key for the `Dshafik\MySQL::$connections` array.

For the latter you'll need to extend the driver and override the constructor, because while DBAL allows you to send it a premade PDO connection, the individual drivers won't.

Now you can already use the PDO example and the `mysql_` functions interchangeably.

While the `fetchAll` method will work, it won't automatically expand `DateTime` objects and arrays of parameters for you, and the `unlike` method was also custom. I [wrote about how to implement that](/blog/2017/02/03/extending-dbal-for-fun-and-profit/) a while back.

# Transactionals

One thing that DBAL does very well is transactionals. Transactionals allow you to wrap a piece of code in a closure and it will automatically create a transaction around it. If an exception is thrown it will roll back the transaction upon reaching the end of the closure.

The neat thing about this is that it keeps track of the nesting level, so you can nest these as many times as you want and it will always roll back if something goes wrong. Of course, you can't use this in conjunction with transactions started through `mysql_` because they won't keep track of the nesting level, so they'll need to be replaced.

Also, while DBAL will throw exceptions if encountering an error like a unique constraint or syntax error, mysqli will not. So you'll still need to add error checking to any legacy functions that you want to work with these transactionals.

Alternatively, you could copy-paste the shim and rewrite it to throw exceptions whenever something goes wrong.

# Finishing up

Eventually once you've moved all your `mysql_` over to DBAL, you can either drop back to PDO if you've been strict about keeping to it, or (more likely) stick with DBAL since you've gotten used to the more pleasant API.

Total time to set it up should be a few hours. Total time to migrate all your queries? There's the question for you...
