It's a sign of a mature community when it settles on common tools to solve common problems. Doctrine's DBAL is one such tool. The competing third party database layers for PHP include AdoDb and the Typo3 layer built on top of it.

AdoDb development seems somewhat slow, with the last commit at the time of writing being over a month ago. The Typo3 layer built on top of it meanwhile was rendered moot by it's being replaced in October by - you guessed it - Doctrine DBAL.

With some searching I've not been able to find any other database layer recent or relevant enough to mention. It's safe to say that doctrine/dbal is the database king of the PHP ecosystem. I'll refer to it simply as "DBAL" from here on in.

## It's not all perfect

DBAL is an excellent library, but it still has some issues.

First off, the way it re-throws PDO exceptions when PDO fails to connect leads to an annoying security vulnerability. Since the PDO constructor takes the username and password directly and throws an exception on a connection or authentication failure, a misconfigured database connection will leak login credentials in plaintext through the exception stack trace by default.

This is less a flaw of DBAL and more a flaw of PDO itself. Luckily, we can fix it. While in PDO the database connection is made as soon as it's constructed, DBAL waits to construct the PDO instance until the user makes his first query. This lazy instantiation means a request without a query won't make a database connection at all, and offers some nice benefits for things like master-slave connections.

Thanks to this property, creating the DBAL instance is unlikely to throw an exception, and with one little override we can get rid of the stack trace problem. A quick look at `\Doctrine\DBAL\Connection` shows the first call to a function containing the username and password is made in [`Connection::connect`][1] - so if we want to keep the credentials out of our stack trace we have to rethrow from the connect method itself.

{% highlight php %}
<?php

public function connect()
{
    try {
        return parent::connect();
    }
    catch (\Exception $e) {
        throw new \Exception($e->getMessage());
    }
}
{% endhighlight %}

By throwing a brand new exception and only copying in the message from the original one, we ensure the stack trace doesn't contain the database credentials. Something similar can be done in `\Doctrine\DBAL\Connections\MasterSlaveConnection::connectTo`.

## Prepared and unescaped

While prepared statements handle the heavy lifting that was once delegated to horrors like `mysql_real_escape_string`, there are some scenarios where prepared statements won't help you. Database names, table names, and column names are one example, but DBAL provides a wonderful method to escape these for you: [`Connection::quoteIdentifier`][2] turns `db.table` into `` `db`.`table` `` on MySQL. (Though it may not be secure, and the documentation warns you that this is a stupid idea and likely to lead to problems)

The other notable area that prepared statements won't help you is in a `LIKE`. Similarly to `REGEX` and other operations who's behaviour is a component of a string by definition, user input into a `LIKE` is not escaped (Since prepared statements don't escape in the first place) and can lead to large slowdowns and at worst a denial of service vulnerability.

Escaping strings for use in `LIKE` isn't hard, but it is verbose. Seems like the perfect thing to throw into our `ConnectionForFunAndProfit`:

{% highlight php %}
<?php

public function unlike($string)
{
    return str_replace(array('\\', '_', '%'), array('\\\\', '\\_', '\\%'), $string);
}
{% endhighlight %}

## The array of gold at the end of the rainbow

DBAL has a fantastic type system that lets you take objects, arrays, and other wonderful things, and automatically converts them to the right type for storage into your database. Now while you can obviously add new types by extending `executeQuery` and `executeUpdate` and altering the parameters before passing them on, that's not what I'm going to show you here.

Despite these wonderful types DBAL doesn't use them by default. If you want it to expand an identifier into an array for instance you need to supply the type:

{% highlight php %}
<?php

$dbal->fetchAll(
    'SELECT * FROM users WHERE active = ? AND id IN(?)',
    [true, [1, 2, 3, 4]],
    [null, \Doctrine\DBAL\Connection::PARAM_INT_ARRAY]
);
{% endhighlight %}

Well that kinda sucks. Let's do that automatically. Overload the `executeQuery` and `executeUpdate` methods and run this to set `PARAM_STR_ARRAY` and `DATETIMETZ` automatically:

{% highlight php %}
<?php

protected function implicitTypes($params, $types)
{
    if (is_int(key($params))) {
        $params = array_values($params);
        $types = array_values($types);
    }

    foreach ($params as $key => $param) {
        if (isset($types[$key])) {
            continue;
        }

        if (is_array($param)) {
            $types[$key] = \Doctrine\DBAL\Connection::PARAM_STR_ARRAY;
        } elseif ($param instanceof \DateTime) {
            $types[$key] = \Doctrine\DBAL\Types\Type::DATETIMETZ;
        } else {
            $types[$key] = null;
        }
    }

    return $types;
}
{% endhighlight %}

Now your `fetch*`, `insert`, `update`, `delete`, `executeQuery` and `executeUpdate` calls will automatically transform arrays into multiple parameters, and `DateTime` objects into strings. You can always override these by supplying a type explicitly of course.

This last one's clearly a dirty hack that needs to be tweaked for your use case, but boy does it make things easier!

## The final product

So we've fixed a potential security vulnerability, added a helper function for escaping `LIKE` strings, and automatically handled array and `DateTime` parameters. Here's the final file:

<script src="https://gist.github.com/jnvsor/554570e314eb11c2302d8b56bd280e51.js"></script>


[1]: https://github.com/doctrine/dbal/blob/7068f20832671d4da6623de79675fd862a0b992b/lib/Doctrine/DBAL/Connection.php#L360
[2]: https://github.com/doctrine/dbal/blob/7068f20832671d4da6623de79675fd862a0b992b/lib/Doctrine/DBAL/Connection.php#L753
