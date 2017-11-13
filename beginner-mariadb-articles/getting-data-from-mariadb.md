# Getting Data from MariaDB

## Basic Elements

```mysql
SELECT *
FROM books;

SELECT isbn, title, author_id
FROM books;

SELECT isbn, title, author_id
FROM books
LIMIT 5;

SELECT isbn, title, author_id
FROM books
LIMIT 5, 10;
```

## Selectivity and Order

```mysql
SELECT isbn, title
FROM books
WHERE author_id = 4729
LIMIT 5;

SELECT isbn, title
FROM books
WHERE author_id = 4729
ORDER BY title ASC
LIMIT 5;
```

## Friendlier and More Complicated

```mysql
SELECT isbn, title, CONCAT(name_first, ' ', name_last) AS author
FROM books
JOIN authors
-- ON authors.authod_id = books.authod_id
USING (author_id)
WHERE name_last = 'Dostoevsky'
ORDER BY title ASC
LIMIT 5;

SELECT isbn, title, CONCAT(name_first, ' ', name_last) AS author
FROM books
JOIN authors
USING (authod_id)
WHERE name_last
LIKE 'Dostoevsk%'
ORDER BY title ASC
LIMIT 5;
```

## Some Flags

If we would only want the first occurrence of a particular criteria to be
displayed, we could add the `DISTINCT` option.

```mysql
SELECT DISTINCT isbn, title
FROM books
JOIN authors
USING (author_id)
WHERE name_last = 'Dostoevsky'
ORDER BY title;
```

If we're retrieving data from an extremely busy database, by default any other
SQL statements entered simultaneously which are changing or updating data will
be executed before a SELECT statement. SELECT statements are considered to be
of lower priority. However, if we would like a particular SELECT statement to
be given a higher priority, we can add the keyword `HIGH_PRIORITY`.

```mysql
SELECT DISTINCT HIGH_PRIORITY isbn, title
FROM books
JOIN authors
USING (author_id)
WHERE name_last = 'Dostoevsky'
ORDER BY title;
```

If we add the `SQL_CALC_FOUND_ROWS` flag just before the column list, MariaDB
will calculate the number of columns found even if there is a LIMIT clause.

To retrieve this information, though, we will have to use the `FOUND_ROWS()`
function like below. This value is temporary and will be lost if the connection
is terminated. It cannot be retrieved by any other client session. It relates
only to the current session and the value for the variable when it was last
calculated.

```mysql
SELECT SQL_CALC_FOUND_ROWS isbn, title
FROM books
JOIN authors
USING (author_id)
WHERE name_last = 'Dostoevsky'
LIMIT 5;

SELECT FOUND_ROWS();
```
