---
layout: post
title: Configure LAMP (Linux, Apache, MySQL/MariaDB & PHP) on RedHat Linux
date: 2023-07-09
categories:
  - Web Development
---

**Note : The scope of this LAMP configuration guide is the RedHat Linux operating system. The command or path of the file might vary depending on the Linux distribution (Linux distro) that you are using. Other Linux distro such as Ubuntu, might have different paths for the *php.ini* file. This configuration guide will assume that you are familiar with basic Linux commands and have a basic understanding of dynamic websites.**

LAMP is an acronym that stands for Linux, Apache, MySQL/MariaDB, and PHP. It is a popular and widely-used technology stack for web development, especially in the open-source community.

- Linux is an operating system just like Windows and MacOS. Linux is an open-source operating system which means that its source code is freely available for anyone to view, modify, and distribute. In simple term, everyone have access to the source code of Linux operating system. This allows users to study how Linux operating system works.
- Apache is a widely used open-source web server software. The Apache HTTP server, often referred to as simply Apache, is designed to server web pages and handle HTTP requests from clients, such as web browser. It is a cross-platform, meaning it can run on various operating system, including Windows, Linux, MacOS and other operating system. It supports multiple programming language, including PHP and Python, and making it compatible with a wide range of web development framework. In addition to serving static web pages, Apache can also handle dynamic content generation, virtual hosting (hosting multiple websites on a single server), SSL/TLS encryption, and other advanced features.
- MySQL and MariaDB are both relational database management systems (RDBMS) that are widely used in the industry. MySQL is an open-source RDBMS that was initially developed by MySQL AB. MySQL was later acquired by Sun Microsystems, which was subsequently acquired by Oracle Corporation in 2010. Oracle now owns and develops MySQL. MariaDB is also an open-source RDBMS that was created as a fork of MySQL. It was developed by some of the original developers of MySQL after concerns arose about the acquisition of MySQL by Oracle. MariaDB aims to be a drop-in replacement for MySQL, meaning that applications and tools that are compatible with MySQL should also work with MariaDB without any significant modifications. MySQL and MariaDB have slightly different features and extensions. While they both support standard SQL, MariaDB has some additional features and performance optimizations that are not present in MySQL.
- PHP (Hypertext Preprocessor) is a popular open-source scripting language primarily used for web development. PHP is designed to be embedded within HTML code and executed on the server-side. It follows a server-client architecture, where PHP code is processed on the server, and the resulting output is sent to the client's web browser for display. This approach allows dynamic generation of web content and enables interaction with databases, file systems, and other server-side operations.

## Installing and Configuring Apache

1) Installing Apache Web Server :
```bash
sudo dnf install httpd
```

2) Enable *httpd* service and make it start automatically after booting :
```bash
sudo systemctl enable --now httpd
```

3) Check the status of the service :
```bash
systemctl status httpd
```

4) Allow the HTTP traffic through the firewall and make this to be permanent :
```bash
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload
```

5) Now you can access the default web page by using URL of your IP address or localhost address. 
- For example : http://192.168.1.15 or http://127.0.0.1


## Installing and Configuring MariaDB

1) Installing MariaDB :
```bash
sudo dnf install mariadb-server mariadb
```

2) 2) Enable *mariadb* service and make it start automatically after booting :
```bash
sudo systemctl enable --now mariadb
```

3) Check the status of the service :
```bash
systemctl status mariadb
```

4) Create the root password for MySQL/MariaDB :
```bash
sudo mysql_secure_installation
```

5) Login to MySQL :
```bash
mysql -u root -p
```

6) Create a database, use that database and create table for that database :
- *Content, for example : id INT, username VARCHAR(50), userEmail VARCHAR(50)*
- *'INT' and 'VARCHAR' is a data type*
- *'50' is buffer size for that data*
```bash
CREATE DATABASE <database name>;
USE <database name>;
CREATE TABLE <table name> (<content name and data type>);
```
For example :
```bash
CREATE DATABASE db_contact;
USE db_contact;
CREATE TABLE tbl_contact (Id INT, contactName VARCHAR(50), contactEmail VARCHAR(50))
```

## Installing and Configuring PHP

1) Installing PHP and its dependencies :
- *The default PHP version available in RedHat repositories in PHP 8.0*
```bash
sudo dnf install php-{common,gmp,fpm,curl,intl,pdo,mbstring,gd,xml,cli,zip,mysqli}
```

- *It is also possible to install other PHP versions. This can be done by adding the Remi and EPEL repositories :* 
```bash
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

2) To configure PHP, open the *php.ini* file :
- PHP's initialization file, generally called php.ini, is responsible for configuring many of the aspects of PHP's behavior. Make the desired settings, for example, Resource Limits such as *max_execution_time* and File Uploads such as *upload_max_filesize*.
```bash
sudo nano /etc/php.ini
```

3) Edit *httpd.conf* file to allow Apache to load PHP :
- *httpd.conf* file is the main Apache HTTP server configuration file
```bash
sudo nano /etc/httpd/conf/httpd.conf

	# LoadModule foo_module modules/mod_foo.so
	AddHandler php-script .php
```

## PHP Application with Database Connectivity

1) Create an HTML form for connecting to the database :
```bash
sudo mkdir /var/www/html/<directory_name>
sudo nano /var/www/html/<directory_name_created_above>/<file_name>.html
```
For example :
```bash
sudo mkdir /var/www/html/contacts
sudo nano /var/www/html/contacts/contact.html
```
```html
	<!DOCTYPE html>
	<html xmlns="http://www.w3.org/1999/xhtml">
	<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title>Contact Form - PHP/MySQL Demo Code</title>
	</head>
	
	<body>
	<fieldset>
	<legend>Contact Form</legend>
	<form name="frmContact" method="post" action="contact.php">
	<p>
	<label for="Name">Name </label>
	<input type="text" name="txtName" id="txtName">
	</p>
	<p>
	<label for="email">Email</label>
	<input type="text" name="txtEmail" id="txtEmail">
	</p>
	<p>
	<label for="phone">Phone</label>
	<input type="text" name="txtPhone" id="txtPhone">
	</p>
	<p>
	<label for="message">Message</label>
	<textarea name="txtMessage" id="txtMessage"></textarea>
	</p>
	<p>&nbsp;</p>
	<p>
	<input type="submit" name="Submit" id="Submit" value="Submit">
	</p>
	</form>
	</fieldset>
	</body>
	</html>
```

2) Create PHP script to save data from HTML form to your database :
```bash
sudo nano /var/www/html/<directory_name>/<file_name>.php
```
For example :
```bash
sudo nano /var/www/html/contacts/contact.php
```
```php
	<?php
	// database connection code
	// $con = mysqli_connect('localhost', 'database_user', 'database_password','database');
	$con = mysqli_connect('localhost', 'root', '<root_password>','db_contact');
	
	// get the post records
	$txtName = $_POST['txtName'];
	$txtEmail = $_POST['txtEmail'];
	$txtPhone = $_POST['txtPhone'];
	$txtMessage = $_POST['txtMessage'];
	
	// database insert SQL code
	$sql = "INSERT INTO `tbl_contact` (`Id`, `fldName`, `fldEmail`, `fldPhone`, `fldMessage`) VALUES ('0', '$txtName', '$txtEmail', '$txtPhone', '$txtMessage')";
	
	// insert in database
	$rs = mysqli_query($con, $sql);
	if($rs) {
		echo "Contact Records Inserted";
	}
	
	?>
```

## Accessing Web Page

1) Restart Apache service :
```bash
sudo systemctl restart httpd
```

2) Now you can access the web page using the URL of your IP address, localhost address or your domain name that have been set up.
- http://192.168.1.15 or http://127.0.0.1 or http://your_domain_name
