# PHP 101 (Lesson 13, Part 2): The Trashman Cometh

## A Regular Guy

So far, the validation routines have been fairly simple- checking dates, checking for required values, and checking data type or size. Often, however, you need more sophisticated validation – for example, to test whether an email address or telephone number is written in the correct format. To accomplish these more complex validation tasks, clever PHP programmers turn to **regular expressions**.

Regular expressions, aka *regex*, are a powerful tool for **pattern matching and substitution**. They are commonly associated with almost all UNIX-based tools, including editors like vi, scripting languages like Perl and PHP, and shell programs like awk and sed. You'll even find them in client-side scripting languages like JavaScript. Kinda like Madonna, their popularity cuts across languages and territorial boundaries.

A regular expression lets you build patterns using a set of special characters. These patterns can then be compared with text in a file, data entered into an application, or input from a form filled in by users on a web site. Depending on whether or not there's a match, appropriate program code can be executed. Regular expressions thus play an important role in the decision-making routines of web applications.

A regular expression can be as simple as this:

```
/love/
```

All this does is match the pattern *love* in the text it's applied to. Like many other things in life, it's simpler to get your mind around the pattern than the concept - but that's neither here nor there.

How about something a little more complex? The pattern ```/fo+/``` would match the words *fool*, footsie and *four-seater*. Try it:

```php
<?php
$array = array('fool', 'footsie', 'four-seater');
foreach ($array as $element) {
    if (preg_match('/fo+/', $element)) echo "$element gives a match<br />\n";
}
?>
```

And although it's a pretty silly example, you have to admit it's realistic – after all, who but fools in love would play footsie in a four-seater?

The ```+``` symbol used in the expression is called a **metacharacter** – a character that has a special meaning when used within a pattern. The ```+``` metacharacter is used to match **one or more occurrences of the preceding character** - in the example above, that would be the letter ```f``` followed by one or more occurrences of the letter ```o```.

Similar to the ```+``` metacharacter are ```*``` and ```?```, which are used to match **zero or more occurrences of the preceding character**, and **zero or one occurrence of the preceding character**, respectively. So ```/ab*/``` would match *aggressive*, *absolutely* and *abbey*, while ```/Ron?/``` would match *Ronald*, *Roger* and *Roland*, though not *Rimbaud* or *Mona*.

In case all this seems a little too imprecise, you can also **specify a range for the number of matches**. For example, the regular expression ```/ron{2,6}/``` would match *ronny* and *ronnnnnny!*, but not *ron*. The numbers in the curly braces represent the lower and upper values of the range to match; you can leave out the upper limit for an open-ended range match.

Just as you can specify a range for the number of characters to be matched, you can also **specify a range of characters**. For example, the range ```/[A-Z]/``` would match any string containing an upper-case alphabetic character, while ```/[a-z]/``` would match any lowercase letters, and ```/[0-9]/``` would match all numbers between 0 and 9.

Using these three character ranges, it's pretty easy to create a regular expression to **match an ordered alphanumeric field**: ```/([a-z][A-Z][0-9])+/``` would match an alphanumeric string given the same character type order, such as *aB2*, but not *abc*. Note the parentheses around the patterns – contrary to what you might think, these are not there purely to confuse you; they come in handy when **grouping sections of a regular expression** together.

Of course, this is just the tip of the regular expression iceberg. There are many more metacharacters, and innumerable ways in which they can be combined to create powerful pattern-matching rules. For an in-depth introduction, take a look at [http://www.melonfire.com/community/columns/trog/article.php?id=2](http://www.melonfire.com/community/columns/trog/article.php?id=2), the reference pages at [http://it.metr.ou.edu/regex/](http://it.metr.ou.edu/regex/), and the PHP manual pages at [http://www.php.net/manual/en/ref.regex.php](http://www.php.net/manual/en/ref.regex.php) and [http://www.php.net/manual/en/ref.pcre.php](http://www.php.net/manual/en/ref.pcre.php). You can find a bunch of sample regular expressions for all manner of applications at [http://www.regexlib.com/](http://www.regexlib.com/).

## A Pattern Emerges

In PHP, regular expression matching takes place with the ```ereg()``` or ```preg_match()``` functions (```ereg()``` also comes in a case-insensitive version called ```eregi()```). These functions, which differ marginally from each other in their semantics, can be used to test user input against pre-defined patterns and thus catch invalid data before it gets into your application. The most common example of regex usage in PHP is, of course, the email address validator... and since I'm a slave to tradition, that's also my first example. Take a look:

```php
<html>
    <head></head>
    <body>
        <?php
        if (!isset($_POST['submit'])) {
        ?>
            <form action = '<?php $_SERVER['PHP_SELF'] ?>' method = 'post'>
                Email address:
                <br />
                <input type = 'text' name = 'email'>
                <input type = 'submit' name = 'submit' value = 'Save'>
            </form>
        <?php
        } else {
            // check email address
            if (!ereg('^([a-zA-Z0-9])+([\.a-zA-Z0-9_-])*@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-]+)*\.([a-zA-Z]{2,6})$', $_POST['email'])) {
                die("Dunno what that is, but it sure isn't an email address!");
            }

            // process the data
            echo "The email address {$_POST['email']} has a valid structure. Doesn't mean it works!";
        }
        ?>
    </body>
</html>
```

Here, the pattern ```/^([a-zA-Z0-9])+([\.a-zA-Z0-9_-])*@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-]+)*\.([a-zA-Z]{2,6})$/``` (try saying that fast!) is a regular expression that matches **the basic format for a user@host email address**. Input which matches this pattern will be accepted; input which doesn't will trigger a piercing siren. Notice that ```ereg()``` doesn't need the same delimiters as the faster ```preg_match()```, which complains if it doesn't get a ```/``` at each end of the expression.

Here's another example, this one good for testing **international phone numbers**:

```php
<html>
    <head></head>
    <body>
        <?php
        if (!isset($_POST['submit'])) {
        ?>
            <form action = '<?php $_SERVER['PHP_SELF'] ?>' method = 'post'>
                Phone number (with country/area codes):
                <br />
                <input type = 'text' name = 'tel'>
                <input type = 'submit' name = 'submit' value = 'Save'>
            </form>
        <?php
        } else {
            // check phone number
            if (!preg_match('/^(\+|00)[1-9]{1,3}(\.|\s|-)?([0-9]{1,5}(\.|\s|-)?){1,3}$/', $_POST['tel'])) {
                die ("Dunno what that is, but it sure isn't an international phone number!");
            }

            // process the data
            echo "{$_POST['tel']} has a valid structure. Doesn't mean it works!";
        }
        ?>
    </body>
</html>
```

If you play with this a bit, you'll see that it'll accept any of the numbers *+1.212.1234.4567*, *+44 1865 123456* and *0091 11 1234 5678*… even though each is formatted differently. Mostly this is because of my use of the ```|``` separator in the regular expression, which functions as logical OR and makes it possible to create a pattern that **supports alternatives internally**. Obviously you can tighten the pattern up as necessary. For example, if you're in India and your application only supports Indian phone numbers, you can fix the pattern so that it expects ```91``` (India's country code) as the first two digits of the number.

It's interesting to try rewriting some of our earlier validation routines using regular expressions. Here's an alternative version of one of the early examples in this tutorial, rewritten to use ```ereg()``` instead of ```intval()```, ```is_numeric()``` and ```isset()```:

```php
<html>
    <head></head>
    <body>
        <?php
        if (!isset($_POST['submit'])) {
        ?>
            <form action='<?php $_SERVER['PHP_SELF'] ?>' method='post'>
                How many sandwiches would you like? (min 1, max 9)
                <br />
                <input type='text' name='quantity'>
                <br />
                <input type='submit' name='submit' value='Save'>
            </form>
        <?php
        } else {
            // check for required data
            if (!ereg('^[1-9]$', $_POST['quantity'])) {
                die('ERROR: That is an invalid quantity!');
            }

            // process the data
            echo "I'm making you {$_POST['quantity']} sandwiches. Hope you can eat them all!";
        }
        ?>
    </body>
</html>
```

Notice how a single regular expression here replaces four separate tests in the earlier version, and how much more compact the result is. It's precisely this power and flexibility that make regular expressions such an important part of the input validation toolkit.

## Back to Class

Now that you know the basics of input validation, it should be clear to you that this is a task you'll be performing often. It therefore makes sense to create a reusable library of functions for input validation, which you can use every time an application needs its input checked for errors. That's precisely what I'm going to do next - create a PHP class that exposes basic object methods for data validation and error handling, and then use it to validate a form.

Here's the class definition, ```class.formValidator.php```, written for PHP 5. You could adapt it to PHP 4 by simply getting rid of the ```public``` and ```private``` markers on the class methods and making the private ```errorList``` property a ```var```. The rest of the following scripts run under either PHP version.

```php
<?php
// PHP 5
// class definition
// class encapsulating data validation functions
class formValidator {

    // define properties
    private $_errorList;

    // define methods
    // constructor
    public function __construct() {
        $this->resetErrorList();
    }

    // initialize error list
    private function resetErrorList() {
        $this->_errorList = array();
    }

    // check whether input is empty
    public function isEmpty($value) {
        return (!isset($value) || trim($value) == '') ? true : false;
    }

    // check whether input is a string
    public function isString($value) {
        return is_string($value);
    }

    // check whether input is a number
    public function isNumber($value) {
        return is_numeric($value);
    }

    // check whether input is an integer
    public function isInteger($value) {
        return (intval($value) == $value) ? true : false;
    }

    // check whether input is alphabetic
    public function isAlpha($value) {
        return preg_match('/^[a-zA-Z]+$/', $value);
    }

    // check whether input is within a numeric range
    public function isWithinRange($value, $min, $max) {
        return (is_numeric($value) && $value >= $min && $value <= $max) ? true : false;
    }
    
    // check whether input is a valid email address
    public function isEmailAddress($value) {
        return eregi('^([a-z0-9])+([\.a-z0-9_-])*@([a-z0-9_-])+(\.[a-z0-9_-]+)*\.([a-z]{2,6})$', $value);
    }

    // check if a value exists in an array
    public function isInArray($array, $value) {
        return in_array($value, $array);
    }

    // add an error to the error list
    public function addError($field, $message) {
        $this->_errorList[] = array('field' => $field, 'message' => $message);
    }

    // check if errors exist in the error list
    public function isError() {
        return (sizeof($this->_errorList) > 0) ? true : false;
    }

    // return the error list to the caller
    public function getErrorList() {
        return $this->_errorList;
    }

    // destructor
    // de-initialize error list
    public function __destruct() {
        unset($this->_errorList);
    }

// end class definition
}

?>
```

Stripped down to its bare bones, this ```formValidator``` class consists of two primary components.

The first is a series of methods that accept the data to be validated, test this data to see whether or not it is valid (however "valid" may be defined within the scope of the method), and return a Boolean result code. Here's a list of the supported methods:

* ```isEmpty()``` – tests if a value is an empty string
* ```isString()``` – tests if a value is a string
* ```isNumber()``` – tests if a value is a numeric string
* ```isInteger()``` – tests if a value is an integer
* ```isAlpha()``` – tests if a value consists only of alphabetic characters
* ```isEmailAddress()``` – tests if a value is an email address
* ```isWithinRange()``` – tests if a value falls within a numeric range
* ```isInArray()``` – tests if a value exists in an array

Obviously, the list above is not exhaustive – you should feel free to add to it as per your own requirements.

In earlier examples in this tutorial, I set things up so that the data validation routine would terminate script processing immediately with ```die()``` if it encountered an input error. In the real world, such abrupt termination on the first error is not usually a good idea; instead, it's more efficient to process *all* the user's input, identify *all* the errors, and then list them for the user to correct at once.

That's where the second component of this class comes in. It's a PHP array that holds a list of all the errors encountered during the validation process, and some methods to manipulate this structure. Here's a list:

* ```isError()``` – check if any errors exist in the error list
* ```addError()``` – add an error to the error list
* ```getErrorList()``` – retrieve the current list of errors
* ```resetErrorList()``` – reset the error list

This might all seem somewhat abstruse to you at the moment. Let's jump into a practical example and all the code above will begin to make more sense. First, we need a straightforward HTML form:

```php
<html>
    <head></head>
    <body>
        <b>Fields marked with * are mandatory</b>
        <form action='processor.php' method='post'>
            <b>Name*:</b>
            <br />
            <input type='text' name='name' size='15'>
            <p />
            <b>Age*:</b>
            <br />
            <input type='text' name='age' size='2' maxlength='2'>
            <p />
            <b>Email address*:</b>
            <br />
            <input type='text' name='email' size='30'>
            <p />
            <b>Sex*:</b>
            <br />
            <input type='radio' name='sex' value='m'>Male
            <input type='radio' name='sex' value='f'>Female
            <p />
            <b>Color*:</b>
            <br />
            <select name='color'>
                <option value=''>-select one-</option>
                <option value='r'>Red</option>
                <option value='g'>Green</option>
                <option value='b'>Blue</option>
                <option value='s'>Silver</option>
            </select>
            <p />
            <b>Insurance*:</b>
            <br />
            <select name='insurance'>
                <option value=''>-select one-</option>
                <option value='1'>Basic</option>
                <option value='2'>Enhanced</option>
                <option value='3'>Premium</option>
            </select>
            <p />
            <b>Optional features:</b>
            <br />
            <input type='checkbox' name='options[]' value='PSTR'>Power steering
            <input type='checkbox' name='options[]' value='AC'>Air-conditioning
            <input type='checkbox' name='options[]' value='4WD'>Four-wheel drive
            <input type='checkbox' name='options[]' value='SR'>Sun roof
            <input type='checkbox' name='options[]' value='LUP'>Leather upholstery
            <p />
            <input type='submit' name='submit' value='Save'>
        </form>
    </body>
</html>
```

Now, we need a PHP script to process the input sent through this form, using my new ```formValidator``` object. Save this as ```processor.php```:

```php
<?php
// include file containing class
include('class.formValidator.php');

// instantiate object
$fv = new formValidator();

// start checking the data
// check name
if ($fv->isEmpty($_POST['name'])) {
    $fv->addError('Name', 'Please enter your name');
}

// check age and age range
if (!$fv->isNumber($_POST['age'])) {
    $fv->addError('Age', 'Please enter your age');
} else if (!$fv->isWithinRange($_POST['age'], 1, 99)) {
    $fv->addError('Age', 'Please enter an age value in the numeric range 1-99');
}

// check sex
if (!isset($_POST['sex'])) {
    $fv->addError('Sex', 'Please select your gender');
}

// check email address
if (!$fv->isEmailAddress($_POST['email'])) {
    $fv->addError('Email address', 'Please enter a valid email address');
}

// check color
if ($fv->isEmpty($_POST['color'])) {
    $fv->addError('Color', 'Please select one of the listed colors');
}

// check insurance type
if ($fv->isEmpty($_POST['insurance'])) {
    $fv->addError('Insurance', 'Please select one of the listed insurance types');
}

// check optional features
if (isset($_POST['options'])) {
    if ($fv->isInArray($_POST['options'], '4WD') && !$fv->isInArray($_POST['options'], 'PSTR')) {
        $fv->addError('Optional features', 'Please also select Power Steering if you would like Four-Wheel Drive');
    }
}
// check to see if any errors were generated
if ($fv->isError()) {
    // print errors
    echo '<b>The operation could not be performed because one or more error(s) occurred.</b> <p /> Please resubmit the form after making the following changes:';
    echo '<ul>';
    foreach ($fv->getErrorList() as $e) {
        echo '<li>'.$e['field'].': '.$e['message'];
    }
    echo '</ul>';
} else {
    // do something useful with the data
    echo 'Data OK';
}
?>
```

As the listing above illustrates, the kind of methods exposed by my ```formValidator()``` object come in very handy to verify the user's input. In all cases, the ```isEmpty()``` method is used to test if required fields have been filled in, while the ```isEmailAddress()``` and ```isWithinRange()``` methods are used for more precise validation. The ```isInArray()``` method, very useful for check boxes and multiple-select lists, is also a great way to enforce associative rules and link specific choices together.

It's important to note that the ```formValidator``` class created above has nothing to do with the visual presentation of either the form or the form's result page. Its methods merely test the input sent to them and return a result code; how that result code is interpreted is entirely up to the developer. In the script above, a ```foreach()``` loop iterates over the list of errors and prints them in a bulleted list; however, you could just as easily display the errors in a table or write them to a log file in a custom format. I'll leave it to you to experiment with the possibilities.

That's about it for this episode of PHP 101. But hey, don't be depressed – I'll be back soon and, next time, I'm going to be taking everything I've taught you and using it to build a real-world PHP/MySQL web application. Make sure you don't miss that!

