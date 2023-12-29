# LEMP-STACK
LEMP Stack Setup Guide
Introduction
This guide provides step-by-step instructions on setting up a LEMP (Linux, Nginx, MySQL, PHP) stack on an Ubuntu 20.04 instance in the cloud. Follow these instructions to create a web server capable of hosting PHP applications with a MySQL database.

Date: Monday, December 18, 2023
Time: 10:46 PM

**Step 1: Setting Up Nginx**

1. Create an Ubuntu instance with the specifications: t2medium, allowing SSH and HTTP traffic.

2. After instance creation, stop it and navigate to "Actions" > "Instance setting" > "Modify metadata options" > "Optional" and save.

3. Open a terminal and run the following commands:

sudo apt update
sudo apt install nginx

4. Confirm the installation when prompted.

5. Verify Nginx installation:

sudo systemctl status nginx

6. Check local access:

curl http://localhost:80

7. Test access from the internet using your public IP address.

**Step 2: Installing MySQL**

1. Install MySQL:

sudo apt install mysql-server

2. Confirm installation when prompted.

3. Log in to MySQL console:

sudo mysql

4. Set root password and run security script:

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
exit
sudo mysql_secure_installation

**Step 3: Installing PHP**

1. Install PHP and required modules:

sudo apt install php-fpm php-mysql

**Step 4: Configuring Nginx for PHP**

1. Create a directory for your project:

sudo mkdir /var/www/projectLEMP

2. Set ownership:

sudo chown -R $USER:$USER /var/www/projectLEMP

3. Open Nginx configuration file:

sudo nano /etc/nginx/sites-available/projectLEMP

4. Paste the following configuration:

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}

5. Save and close the file.

6. Activate the configuration:

sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

7. Test configuration:

sudo nginx -t

8. Disable default Nginx host:

sudo unlink /etc/nginx/sites-enabled/default

9. Reload Nginx:

sudo systemctl reload nginx

10. Create a test index.html file:

sudo sh -c "echo 'Hello LEMP from hostname $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) with public IP $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)' > /var/www/projectLEMP/index.html"

11. Test your website in the browser using your public IP address.

**Step 5: Testing PHP with Nginx**

1. Create a test PHP file:

sudo nano /var/www/projectLEMP/info.php

2. Add the following content:

<?php
phpinfo();

3. Access the page in your browser:

http://<Public-DNS-Name>:80/info.php

4. Remove the test PHP file:

sudo rm -rf /var/www/projectLEMP/info.php

****Step 6: Retrieving Data from MySQL Database with PHP****

1. Connect to MySQL console:

sudo mysql -p

2. Create a new database:

sudo mysql -u root -p -e "CREATE DATABASE example_database;"

3. Create a new user and grant privileges:

mysql> CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.2';
mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';

4. Test the new user:

mysql -u example_user -p

5. Create a test table and insert data:

CREATE TABLE example_database.todo_list (
    item_id INT AUTO_INCREMENT,
    content VARCHAR(255),
    PRIMARY KEY(item_id)
);

INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
INSERT INTO example_database.todo_list (content) VALUES ("My second important item");
INSERT INTO example_database.todo_list (content) VALUES ("My third important item");
INSERT INTO example_database.todo_list (content) VALUES ("and this only one too");

SELECT * FROM example_database.todo_list;

6. Create a PHP script to retrieve data:

sudo nano /var/www/projectLEMP/todo_list.php

<?php
$user = "example_user";
$password = "PassWord.2";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}

**7. Access the page in your browser:**

http://<Public_domain_or_IP>/todo_list.php

8. Clean up:

sudo rm -rf /var/www/projectLEMP/todo_list.php

Your LEMP stack is now set up and ready for hosting PHP applications with a MySQL database.
