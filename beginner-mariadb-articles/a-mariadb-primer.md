# A MariaDB Primer

## Logging into MariaDB

```shell
$ mysql -u //user_name// -p -h //ip_address// //db_name//
```

## The Basics of a Database

```mysql
CREATE DATABASE IF NOT EXISTS test;

USE test;

CREATE TABLE IF NOT EXISTS books (
    BookID INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
    Title VARCHAR(100) NOT NULL,
    SeriesID INT, AuthorID INT);

CREATE TABLE IF NOT EXISTS authors (
    id INT NOT NULL PRIMARY KEY AUTO_INCREMENT);

CREATE TABLE IF NOT EXISTS series (
    id INT NOT NULL PRIMARY KEY AUTO_INCREMENT);

INSERT INTO books (Title, SeriesID, AuthorID)
VALUES ('The Fellowship of the Ring', 1, 1),
    ('The Two Towers', 1, 1),
    ('The Return of the King', 1, 1),
    ('The Sum of All Men', 2, 2),
    ('Bortherhood of the Wolf', 2, 2),
    ('Wizardborn', 2, 2),
    ('The Hobbbit', 0, 1);

SHOW TABLES;

DESCRIBE books;

SELECT * FROM books;
```

## Inserting Data

```mysql
INSERT INTO books (Title, SeriesID, AuthorID)
VALUES ('Lair of Bones', 2, 2);
```

## Modifying Data

```mysql
UPDATE books
SET Title = 'The Hobbit'
WHERE BookID = 7;
```
