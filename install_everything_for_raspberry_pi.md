# Install Apache2 on Raspberry

```
sudo apt install apache2 -y
```

## List items in directory

```
cd /var/www/html
ls -al
```

## Get IP address

```
hostname -I
```

Use IP in output or `localhost` to access Apache.

# Install PHP

```
sudo apt install php -y
```

## Testing it out

Create `index.php` in `/var/www/html`. Write simple code:

```html
<?php echo “Hello student in IoT class”;?>
```

Access web browser at `localhost/index.php` to test the PHP file.

# Install MySQL

```
sudo apt install mariadb-server php-mysql -y
sudo service apache2 restart
```

Secure MySQL installation. Follow recommended:

```
sudo mysql_secure_installation
```

# Install phpMyAdmin

Download:

```
wget https://files.phpmyadmin.net/phpMyAdmin/4.9.0.1/phpMyAdmin-4.9.0.1-all-languages.zip
```

Extract:

```
sudo unzip phpMyAdmin-4.9.0.1-all-languages.zip -d /opt
```

Rename:

```
sudo mv -v /opt/phpMyAdmin-4.9.0.1-all-languages /opt/phpMyAdmin
```

Change ownership:

```
sudo chown -Rfv www-data:www-data /opt/phpMyAdmin
```

Configure:

```
sudo nano /etc/apache2/sites-available/phpmyadmin.conf
```

Edit the `phpmyadmin.conf` file:

```
<VirtualHost *:9000>
ServerAdmin webmaster@localhost
DocumentRoot /opt/phpMyAdmin
<Directory /opt/phpMyAdmin> Options Indexes FollowSymLinks
AllowOverride none
Require all granted
</Directory>
ErrorLog ${APACHE_LOG_DIR}/error_phpmyadmin.log
CustomLog ${APACHE_LOG_DIR}/access_phpmyadmin.log combined
</VirtualHost>
```

Edit `ports.conf`:

```
sudo nano /etc/apache2/ports.conf
```

Add `Listen 9000` below `Listen 80`.

## Enable phpMyAdmin site

```
sudo a2ensite phpmyadmin.conf
sudo systemctl restart apache2
```

Access phpMyAdmin from the web browser using the port 9000 from the localhost.

## Create new phpMyAdmin user

```
sudo mysql --user=root --password
```

In MySQL console:

```
GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;
```

Replace `username` and `password` with desired username and password.

# Using Python to access MySQL

Install MySQL lib for Python:

```
sudo apt-get install python3-mysqldb
```

Assume that we have the Database with name IoT_SU and table name DataInfor with 2 columns as “ID” and “temperature” where ID is auto number and temperature is text:

Example Python code:

```python
import MySQLdb
dbConn = MySQLdb.connect("localhost", "account_db", "password", "IoT_SU") or die("Could not connect to the database")
print(dbConn)
data=25
with dbConn:
    cursor = dbConn.cursor()
    cursor.execute("INSERT INTO DataInfor (Temperature) values (%s)" % (data))
    dbConn.commit()
    cursor.close()
```
If it returns the error like `Attribute error: __Enter__` then it's due to `with()` incompatibility with Python 3.9 (an old Python version). Use the following Python code instead to test:

```Python
import MySQLdb

# establish a database connection
dbConn = MySQLdb.connect("localhost", "root", "123456", "IoT_SU")

# check if the connection was successful
if not dbConn:
    print("Could not connect to the database")
else:
    print("Database connection established")

# define the data to be inserted
data = 25

# try to execute the database operation
try:
    # create a cursor object
    cursor = dbConn.cursor()

    # execute the SQL query to insert the data
    cursor.execute("INSERT INTO DataInfor (Temperature) VALUES (%s)", (data,))

    # commit the changes to the database
    dbConn.commit()

    # print a success message
    print("Data inserted successfully")

# handle any potential exceptions that may occur
except Exception as e:
    # rollback the changes to the database
    dbConn.rollback()

    # print an error message
    print("Error inserting data into the database:", e)

# close the cursor and the database connection
finally:
    cursor.close()
    dbConn.close()
```

