# Adding and Changing Data in MariaDB

## Adding Data

```mysql
INSERT table1
VALUES ('text1', 'text2', 'text3');

INSERT INTO table1 (col3, col1)
VALUES ('text3', 'text1');
```

Notice that the keyword `INTO` was added here. This is optional and has no
effect on MariaDB. It's only a matter of grammatical perference.

If you're going to insert data into a table and want to specify all of the
values except one (say the key column since it's an auto-incremented one), then
you could just give a value of `DEFAULT` to keep from having to list the columns.

Incidentally, you can give the column names even if you're naming all of them.
It's just unnecessary unless you're going to reorder them.

Multiple row insertions can be done like so:

```mysql
INSERT IGNORE
INTO table2
VALUES ('id1', 'text', 'text'),
    ('id2', 'text', 'text'),
    ('id2', 'text', 'text');
```

We are attempting to insert three rows of data into table2 for which the first
column happens to be a `UNIQUE` key field. Since the statement has an `IGNORE`
flag, duplicates will be ignored and not inserted, but the other rows will
still be inserted. So, the first and second rows above will be inserted and
the third one won't.

## Priority

An INSERT statement takes priority over read statements. An INSERT will lock
the table and force other clients to wait until it's finished. On a busy
MariaDB server that has many simultaneous requests for data, this could cause
users to experience delays when you run a script that performs a series of
INSERT statements. If you don't want user requests to be put on hold and you
can wait to insert the data, you could use the `LOW_PRIORITY` flag:

```mysql
INSERT LOW_PRIORITY
INTO table1
VALUES ('text1', 'text2', 'text3');
```

The `LOW_PRIORITY` flag will put the INSERT statement in queue, waiting for all
current and pending requests to be completed before it's performed. If new
requests are made while a low priority statement is waiting, then they are put
ahead of it in the queue. MariaDB does not begin to execute a low priority
statement until there are no other requests waiting. Once the transaction
begins, though, the table is locked and any other requests for data from the
table that come in after it starts must wait until it's completed. Because it
locks the table, low priority statements will prevent simultaneous insertions
from other clients even if you're dealing with a MyISAM table. Incidentally,
notice that the `LOW_PRIORITY` flag comes before the `INTO`.

One potential invonvenience with an `INSERT LOW_PRIORITY` statement is that the
client will be tied up waiting for the statement to be completed successfully.
So if you're inserting data into a busy server with a low priority setting
using the mysql client, your client could be locked up for minutes, maybe hours
depending on how busy your server is at the time. As an alternative either to
making other clients with read requests wait or to having your client wait,
you can use the `DELAYED` flag instead of the `LOW_PRIORITY` flag:

```mysql
INSERT DELAYED
INTO table1
VALUES ('text1', 'text2', 'text3');
```

MariaDB will take the request as a low priority one and put it on its list of
tasks to perform when it has a break. However, it will immediately release the
client so that the client can go on to enter other SQL statements or even exit.

Another advantage of this method is that multiple `INSERT DELAYED` requests are
batched together for block insertion when there is a gap, making the process
potentially faster than  `INSERT LOW_PRIORITY`. The flaw in this choice,
however, is that the client is never told if a delayed insertion is
successfully made or not. The client is informed of error messages when the
statement is entered — the statement has to be valid before it will be queued —
but it's not told of problems that occur after it's accepted. This brings up
another flaw: delayed insertions are stored in the server's memory. So if the
MariaDB daemon (mysqld) dies or is manually killed, then the transactions are
lost and the client is not notified of the failure. So DELAYED is not always a
good alternative.

## Contingent Additions

```mysql
INSERT INTO softball_team
(last, first, telephone)
    SELECT name_last, name_first, tel_home
    FROM company.employees
    WHERE softball = 'Y';
```

In this SQL statement the columns in which data is to be inserted into are
listed, then the complete SELECT statement follows with the appropriate WHERE
clause to determine if an employee is on the softball team. Since we're
executing this statement from the new database and since the table employees is
in a separate database called company, we have to specify it as you see here.
By the way, `INSERT...SELECT` statements cannot be performed on the same table.

## Replacement Data

When you're adding massive amounts of data to a table that has a key field, as
mentioned earlier, you can use the `IGNORE` flag to prevent duplicates from
being inserted, but still allow unique rows to be entered. However, there may
be times when you actually want to replace the rows with the same key fields
with the new ones. In such a situation, istead of using INSERT you can use a
REPLACE statement:

```mysql
REPLACE LOW_PRIORITY
INTO table2 (id, col1, col2)
VALUES ('id1', 'text', 'text'),
    ('id2', 'text', 'text'),
    ('id3', 'text', 'text');
```

Notice that the syntax is the same as an INSERT statement. The flags all have
the same effect, as well. Also multiple rows may be inserted, but there's no
need for the `IGNORE` flag since duplicates won't happen — the origals are just
overwritten. Actually, when a row is replaced, it's first deleted completely
and the new row is then inserted. Any columns without values in the new row
will be given the default values for the columns. None of the values of the old
row are kept. Incidentally, REPLACE will also allow you to combine it with a
SELECT statement as we saw with the INSERT statement earlier.

## Updating Data

If you want to change the data contained in existing records, but only for
certain columns, then you would need to use an UPDATE statement.

```mysql
UPDATE LOW_PRIORITY table3
SET col1 = 'text-a', col2 = 'text-b'
WHERE id < 100;
```

In the SQL statement here, we are changing the value of the two columns named
individually using the `SET` clause. Incidentally, the `SET` clause optionally
can be used in `INSERT` and `REPLACE` statements, but it eliminates the
multiple row option. In the statement above, we're also using a `WHERE` clause
to determine which records are changed: only rows with an id that has a values
less than 100 are updated. Notice that the `LOW_PROORITY` flag can be used with
this statement, too. The `IGNORE` flag can be used, as well.

A useful feature of the UPDATE statement is that it allows the use of the
current value of a column to update the same column. For instance, suppose you
want to add one day to the value of a date column where the date is a Sunday.
You could do the following:

```mysql
UPDATE table5
SET col_date = DATE_ADD(col_date, INTERVAL 1 DAY)
WHERE DAYOFWEEK(col_date) = 1;
```

There are a couple more twists that you can now do with the UPDATE statement:
if you want to update the rows in a specific order, you can add an ORDER BY
clause. You can also limit the number of rows that are updated with a LIMIT
clause. Below is an example of both of these clauses:

```mysql
UPDATE LOW_PRIORITY table3
SET col1 = 'text-a', col2 = 'text-b'
WHERE id < 100
ORDER BY col3 DESC
LIMIT 10;
```

If you want to refer to multiple tables in one UPDATE statement, you can do so
like this:

```mysql
UPDATE table3, table4
SET table3.col1 = table4.col1
WHERE table3.id = table4.id;
```

Here we see a join between the two tables named. In table3, the value of col1
is set to the value of the same column in table4 where the values of id from
each match. We're not updating both tables here; we're just accessing both. We
must specify the table name for each column to prevent an ambiguity error.
Incidentally, ORDER BY and LIMIT clauses aren't allowed with multiple table
updates.

There's another combination that you can do with the INSERT statement that we
didn't mention earlier. It involves the UPDATE statement. When inserting
multiple rows of data, if you want to note which rows had potentially duplicate
entries and which ones are new, you could add a column called status and change
it's value accordingly with a statement like this one:

```mysql
INSERT IGNORE INTO table1 (id, col1, col2, status)
VALUES ('1012', 'text', 'text', 'new'),
    ('1025', 'text', 'text', 'new'),
    ('1030', 'text', 'text', 'new')
ON DUPLICATE KEY
UPDATE status = 'old';
```

Because of the `IGNORE` flag, errors will not be generated, duplicates won't be
inserted or replaced, but the rest will be added. Because of the ON DUPLICATE
KEY, the column status of the original row will be set to old when there are
duplicate entry attempts. The rest will be inserted and their status set to new.
