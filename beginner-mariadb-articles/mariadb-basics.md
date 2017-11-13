# MariaDB Basics

## Connecting to MariaDB

```shell
$ mysql -u root -p -h localhost
```

## Creating a Structure

```mysql
CREATE DATABASE bookstore;

USE bookstore;

CREATE TABLE books ('
    isbn VARCHAR(20) PRIMARY KEY,
    title VARCHAR(50),
    author_id INT,
    publisher_id INT,
    year_pub VARCHAR(4),
    description TEXT);

CREATE TABLE authors (
    author_id INT AUTO_INCREMENT PRIMARY KEY,
    name_last VARCHAR(50),
    name_first VARCHAR(50),
    country VARCHAR(50));
```

## Minor Items

SQL statements end with a semi-colon (or a `\G`). You can spread an SQL
statement over multiple lines. However, it won't be passed to the server by the
client until you terminate it with a semi-colon and hit [Enter]. To cancel an
SQL statement once you've started typing it, enter `\c` and press [Enter].

## Entering Data

```mysql
INSERT INTO authors (name_last, name_first, country)
VALUES ('Kafka', 'Franz', 'Czech Republic');

INSERT INTO books (title, author_id, isbn, year_pub)
VALUES ('The Castle', '1', '0805211063', '1998');

INSERT INTO books (title, author_id, isbn, year_pub)
VALUES ('The Trial', '1', '0805210407', '1995'),
    ('The Metamorphosis', '1', '0553213695', '1995'),
    ('America', '1', '0805210644', '1995');

SELECT title
FROM books
LIMIT 5;

SELECT title, name_last
FROM books
JOIN authors
USING (author_id);

SELECT title AS 'Kafka Books'
FROM books
JOIN authors
USING (author_id)
WHERE name_last = 'Kafka';
```

## Changing & Deleting Data

```mysql
UPDATE books
SET title = 'Amerika'
WHERE isbn = '0805210644';

DELETE FROM books
WHERE author_id = '2034';
```
