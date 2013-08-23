# PHP 101 (Lesson 14, Part 2): Going to the Polls

## Adding More...

The next step in building this application is to provide the **administrator** with an easy way to add and delete questions and answers from the MySQL database. Consider the script *admin.php*, which provides the starting point for these tasks:

```php
<html>
    <head></head>
    <body>
        <h2>Administration</h2>
        <h4>Current Questions:</h4>
        <table border = '0' cellspacing = '10'>
            <?php
            // include configuration file
            include('config.php');

            // open database connection
            $connection = mysql_connect($host, $user, $pass) or die('ERROR: Unable to connect!');

            // select database
            mysql_select_db($db) or die('ERROR: Unable to select database!');

            // generate and execute query
            $query = 'SELECT qid, qtitle, qdate FROM questions ORDER BY qdate DESC';
            $result = mysql_query($query) or die('ERROR: $query. '.mysql_error());
            // if records are present 
            if (mysql_num_rows($result) > 0) {
                // iterate through resultset
                // print question titles
                while($row = mysql_fetch_object($result)) {
                ?>
                <tr>
                    <td><?php echo $row->qtitle; ?></td>
                    <td><font size = '-2'><a href = 'view.php?qid=<?php echo $row->qid; ?>'>view report</a></font></td>
                    <td><font size = '-2'><a href = 'delete.php?qid=<?php echo $row->qid; ?>'>delete</a></font></td>
                </tr>
                <?php
                }
            } else {
                // if no records are present, display message
                ?>
                <font size='-1'>No questions currently configured</font>
                <?php
            }

            // close connection
            mysql_close($connection);
            ?>
        </table>
        <h4>Add New Question:</h4>
        <form action = 'add.php' method ='post'>
            <table border = '0' cellspacing = '5'>
                <tr>
                    <td>Question</td>
                    <td><input type = 'text' name = 'qtitle'></td>
                </tr>
                <tr>
                    <td>Option #1</td>
                    <td><input type = 'text' name = 'options[]'></td>
                </tr>
                <tr>
                    <td>Option #2</td>
                    <td><input type = 'text' name = 'options[]'></td>
                </tr>
                <tr>
                    <td>Option #3</td>
                    <td><input type = 'text' name = 'options[]'></td>
                </tr>
                <tr>
                    <td>Option #4</td>
                    <td><input type = 'text' name = 'options[]'></td>
                </tr>
                <tr>
                    <td>Option #5</td>
                    <td><input type = 'text' name = 'options[]'></td>
                </tr>
                <tr>
                    <td colspan = '2' align = 'right'><input type = 'submit' name = 'submit' value = 'Add Question'></td>
                </tr>
            </table>
        </form>
    </body>
</html>
```

Here's what it looks like:

![Poll admin example](http://alemohamad.com/github/lesson14.02.png)

As you can see, there are two sections in this script. The first half connects to the database and prints a list of all available questions, with "view report" and "delete" links next to each (more on this these shortly). The second half contains a simple form for the administrator to add a new question and up to five possible answers.

Once the form is submitted, the data entered by the administrator gets ```POST```-ed to the script *add.php*, which validates it and saves it to the database. Here's the code:

```php
<html>
    <head></head>
    <body>
        <h2>Administration</h2>
        <?php
        if (isset($_POST['submit'])) {
            // check form input for errors

            // check title
            if (trim($_POST['qtitle']) == '') {
                die('ERROR: Please enter a question');
            }

            // clean up options
            // add valid ones to a new array
            foreach ($_POST['options'] as $o) {
                if (trim($o) != '') {
                    $atitles[] = $o;
                }
            }

            // check for at least two options    
            if (sizeof($atitles) <= 1) {
                die('ERROR: Please enter at least two answer choices');
            }

            // include configuration file
            include('config.php');

            // open database connection
            $connection = mysql_connect($host, $user, $pass) or die('ERROR: Unable to connect!');

            // select database
            mysql_select_db($db) or die('ERROR: Unable to select database!');
    
            // generate and execute query to insert question
            $query = "INSERT INTO questions (qtitle, qdate) VALUES ('{$_POST['qtitle']}', NOW())";
            $result = mysql_query($query) or die("ERROR: $query.".mysql_error());

            // get the ID of the inserted record
            $qid = mysql_insert_id();

            // reset variables
            unset($query);
            unset($result);

            // now insert the options
            // linking each with the question ID 
            foreach ($atitles as $atitle) {
                $query = "INSERT INTO answers (qid, atitle, acount) VALUES ('$qid', '$atitle', '0')";
                $result = mysql_query($query) or die("ERROR: $query. ".mysql_error());
            }

            // close connection
            mysql_close($connection);

            // print success message
            echo "Question successfully added to the database! Click <a href='admin.php'>here</a> to return to the main page";
        } else {
            die('ERROR: Data not correctly submitted');
        }
        ?>
    </body>
</html>
```

This script has a lot of things happening in it, so let's go through it step-by-step.

The first order of business is to **sanitize** the data entered by the user. There are a bunch of lines of code at the top of the script that do this, by checking for a question title and verifying that at least two answer choices are present. Notice my use of the ```trim()``` function to weed out any input that contains only empty spaces, and the ```sizeof()``` function that verifies the presence of at least two valid answer choices in the ```$POST['options']``` array. Any failure here results in an error message, and the script will refuse to proceed further.

Assuming all the data is acceptable, the next step is to **save** it to the database. First, the question is saved to the questions table via an ```INSERT``` query. The ID generated by this ```INSERT``` query is retrieved via the ```mysql_insert_id()``` function, and used to link the answer choices to the question when saving them to the answers table. Since there will be more than one answer choice for each question, a ```foreach()``` loop is used to repeatedly run an ```INSERT``` query – once for each possible answer choice (with MySQL 4.1 and the PHP 5 *mysqli* extension, you could instead use a prepared query here – feel free to experiment with this alternative yourself).

That takes care of adding questions and answers. Now, what about **removing** them?

Well, go back and take a look at the *admin.php* script. You'll see that, next to each question displayed, there is a "delete" link, which points to the script *delete.php*. You'll also see that this script is passed an input parameter, the question ID, on the URL itself. It's clear, then, that *delete.php* can use this input parameter to identify the corresponding question in the ```questions``` table (as well as its answers – the question ID is common to both tables, remember) and run a ```DELETE``` query to erase this data from the system.

Here's the code that actually does the work:

```php
<html>
    <head></head>
    <body>
        <h2>Administration</h2>
        <?php
        if ($_GET['qid'] && is_numeric($_GET['qid'])) {

            // include configuration file
            include('config.php');

            // open database connection
            $connection = mysql_connect($host, $user, $pass) or die('ERROR: Unable to connect!');

            // select database
            mysql_select_db($db) or die('ERROR: Unable to select database!');
    
            // generate and execute query
            $query = "DELETE FROM answers WHERE qid = '".$_GET['qid']."'";
            $result = mysql_query($query) or die("ERROR: $query. ".mysql_error());

            // generate and execute query
            $query = "DELETE FROM questions WHERE qid = '".$_GET['qid']."'";
            $result = mysql_query($query) or die("ERROR: $query. ".mysql_error());

            // close connection
            mysql_close($connection);

            // print success message
            echo "Question successfully removed from the database! Click <a href = 'admin.php'>here</a> to return to the main page";
        } else {
            die('ERROR: Data not correctly submitted');
        }
        ?>
    </body>
</html>
```

As you can see, the question ID passed through the ```GET``` method is retrieved by the script, and used inside two ```DELETE``` queries to remove all the records linked to that ID.

## Playing the Numbers

Now for possibly the most interesting section of this tutorial: Item #3. Obviously, once you have users and votes coming in, you'd like to see **reports** of how the votes are distributed. This involves connecting to the database, using the question ID to extract the correct record set, calculating the total number of votes and the percentage each option has of the total, and displaying this information in a table.

Here's what all that looks like in PHP:

```php
<html>
    <head></head>
    <body>
        <h2>Administration</h2>
        <?php
        if ($_GET['qid'] && is_numeric($_GET['qid'])) {
            // include configuration file
            include('config.php');

            // open database connection
            $connection = mysql_connect($host, $user, $pass) or die('ERROR: Unable to connect!');

            // select database
            mysql_select_db($db) or die('ERROR: Unable to select database!');

            // get the question 
            $query = "SELECT qtitle FROM questions WHERE qid = '" . $_GET['qid']."'";
            $result = mysql_query($query) or die("ERROR: $query. " . mysql_error());
            $row = mysql_fetch_object($result);
            echo '<h3>' . $row->qtitle . '</h3>';

            // reset variables
            unset($query);
            unset($result);
            unset($row);

            // find out if any votes have been cast
            $query = "SELECT qid, SUM(acount) AS total FROM answers GROUP BY qid HAVING qid = " . $_GET['qid'];
            $result = mysql_query($query) or die("ERROR: $query. " . mysql_error());
            $row = mysql_fetch_object($result);
            $total = $row->total;

            // if votes have been cast
            if ($total > 0) {
                // reset variables
                unset($query);
                unset($result);
                unset($row);

                // get individual counts
                $query = "SELECT atitle, acount FROM answers WHERE qid = '" . $_GET['qid']."'";
                $result = mysql_query($query) or die("ERROR: $query. " . mysql_error());

                // if records present
                if (mysql_num_rows($result) > 0) {
                    // print vote results
                    echo '<table border=1 cellspacing=0 cellpadding=15>';

                    // iterate through data
                    // print absolute and percentage totals
                    while($row = mysql_fetch_object($result)) {
                        echo '<tr>';
                        echo '<td>' . $row->atitle . '</td>';
                        echo '<td>' . $row->acount . '</td>';
                        echo '<td>' . round(($row->acount/$total) * 100, 2) . '%</td>';
                        echo '</tr>';
                    }
                    // print grand total
                    echo '<tr>';
                    echo '<td><u>TOTAL</u></td>';
                    echo '<td>' . $total . '</td>';
                    echo '<td>100%</td>';
                    echo '</tr>';
                    echo '</table>';
                }
            } else {
                // if votes have not been cast
                echo 'No votes cast yet';
            }

            // close connection
            mysql_close($connection);
        } else {
            die('ERROR: Data not correctly submitted');
        }
        ?>
    </body>
</html>
```

Here's an example of what the output might look like:

![Poll results example](http://alemohamad.com/github/lesson14.03.png)

This script, *view.php*, is activated from *admin.php* in much the same way as *delete.php* – a question ID is passed to it as an input parameter, and that ID is used to retrieve the corresponding answers and the votes each one has gathered. Once the answer set has been retrieved, the total number of votes submitted can be calculated, and the percentage share of each option in the total vote can be obtained. This data is then displayed in a simple HTML table.

You need to be careful when **converting** the absolute numbers into percentages – if there aren't any votes yet, you can get some pretty strange division by ```zero``` errors. To avoid this, the second query in the script uses MySQL's ```SUM()``` function and ```GROUP BY``` clause to obtain the total number of votes for a particular question. If this total is ```0```, no votes have yet been cast, and a message to that effect is displayed; if the total is greater than ```0```, the individual percentages are calculated.

## Exit Poll

The way things are currently set up, a single user can vote for a particular option more than once, thereby contravening one of the basic principles of democracy: one citizen, one vote. Although it's unlikely that many users would have the patience or inclination to do this; however, it *is* a hole, and *should* be plugged.

I've decided to set a **cookie** on the voter's system once the vote has successfully been cast. With the addition of a few lines of script, I can now check for the presence or absence of this cookie whenever a user tries to vote, and thereby decide whether or not to accept the vote.

Here's the code, which gets added to the very top of *user_submit.php*:

```php
<?php
// check if a cookie exists for this question
// deny access if it does
if (isset($_COOKIE) && !empty($_COOKIE)) {
    if ($_COOKIE['lastpoll'] && $_COOKIE['lastpoll'] == $_POST['qid']) {
        die('ERROR: You have already voted in this poll');
    }
}
// set cookie
setCookie('lastpoll', $_POST['qid'], time() + 2592000);
?>
```

With this in place, when a user votes, a cookie is set on the client browser, containing the ID for the question the user voted on. At each subsequent vote attempt, the script will first check for the presence of the cookie and, if it exists, the value of the cookie variable ```$_COOKIE['lastpoll']```. Only if the cookie is absent (indicating that this is a first-time voter) or the value of ```$_COOKIE['lastpoll']``` is different from the ID of the current poll question (indicating that the user has voted previously, but in response to a different question), will the vote be accepted.

This is by no means foolproof: any reasonably adept user can delete the cookie from the client's cache and vote again – but it does add a layer of security to the process. The ideal method, of course, would be to track voters on the server itself and deny votes to those who have already voted; and indeed, this is a feasible alternative if the site requires users to register with unique usernames before accessing its online polls.

Well, that's about it. Hopefully, this exercise has given you some insight into how PHP can be used to build a simple web application, and illustrated its power and flexibility as a rapid development tool for the web medium. Come back soon for [the final PHP 101](http://devzone.zend.com/24/php-101-part-15-no-news-is-good-news/), and one more do-it-yourself application!

