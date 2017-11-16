# Altering Tables in MariaDB

## Before Begining

To backup the clients table with mysqldump, we will enter the following from
the command-line:

```shell
$ mysqldump --user='username' --password='password' --add-locks db table > table.sql
```

If the table should need to be restored, the following can be run from the shell:

```shell
mysql --user='username' --password='password' db < table.sql
```

## Basic Addition and More

In order to add a column to an existing MariaDB table, one would use the ALTER
TABLE statement.

```mysql
ALTER TABLE clients
ADD COLUMN status CHAR(2);
```

Additionally, specifying the location of the new column is allowed.

```mysql
ALTER TABLE clients
ADD COLUMN address2 VARCHAR(25)
AFTER address;

ALTER TABLE clients
ADD COLUMN address3 VARCHAR(25)
FIRST;
```

## Changing One's Mind

```mysql
ALTER TABLE clients
CHANGE status status ENUM('AC', 'IA;
```

Notice that the column name status is specified twice. Although the column name
isn't being changed, it still must be respecified. To change the column name
(from `status` to `active`), while leaving the enumerated list the same, we
specify the new column name in the second position:

```mysql
ALTER TABLE clients
CHANGE status active ENUM('AC', 'IA');
```

Here we have the current column name and then the new column name, along with
the data type specifications (i.e., `ENUM`), even though the result is only a
name change. With the `CHANGE` clause everything must be stated, even items
that are not to be changed.

```mysql
ALTER TABLE clients
CHANGE address address1 VARCHAR(40),
MODIFY active ENUM('yes', 'no', 'AC', 'IA');

UPDATE clients
SET active = 'yes'
WHERE active = 'AC';

UPDATE clients
SET active = 'no'
WHERE active = 'IA';

ALTER TABLE clients
MODIFY active ENUM('yes', 'no');
```

The first SQL statement above changes address and modifies active in
preparation for the transition. Notice the use of a `MODIFY` clause. It works
the same as `CHANGE`, but it is only used for changing data types and not
column names. Therefore, the column name isn't rerespecified. Notice also that
there is a comma after the CHANGE clause. You can string several `CHANGE` and
`MODIFY` clauses together with comma separators. We've enumerated both the new
choices and the old ones to be able to migrate the data. The two UPDATE
statements  are designed to adjust the data accordingly and the last ALTER
TABLE statement is to remove the old enumerated choices for the status column.

```mysql
ALTER TABLE clients
DROP client_type;
```

This deletes `client_type` and its data, but not the whole table, obviously.
Nevertheless, it is a permanent and non-reversible action; there won't be a
confirmation request when using the mysql client. This is how it is with all
MariaDB DROP statements and clauses.

## The Default

To be able to specify a default value other than NULL, an ALTER TABLE statement
can be entered with a `SET` clause.

```mysql
ALTER TABLE clients
ALTER state SET DEFAULT 'LA';
```

Notice that the second line starts with `ALTER` and not `CHANGE`. If we change
our mind about having a default value for state, we would enter the following
to reset it back to NULL (or whatever the initial default value would be based
on the data type):

```mysql
ALTER TABLE clients
ALTER state DROP DEFAULT;
```

This particular `DROP` doesn't delete data, by the way.

## Indexes

What most newcomers to MariaDB don't seem to realize is that the index is
separate from the indexed column. To illustrate, let's take a look at the index
for clients using the SHOW INDEX statement:

```mysql
MariaDB [bookstore]> SHOW INDEX FROM clients \G;
*************************** 1. row ***************************
        Table: clients
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: cust_id
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
```

The text above show that behind the scenes there is an index associated with
`cust_id`. The column `cust_id` is not the index. Incidentally, the `\G` at the
end of the SHOW INDEX statement is to display the results in portrait instead
of landscape format. Before the name of an indexed column can be changed, the
index related to it must be eliminated. The index is not automatically changed
or deleted. So, a `DROP` clause for the index must be entered first and then a
`CHANGE` for the column name can be made along with the establishing of a new
index:

```mysql
ALTER TABLE clients
DROP PRIMARY KEY,
CHANGE cust_id client_id INT PRIMARY KEY;
```

The order of these clauses is necessary. The index must be dropped before the
column can be renamed. The syntax here is for a `PRIMARY KEY`. There are other
types of indexes, of course. To change a column that has an index type other
than a `PRIMARY KEY`. Assuming for a moment that `cust_id` has a `UNIQUE` index,
this is what we would enter to change its name:

```mysql
ALTER TABLE clients
DROP UNIQUE cust_id
CHANGE cust_id client_id INT UNIQUE;
```

Although the index type can be changed easily, MariaDB won't permit you to do
so when there are duplicate rows of data and when going from an index that
allows duplicates(e.g., `INDEX`) to one that doesn't (e.g., `UNIQUE`). If you
actually do want to eliminate the duplicates, though, you can add the `IGNORE`
flag to force the duplicates to be deleted:

```mysql
ALTER IGNORE TABLE clients
DROP INDEX cust_id
CHANGE cust_id client_id INT UNIQUE;
```

In this example, we're not only changing the indexed column's name, but we're
also changing the index type from `INDEX` to `UNIQUE`. And, again, the `IGNORE`
flag tells MariaDB to ignore any records with duplicate values for `cust_id`.

## Renaming & Shifting Tables

```mysql
RENAME TABLE clients
TO client_addresses;
```

The RENAME TABLE statement will also allows a table to be moved to another
database just by adding the receiving database's name in front of the new table
name, separated by a dot. Of course, you can move a table without renaming it.

```mysql
RENAME TABLE client_addresses
TO db2.client_addresses;
```

Sometimes developers want to resort the data somewhat permanently to the data
within the table based on a particular column or columns.

```mysql
ALTER TABLE client_addresses
ORDER BY city, name;
```

Now when the developer enters a SELECT statement without an ORDER BY clause,
the results are already ordered by the default of city and then name, at least
until more data is added to the table.

> **ORDER BY ignored as there is a user-defined clustered index in the table**
>
> 0. [Stack Overflow][stackoverflow]
>
> 1. [Row Order for MyISAM Tables][mysqlrefman]

[stackoverflow]: https://stackoverflow.com/a/29781290/5631625 ''
[mysqlrefman]: https://dev.mysql.com/doc/refman/5.7/en/alter-table.html 'Row Order for MyISAM Tables'
