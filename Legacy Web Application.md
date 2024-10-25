
by setting up a Web Application, we use these topics which have learned so far: 

- SQL
- Programming
- JavaScript
- Linux + bash
- Network  = Web Server


###  what is even a Web Application? 

>web application is type of website, which has **static + dynamic contents** .
>users have different contents, based on their activities on that website.

##### static website -->  index.html
##### Web Application -->  static + dynamic contents

- why web application is important?
	because it's the playground. as a web app security engineer, you must know what is going on in a web application. 
	\* the more you know about Web Application, more professional you become.  

---

#### a Web Application requires: 

- ##### server-side programming language
	--> PHP, Python, Go, NodeJS, Java

- ##### Database
	-->  MySQL, SQLServer, MongoDB, etc

- ##### HTML + JS (showing contents to client)
	--> JQuery, React, etc

- ##### Web-Server 
	-->  integration 

#### flow: 
Me <--> web-server <--> PHP + MySQL + etc  
this flow could be much complicated. 

---

## Legacy & Modern web applications

--->  **Legacy**:  PHP + Web-Server (document_root) + JavaScript(vanilla) + DB 
--->  **Modern**: NodeJS + Express + JavaScript (vanilla, react, vue, etc) + DB 

\* PHP is also used in modern web apps: (*search about it*)
	PHP + web-server (document_root) + jquery + React 

\* hint: as a hacker in this field, once you have a *good understanding of Programming*, later you can work with various languages.


---

### set-up

```bash
apt update
apt install apache2
apt install php #libapache2-mod-php8.1 module integrates php & apache2
```

- after installing PHP in server: 

```bash
cd /var/www/html 
vim index.php #it's better to remove default index.html (why?) 
```

- in vim: 

```php
<?php 
echo '0123';
?> 
```

- in order to let php engine understand the syntax, we use angle brackets `<>`  and `?` to begin and finish the code. 
- `echo` is similar to `print()` in Python.


- *if PHP is not installed* on the server, and you open a `.php` file, Apache (or any web-server) can not respond the content inside the php file.  --> in this case, you can see the the content only by the *source code* (ctrl + u)
- *But when PHP is installed*, the module mentioned above, integrates PHP and Apache. then Apache can responds the `0123` on browser.

- note that, even if PHP is not installed on server, Chrome can show the content of .php file. (but firefox can't)


##### Me (client) apache -->  
	- if (.php exists) --> PHP engine --> parse --> response
	- else --> static resource (e.g : .html )

- if I put the content of index.php into a .html file, the browser responds the php source code( the content can not be responded. ) --> browsers don't show the source code in this case

```bash
cp index.php test.html #try urself
```

### *This process is called PHP + Apache Integration*

---

a web application need at least this 3 technologies: 
---> php + apache + MySQL (a Database)


## lets install MySQL 

*4 Important Concept:*
- MySQL --> for the maintenance of data
- who saves the data in MySQL ?  -->  the web application (PHP in fact)
- How PHP connect to the DB ?     -->  by a *user* --> `mysql -u root -p`
- MySQL put data into a database  --> so I need to create a Database

```bash
apt install mysql-server
```

- now, I need to create a DB, create a USER 
- USER `root` is dangerous to connect to web app. we should do it with USER which has low permission. 

```sql
mysql -u root --default root has no pass, (just click enter for pass)
CREATE database voorivex_weblog;
CREATE USER 'voorivex'@'localhost' IDENTIFIED BY 'mamad123';
GRANT ALL PRIVILEGES ON voorivex_weblog.* TO 'voorivex'@'localhost';
FlUSH PRIVILEGES;
exit
```

\* localhost here, is because PHP and SQL are in the same machine.
now I run MySQL with voorivex user :

```SQL
mysql -u root -p -- password: mamad123
show databases;
```

now I clone this repository: [voorivex-weblog](https://github.com/Voorivex/voorivex-weblog-owasp)

```bash
git clone https://github.com/Voorivex/voorivex-weblog-owasp
mv voorivex-weblog-owasp/* . #moves everything to current directory
rm -rf voorivex-weblog-owasp
```

- now, import `database.sql` to our database in the server
- I use root for importing (why?)

```bash
mysql -u root -p voorivex_weblog < database.sql
```

```sql
mysql -u voorivex -p
use voorivex_weblog;
show tables;
describe users;
```


#### note that PHP needs User + Pass for accessing the Database
-->  the credentials of MySQL is located in `db.php` 
-->  this user and password MUST BE the same with in the user MySQL

```bash
root@personal-host:/var/www/html# cat db.php
<?php
$db_servername = "localhost"; // Replace with your MySQL server hostname or IP address
$db_username = "voorivex"; // Replace with your MySQL username
$db_password = "mamad123"; // Replace with your MySQL password
$db_database = "voorivex_weblog"; // Replace with the name of your MySQL database
$backupPath = "/var/www/html/admin/backups/";

// Create a connection
$conn = new mysqli($db_servername, $db_username, $db_password, $db_database);

// Check the connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

// echo "Connected successfully";

```

lets make the back-up directory

```bash
mkdir -p /var/www/html/admin/backups/
```


- now if you try browsing the website (VPS IP), you see nothing. Because there's an error in your application.

```bash
cat /var/log/apache2/error.log
```

- this `error.log` file contains important logs. 
- you should know how to work with it and read errors
	--> job interview, observing errors


```bash
vim /etc/php/8.1/apache2/php.ini
```

what is `php.ini` file? 
	-->  it's the PHP configuration file in Apache. 
	in this file search for `display_error` and change it's value to `on`.
	*then, restart Apache -->*  `service apache2 restart`
	with this edit, you can see php errors on the browser.


---

### Concepts:

Me (Client) <--> HTTP <-->  Apache <-->  PHP <--> MySQL 

- apache parses the HTTP packet line by line, and give it to PHP.
- User can send data from everywhere of the HTTP packet. (in this web app, the login credentials are sent in body)
- capture the traffic with burp

---
## Reading PHP source codes

*Let's start with index.php*

```php
<?php

session_start();

include 'header.php';

include 'db.php';

?>```

- session_start();  --->  creates a cookie (session), when  you open index.php file.
- include --->  same as import in python. --> source code includes in the current file

\- cookie and session are important concepts. If they don't configure properly, could cause to insecurity. 

state    -->  saving
session -->  server-side/ temporary/ more secure
cookie   -->  client-side/ have persistent/ less secure 
\* read more about them in book or online.

-->  in this web app, the login credentials sent like this (in bode section of HTTP packet): 
```bash
username=test&password=test
```

---


### Important Concepts 

\- in the web application, you can send data from *ALL PART* of the *HTTP PACKET* ! (not only the body part) --> 
\- the thing is, if the web app is ready for accepting data (on that specific part or not)

\- **How to feed inputs in a web application?**
\- **integration between PHP and MySQL**

\- in PHP, can use HTML (as static resource) 

---
### db.php

- web applications need to connect to the database. 
- this file, contains username, password, and the data required to connect PHP to MySQL.
- *Line 9, is responsible to connect PHP to MySQL. --> this is a critical point in security of web applications!*
- `die` function stops all upcoming codes to be executed (stops the program)


```php
<?php

$db_servername = "localhost"; // Replace with your MySQL server hostname or IP address

$db_username = "voorivex"; // Replace with your MySQL username

$db_password = "mamad123"; // Replace with your MySQL password

$db_database = "voorivex_weblog"; // Replace with the name of your MySQL database

$backupPath = "/var/www/html/admin/backups/";

// Create a connection

$conn = new mysqli($db_servername, $db_username, $db_password, $db_database); // --> Line 9

// Check the connection

if ($conn->connect_error) {
	die("Connection failed: " . $conn->connect_error);

}
// echo "Connected successfully";

?>
```

note: this application is similar to a WordPress application

---

### login.php

--> **in concept of security, you have to know where are the inputs of the web application?** 
--> **you must know who is the (data) sender ? who is the receiver** 

#### ***there are ONLY 2 main sender on browsers (client-side): HTML & JavaScript***

- extensions in fact, are also senders 
- in old days, there were other senders (like java applet), but now, only HTML & JS can send data to the server. 


this is the login html source code of website: 
- the `required`  attribute ensures that the user must enter a value in the input field before submitting the form. --> it can not be sent empty.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Voorivex Weblog System</title>
<link rel="stylesheet" href="[/statics/styles.css](http://194.120.24.24/statics/styles.css)">
<!-- Add any additional CSS or JavaScript links here -->
<body><h1>Login</h1><form action="login.php" method="post">
<label for="username">Username:</label>
<input type="text" id="username" name="username" required><br>

<label for="password">Password:</label>
<input type="password" id="password" name="password" required><br>
<input type="submit" value="Login"><br>
<a href='[register.php](http://194.120.24.24/register.php)'>Need a registration?</a><br>
</form>
</body>
<footer>
<p>&copy; 2023 Voorivex Weblog system. All rights reserved.</p>
</footer>
</body>
</html>
```

***when it comes to the sender in web applications (HTML & JavaScript), these 3 critical questions must be answered:*** 

1. ***where data is being sent?*** 
2. ***content type***
	***--> field name***
3. ***trigger point*** 


in this (login) code , the answers are the followings: 

1. `login.php` --> form tags and their `action` attributes are responsible to get data 
2. if in an *HTML form*, the *content-type* is not mentioned, it is `Content-Type: application/x-www-form-urlencoded`
3. trigger point is the login button. 
4. in this code : 

```html
<input type="password" id="password" name="password" required>
```

the `name` attribute is important to us, other attributes like id and type are not.
	why it is important?
	because it changes the login credentials values, when logging information sent to server 
	if you change the `name = mamad` , mamad will be the value for password.

```bash
username=test&kosmos=test
```

\* note that some content-types might have not field name.
\* in this web application, the application receives data by these codes: 

```php
// Check if the form is submitted

if ($_SERVER["REQUEST_METHOD"] == "POST") {

// Get user input from the form

	$username = $_POST["username"];
	$password = $_POST["password"];
```

- the first line, which has `$_SERVER` **is a=n important array (list) about server** configurations. 
- the `["REQUEST_METHOD"]` indicates the method, which need to be used 

- last 2 lines, are responsible for *receiving user inputs*. this is a common way to save url-encoded content-types.  


----


#### login query 

this line is the login query. If both username and password were available in a row, login will be processed. 

```php
$sql = "SELECT * FROM users WHERE username = '$username' AND password '$password'";
```

---

### source code auditing 

- read the source code, try to understand each line functioning . 
- you can make a debugger after certain lines, to check if they work or not

---

 *these concepts are crucial to know and understand*:

- ***How to feed inputs in a Web Application?***
- ***integration between PHP and MySQL***
	- ***where data is being sent?*** 
	-  ***content type***
			***---> field name***
	-  ***trigger point*** 
- ***behind the scene of Login***
- ***behind the scene of Register***
- ***behind the scene of Change Password*** 
- ***behind the scene of Forget Password*** 

