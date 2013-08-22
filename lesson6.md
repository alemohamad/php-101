# PHP 101 (Lesson 6): Functionally Yours

## A Little Knowledge

If you've been taking your regular dose of PHP 101, you know now enough about PHP to write simple programs of your own. However, these programs will be "procedural" or linear – the statements in them will be executed sequentially, one after another – simply because that's the only programming style I've used so far.

You know what they say about a little knowledge being a dangerous thing… as your PHP scripts become more and more complex, it's only a matter of time before you bump your head against the constraints of the procedural method, and begin looking for a more efficient way of structuring your PHP programs.

That's where Part Six of PHP 101 comes in. In this tutorial I'm going to introduce you to a new way of doing things, where code doesn't run in a straight line, but twists, leaps and bounds across the landscape of your script. Most of this activity is accomplished through a programming construct called a "function", and this tutorial teaches you how to build them (once), use them (many times), pass them arguments and have them return values, and generally make your scripts more compact, efficient and maintainable.

## In Plain English

Ask a geek to define the term "function", and he'll probably mumble something about a function being "a block of statements that can be grouped together as a named entity." Since this is a tutorial on PHP, not an introductory course in Greek, I'll translate that for you: a function is simply a set of program statements which perform a specific task, and which can be "called", or executed, from anywhere in your program.

Every programming language comes with its own built-in functions, and typically also allows developers to define their own functions. For example, if I had a profit statement for the year on my desk, and I wanted to inflate each number by 35%, I could call my neighborhood accounting firm and get them to do it for me… or I could write a simple PHP function called ```cheatTheShareholders()``` and have it do the work for me (it's faster, plus PHP doesn't bill by the hour).

There are three important reasons why functions are a Good Thing&#8482;. First: user-defined functions allow you to separate your code into easily identifiable subsections – which are easier to understand and debug. Second: functions make your program modular, allowing you to write a piece of code once and then re-use it multiple times within the same program. And third: functions simplify code updates or changes, because the change needs only to be implemented in a single place (the function definition). Functions thus save time, money and electrons... and I know the electrons at least will thank you!

## Monday Morning Blues

To see how a function works, look at the following example:

```php
<?php

// define a function
function myStandardResponse() {
    echo "Get lost, jerk!<br /><br />";
}

// on the bus
echo "Hey lady, can you spare a dime? <br />";
myStandardResponse();

// at the office
echo "Can you handle Joe's workload, in addition to your own, while he's in Tahiti for a month? You'll probably need to come in early and work till midnight, but we are confident you can handle it. Oh, and we can't pay you extra because of budgetary constraints...<br />";
myStandardResponse();

// at the party
echo "Hi, haven't I seen you somewhere before?<br />";
myStandardResponse();

?>
```

Here's what the output might look like:

```
Hey lady, can you spare a dime?
Get lost, jerk!

Can you handle Joe's workload, in addition to your own, while he's in Tahiti for a month?
You'll probably need to come in early and work till midnight, but we are confident you can
handle it. Oh, and we can't pay you extra because of budgetary constraints...
Get lost, jerk!

Hi, haven't I seen you somewhere before?
Get lost, jerk!
```

(Sure it's rude, but it does demonstrate how a function allows you to reuse pieces of code.)

The first thing I've done in the script above is define a new function, with the ```function``` keyword. This keyword is followed by the name of the function, which in this case is ```myStandardResponse()```. All the program code attached to that function is then placed within a pair of curly braces – and this program code can contain loops, conditional statements, calls to other user-defined functions, or calls to other PHP functions.

Of course, defining a function is only half of the puzzle; for it to be of any use at all, you need to "invoke" it. In PHP, as in a million other languages, this is accomplished by calling the function by its name, as I've done in the example above. Calling a user-defined function is identical to calling a built-in PHP function like ```echo()``` or ```explode()```.

Here's the typical format for a function:

```php
function function_name (optional function arguments) {
    statement 1...
    statement 2...
    .
    .
    .
    statement n...
}
```

## Having an Argument... or Two

Functions like the one you saw in the previous section print the same value every time you invoke them. While this is interesting the first six times, it can get boring on the seventh. What we need to do, to make these boring, unintelligent functions a little more exciting, is get them to return a different value each time they are invoked.

Enter arguments.

Arguments work by using a placeholder to represent a certain variable within a function. Values for this variable are provided to the function at run-time from the main program. Since the input to the function will differ at each invocation, so will the output.

To see how this works, look at the following function, which accepts a single argument and then prints it back after a calculation:

```php
<?php
// define a function
function getCircumference($radius) {
    echo "Circumference of a circle with radius $radius is " . sprintf("%4.2f", (2 * $radius * pi())) . "<br />";
}

// call a function with an argument
getCircumference(10);

// call the same function with another argument
getCircumference(20);
?>
```

In this example, when the ```getCircumference()``` function is called with an argument, the argument is assigned to the placeholder variable $radius within the function, and then acted upon by the code within the function definition.

It's also possible to pass more than one argument to a function. This is done using a comma-separated list, as the following example shows:

```php
<?php
// define a function
function changeCase($str, $flag) {
    /* check the flag variable and branch the code */
    switch($flag) {
        case 'U':
            print strtoupper($str)."<br />";
            break;
        case 'L':
            print strtolower($str)."<br />";
            break;
        default:
            print $str."<br />";
            break;
    }
}

// call the function
changeCase("The cow jumped over the moon", "U");
changeCase("Hello Sam", "L");
?>
```

Here, depending on the value of the second argument, program flow within the function moves to the appropriate branch and manipulates the first argument.

Note that there is no requirement to specify the data type of the argument being passed to a function. Since PHP is a dynamically-typed language, it automatically identifies the variable type and acts on it appropriately.

## Circles in the Sand

The functions on the previous page simply printed their output to the screen. But what if you want the function to do something else with the result? Well, in PHP, you can have a function return a value, such as the result of a calculation, to the statement that called it. This is done using a ```return``` statement within the function, as shown below:

```php
<?php
// define a function
function getCircumference($radius) {
    // return value
    return (2 * $radius * pi());
}
/* call a function with an argument and store the result in a variable */
$result = getCircumference(10);
/* call the same function with another argument and print the return value */
print getCircumference(20);
?>
```

Here, the argument passed to the ```getCircumference()``` function is processed, and the result is returned to the main program, where it may be captured in a variable, printed, or dealt with in other ways.

You can even use the result of a function inside another function, as illustrated in this minor revision of the example above:

```php
<?php
// define a function
function getCircumference($radius) {
    // return value
    return (2 * $radius * pi());
}
// print the return value after formatting it
print "The answer is ".sprintf("%4.2f", getCircumference(20));
?>
```

Return values need not be numbers or strings alone: a function can just as easily return an array (remember them?), as demonstrated in the following example:

```php
<?php
/* define a function that can accept a list of email addresses */
function getUniqueDomains($list) {
    /* iterate over the list, split addresses and add domain part to another array */
    $domains = array();
    foreach ($list as $l) {
        $arr = explode("@", $l);
        $domains[] = trim($arr[1]);
    }
    // remove duplicates and return
    return array_unique($domains);
}
// read email addresses from a file into an array
$fileContents = file("data.txt");
/* pass the file contents to the function and retrieve the result array */
$returnArray = getUniqueDomains($fileContents);
// process the return array
foreach ($returnArray as $d) {
    print "$d, ";
}
?>
```

Assuming the file looked like this,

```php
test@test.com
a@x.com
zooman@deeply.bored.org
b@x.com
guess.me@where.ami.net
testmore@test.com
```

the output of the script above would look like this:

```php
test.com, x.com, deeply.bored.org, where.ami.net, 
```

Note that the return statement terminates program execution inside a function.

## Marching Order

The order in which arguments are passed to a function can be important. The following example requires that the name is passed as the first argument, and the place as the second.

```php
<?php
// define a function
function introduce($name, $place) {
    print "Hello, I am $name from $place";
}
// call function
introduce("Moonface", "The Faraway Tree");
?>
```

This is the output:

```php
Hello, I am Moonface from The Faraway Tree
```

In this example, if you reversed the order in which arguments were passed to the function, this is what you'd see:

```php
Hello, I am The Faraway Tree from Moonface
```

And look what happens if you forget to pass a required argument altogether:

```php
Warning: Missing argument 2 for introduce() in xx.php on line 3
Hello, I am Moonface from 
```

In order to avoid such errors, PHP allows you to specify default values for all the arguments in a user-defined function. These default values are used if the function invocation is missing some arguments. Here's an example:

```php
<?php
// define a function
function introduce($name="John Doe", $place="London") {
    print "Hello, I am $name from $place";
}
// call function
introduce("Moonface");
?>
```

In this case the output would be:

```php
Hello, I am Moonface from London
```

Notice that the function has been called with only a single argument, even though the function definition requires two. However, since default values are present for each argument in the function, the missing argument is replaced by the default value for that argument, and no error is generated.

## The Amazing Shrinking Argument List

All the examples on the previous page have one thing in common: the number of arguments in the function definition is fixed. However, PHP 4.x also supports variable-length argument lists, by using the ```func_num_args()``` and ```func_get_args()``` commands. For want of a better name, these functions are called "function functions". Try wrapping your tongue around that while you look at the next example, which demonstrates how they can be used:

```php
<?php
// define a function
function someFunc() {
    // get the arguments
    $args = func_get_args();
    
    // print the arguments
    print "You sent me the following arguments:";
    foreach ($args as $arg) {
        print " $arg ";
    }
    print "<br />";
}
// call a function with different arguments
someFunc("red", "green", "blue");
someFunc(1, "soap");
?>
```

Hmmm... if you're sneaky, you might have tried to pass ```someFunc()``` an array, and found that instead of displaying the elements of the array, it simply said "Array". You can fix this by adding a quick test for array arguments inside the function, as in this rewrite:

```php
<?php
// define a function
function someFunc() {
    // get the number of arguments passed
    $numArgs = func_num_args();
    
    // get the arguments
    $args = func_get_args();
    
    // print the arguments
    print "You sent me the following arguments: ";
    for ($x = 0; $x < $numArgs; $x++) {
        print "<br />Argument $x: ";
        /* check if an array was passed and, if so, iterate and print contents */
        if (is_array($args[$x])) {
            print " ARRAY ";
            foreach ($args[$x] as $index => $element) {
                print " $index => $element ";
            }
        }
        else {
            print " $args[$x] ";
        }
    }
}
// call a function with different arguments
someFunc("red", "green", "blue", array(4,5), "yellow");
?>
```

## Going Global

Let's now talk a little bit about the variables used within a function, and their relationship with variables in the outside world. Usually, the variables used within a function are "local" – meaning that the values assigned to them, and the changes made to them, are restricted to the function space alone.

Consider this simple example:

```php
<?php
// define a variable in the main program
$today = "Tuesday";
// define a function
function getDay() {
    // define a variable inside the function
    $today = "Saturday";
    // print the variable
    print "It is $today inside the function<br />";
}
// call the function
getDay();
// print the variable
print "It is $today outside the function";
?>
```

When you run this script, here is what you will see:

```php
It is Saturday inside the function
It is Tuesday outside the function
```

In other words, the variable inside the function is insulated from the identically-named variable in the main program. Variables inside a function are thus aptly called "local" variables because they only exist within the function in which they are defined.

The reverse is also true: variables defined inside a function cannot be "seen" outside it. To illustrate, take a look at the next example and its output (or the lack of it):

```php
<?php
// define a function
function getDay() {
    // define a variable inside the function
    $today = "Saturday";
}
getDay();
print "Today is $today";
?>
```

Here is the output:

```php
Today is 
```

Depending on the error_reporting you have set up in php.ini, you might also see an error message:

```php
Notice: Undefined variable: today in x1.php on line 10
```

However, I didn't say this can't be overcome. To have variables within a function accessible from outside it (and vice-versa), all you need to do is first declare them "global" with the – you guessed it! – global keyword.

Here is a rewrite of the earlier example, this time declaring the ```$today``` variable global:

```php
<?php
// define a variable in the main program
$today = "Tuesday";
// define a function
function getDay() {
    // make the variable global
    global $today;
    // define a variable inside the function
    $today = "Saturday";
    // print the variable
    print "It is $today inside the function<br />";
}
// print the variable
print "It is $today before running the function<br />";
// call the function
getDay();
// print the variable
print "It is $today after running the function";
?>
```

And here is the output:

```php
It is Tuesday before running the function
It is Saturday inside the function
It is Saturday after running the function
```

Thus, once a variable is declared global, it is available at the global level, and can be manipulated both inside and outside a function.

PHP also comes with so-called superglobal variables – variables that are always available, regardless of whether you're inside a function or outside it. You've already seen some of these special variables in action: the ```$_SERVER```, ```$_POST``` and ```$_GET``` variables are all superglobals, which is why you can access things like the currently-executing script's name or form values even inside a function.

Superglobals are a Good Thing&#8482;, because they're always there when you need them,
and you don't need to jump through any hoops to use the data stored inside them. Read more
about superglobals and variable scope at [http://www.php.net/manual/en/language.variables.predefined.php](http://www.php.net/manual/en/language.variables.predefined.php) and [http://www.php.net/manual/en/language.variables.scope.php](http://www.php.net/manual/en/language.variables.scope.php).

## Checking References

Any discussion about variables in and out of functions would be incomplete without some mention of the difference between "passing by reference" and "passing by value". So far, all the examples you've seen have involved passing arguments to a function "by value" - meaning that a copy of the variable was passed to the function, while the original variable remained untouched. However, PHP also allows you to pass "by reference" – meaning that instead of passing a value to a function, you pass a reference to the original variable, and have the function act on that instead of a copy.

Confusing? Well, this is probably easier to understand with an example. Let's start with this:

```php
<?php
// create a variable
$today = "Saturday";
// function to print the value of the variable
function setDay($day) {
    $day = "Tuesday";
    print "It is $day inside the function<br />";
}
// call function
setDay($today);
// print the value of the variable
print "It is $today outside the function";
?>
```

You've already seen this before, and you already know what the output is going to say:

```php
It is Tuesday inside the function
It is Saturday outside the function
```

This is because when the ```getDay()``` function is invoked, it passes the value "Saturday" to the function ("passing by value"). The original variable remains untouched; only its content is sent to the function. The function then acts on the content, modifying and displaying it.

Now, look at how "passing by reference" works:

```php
<?php
// create a variable
$today = "Saturday";
// function to print the value of the variable
function setDay(&$day) {
    $day = "Tuesday";
    print "It is $day inside the function<br />";
}
// call function
setDay($today);
// print the value of the variable
print "It is $today outside the function";
?>
```

Notice the ampersand (&) before the argument in the function definition. This tells PHP to use the variable reference instead of the variable value. When such a reference is passed to a function, the code inside the function acts on the reference, and modifies the content of the original variable (which the reference is pointing to) rather than a copy. If you then try retrieving the value of the original variable outside the function, it returns the modified value:

```php
It is Tuesday inside the function
It is Tuesday outside the function
```

Now you understand why I said no discussion about variables would be complete without mentioning the two ways of passing variables. This, of course, is what the ```global``` keyword does inside a function: use a reference to ensure that changes to the variable inside the function also reflect outside it. The PHP manual puts it best when it says "…when you declare a variable as ```global $var``` you are in fact creating a reference to a global variable". For more examples, read all about references at [http://www.zend.com/manual/language.references.php](http://www.zend.com/manual/language.references.php).

And that just about concludes this tutorial. This time you've taken a big step towards better software design by learning how to abstract parts of your PHP code into reusable functions. You now know how to add flexibility to your functions by allowing them to accept different arguments, and how to obtain one (or more) return values from them. Finally, you've learned a little bit about how PHP treats variables inside and outside functions.

In [Part Seven](http://devzone.zend.com/10/php-101-part-7-the-bear-necessities), I'll be showing you how to group related functions together into classes, and also telling you all about the cool new features in the PHP 5 object model. You definitely don't want to miss that one!

