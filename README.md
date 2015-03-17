# flask-sakila

Please note this is not a tutorial, I have wrote it in that style so you can follow along. If you get into trouble (like I did) try the mailing list or just google it. You will find that you will actually learn more from researching it and getting into tight spots. ;) 

I built this app with the Flask micro-framework to be used as part of a series of applications that I will be 
performing tests on. This is a Flask version of the Ruby on Rails sakila application: https://github.com/archerydwd/ror_sakila & the Chicago Boss version is here: https://github.com/archerydwd/cb_sakila

I am going to be performing tests on this app using some load testing tools such as Gatling & Tsung. 

Once I have tested this application and the other verisons of it, I will publish the results, which can then be used as a benchmark for others when trying to choose a framework.

You can build this app using a framework of your choosing and then follow the testing mechanisms that I will describe and then compare the results against my benchmark to get an indication of performance levels of your chosen framework.

=
###Install Python

At time of writing this the Python version was: 3.4.3 and the Flask version was: 0.10.1

**On OSX** 

Follow the link: https://www.python.org/ftp/python/3.4.3/python-3.4.3-macosx10.6.pkg
Open this and follow the instructions.

**On Linux**

```
wget http://www.python.org/ftp/python/3.4.3/Python-3.4.3.tgz
tar -xzf Python-3.4.3.tgz  
cd Python-3.4.3

./configure  
make  
sudo make install
```

=
###Install pip on linux

We will use pip to install flask.

```
sudo apt-get install python-pip
```

=
###Install Flask

```
sudo pip install Flask
```

=
###Install MySQL

**On Linux**

When it asks for a password, you can set it to what you want. For the purposes of this I will use the password "password"

```
sudo apt-get install mysql-server
```

Start the MySQL Server

```
sudo etc/init.d/apache2 start
```

**Install msql-connector-python**

```
sudo pip install mysql-connector-python --allow-external mysql-connector-python
```

**On OSX**

```
brew install mysql-connector-c
sudo pip install mysql-python
```

=
###Create the database

To create the database, we need to login and enter a few commands. Please note, if this is your first time using mysql, the first time you login and enter a password, this acts as setting a password. If you don't want to set a password (bad idea) just hit enter when it requests the password.

```
mysql -u root -p
create database flask_sakila;
use flask_sakila;

source /home/darren/git/flask_sakila/flask_sakila_dump.sql
```

Then to check that this has indeed worked, you can enter the following command and you should see a list of the tables in the database:

```
show tables;
```

=
###Building the sakila app


