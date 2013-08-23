# PHP 101 (Lesson 9, Part 2): SQLite My Fire!

## Not My Type

Whilst on the topic of ```INSERT```, remember my statement a couple pages back about how SQLite is typeless and so you can insert values of any type into any field? There is one important exception to this rule: a field marked as ```INTEGER PRIMARY KEY```. In SQLite, fields marked as ```INTEGER PRIMARY KEY``` do two important things: they provide a unique numeric identifier for each record in the table, and if you insert a ```NULL``` value into them, SQLite automatically inserts a value that is 1 greater than the largest value already present in that field.

```INTEGER PRIMARY KEY``` fields in SQLite thus perform the equivalent of ```AUTO_INCREMENT``` fields in MySQL, and are a convenient way of automatically numbering your records. Obviously, you can't insert non-numeric values into such a field, which is why I said they were an exception to the typeless rule. Read more about this at [http://www.sqlite.org/datatypes.html](http://www.sqlite.org/datatypes.html).

Since the ```books``` table used in the previous example already contains such a field (the id field), it's clear that every ```INSERT``` into it with a ```NULL``` value for that field generates a new record number. If you'd like to retrieve this number, PHP has a way to do that too – just use the ```sqlite_last_insert_rowid()``` function, which returns the ID of the last inserted row (equivalent to the ```mysql_insert_id()``` function in PHP's MySQL API).

To see this in action, update the ```if()``` loop in the middle of the previous script to include a call to ```sqlite_last_insert_rowid()```, as follows:

```php
<?php
// check to see if the form was submitted with a new record
if (isset($_POST['submit'])) {
    // make sure both title and author are present
    if (!empty($_POST['title']) && !empty($_POST['author'])) {
        // generate INSERT query
        $insQuery = "INSERT INTO books (title, author) VALUES (\"" . sqlite_escape_string($_POST['title']) . "\", \"" . sqlite_escape_string($_POST['author']) . "\")";
        // execute query
        $insResult = sqlite_query($handle, $insQuery) or die("Error in query: " . sqlite_error_string(sqlite_last_error($handle)));
        // print success message
        echo "<i>Record successfully inserted with ID " . sqlite_last_insert_rowid($handle) . "!</i><p />";
    } else {
        // missing data
        // display error message
        echo "<i>Incomplete form input. Record not inserted!</i><p />";
    }
}
?>
```

If you need to, you can also find out how many rows were affected using the ```sqlite_changes()``` function – try it for yourself and see!

## Starting From Scratch

You'll remember, from the beginning of this tutorial, that I suggested you initialize the ```library.db``` database using the SQLite commandline program. Well, that isn't the only way to create a fresh SQLite database – you can use PHP itself to do this, by issuing the necessary ```CREATE TABLE``` and ```INSERT``` commands through the ```sqlite_query()``` function. Here's how:

```php
<?php
// set path of database file
$db = $_SERVER['DOCUMENT_ROOT'] . "/../library2.db";
// open database file
$handle = sqlite_open($db) or die("Could not open database");
// create database
sqlite_query($handle, "CREATE TABLE books (id INTEGER PRIMARY KEY, title VARCHAR(255) NOT NULL, author VARCHAR(255) NOT NULL)") or die("Error in query: " . sqlite_error_string(sqlite_last_error($handle)));
// insert records
sqlite_query($handle, "INSERT INTO books (title, author) VALUES ('The Lord Of The Rings', 'J.R.R. Tolkien')") or die("Error in query: " . sqlite_error_string(sqlite_last_error($handle)));
sqlite_query($handle, "INSERT INTO books (title, author) VALUES ('The Murders In The Rue Morgue', 'Edgar Allan Poe')") or die("Error in query: " . sqlite_error_string(sqlite_last_error($handle)));
sqlite_query($handle, "INSERT INTO books (title, author) VALUES ('Three Men In A Boat', 'Jerome K. Jerome')") or die("Error in query: " . sqlite_error_string(sqlite_last_error($handle)));
sqlite_query($handle, "INSERT INTO books (title, author) VALUES ('A Study In Scarlet', 'Arthur Conan Doyle')") or die("Error in query: " . sqlite_error_string(sqlite_last_error($handle)));
sqlite_query($handle, "INSERT INTO books (title, author) VALUES ('Alice In Wonderland', 'Lewis Carroll')") or die("Error in query: " . sqlite_error_string(sqlite_last_error($handle)));
// print success message
echo "<i>Database successfully initialized!";
// all done
// close database file
sqlite_close($handle);
?>
```

Or, in PHP 5, you can use the object-oriented approach:

```php
<?php
// set path of database file
$file = $_SERVER['DOCUMENT_ROOT'] . "/../library3.db";
// create database object
$db = new SQLiteDatabase($file) or die("Could not open database");
// create database
$db->query("CREATE TABLE books (id INTEGER PRIMARY KEY, title VARCHAR(255) NOT NULL, author VARCHAR(255) NOT NULL)") or die("Error in query");
// insert records
$db->query("INSERT INTO books (title, author) VALUES ('The Lord Of The Rings', 'J.R.R. Tolkien')") or die("Error in query");
$db->query("INSERT INTO books (title, author) VALUES ('The Murders In The Rue Morgue', 'Edgar Allan Poe')") or die("Error in query");
$db->query("INSERT INTO books (title, author) VALUES ('Three Men In A Boat', 'Jerome K. Jerome')") or die("Error in query");
$db->query("INSERT INTO books (title, author) VALUES ('A Study In Scarlet', 'Arthur Conan Doyle')") or die("Error in query");
$db->query("INSERT INTO books (title, author) VALUES ('Alice In Wonderland', 'Lewis Carroll')") or die("Error in query");
// print success message
echo "<i>Database successfully initialized!";
// all done
// destroy database object
unset($db);
?>
```

## A Few Extra Tools

Finally, the SQLite API also includes some ancillary functions, to provide you with information on the SQLite version and encoding, and on the error code and message generated by the last failed operation. The following example demonstrates the ```sqlite_libversion()``` and ```sqlite_libencoding()``` functions, which return the version number and encoding of the linked SQLite library respectively:

```php
<?php
// version
echo "SQLite version: ".sqlite_libversion()."<br />";
// encoding
echo "SQLite encoding: ".sqlite_libencoding()."<br />";
?>
```

When things go wrong, reach for the ```sqlite_last_error()``` function, which returns the last error code returned by SQLite. Of course, this error code – a numeric value – is not very useful in itself; to convert it to a human-readable message, couple it with the ```sqlite_error_string()``` function. Consider the following example, which illustrates by attempting to run a query with a deliberate error in it:

```php
<?php
// set path of database file
$db = $_SERVER['DOCUMENT_ROOT'] . "/../library.db";
// open database file
$handle = sqlite_open($db) or die("Could not open database");
// generate query string
// query contains a deliberate error
$query = "DELETE books WHERE id = 1";
// execute query
$result = sqlite_query($handle, $query) or die("Error in query: " . sqlite_error_string(sqlite_last_error($handle)));
// all done
// close database file
sqlite_close($handle);
?>
```

Here's what the output looks like:

![SQLite error example](http://alemohamad.com/github/lesson9.02.png)

Note that although they might appear similar, the ```sqlite_last_error()``` and ```sqlite_error_string()``` functions don't work in exactly the same way as the ```mysql_errno()``` and ```mysql_error()``` functions. The ```mysql_errno()``` and ```mysql_error()``` functions can be used independently of each other to retrieve the last error code and message respectively, but the ```sqlite_error_string()``` is dependent on the error code returned by ```sqlite_last_error()```.

If your appetite has been whetted, you can read more about the things PHP can do with SQLite in Zend's [PHP 5 In Depth](http://devzone.zend.com/php5/articles/php5-sqlite.php) section.

And that's about all I have for you in this tutorial. More secrets await you in [Part 10](http://devzone.zend.com/16/php-101-part-10-a-session-in-the-cookie-jar) of PHP 101, so make sure you come back for that one!

