# PHP 101 (Lesson 7, Part 2): The Bear Necessities

## Ending On A High Note

Just as there are constructors, so also are there **destructors**. Destructors are object methods which are called when the last reference to an object in memory is destroyed, and they are usually tasked with clean-up work – for example, closing database connections or files, destroying a session and so on. Destructors are only available in PHP 5, and must be named ```__destruct()```. Here's an example:

```php
<?php

// PHP 5
// class definition

class Bear {

    // define properties
    public $name;
    public $weight;
    public $age;
    public $sex;
    public $colour;

    // constructor
    public function __construct() {
        $this->age = 0;
        $this->weight = 100;
        $this->colour = "brown";
    }

    // destructor
    public function __destruct() {
        echo $this->name . " is dead. He was " . $this->age . " years old and " . $this->weight . " units heavy. Rest in peace!";
    }

    // define methods
    public function eat($units) {
        echo $this->name . " is eating " . $units . " units of food...";
        $this->weight += $units;
    }

    public function run() {
        echo $this->name . " is running...";
    }

    public function kill() {
        echo $this->name . " is killing prey...";
    }
}

// create instance of Bear()
$daddy = new Bear;
$daddy->name = "Daddy Bear";
$daddy->age = 10;
$daddy->kill();
$daddy->eat(2000);
$daddy->run();
$daddy->eat(100);
?>
```

Here, once the script ends, no reference will exist for ```$daddy```, and so the destructor will be called automatically. The output would look like this:

```
Daddy Bear is killing prey...
Daddy Bear is eating 2000 units of food...
Daddy Bear is running...
Daddy Bear is eating 100 units of food...
Daddy Bear is dead. He was 10 years old and 2200 units heavy. Rest in peace!
```

## Discovering New Things

PHP 4 and PHP 5 come with a bunch of functions designed to let you discover object properties and methods, and find out which class an object belongs to. The first two of these are the ```get_class()``` and ```get_parent_class()``` functions, which tell you the name of the classes which spawned a particular object. Consider the following class definition:

```php
<?php

// PHP 5
// base class

class Bear {

    public $name;
    public $weight;

    // constructor
    public function __construct() {
    }

    // define methods
    public function eat() {
    }

    public function run() {
    }

    public function sleep() {
    }
}

// derived class
class GrizzlyBear extends Bear {
    public function kill() {
    }
}
?>
```

And now consider the following script, which uses ```get_class()``` and ```get_parent_class()``` to retrieve the class name from an instance:

```php
<?php
$joe = new GrizzlyBear;
$joe->name = "Joe Bear";
$joe->weight = 1000;
echo "Class: " . get_class($joe);
echo "Parent class: " . get_parent_class(get_class($joe));
?>
```

You can view all the properties exposed by a class with ```get_class_vars()```, and all its methods with ```get_class_methods()``` function. To view properties of the specific object instance, use ```get_object_vars()``` instead of ```get_class_vars()```. Here is an example:

```php
<?php

// create instance
$joe = new GrizzlyBear;
$joe->name = "Joe Bear";
$joe->weight = 1000;

// get class name
$className = get_class($joe);

// get class properties
echo "Class properties:";
print_r(get_class_vars($className));

// get class methods
echo "Class methods:";
print_r(get_class_methods($className));

// get this instance's properties
echo "Instance properties:";
print_r(get_object_vars($joe));

?>
```

and here is some sample output:

```php
Class properties:
Array
(
    [name] => 
    [weight] => 
)

Class methods:
Array
(
    [0] => kill
    [1] => __construct
    [2] => eat
    [3] => run
    [4] => sleep
)

Instance properties:
Array
(
    [name] => Joe Bear
    [weight] => 1000
)
```

As noted in one of the previous segments of this tutorial, the ```print_r()``` function allows you to look inside any PHP variable, including an object. It's extremely useful, so note it down for future reference.

## Access Denied

And now that you know the basics of how objects work in PHP, let's wrap this up with a real-world example. Consider the following userAuth() class, which exposes methods to validate a user login using an encrypted password file such as /etc/passwd or .htaccess, both of which are used on Unix systems (i.e. most of the Internet). I'll assume here that the passwords in the password file are encrypted with MD5, and use a 12-character salt beginning with $1$:

```php
<?php

// PHP 5
// class definition

class userAuth {

    // define properties
    public $username;
    private $passwd;
    private $passwdFile;
    private $_resultCode;

    // constructor
    // must be passed username and password
    public function __construct($username, $password) {
            $this->username = $username;
            $this->passwd = $password;
            $this->_resultCode = -1;
    }

    // used to set file to read for password data
    public function setPasswdFile($file) {
        $this->passwdFile = $file;
    }

    // returns: -1 if user does not exist
    //           0 if user exists but password is incorrect
    //           1 if username and password are correct
    public function getResultCode() {
        return $this->_resultCode;
    }

    public function authenticateUser() {
        // make sure that the script has permission to read this file!
        $data = file($this->passwdFile);

        // iterate through file
        foreach ($data as $line) {
            $arr = explode(":", $line);

            // if username matches
            // test password
            if ($arr[0] == $this->username) {
                // if match, user/pass combination is correct
                // return 1
                if ($arr[1] == crypt($this->passwd, $arr[1])) {
                    $this->_resultCode = 1;
                    break;
                // otherwise return 0
                } else {
                    $this->_resultCode = 0;
                    break;
                }
            }
        }
    }
    // end class definition
}
?>
```

Most of this should be clear to you from the examples in previous pages. In case it isn't, the following script should help you understand what's happening:

```php
<?php

// create instance
$ua = new userAuth("joe", "secret");
// set password file
$ua->setPasswdFile("passwd.txt");
// perform authentication
$ua->authenticateUser();
// check result code and display message

switch ($ua->getResultCode()) {
    case -1:
        echo "Could not find your user account";
        break;
    
    case 0:
        echo "Your password was incorrect";
        break;
    
    case 1:
        echo "Welcome, ".$ua->username;
        break;
}        
?>
```

Here, the username and password is passed to the object constructor, as is the name and path of the file containing authentication credentials. The ```authenticateUser()``` method takes care of parsing the password file and checking if the user exists and the password is correct. Depending on what it finds, a result code is generated and stored in the private variable ```$_resultCode```. This variable can be read through the ```getResultCode()``` method, and an appropriate message displayed. And since this entire thing is neatly encapsulated in a class, I can take it anywhere, use it in any script – even inside another application – and extend it to support different types of authentication schemes and containers.

There's a lot more you can do with objects, especially in PHP 5; I've restrained myself here because I didn't want to confuse you too much with talk of overloading, abstract classes and static methods. If you're interested, however, drop by [http://www.php.net/manual/en/language.oop.php](http://www.php.net/manual/en/language.oop.php) for more. And make sure you come back for [Part Eight](http://devzone.zend.com/12/php-101-part-8-databases-and-other-animals_part-1) of PHP 101, because I'm going to show you how to hook your scripts up to a MySQL database.
