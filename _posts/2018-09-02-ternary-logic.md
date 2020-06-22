---
title: When computers stop thinking in binary
---

In this article I'm going to give a grounds up re-education on logic in computers, aimed at programmers with beginner experience in relational databases. It will start out with a tame recap of boolean logic, but get fairly hairy toward the end.

## A tiny primer on computer logic

Computers by and large have 2 logical states. True and false. 1 and 0. On and off. Yes and no. Through these states they're able to make logical decisions given the right programming.

I'm going to use SQL to show you some of the basics of logic:

```sql
> SELECT TRUE;
+------+
| TRUE |
+------+
|    1 |
+------+

> SELECT FALSE;
+-------+
| FALSE |
+-------+
|     0 |
+-------+
```

What we have here is an ASCII table of results, where the top is the column name, and the bottom is a row of data.

We can clearly see that `TRUE` is 1 and `FALSE` is 0. We can also do some basic comparisons to check whether something is true or false, where `=` checks for equality and `!=` checks for inequality.

```sql
> SELECT 0 = TRUE;
+----------+
| 0 = TRUE |
+----------+
|        0 |
+----------+

> SELECT 1 = TRUE;
+----------+
| 1 = TRUE |
+----------+
|        1 |
+----------+

> SELECT 1 != TRUE;
+-----------+
| 1 != TRUE |
+-----------+
|         0 |
+-----------+

> SELECT 0 != TRUE;
+-----------+
| 0 != TRUE |
+-----------+
|         1 |
+-----------+
```

We can compare different values and get boolean results from them:

```sql
> SELECT 10 > 15;
+---------+
| 10 > 15 |
+---------+
|       0 |
+---------+

> SELECT 10 < 15;
+---------+
| 10 < 15 |
+---------+
|       1 |
+---------+
```

## What is truth?

Let's turn away from the database for a minute and look at a programming language:

```python
>>> if 1:
...     print('Yay!')
... else:
...     print('Boo!')
... 
Yay!
>>> if 0:
...     print('Yay!')
... else:
...     print('Boo!')
... 
Boo!
>>> if 10:
...     print('Yay!')
... else:
...     print('Boo!')
... 
Yay!
>>> if -1:
...     print('Yay!')
... else:
...     print('Boo!')
... 
Yay!
```

Here we can see that -- in programming at least -- everything that isn't 0 is considered true. Let's see if that's the same in SQL:

```sql
> SELECT 10 = TRUE;
+-----------+
| 10 = TRUE |
+-----------+
|         0 |
+-----------+
> SELECT 10 != TRUE;
+------------+
| 10 != TRUE |
+------------+
|          1 |
+------------+
```

Huh. 10 is not true in SQL. Or is it? We have an `IF` in SQL as well, so let's see what it says:

```sql
> SELECT IF(TRUE, "Yay!", "Boo!");
+--------------------------+
| IF(TRUE, "Yay!", "Boo!") |
+--------------------------+
| Yay!                     |
+--------------------------+

> SELECT IF(FALSE, "Yay!", "Boo!");
+---------------------------+
| IF(FALSE, "Yay!", "Boo!") |
+---------------------------+
| Boo!                      |
+---------------------------+
```

Here we have an `IF` checking a value and returning either 'Yay!' or 'Boo!'. It should work with numbers too:

```sql
> SELECT IF(1, "Yay!", "Boo!");
+-----------------------+
| IF(1, "Yay!", "Boo!") |
+-----------------------+
| Yay!                  |
+-----------------------+

> SELECT IF(0, "Yay!", "Boo!");
+-----------------------+
| IF(0, "Yay!", "Boo!") |
+-----------------------+
| Boo!                  |
+-----------------------+

> SELECT IF(10, "Yay!", "Boo!");
+------------------------+
| IF(10, "Yay!", "Boo!") |
+------------------------+
| Yay!                   |
+------------------------+

> SELECT IF(-1, "Yay!", "Boo!");
+------------------------+
| IF(-1, "Yay!", "Boo!") |
+------------------------+
| Yay!                   |
+------------------------+
```

It's not that `TRUE` and `FALSE` are different concepts in databases, it's that `TRUE` and `FALSE` are just keywords that become the numbers 1 and 0, like in the first example.

```sql
> SELECT 10 = TRUE;
+-----------+
| 10 = TRUE |
+-----------+
|         0 |
+-----------+
> SELECT 10 = 1;
+--------+
| 10 = 1 |
+--------+
|      0 |
+--------+
```

## Ternary logic

Now comes ternary logic - a logic system that has more than just true and false. I'm using mariaDB. While it should more or less be the same for other relational databases, database compatability invokes flashbacks to early 2000's web browsers, so I daren't claim it's universal...

In relational databases there is a concept of 'missing data'. The [wikipedia page on ternary logic](https://en.wikipedia.org/wiki/Three-valued_logic) uses the term 'Unknown' to describe it, which suits our purposes just fine.

Unlike in programming languages, where `null` is often just a special type of 0, in SQL `NULL` means 'Unknown'. This gives rise to a third logical value.

```sql
> SELECT NULL;
+------+
| NULL |
+------+
| NULL |
+------+
```

As you can see, `NULL` is an actual value, as opposed to `TRUE` and `FALSE` which evaluate to 1 and 0.

Let's do the same comparisons we did earlier with `NULL` and see what happens:

```sql
> SELECT 0 = NULL;
+----------+
| 0 = NULL |
+----------+
|     NULL |
+----------+

> SELECT 1 = NULL;
+----------+
| 1 = NULL |
+----------+
|     NULL |
+----------+

> SELECT 1 != NULL;
+-----------+
| 1 != NULL |
+-----------+
|      NULL |
+-----------+

> SELECT 0 != NULL;
+-----------+
| 0 != NULL |
+-----------+
|      NULL |
+-----------+

> SELECT 10 < NULL;
+-----------+
| 10 < NULL |
+-----------+
|      NULL |
+-----------+

> SELECT 10 > NULL;
+-----------+
| 10 > NULL |
+-----------+
|      NULL |
+-----------+

> SELECT 10 = NULL;
+-----------+
| 10 = NULL |
+-----------+
|      NULL |
+-----------+
```

Weird huh? `X = Y` and `X != Y` should never have the same result in a boolean logic system, but with ternary logic it can happen. Go back and read that again but replace `NULL` with 'Unknown' in your head.

It makes sense that the result of asking whether X is equal to an unknown value is 'Unknown'. Asking whether X is not equal to an unknown value is also 'Unknown'. This is the tricky part of ternary logic!

Of course, if you have two unknown values, comparing them gives you an unknown result as well:

```sql
> SELECT NULL = NULL;
+-------------+
| NULL = NULL |
+-------------+
|        NULL |
+-------------+
```

But as we've seen with `TRUE` and `FALSE` logic in SQL can be a bit tricky, so let's see what `IF` has to say about it:

```sql
> SELECT IF(1 = NULL, "Yay!", "Boo!");
+------------------------------+
| IF(1 = NULL, "Yay!", "Boo!") |
+------------------------------+
| Boo!                         |
+------------------------------+

> SELECT IF(1 != NULL, "Yay!", "Boo!");
+-------------------------------+
| IF(1 != NULL, "Yay!", "Boo!") |
+-------------------------------+
| Boo!                          |
+-------------------------------+
```

Huh. These are both false! Or are they? We know that `1 = NULL` and `1 != NULL` both evaluate to `NULL`, so what does `NULL` do in an `IF`?

```sql
> SELECT IF(NULL, "Yay!", "Boo!");
+--------------------------+
| IF(NULL, "Yay!", "Boo!") |
+--------------------------+
| Boo!                     |
+--------------------------+
```

Aha! It's casting the `NULL` to `FALSE`!

Well, no. Not quite.

`NULL = TRUE` may be `NULL`, but that's because `TRUE` is the value 1. However, `NULL` is distinct from logical true. For example, unlike true and false, logically notting `NULL` just gets us more `NULL`:

```sql
> SELECT !TRUE;
+-------+
| !TRUE |
+-------+
|     0 |
+-------+

> SELECT !FALSE;
+--------+
| !FALSE |
+--------+
|      1 |
+--------+

> SELECT !NULL;
+-------+
| !NULL |
+-------+
|  NULL |
+-------+
```

Coming from a boolean logic system it's tempting to think of true as "Anything that isn't false" but that's not the case in ternary logic. The key takeaway is that `IF` really *is* checking for true.

## Missing data is literally `NULL`

Since `NULL` means 'Unknown' it's the perfect means to represent missing data. I'm going to set up a table of customers and addresses and see what happens.

```sql
> CREATE TABLE customers (
  id int NOT NULL AUTO_INCREMENT,
  email varchar(256) NOT NULL,
  name text NOT NULL,
  PRIMARY KEY (id),
  UNIQUE KEY email (email)
);

> CREATE TABLE addresses (
  id int NOT NULL AUTO_INCREMENT PRIMARY KEY,
  customer_id int NOT NULL,
  address text NOT NULL,
  FOREIGN KEY (customer_id) REFERENCES customers (id)
);
```

We have customers with unique email addresses, and addresses that have to have a customer attached to them. Let's see what happens when we select customers and their addresses:

```sql
> SELECT * FROM customers
LEFT JOIN addresses
ON addresses.customer_id = customers.id;
+----+------------------+------+------+-------------+---------------------------+
| id | email            | name | id   | customer_id | address                   |
+----+------------------+------+------+-------------+---------------------------+
|  2 | bill@example.com | Bill |    1 |           2 | 1 fictional street 1234AB |
|  2 | bill@example.com | Bill |    2 |           2 | 221b baker street         |
|  1 | bob@example.com  | Bob  | NULL |        NULL | NULL                      |
+----+------------------+------+------+-------------+---------------------------+
```

Now we've got Bill's information, but there are no addresses for Bob in the database. What better way to represent a missing address than `NULL` -- unknown!

But this can have a performance implication as well. If we decided to join another table based on the address, the database doesn't need to even look at it when the address is empty, because `X = NULL` will always be `NULL`!

## Abusing ignorance for database design

All these addresses are annoying for our customers. Let's give them a list of their addresses and let them pick a main address that will be used for future orders.

```sql
> ALTER TABLE addresses ADD main tinyint NOT NULL DEFAULT '0';
```

Now with the main set we can look for a specific customer's addresses:

```sql
> SELECT * FROM addresses WHERE customer_id = 2;
+----+-------------+---------------------------+------+
| id | customer_id | address                   | main |
+----+-------------+---------------------------+------+
|  1 |           2 | 1 fictional street 1234AB |    1 |
|  2 |           2 | 221b baker street         |    1 |
|  3 |           2 | sqrt(-1) abracadabra lane |    0 |
+----+-------------+---------------------------+------+

> SELECT * FROM addresses WHERE customer_id = 2 AND main = 1;
+----+-------------+---------------------------+------+
| id | customer_id | address                   | main |
+----+-------------+---------------------------+------+
|  1 |           2 | 1 fictional street 1234AB |    1 |
|  2 |           2 | 221b baker street         |    1 |
+----+-------------+---------------------------+------+
```

Oh no! We've got multiple main addresses! Let's fix that...

```sql
> UPDATE addresses SET main = 0 WHERE id = 2;
```

A database should really have constraints in place to prevent this from happening. Let's put a unique index on the `customer_id` and `main` columns so there can only be one main address:

```sql
> ALTER TABLE addresses ADD UNIQUE customer_id_main (customer_id, main);
ERROR 1062 (23000): Duplicate entry '2-0' for key 'customer_id_main'
```

Oh no! We only have 2 values in main (0 and 1) and we have 3 rows, so we can't make a unique constraint! But how does it know that it's not unique? Does it just compare all the values?

Aha! Let's make it a nullable field and set main to `NULL` when it's not default. The unique index won't be able to distinguish `NULL` from other `NULL`s and it'll work:

```sql
> ALTER TABLE addresses CHANGE main main tinyint NULL;
> UPDATE addresses SET main = NULL WHERE main = 0;
> ALTER TABLE addresses ADD UNIQUE customer_id_main (customer_id, main);
```

Hooray! Now we have a unique constraint!

Of course there was nothing stopping us from using `main = 1`, `main = 2`, `main = 3` and so on, but now we can have infinite alternative addresses without needing to keep track of which values we already used.

## Conclusion

Ternary logic is invaluable for data storage, since the data you're looking for might not exist. Be aware of the logic surrounding `NULL` to improve your ability to use databases.

See also: [wikipedia](https://en.wikipedia.org/wiki/Three-valued_logic)
