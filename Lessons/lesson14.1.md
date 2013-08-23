# PHP 101 (Lesson 14, Part 1): Going to the Polls

## The Real World

In the course of this series, I've taken you on a tour of PHP, teaching you everything you need to know to get started with this extremely powerful toolkit. You've learned how to process arrays, write functions, construct objects, and throw exceptions. You've also learned how to read user input from forms, search databases, and use cookies and sessions to maintain state. You're no longer the timid PHP newbie you used to be, but a bold and powerful PHP warrior, ready to take on anything the world (or your boss) throws at you...

There's only one drawback. Sure, you have all the weaponry... but you haven't ever used it in the real world. That's where these concluding segments of PHP 101 come in.

Over the final two chapters of this tutorial, I'm going to guide you through the process of creating two real-world PHP applications. Not only will this introduce you to practical application development with PHP, but it will also give you an opportunity to try out all the theory you've imbibed over the past weeks.

Drivers, start your engines!

## Burning Questions

The first application is fairly simple. It's a **polling system** for a web site, one which allows you to quickly measure what your visitors think about controversial issues (Kerry versus Bush, *to-mah-to* versus *to-mae-to*, that kind of thing). This online polling mechanism is fairly popular, because it lets you find out what your visitors are thinking, and makes your web site more dynamic and interactive.

I'm sure you've seen such a system in action on many web portals, and have a fairly clear mind's-eye picture of how it works. Nevertheless, it's good practice to write down exactly what the end product is supposed to do before you begin writing even a single line of code (geeks call this **defining requirements**).

1. There needs to be a mechanism by which the user can view a question, and then select from a list of possible answers. This "vote" then needs to be captured by the system, and added to the existing tally of votes for that question.
2. There needs to be a way for the site administrator to add new questions, or delete old ones. A MySQL database is a good place to store these questions and answers, but the administrator may not necessarily be proficient enough in SQL to change this data manually. Therefore, a form-based interface should be provided, to make the task simple and error-free.
3. Obviously, there also needs to be a way to view reports of the votes submitted for each question and its answers. The report would contain a count of the total votes registered for a question, as well as a breakdown of the votes each answer received.

An important question here is: Does it make sense to fix the number of available choices for each question? In my opinion, it doesn't, because the number of available choices is likely to change with each question. It's better to leave this number variable, and to allow the poll administrator to add as many choices per question as appropriate. We can, however, define an upper limit on the number of possible choices for each question – for argument's sake let's say five.

With this basic outline in mind, the next step is to **design a database** that supports these requirements.

## Designer Databases

This is a good time for you to [download the source code](http://devzone.zend.com/content/articles/663/php101-14-code.zip) for this application, so that you can refer to it throughout this tutorial. (Note that you will need a MySQL server and a PHP-capable Web server to run this code.)

Here's the database which I'll be using for this application, stored in *db.sql*:

```sql
#
# Table structure for table `questions`
#
CREATE TABLE `questions` (
  `qid` tinyint(3) unsigned NOT NULL auto_increment,
  `qtitle` varchar(255) NOT NULL default '',
  `qdate` date NOT NULL default '0000-00-00',
  PRIMARY KEY  (`qid`)
);

#
# Table structure for table `answers`
#
CREATE TABLE `answers` (
  `aid` tinyint(3) unsigned NOT NULL auto_increment,
  `qid` tinyint(4) NOT NULL default '0',
  `atitle` varchar(255) NOT NULL default '',
  `acount` int(11) NOT NULL default '0',
  PRIMARY KEY  (`aid`)
);
```

As you can see, this is pretty simple: one table for the questions, and one for the answers. The two tables are linked to each other by means of the ```qid``` field. With this structure, it's actually possible to have an infinite numbers of answers to each question. (This is not what we want – we'd prefer this number to be five or less – but the logic to implement this rule is better placed at the **application layer** than at the **database layer**).

To get things started, and to give you a better idea of how this structure plays in real life, let's ```INSERT``` a question into the database, together with three possible responses:

```sql
INSERT INTO `questions` VALUES (1, 'What version of PHP are you using?', '2004-10-15');
INSERT INTO `answers` VALUES (1, 1, 'PHP 3.x', 0);
INSERT INTO `answers` VALUES (2, 1, 'PHP 4.x', 0);
INSERT INTO `answers` VALUES (3, 1, 'PHP 5.x', 0);
```

Alternatively, you could create a new database and type source ```db.sql``` from the command prompt to load the table structures and data directly.

## Rocking the Vote

With the database taken care of, it's time to put together the web pages that the user sees. The first of these is *user.php*, which connects to the database to get the latest poll question and displays it together with all its possible responses. Take a look:

```php
<html>
    <head></head>
    <body>
        <?php
        // include configuration file
        include('config.php');

        // open database connection
        $connection = mysql_connect($host, $user, $pass) or die('ERROR: Unable to connect!');

        // select database
        mysql_select_db($db) or die('ERROR: Unable to select database!');

        // generate and execute query
        $query = "SELECT qid, qtitle FROM questions ORDER BY qdate DESC LIMIT 0, 1";
        $result = mysql_query($query) or die("ERROR: $query.".mysql_error());

        // if records are present
        if (mysql_num_rows($result) > 0) {
            $row = mysql_fetch_object($result);

            // get question ID and title
            $qid = $row->qid;

            echo '<h2>'.$row->qtitle .'</h2>';
            echo "<form method = post action = 'user_submit.php'>";

            // get possible answers using question ID
            $query = "SELECT aid, atitle FROM answers WHERE qid = '$qid'";
            $result = mysql_query($query) or die("ERROR: $query.".mysql_error());

            if (mysql_num_rows($result) > 0) {
                // print answer list as radio buttons
                while ($row = mysql_fetch_object($result)) {
                    echo "<input type = radio name = aid value = '".$row->aid."'>'".$row->atitle."'</input><br />";
                }
                echo "<input type = hidden name = qid value = '".$qid."'>";
                echo "<input type = submit name = submit value = 'Vote!'>";
            }
            echo '</form>';
        } else {
            // if no records present, display message
            echo '<font size="-1">No questions currently configured</font>';
        }

        // close connection
        mysql_close($connection);
        ?>
    </body>
</html>
```

Pay special attention to the SQL query I'm running: I'm using the ```ORDER BY```, ```DESC``` and ```LIMIT``` keywords to ensure that I get the latest record (question) from the ```questions``` table. Once the query returns a result, the record ID is used to get the corresponding answer list from the answers table. A ```while()``` loop is then used to print the answers as a series of radio buttons. The record ID corresponding to each answer is attached to its radio button; when the form is submitted, this identifier will be used to ensure that the correct counter is updated.

Note that if the database is empty, an error message is displayed. In this example, we've already inserted one question into the database, so you won't see it at all; however, it's good programming practice to ensure that all eventualities are accounted for, even the ones that don't occur that very often.

The file *config.php* included at the top of the script contains the access parameters for the MySQL database. This data has been placed in a separate file to make it easy to change it if you move the application to a new server. Take a look inside:

```php
<?php
// database access parameters
$host = 'localhost';
$user = 'guest';
$pass = 'guessme';
$db = 'db3';
?>
```

Here's what the form looks like:

![Poll example](http://alemohamad.com/github/lesson14.01.png)

Okay, now you've got the poll displayed. Users are lining up to participate, and clicks are being generated by the millions. What do you do with them?

The answer lies in the script that gets activated when a user casts a vote and submits the form described earlier. This script, *user_submit.php*, takes care of updating the vote counter for the appropriate question/answer combination. Take a look:

```php
<html>
    <head></head>
    <body>
        <?php
        if (isset($_POST['submit'])) {
            if (!isset($_POST['aid'])) {
                die('ERROR: Please select one of the available choices');
            }

            // include configuration file
            include('config.php');

            // open database connection
            $connection = mysql_connect($host, $user, $pass) or die('ERROR: Unable to connect!');

            // select database
            mysql_select_db($db) or die('ERROR: Unable to select database!');
    
            // update vote counter
            $query = "UPDATE answers SET acount = acount + 1 WHERE aid = ".$_POST['aid']." AND qid = ".$_POST['qid'];
            $result = mysql_query($query) or die("ERROR: $query. ".mysql_error());

            // close connection
            mysql_close($connection);

            // print success message    
            echo 'Your vote was successfully registered!';
        } else {
            die('ERROR: Data not correctly submitted');
        }
        ?>
    </body>
</html>
```

This script first checks to ensure that an answer has been selected, by verifying the presence of the answer ID ```$_POST['aid']```. Assuming the ID is present, the script updates the database to reflect the new vote and displays an appropriate message.

Now, flip back through your notebook and look at the initial requirement list. Yup, you can cross off Item #1. Onwards to Item #2...
