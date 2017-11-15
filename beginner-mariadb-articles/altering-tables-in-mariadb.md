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
