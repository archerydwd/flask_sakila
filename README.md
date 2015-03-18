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

>touch main.py

>vim main.py

```
from flask import Flask, render_template, request, url_for, redirect, flash
import datetime
from MyUtils import UseDatabase

app = Flask(__name__)

@app.route('/')
def index():
	return(render_template('index.html', heading="Index page", index_actor_link=url_for("actors_index"), index_address_link=url_for("addresses_index"), index_category_link=url_for("categories_index"), index_city_link=url_for("cities_index"), index_country_link=url_for("countries_index"), index_customer_link=url_for("customers_index"), index_film_link=url_for("films_index"), index_filmtext_link=url_for("filmtexts_index"), index_inventory_link=url_for("inventories_index"), index_language_link=url_for("languages_index"), index_payment_link=url_for("payments_index"), index_rental_link=url_for("rentals_index"), index_staff_link=url_for("staffs_index"), index_store_link=url_for("stores_index")))

# =============================================================================================
# ACTORS
# =============================================================================================

# ACTORS INDEX
@app.route('/actors/index', methods=['GET', 'POST'])
def actors_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from actors"""
		cursor.execute(SQL)
		actors_list = cursor.fetchall()
	return(render_template('/actors/index.html', heading="Listing actors", actors=actors_list, show_actor_link=url_for("show_actor"), update_actor_link=url_for("update_actor"), delete_actor_link=url_for("delete_actor"), create_actor_link=url_for("create_actor")))

# ACTORS DELETE
@app.route('/actors/delete', methods=['POST'])
def delete_actor():
	SQL = """delete from actors where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("actors_index")))	

# ACTORS CREATE
@app.route('/actors/create', methods=['GET', 'POST'])
def create_actor():
	return(render_template("/actors/create.html", heading="Create a new Actor", index_actor_link=url_for("actors_index"), save_actor_link=url_for("save_actor")))

# ACTORS CREATE SAVE
@app.route('/actors/save', methods=['POST'])
def save_actor():
	INSERT = """insert into actors (first_name, last_name) VALUES (%s, %s)"""
	all_ok = True
	if len(request.form['first_name']) == 0:
		all_ok = False
		flash("Sorry the actor's first name cannot be empty. Try again")
	if len(request.form['last_name']) == 0:
		all_ok = False
		flash("Sorry the actor's last name cannot be empty. Try again")
	if all_ok:
		with UseDatabase(app.config) as cursor:
			cursor.execute(INSERT, (request.form['first_name'], request.form['last_name']))
			SELECTALL = """select * from actors"""
			cursor.execute(SELECTALL)
			actors_list = cursor.fetchall()
		return(render_template('/actors/index.html', heading="Listing actors", actors=actors_list, show_actor_link=url_for("show_actor"), update_actor_link=url_for("update_actor"), delete_actor_link=url_for("delete_actor"), create_actor_link=url_for("create_actor")))
	else:
		return(redirect(url_for("create_actor")))

# ACTORS UPDATE
@app.route('/actors/update', methods=['POST'])
def update_actor():
	with UseDatabase(app.config) as cursor:
		ACTOR = """select * from actors where id = %s"""
		cursor.execute(ACTOR, (request.form['id'],))
		actor_info = cursor.fetchall()
	return(render_template("/actors/update.html", heading="Editing actor", actor=actor_info, show_actor_link=url_for("show_actor"), index_actor_link=url_for("actors_index"), update_actor_save_link=url_for("save_actor_changes")))

# ACTORS UPDATE SAVE
@app.route('/actors/update_save', methods=['POST'])
def save_actor_changes():
	with UseDatabase(app.config) as cursor:
		UPDATE = """update actors set first_name = %s, last_name = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['first_name'], request.form['last_name'], request.form['id']))
		SELECT = """select * from actors"""
		cursor.execute(SELECT)
		actors_list = cursor.fetchall()
	return(render_template('/actors/index.html', heading="Listing actors", actors=actors_list, show_actor_link=url_for("show_actor"), update_actor_link=url_for("update_actor"), delete_actor_link=url_for("delete_actor"), create_actor_link=url_for("create_actor")))

# ACTORS SHOW
@app.route('/actors/show', methods=['POST'])
def show_actor():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from actors where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		actor_info = cursor.fetchall()
	return(render_template('/actors/show.html', actor=actor_info, index_actor_link=url_for("actors_index"), update_actor_link=url_for("update_actor")))

# =============================================================================================
# ADDRESSES
# =============================================================================================

# ADDRESSES INDEX
@app.route('/addresses/index', methods=['GET', 'POST'])
def addresses_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from addresses"""
		cursor.execute(SQL)
		addresses_list = cursor.fetchall()
	return(render_template('/addresses/index.html', heading="Listing addresses", addresses=addresses_list, show_address_link=url_for("show_address"), update_address_link=url_for("update_address"), delete_address_link=url_for("delete_address"), create_address_link=url_for("create_address")))

# ADDRESSES DELETE
@app.route('/addresses/delete', methods=['POST'])
def delete_address():
	SQL = """delete from addresses where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("addresses_index")))	

# ADDRESSES CREATE
@app.route('/addresses/create', methods=['GET', 'POST'])
def create_address():
	return(render_template("/addresses/create.html", heading="Create a new Address", index_address_link=url_for("addresses_index"), save_address_link=url_for("save_address")))

# ADDRESSES CREATE SAVE
@app.route('/addresses/save', methods=['POST'])
def save_address():
	INSERT = """insert into addresses (address, district, city_id, postal_code, phone) VALUES (%s, %s, %s, %s, %s)"""
	all_ok = True
	if len(request.form['address']) == 0:
		all_ok = False
		flash("Sorry the address cannot be empty. Try again")
	if len(request.form['district']) == 0:
		all_ok = False
		flash("Sorry the district cannot be empty. Try again")
	if len(request.form['city_id']) == 0:
		all_ok = False
		flash("Sorry the city id cannot be empty. Try again")
	if len(request.form['postal_code']) == 0:
		all_ok = False
		flash("Sorry the postal code cannot be empty. Try again")
	if len(request.form['phone']) == 0:
		all_ok = False
		flash("Sorry the phone cannot be empty. Try again")
	if all_ok:
		with UseDatabase(app.config) as cursor:
			cursor.execute(INSERT, (request.form['address'], request.form['district'], request.form['city_id'], request.form['postal_code'], request.form['phone'],))
			SELECTALL = """select * from addresses"""
			cursor.execute(SELECTALL)
			addresses_list = cursor.fetchall()	
		return(render_template('/addresses/index.html', heading="Listing addresses", addresses=addresses_list, show_address_link=url_for("show_address"), update_address_link=url_for("update_address"), delete_address_link=url_for("delete_address"), create_address_link=url_for("create_address")))
	else:
		return(redirect(url_for("create_address")))

# ADDRESSES UPDATE
@app.route('/addresses/update', methods=['POST'])
def update_address():
	with UseDatabase(app.config) as cursor:
		ADDRESS = """select * from addresses where id = %s"""
		cursor.execute(ADDRESS, (request.form['id'],))
		address_info = cursor.fetchall()
	return(render_template("/addresses/update.html", heading="Editing address", address=address_info, show_address_link=url_for("show_address"), index_address_link=url_for("addresses_index"), update_address_save_link=url_for("save_address_changes")))

# ADDRESSES UPDATE SAVE
@app.route('/addresses/update_save', methods=['POST'])
def save_address_changes():
	with UseDatabase(app.config) as cursor:
		UPDATE = """update addresses set address = %s, district = %s, city_id = %s, postal_code = %s, phone = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['address'], request.form['district'], request.form['city_id'], request.form['postal_code'], request.form['phone'], request.form['id']))
		SELECT = """select * from addresses"""
		cursor.execute(SELECT)
		addresses_list = cursor.fetchall()
	return(render_template('/addresses/index.html', heading="Listing addresses", addresses=addresses_list, show_address_link=url_for("show_address"), update_address_link=url_for("update_address"), delete_address_link=url_for("delete_address"), create_address_link=url_for("create_address")))

# ADDRESSES SHOW
@app.route('/addresses/show', methods=['POST'])
def show_address():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from addresses where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		address_info = cursor.fetchall()
	return(render_template('/addresses/show.html', address=address_info, index_address_link=url_for("addresses_index"), update_address_link=url_for("update_address")))

# =============================================================================================
# CATEGORIES
# =============================================================================================

# CATEGORIES INDEX
@app.route('/categories/index', methods=['GET', 'POST'])
def categories_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from categories"""
		cursor.execute(SQL)
		categories_list = cursor.fetchall()
	return(render_template('/categories/index.html', heading="Listing categories", categories=categories_list, show_category_link=url_for("show_category"), update_category_link=url_for("update_category"), delete_category_link=url_for("delete_category"), create_category_link=url_for("create_category")))

# CATEGORIES DELETE
@app.route('/categories/delete', methods=['POST'])
def delete_category():
	SQL = """delete from categories where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("categories_index")))	

# CATEGORIES CREATE
@app.route('/categories/create', methods=['GET', 'POST'])
def create_category():
	return(render_template("/categories/create.html", heading="Create a new Category", index_category_link=url_for("categories_index"), save_category_link=url_for("save_category")))

# CATEGORIES CREATE SAVE
@app.route('/categories/save', methods=['POST'])
def save_category():
	INSERT = """insert into categories (name) VALUES (%s)"""
	all_ok = True
	if len(request.form['name']) == 0:
		all_ok = False
		flash("Sorry the categorie's name cannot be empty. Try again")
	if all_ok:
		with UseDatabase(app.config) as cursor:
			cursor.execute(INSERT, (request.form['name'],))
			SELECTALL = """select * from categories"""
			cursor.execute(SELECTALL)
			categories_list = cursor.fetchall()
		return(render_template('/categories/index.html', heading="Listing categories", categories=categories_list, show_category_link=url_for("show_category"), update_category_link=url_for("update_category"), delete_category_link=url_for("delete_category"), create_category_link=url_for("create_category")))
	else:
		return(redirect(url_for("create_category")))

# CATEGORIES UPDATE
@app.route('/categories/update', methods=['POST'])
def update_category():
	with UseDatabase(app.config) as cursor:
		CATEGORY = """select * from categories where id = %s"""
		cursor.execute(CATEGORY, (request.form['id'],))
		category_info = cursor.fetchall()
	return(render_template("/categories/update.html", heading="Editing category", category=category_info, show_category_link=url_for("show_category"), index_category_link=url_for("categories_index"), update_category_save_link=url_for("save_category_changes")))

# CATEGORIES UPDATE SAVE
@app.route('/categories/update_save', methods=['POST'])
def save_category_changes():
	with UseDatabase(app.config) as cursor:
		UPDATE = """update categories set name = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['name'], request.form['id']))
		SELECT = """select * from categories"""
		cursor.execute(SELECT)
		categories_list = cursor.fetchall()
	return(render_template('/categories/index.html', heading="Listing categories", categories=categories_list, show_category_link=url_for("show_category"), update_category_link=url_for("update_category"), delete_category_link=url_for("delete_category"), create_category_link=url_for("create_category")))

# CATEGORIES SHOW
@app.route('/categories/show', methods=['POST'])
def show_category():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from categories where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		category_info = cursor.fetchall()
	return(render_template('/categories/show.html', category=category_info, index_category_link=url_for("categories_index"), update_category_link=url_for("update_category")))

# =============================================================================================
# CITIES
# =============================================================================================

# CITIES INDEX
@app.route('/cities/index', methods=['GET', 'POST'])
def cities_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from cities"""
		cursor.execute(SQL)
		cities_list = cursor.fetchall()
	return(render_template('/cities/index.html', heading="Listing cities", cities=cities_list, show_city_link=url_for("show_city"), update_city_link=url_for("update_city"), delete_city_link=url_for("delete_city"), create_city_link=url_for("create_city")))

# CITIES DELETE
@app.route('/cities/delete', methods=['POST'])
def delete_city():
	SQL = """delete from cities where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("cities_index")))	

# CITIES CREATE
@app.route('/cities/create', methods=['GET', 'POST'])
def create_city():
	return(render_template("/cities/create.html", heading="Create a new City", index_city_link=url_for("cities_index"), save_city_link=url_for("save_city")))

# CITIES CREATE SAVE
@app.route('/cities/save', methods=['POST'])
def save_city():
	INSERT = """insert into cities (city, country_id) VALUES (%s, %s)"""
	all_ok = True
	if len(request.form['city']) == 0:
		all_ok = False
		flash("Sorry the city name cannot be empty. Try again")
	if len(request.form['country_id']) == 0:
		all_ok = False
		flash("Sorry the country's id cannot be empty. Try again")
	if all_ok:
		with UseDatabase(app.config) as cursor:
			cursor.execute(INSERT, (request.form['city'], request.form['country_id']))
			SELECTALL = """select * from cities"""
			cursor.execute(SELECTALL)
			cities_list = cursor.fetchall()
		return(render_template('/cities/index.html', heading="Listing cities", cities=cities_list, show_city_link=url_for("show_city"), update_city_link=url_for("update_city"), delete_city_link=url_for("delete_city"), create_city_link=url_for("create_city")))
	else:
		return(redirect(url_for("create_city")))

# CITIES UPDATE
@app.route('/cities/update', methods=['POST'])
def update_city():
	with UseDatabase(app.config) as cursor:
		CITY = """select * from cities where id = %s"""
		cursor.execute(CITY, (request.form['id'],))
		city_info = cursor.fetchall()
	return(render_template("/cities/update.html", heading="Editing city", city=city_info, show_city_link=url_for("show_city"), index_city_link=url_for("cities_index"), update_city_save_link=url_for("save_city_changes")))

# CITIES UPDATE SAVE
@app.route('/cities/update_save', methods=['POST'])
def save_city_changes():
	with UseDatabase(app.config) as cursor:
		UPDATE = """update cities set city = %s, country_id = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['city'], request.form['country_id'], request.form['id']))
		SELECT = """select * from cities"""
		cursor.execute(SELECT)
		cities_list = cursor.fetchall()
	return(render_template('/cities/index.html', heading="Listing cities", cities=cities_list, show_city_link=url_for("show_city"), update_city_link=url_for("update_city"), delete_city_link=url_for("delete_city"), create_city_link=url_for("create_city")))

# CITIES SHOW
@app.route('/cities/show', methods=['POST'])
def show_city():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from cities where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		city_info = cursor.fetchall()
	return(render_template('/cities/show.html', city=city_info, index_city_link=url_for("cities_index"), update_city_link=url_for("update_city")))

# =============================================================================================
# COUNTRIES
# =============================================================================================

# COUNTRIES INDEX
@app.route('/countries/index', methods=['GET', 'POST'])
def countries_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from countries"""
		cursor.execute(SQL)
		countries_list = cursor.fetchall()
	return(render_template('/countries/index.html', heading="Listing countries", countries=countries_list, show_country_link=url_for("show_country"), update_country_link=url_for("update_country"), delete_country_link=url_for("delete_country"), create_country_link=url_for("create_country")))

# COUNTRIES DELETE
@app.route('/countries/delete', methods=['POST'])
def delete_country():
	SQL = """delete from countries where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("countries_index")))	

# COUNTRIES CREATE
@app.route('/countries/create', methods=['GET', 'POST'])
def create_country():
	return(render_template("/countries/create.html", heading="Create a new Country", index_country_link=url_for("countries_index"), save_country_link=url_for("save_country")))

# COUNTRIES CREATE SAVE
@app.route('/countries/save', methods=['POST'])
def save_country():
	INSERT = """insert into countries (country) VALUES (%s)"""
	all_ok = True
	if len(request.form['country']) == 0:
		all_ok = False
		flash("Sorry the country name cannot be empty. Try again")
	if all_ok:
		with UseDatabase(app.config) as cursor:
			cursor.execute(INSERT, (request.form['country'],))
			SELECTALL = """select * from countries"""
			cursor.execute(SELECTALL)
			countries_list = cursor.fetchall()
		return(render_template('/countries/index.html', heading="Listing countries", countries=countries_list, show_country_link=url_for("show_country"), update_country_link=url_for("update_country"), delete_country_link=url_for("delete_country"), create_country_link=url_for("create_country")))
	else:
		return(redirect(url_for("create_country")))

# COUNTRIES UPDATE
@app.route('/countries/update', methods=['POST'])
def update_country():
	with UseDatabase(app.config) as cursor:
		COUNTRY = """select * from countries where id = %s"""
		cursor.execute(COUNTRY, (request.form['id'],))
		country_info = cursor.fetchall()
	return(render_template("/countries/update.html", heading="Editing country", country=country_info, show_country_link=url_for("show_country"), index_country_link=url_for("countries_index"), update_country_save_link=url_for("save_country_changes")))

# COUNTRIES UPDATE SAVE
@app.route('/countries/update_save', methods=['POST'])
def save_country_changes():
	with UseDatabase(app.config) as cursor:
		UPDATE = """update countries set country = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['country'], request.form['id']))
		SELECT = """select * from countries"""
		cursor.execute(SELECT)
		countries_list = cursor.fetchall()
	return(render_template('/countries/index.html', heading="Listing countries", countries=countries_list, show_country_link=url_for("show_country"), update_country_link=url_for("update_country"), delete_country_link=url_for("delete_country"), create_country_link=url_for("create_country")))

# COUNTRIES SHOW
@app.route('/countries/show', methods=['POST'])
def show_country():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from countries where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		country_info = cursor.fetchall()
	return(render_template('/countries/show.html', country=country_info, index_country_link=url_for("countries_index"), update_country_link=url_for("update_country")))

# =============================================================================================
# CUSTOMERS
# =============================================================================================

# CUSTOMERS INDEX
@app.route('/customers/index', methods=['GET', 'POST'])
def customers_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from customers"""
		cursor.execute(SQL)
		customers_list = cursor.fetchall()
	return(render_template('/customers/index.html', heading="Listing customers", customers=customers_list, show_customer_link=url_for("show_customer"), update_customer_link=url_for("update_customer"), delete_customer_link=url_for("delete_customer"), create_customer_link=url_for("create_customer")))

# CUSTOMERS DELETE
@app.route('/customers/delete', methods=['POST'])
def delete_customer():
	SQL = """delete from customers where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("customers_index")))	

# CUSTOMERS CREATE
@app.route('/customers/create', methods=['GET', 'POST'])
def create_customer():
	return(render_template("/customers/create.html", heading="Create a new Customer", index_customer_link=url_for("customers_index"), save_customer_link=url_for("save_customer")))

# CUSTOMERS CREATE SAVE
@app.route('/customers/save', methods=['POST'])
def save_customer():
	if request.form['active'] == 'on':
		checkbox = 1;
	else:
		checkbox = 0;
	INSERT = """insert into customers (store_id, first_name, last_name, email, address_id, active, create_date) VALUES (%s, %s, %s, %s, %s, %s, %s)"""
	all_ok = True
	if len(request.form['first_name']) == 0:
		all_ok = False
		flash("Sorry the customer's first name cannot be empty. Try again")
	if len(request.form['last_name']) == 0:
		all_ok = False
		flash("Sorry the customer's last name cannot be empty. Try again")
	if len(request.form['email']) == 0:
		all_ok = False
		flash("Sorry the customer's email cannot be empty. Try again")
	if all_ok:
		with UseDatabase(app.config) as cursor:
			current_time = datetime.datetime.now().time()
			cursor.execute(INSERT, (request.form['store_id'], request.form['first_name'], request.form['last_name'], request.form['email'], request.form['address_id'], checkbox, current_time.strftime('%Y-%m-%d %H:%M:%S'),))
			SELECTALL = """select * from customers"""
			cursor.execute(SELECTALL)
			customers_list = cursor.fetchall()
		return(render_template('/customers/index.html', heading="Listing customers", customers=customers_list, show_customer_link=url_for("show_customer"), update_customer_link=url_for("update_customer"), delete_customer_link=url_for("delete_customer"), create_customer_link=url_for("create_customer")))
	else:
		return(redirect(url_for("create_customer")))

# CUSTOMERS UPDATE
@app.route('/customers/update', methods=['POST'])
def update_customer():
	with UseDatabase(app.config) as cursor:
		CUSTOMER = """select * from customers where id = %s"""
		cursor.execute(CUSTOMER, (request.form['id'],))
		customer_info = cursor.fetchall()
	return(render_template("/customers/update.html", heading="Editing customer", customer=customer_info, show_customer_link=url_for("show_customer"), index_customer_link=url_for("customers_index"), update_customer_save_link=url_for("save_customer_changes")))

# CUSTOMERS UPDATE SAVE
@app.route('/customers/update_save', methods=['POST'])
def save_customer_changes():
	with UseDatabase(app.config) as cursor:
		if request.form['active'] == 'on':
			checkbox = 1;
		else:
			checkbox = 0;
		UPDATE = """update customers set store_id = %s, first_name = %s, last_name = %s, email = %s, address_id = %s, active = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['store_id'], request.form['first_name'], request.form['last_name'], request.form['email'], request.form['address_id'], checkbox, request.form['id']))
		SELECT = """select * from customers"""
		cursor.execute(SELECT)
		customers_list = cursor.fetchall()
	return(render_template('/customers/index.html', heading="Listing customers", customers=customers_list, show_customer_link=url_for("show_customer"), update_customer_link=url_for("update_customer"), delete_customer_link=url_for("delete_customer"), create_customer_link=url_for("create_customer")))

# CUSTOMERS SHOW
@app.route('/customers/show', methods=['POST'])
def show_customer():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from customers where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		customer_info = cursor.fetchall()
	return(render_template('/customers/show.html', customer=customer_info, index_customer_link=url_for("customers_index"), update_customer_link=url_for("update_customer")))

# =============================================================================================
# FILMS
# =============================================================================================

# FILMS INDEX
@app.route('/films/index', methods=['GET', 'POST'])
def films_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from films"""
		cursor.execute(SQL)
		films_list = cursor.fetchall()
	return(render_template('/films/index.html', heading="Listing films", films=films_list, show_film_link=url_for("show_film"), update_film_link=url_for("update_film"), delete_film_link=url_for("delete_film"), create_film_link=url_for("create_film")))

# FILMS DELETE
@app.route('/films/delete', methods=['POST'])
def delete_film():
	SQL = """delete from films where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("films_index")))	

# FILMS CREATE
@app.route('/films/create', methods=['GET', 'POST'])
def create_film():
	return(render_template("/films/create.html", heading="Create a new Film", index_film_link=url_for("films_index"), save_film_link=url_for("save_film")))

# FILMS CREATE SAVE
@app.route('/films/save', methods=['POST'])
def save_film():
	INSERT = """insert into films (title, description, release_year, language_id, rental_duration, rental_rate, length, replacement_cost, rating, special_features) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s)"""
	all_ok = True
	if len(request.form['title']) == 0:
		all_ok = False
		flash("Sorry the film's title cannot be empty. Try again")
	if len(request.form['description']) == 0:
		all_ok = False
		flash("Sorry the film's description cannot be empty. Try again")
	if all_ok:
		with UseDatabase(app.config) as cursor:
			cursor.execute(INSERT, (request.form['title'], request.form['description'], request.form['release_year'], request.form['language_id'], request.form['rental_duration'], request.form['rental_rate'], request.form['length'], request.form['replacement_cost'], request.form['rating'], request.form['special_features'],))
			SELECTALL = """select * from films"""
			cursor.execute(SELECTALL)
			films_list = cursor.fetchall()
		return(render_template('/films/index.html', heading="Listing films", films=films_list, show_film_link=url_for("show_film"), update_film_link=url_for("update_film"), delete_film_link=url_for("delete_film"), create_film_link=url_for("create_film")))
	else:
		return(redirect(url_for("create_film")))

# FILMS UPDATE
@app.route('/films/update', methods=['POST'])
def update_film():
	with UseDatabase(app.config) as cursor:
		FILM = """select * from films where id = %s"""
		cursor.execute(FILM, (request.form['id'],))
		film_info = cursor.fetchall()
	return(render_template("/films/update.html", heading="Editing film", film=film_info, show_film_link=url_for("show_film"), index_film_link=url_for("films_index"), update_film_save_link=url_for("save_film_changes")))

# FILMS UPDATE SAVE
@app.route('/films/update_save', methods=['POST'])
def save_film_changes():
	with UseDatabase(app.config) as cursor:
		UPDATE = """update films set title = %s, description = %s, release_year = %s, language_id = %s, rental_duration = %s, rental_rate = %s, length = %s, replacement_cost = %s, rating = %s, special_features = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['title'], request.form['description'], request.form['release_year'], request.form['language_id'], request.form['rental_duration'], request.form['rental_rate'], request.form['length'], request.form['replacement_cost'], request.   form['rating'], request.form['special_features'], request.form['id'],))
		SELECT = """select * from films"""
		cursor.execute(SELECT)
		films_list = cursor.fetchall()
	return(render_template('/films/index.html', heading="Listing films", films=films_list, show_film_link=url_for("show_film"), update_film_link=url_for("update_film"), delete_film_link=url_for("delete_film"), create_film_link=url_for("create_film")))

# FILMS SHOW
@app.route('/films/show', methods=['POST'])
def show_film():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from films where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		film_info = cursor.fetchall()
	return(render_template('/films/show.html', film=film_info, index_film_link=url_for("films_index"), update_film_link=url_for("update_film")))

# =============================================================================================
# FILM TEXTS
# =============================================================================================

# FILMS TEXTS INDEX
@app.route('/filmtexts/index', methods=['GET', 'POST'])
def filmtexts_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from filmtexts"""
		cursor.execute(SQL)
		filmtexts_list = cursor.fetchall()
	return(render_template('/filmtexts/index.html', heading="Listing film texts", filmtexts=filmtexts_list, show_filmtext_link=url_for("show_filmtext"), update_filmtext_link=url_for("update_filmtext"), delete_filmtext_link=url_for("delete_filmtext"), create_filmtext_link=url_for("create_filmtext")))

# FILMS TEXTS DELETE
@app.route('/filmtexts/delete', methods=['POST'])
def delete_filmtext():
	SQL = """delete from filmtexts where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("filmtexts_index")))	

# FILMS TEXTS CREATE
@app.route('/filmtexts/create', methods=['GET', 'POST'])
def create_filmtext():
	return(render_template("/filmtexts/create.html", heading="Create a new Film Text", index_filmtext_link=url_for("filmtexts_index"), save_filmtext_link=url_for("save_filmtext")))

# FILMS TEXTS CREATE SAVE
@app.route('/filmtexts/save', methods=['POST'])
def save_filmtext():
	INSERT = """insert into filmtexts (title, description) VALUES (%s, %s)"""
	all_ok = True
	if len(request.form['title']) == 0:
		all_ok = False
		flash("Sorry the film text's title cannot be empty. Try again")
	if len(request.form['description']) == 0:
		all_ok = False
		flash("Sorry the film text's description cannot be empty. Try again")
	if all_ok:
		with UseDatabase(app.config) as cursor:
			cursor.execute(INSERT, (request.form['title'], request.form['description']))
			SELECTALL = """select * from filmtexts"""
			cursor.execute(SELECTALL)
			filmtexts_list = cursor.fetchall()
		return(render_template('/filmtexts/index.html', heading="Listing filmtexts", filmtexts=filmtexts_list, show_filmtext_link=url_for("show_filmtext"), update_filmtext_link=url_for("update_filmtext"), delete_filmtext_link=url_for("delete_filmtext"), create_filmtext_link=url_for("create_filmtext")))
	else:
		return(redirect(url_for("create_filmtext")))

# FILMS TEXTS UPDATE
@app.route('/filmtexts/update', methods=['POST'])
def update_filmtext():
	with UseDatabase(app.config) as cursor:
		FILMTEXT = """select * from filmtexts where id = %s"""
		cursor.execute(FILMTEXT, (request.form['id'],))
		filmtext_info = cursor.fetchall()
	return(render_template("/filmtexts/update.html", heading="Editing film text", filmtext=filmtext_info, show_filmtext_link=url_for("show_filmtext"), index_filmtext_link=url_for("filmtexts_index"), update_filmtext_save_link=url_for("save_filmtext_changes")))

# FILMS TEXTS UPDATE SAVE
@app.route('/filmtexts/update_save', methods=['POST'])
def save_filmtext_changes():
	with UseDatabase(app.config) as cursor:
		UPDATE = """update filmtexts set title = %s, description = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['title'], request.form['description'], request.form['id']))
		SELECT = """select * from filmtexts"""
		cursor.execute(SELECT)
		filmtexts_list = cursor.fetchall()
	return(render_template('/filmtexts/index.html', heading="Listing film texts", filmtexts=filmtexts_list, show_filmtext_link=url_for("show_filmtext"), update_filmtext_link=url_for("update_filmtext"), delete_filmtext_link=url_for("delete_filmtext"), create_filmtext_link=url_for("create_filmtext")))

# FILMS TEXTS SHOW
@app.route('/filmtexts/show', methods=['POST'])
def show_filmtext():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from filmtexts where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		filmtext_info = cursor.fetchall()
	return(render_template('/filmtexts/show.html', filmtext=filmtext_info, index_filmtext_link=url_for("filmtexts_index"), update_filmtext_link=url_for("update_filmtext")))



# =============================================================================================
# INVENTORIES						
# =============================================================================================

# INVENTORIES INDEX
@app.route('/inventories/index', methods=['GET', 'POST'])
def inventories_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from inventories"""
		cursor.execute(SQL)
		inventories_list = cursor.fetchall()
	return(render_template('/inventories/index.html', heading="Listing inventories", inventories=inventories_list, show_inventory_link=url_for("show_inventory"), update_inventory_link=url_for("update_inventory"), delete_inventory_link=url_for("delete_inventory"), create_inventory_link=url_for("create_inventory")))

# INVENTORIES DELETE
@app.route('/inventories/delete', methods=['POST'])
def delete_inventory():
	SQL = """delete from inventories where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("inventories_index")))	

# INVENTORIES CREATE
@app.route('/inventories/create', methods=['GET', 'POST'])
def create_inventory():
	return(render_template("/inventories/create.html", heading="Create a new Inventory", index_inventory_link=url_for("inventories_index"), save_inventory_link=url_for("save_inventory")))

# INVENTORIES CREATE SAVE
@app.route('/inventories/save', methods=['POST'])
def save_inventory():
	INSERT = """insert into inventories (film_id, store_id) VALUES (%s, %s)"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(INSERT, (request.form['film_id'], request.form['store_id']))
		SELECTALL = """select * from inventories"""
		cursor.execute(SELECTALL)
		inventories_list = cursor.fetchall()
	return(render_template('/inventories/index.html', heading="Listing inventories", inventories=inventories_list, show_inventory_link=url_for("show_inventory"), update_inventory_link=url_for("update_inventory"), delete_inventory_link=url_for("delete_inventory"), create_inventory_link=url_for("create_inventory")))

# INVENTORIES UPDATE
@app.route('/inventories/update', methods=['POST'])
def update_inventory():
	with UseDatabase(app.config) as cursor:
		INVENTORY = """select * from inventories where id = %s"""
		cursor.execute(INVENTORY, (request.form['id'],))
		inventory_info = cursor.fetchall()
	return(render_template("/inventories/update.html", heading="Editing inventories", inventory=inventory_info, show_inventory_link=url_for("show_inventory"), index_inventory_link=url_for("inventories_index"), update_inventory_save_link=url_for("save_inventory_changes")))

# INVENTORIES UPDATE SAVE
@app.route('/inventories/update_save', methods=['POST'])
def save_inventory_changes():
	with UseDatabase(app.config) as cursor:
		UPDATE = """update inventories set film_id = %s, store_id = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['film_id'], request.form['store_id'], request.form['id']))
		SELECT = """select * from inventories"""
		cursor.execute(SELECT)
		inventories_list = cursor.fetchall()
	return(render_template('/inventories/index.html', heading="Listing inventories", inventories=inventories_list, show_inventory_link=url_for("show_inventory"), update_inventory_link=url_for("update_inventory"), delete_inventory_link=url_for("delete_inventory"), create_inventory_link=url_for("create_inventory")))

# INVENTORIES SHOW
@app.route('/inventories/show', methods=['POST'])
def show_inventory():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from inventories where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		inventory_info = cursor.fetchall()
	return(render_template('/inventories/show.html', inventory=inventory_info, index_inventory_link=url_for("inventories_index"), update_inventory_link=url_for("update_inventory")))



# =============================================================================================
# LANGUAGES
# =============================================================================================

# LANGUAGES INDEX
@app.route('/languages/index', methods=['GET', 'POST'])
def languages_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from languages"""
		cursor.execute(SQL)
		languages_list = cursor.fetchall()
	return(render_template('/languages/index.html', heading="Listing languages", languages=languages_list, show_language_link=url_for("show_language"), update_language_link=url_for("update_language"), delete_language_link=url_for("delete_language"), create_language_link=url_for("create_language")))

# LANGUAGES DELETE
@app.route('/languages/delete', methods=['POST'])
def delete_language():
	SQL = """delete from languages where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("languages_index")))	

# LANGUAGES CREATE
@app.route('/languages/create', methods=['GET', 'POST'])
def create_language():
	return(render_template("/languages/create.html", heading="Create a new Language", index_language_link=url_for("languages_index"), save_language_link=url_for("save_language")))

# LANGUAGES CREATE SAVE
@app.route('/languages/save', methods=['POST'])
def save_language():
	INSERT = """insert into languages (name) VALUES (%s)"""
	all_ok = True
	if len(request.form['name']) == 0:
		all_ok = False
		flash("Sorry the language's name cannot be empty. Try again")
	if all_ok:
		with UseDatabase(app.config) as cursor:
			cursor.execute(INSERT, (request.form['name'],))
			SELECTALL = """select * from languages"""
			cursor.execute(SELECTALL)
			languages_list = cursor.fetchall()
		return(render_template('/languages/index.html', heading="Listing languages", languages=languages_list, show_language_link=url_for("show_language"), update_language_link=url_for("update_language"), delete_language_link=url_for("delete_language"), create_language_link=url_for("create_language")))
	else:
		return(redirect(url_for("create_language")))

# LANGUAGES UPDATE
@app.route('/languages/update', methods=['POST'])
def update_language():
	with UseDatabase(app.config) as cursor:
		LANGUAGE = """select * from languages where id = %s"""
		cursor.execute(LANGUAGE, (request.form['id'],))
		language_info = cursor.fetchall()
	return(render_template("/languages/update.html", heading="Editing language", language=language_info, show_language_link=url_for("show_language"), index_language_link=url_for("languages_index"), update_language_save_link=url_for("save_language_changes")))

# LANGUAGES UPDATE SAVE
@app.route('/languages/update_save', methods=['POST'])
def save_language_changes():
	with UseDatabase(app.config) as cursor:
		UPDATE = """update languages set name = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['name'], request.form['id']))
		SELECT = """select * from languages"""
		cursor.execute(SELECT)
		languages_list = cursor.fetchall()
	return(render_template('/languages/index.html', heading="Listing languages", languages=languages_list, show_language_link=url_for("show_language"), update_language_link=url_for("update_language"), delete_language_link=url_for("delete_language"), create_language_link=url_for("create_language")))

# LANGUAGES SHOW
@app.route('/languages/show', methods=['POST'])
def show_language():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from languages where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		language_info = cursor.fetchall()
	return(render_template('/languages/show.html', language=language_info, index_language_link=url_for("languages_index"), update_language_link=url_for("update_language")))



# =============================================================================================
# PAYMENTS
# =============================================================================================

# PAYMENTS INDEX
@app.route('/payments/index', methods=['GET', 'POST'])
def payments_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from payments"""
		cursor.execute(SQL)
		payments_list = cursor.fetchall()
	return(render_template('/payments/index.html', heading="Listing payments", payments=payments_list, show_payment_link=url_for("show_payment"), update_payment_link=url_for("update_payment"), delete_payment_link=url_for("delete_payment"), create_payment_link=url_for("create_payment")))

# PAYMENTS DELETE
@app.route('/payments/delete', methods=['POST'])
def delete_payment():
	SQL = """delete from payments where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("payments_index")))	

# PAYMENTS CREATE
@app.route('/payments/create', methods=['GET', 'POST'])
def create_payment():
	return(render_template("/payments/create.html", heading="Create a new Payment", index_payment_link=url_for("payments_index"), save_payment_link=url_for("save_payment")))

# PAYMENTS CREATE SAVE
@app.route('/payments/save', methods=['POST'])
def save_payment():
	INSERT = """insert into payments (customer_id, staff_id, rental_id, amount, payment_date) VALUES (%s, %s, %s, %s, %s)"""
	with UseDatabase(app.config) as cursor:
		current_time = datetime.datetime.now().time()
		cursor.execute(INSERT, (request.form['customer_id'], request.form['staff_id'], request.form['rental_id'], request.form['amount'], current_time.strftime('%Y-%m-%d %H:%M:%S')))
		SELECTALL = """select * from payments"""
		cursor.execute(SELECTALL)
		payments_list = cursor.fetchall()
	return(render_template('/payments/index.html', heading="Listing payments", payments=payments_list, show_payment_link=url_for("show_payment"), update_payment_link=url_for("update_payment"), delete_payment_link=url_for("delete_payment"), create_payment_link=url_for("create_payment")))

# PAYMENTS UPDATE
@app.route('/payments/update', methods=['POST'])
def update_payment():
	with UseDatabase(app.config) as cursor:
		PAYMENT = """select * from payments where id = %s"""
		cursor.execute(PAYMENT, (request.form['id'],))
		payment_info = cursor.fetchall()
	return(render_template("/payments/update.html", heading="Editing payment", payment=payment_info, show_payment_link=url_for("show_payment"), index_payment_link=url_for("payments_index"), update_payment_save_link=url_for("save_payment_changes")))

# PAYMENTS UPDATE SAVE
@app.route('/payments/update_save', methods=['POST'])
def save_payment_changes():
	with UseDatabase(app.config) as cursor:

		UPDATE = """update payments set customer_id = %s, staff_id = %s, rental_id = %s, amount = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['customer_id'], request.form['staff_id'], request.form['rental_id'], request.form['amount'], request.form['id']))
		SELECT = """select * from payments"""
		cursor.execute(SELECT)
		payments_list = cursor.fetchall()
	return(render_template('/payments/index.html', heading="Listing payments", payments=payments_list, show_payment_link=url_for("show_payment"), update_payment_link=url_for("update_payment"), delete_payment_link=url_for("delete_payment"), create_payment_link=url_for("create_payment")))

# PAYMENTS SHOW
@app.route('/payments/show', methods=['POST'])
def show_payment():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from payments where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		payment_info = cursor.fetchall()
	return(render_template('/payments/show.html', payment=payment_info, index_payment_link=url_for("payments_index"), update_payment_link=url_for("update_payment")))

# =============================================================================================
# RENTALS
# =============================================================================================

# RENTALS INDEX
@app.route('/rentals/index', methods=['GET', 'POST'])
def rentals_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from rentals"""
		cursor.execute(SQL)
		rentals_list = cursor.fetchall()
	return(render_template('/rentals/index.html', heading="Listing rentals", rentals=rentals_list, show_rental_link=url_for("show_rental"), update_rental_link=url_for("update_rental"), delete_rental_link=url_for("delete_rental"), create_rental_link=url_for("create_rental")))

# RENTALS DELETE
@app.route('/rentals/delete', methods=['POST'])
def delete_rental():
	SQL = """delete from rentals where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("rentals_index")))	

# RENTALS CREATE
@app.route('/rentals/create', methods=['GET', 'POST'])
def create_rental():
	return(render_template("/rentals/create.html", heading="Create a new Rental", index_rental_link=url_for("rentals_index"), save_rental_link=url_for("save_rental")))

# RENTALS CREATE SAVE
@app.route('/rentals/save', methods=['POST'])
def save_rental():
	INSERT = """insert into rentals (rental_date, inventory_id, customer_id, return_date, staff_id) VALUES (%s, %s, %s, %s, %s)"""
	with UseDatabase(app.config) as cursor:
		rental_date = datetime.datetime.now().time()
		cursor.execute(INSERT, (rental_date.strftime('%Y-%m-%d %H:%M:%S'), request.form['inventory_id'], request.form['customer_id'], rental_date.strftime('%Y-%m-%d %H:%M:%S'), request.form['staff_id']))
		SELECTALL = """select * from rentals"""
		cursor.execute(SELECTALL)
		rentals_list = cursor.fetchall()
	return(render_template('/rentals/index.html', heading="Listing rentals", rentals=rentals_list, show_rental_link=url_for("show_rental"), update_rental_link=url_for("update_rental"), delete_rental_link=url_for("delete_rental"), create_rental_link=url_for("create_rental")))

# RENTALS UPDATE
@app.route('/rentals/update', methods=['POST'])
def update_rental():
	with UseDatabase(app.config) as cursor:
		RENTAL = """select * from rentals where id = %s"""
		cursor.execute(RENTAL, (request.form['id'],))
		rental_info = cursor.fetchall()
	return(render_template("/rentals/update.html", heading="Editing rental", rental=rental_info, show_rental_link=url_for("show_rental"), index_rental_link=url_for("rentals_index"), update_rental_save_link=url_for("save_rental_changes")))

# RENTALS UPDATE SAVE
@app.route('/rentals/update_save', methods=['POST'])
def save_rental_changes():
	with UseDatabase(app.config) as cursor:
		UPDATE = """update rentals set inventory_id = %s, customer_id = %s, staff_id = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['inventory_id'], request.form['customer_id'], request.form['staff_id'], request.form['id']))
		SELECT = """select * from rentals"""
		cursor.execute(SELECT)
		rentals_list = cursor.fetchall()
	return(render_template('/rentals/index.html', heading="Listing rentals", rentals=rentals_list, show_rental_link=url_for("show_rental"), update_rental_link=url_for("update_rental"), delete_rental_link=url_for("delete_rental"), create_rental_link=url_for("create_rental")))

# RENTALS SHOW
@app.route('/rentals/show', methods=['POST'])
def show_rental():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from rentals where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		rental_info = cursor.fetchall()
	return(render_template('/rentals/show.html', rental=rental_info, index_rental_link=url_for("rentals_index"), update_rental_link=url_for("update_rental")))



# =============================================================================================
# STAFF
# =============================================================================================

# STAFF INDEX
@app.route('/staffs/index', methods=['GET', 'POST'])
def staffs_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from staffs"""
		cursor.execute(SQL)
		staffs_list = cursor.fetchall()
	return(render_template('/staffs/index.html', heading="Listing staffs", staffs=staffs_list, show_staff_link=url_for("show_staff"), update_staff_link=url_for("update_staff"), delete_staff_link=url_for("delete_staff"), create_staff_link=url_for("create_staff")))

# STAFF DELETE
@app.route('/staffs/delete', methods=['POST'])
def delete_staff():
	SQL = """delete from staffs where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("staffs_index")))	

# STAFF CREATE
@app.route('/staffs/create', methods=['GET', 'POST'])
def create_staff():
	return(render_template("/staffs/create.html", heading="Create a new Staff", index_staff_link=url_for("staffs_index"), save_staff_link=url_for("save_staff")))

# STAFF CREATE SAVE
@app.route('/staffs/save', methods=['POST'])
def save_staff():
	if request.form['active'] == 'on':
		checkbox = 1;
	else:
		checkbox = 0;
	INSERT = """insert into staffs (first_name, last_name, address_id, email, store_id, active, username, password) VALUES (%s, %s, %s, %s, %s, %s, %s, %s)"""
	all_ok = True
	if len(request.form['first_name']) == 0:
		all_ok = False
		flash("Sorry the staffs first name cannot be empty. Try again")
	if len(request.form['last_name']) == 0:
		all_ok = False
		flash("Sorry the staffs last name cannot be empty. Try again")
	if all_ok:
		with UseDatabase(app.config) as cursor:
			cursor.execute(INSERT, (request.form['first_name'], request.form['last_name'], request.form['address_id'], request.form['email'], request.form['store_id'], checkbox, request.form['username'], request.form['password'],))
			SELECTALL = """select * from staffs"""
			cursor.execute(SELECTALL)
			staffs_list = cursor.fetchall()
		return(render_template('/staffs/index.html', heading="Listing staff", staffs=staffs_list, show_staff_link=url_for("show_staff"), update_staff_link=url_for("update_staff"), delete_staff_link=url_for("delete_staff"), create_staff_link=url_for("create_staff")))
	else:
		return(redirect(url_for("create_staff")))

# STAFF UPDATE
@app.route('/staffs/update', methods=['POST'])
def update_staff():
	with UseDatabase(app.config) as cursor:
		STAFF = """select * from staffs where id = %s"""
		cursor.execute(STAFF, (request.form['id'],))
		staff_info = cursor.fetchall()
	return(render_template("/staffs/update.html", heading="Editing staff", staff=staff_info, show_staff_link=url_for("show_staff"), index_staff_link=url_for("staffs_index"), update_staff_save_link=url_for("save_staff_changes")))

# STAFF UPDATE SAVE
@app.route('/staffs/update_save', methods=['POST'])
def save_staff_changes():
	with UseDatabase(app.config) as cursor:
		if request.form['active'] == 'on':
			checkbox = 1;
		else:
			checkbox = 0;
		UPDATE = """update staffs set first_name = %s, last_name = %s, address_id = %s, email = %s, store_id = %s, active = %s, username = %s, password = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['first_name'], request.form['last_name'], request.form['address_id'], request.form['email'], request.form['store_id'], checkbox, request.form['username'], request.form['password'], request.form['id']))
		SELECT = """select * from staffs"""
		cursor.execute(SELECT)
		staffs_list = cursor.fetchall()
	return(render_template('/staffs/index.html', heading="Listing staffs", staffs=staffs_list, show_staff_link=url_for("show_staff"), update_staff_link=url_for("update_staff"), delete_staff_link=url_for("delete_staff"), create_staff_link=url_for("create_staff")))

# STAFF SHOW
@app.route('/staffs/show', methods=['POST'])
def show_staff():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from staffs where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		staff_info = cursor.fetchall()
	return(render_template('/staffs/show.html', staff=staff_info, index_staff_link=url_for("staffs_index"), update_staff_link=url_for("update_staff")))


# =============================================================================================
# STORES
# =============================================================================================

# STORES INDEX
@app.route('/stores/index', methods=['GET', 'POST'])
def stores_index():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from stores"""
		cursor.execute(SQL)
		stores_list = cursor.fetchall()
	return(render_template('/stores/index.html', heading="Listing stores", stores=stores_list, show_store_link=url_for("show_store"), update_store_link=url_for("update_store"), delete_store_link=url_for("delete_store"), create_store_link=url_for("create_store")))

# STORES DELETE
@app.route('/stores/delete', methods=['POST'])
def delete_store():
	SQL = """delete from stores where id = %s"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(SQL, (request.form['id'],))
	return(redirect(url_for("stores_index")))	

# STORES CREATE
@app.route('/stores/create', methods=['GET', 'POST'])
def create_store():
	return(render_template("/stores/create.html", heading="Create a new Store", index_store_link=url_for("stores_index"), save_store_link=url_for("save_store")))

# STORES CREATE SAVE
@app.route('/stores/save', methods=['POST'])
def save_store():
	INSERT = """insert into stores (address_id) VALUES (%s)"""
	with UseDatabase(app.config) as cursor:
		cursor.execute(INSERT, (request.form['address_id'],))
		SELECTALL = """select * from stores"""
		cursor.execute(SELECTALL)
		stores_list = cursor.fetchall()
	return(render_template('/stores/index.html', heading="Listing stores", stores=stores_list, show_store_link=url_for("show_store"), update_store_link=url_for("update_store"), delete_store_link=url_for("delete_store"), create_store_link=url_for("create_store")))

# STORES UPDATE
@app.route('/stores/update', methods=['POST'])
def update_store():
	with UseDatabase(app.config) as cursor:
		STORE = """select * from stores where id = %s"""
		cursor.execute(STORE, (request.form['id'],))
		store_info = cursor.fetchall()
	return(render_template("/stores/update.html", heading="Editing store", store=store_info, show_store_link=url_for("show_store"), index_store_link=url_for("stores_index"), update_store_save_link=url_for("save_store_changes")))

# STORES UPDATE SAVE
@app.route('/stores/update_save', methods=['POST'])
def save_store_changes():
	with UseDatabase(app.config) as cursor:
		UPDATE = """update stores set address_id = %s where id = %s"""
		cursor.execute(UPDATE, (request.form['address_id'], request.form['id']))
		SELECT = """select * from stores"""
		cursor.execute(SELECT)
		stores_list = cursor.fetchall()
	return(render_template('/stores/index.html', heading="Listing stores", stores=stores_list, show_store_link=url_for("show_store"), update_store_link=url_for("update_store"), delete_store_link=url_for("delete_store"), create_store_link=url_for("create_store")))

# STORES SHOW
@app.route('/stores/show', methods=['POST'])
def show_store():
	with UseDatabase(app.config) as cursor:
		SQL = """select * from stores where id = %s"""
		cursor.execute(SQL, (request.form['id'],))
		store_info = cursor.fetchall()
	return(render_template('/stores/show.html', store=store_info, index_store_link=url_for("stores_index"), update_store_link=url_for("update_store")))


app.config['SECRET_KEY'] = 'thisismysecretkeywhichyouwillneverguesshahahahahahahahaha'

if __name__ == "__main__":
	app.config['DB_HOST'] = '127.0.0.1'
	app.config['DB_USER'] = 'root'
	app.config['DB_PASSWORD'] = 'password'
	app.config['DB'] = 'flask_sakila'
	app.run(debug=True)
```

In the above I have imported all relevant libraries. I have then divided up the file into the seperate parts as per the database's tables. Inside each of these parts I have set up routes and methods for the CRUD actions. At the bottom I have included the if __name__ == "__main__" statement to tell if the app is being run locally or if it's on a platform that runs it automatically. If run locally we need to do app.run inside the if statement. We also set up the connection details inside the if statement.

=
###Database connector

>touch MyUtils.py

>vim MyUtils.py

```
import mysql.connector

class UseDatabase:
	def __init__(self, configuration):
		""" Initialisation code which executes the context manager is CREATED. """
		self.host = configuration['DB_HOST']
		self.user = configuration['DB_USER']
		self.password = configuration['DB_PASSWORD']
		self.db = configuration['DB']

	def __enter__(self):
		""" Set-up code which executes BEFORE the body of the with statement. """
		self.conn = mysql.connector.connect(host=self.host,
		user=self.user,
		password=self.password,
		database=self.db,)
		self.cursor = self.conn.cursor()
		return(self.cursor)

	def __exit__(self, exc_type, exv_value, exc_traceback):
		""" Tear-down code with executes AFTER the body of the with statement. The three extra parameters to __exit__() contain information related to any exception which may have occurred while running the body of the with statement. """
		self.cursor.close()
		self.conn.commit()
		self.conn.close()
```

=
###Templates

We now need to provide templates for our sakila app to work. Flask uses Jinja2 templating, so we are going to build a hierarchical template structure.

**Create the directories**

```
mkdir templates/actors
mkdir templates/addresses
mkdir templates/categories
mkdir templates/cities
mkdir templates/countries
mkdir templates/customers
mkdir templates/films
mkdir templates/filmtexts
mkdir templates/inventories
mkdir templates/languages
mkdir templates/payments
mkdir templates/rentals
mkdir templates/staffs
mkdir templates/stores
```

**Create the files**

```
touch templates/actors/index.html show.html create.html update.html
touch templates/addresses/index.html show.html create.html update.html
touch templates/categories/index.html show.html create.html update.html
touch templates/cities/index.html show.html create.html update.html
touch templates/countries/index.html show.html create.html update.html
touch templates/customers/index.html show.html create.html update.html
touch templates/films/index.html show.html create.html update.html
touch templates/filmtexts/index.html show.html create.html update.html
touch templates/inventories/index.html show.html create.html update.html
touch templates/languages/index.html show.html create.html update.html
touch templates/payments/index.html show.html create.html update.html
touch templates/rentals/index.html show.html create.html update.html
touch templates/staffs/index.html show.html create.html update.html
touch templates/stores/index.html show.html create.html update.html
touch templates/base.html index.html
```

**base.html**

>vim templates/base.html

```
<!doctype html>
<html>
	<head>
		<title>Flask sakila</title>
	</head>
	<body>
		<h1>{{ heading }}</h1>
		{% block body %}

		{% endblock %}
	</body>	
</html>
```

**index.html**

>vim templates/index.html

```
{% extends "base.html" %}

{% block body %}
	<ul>
		<li><a href="{{ index_actor_link }}">Actors</a></li>
		<li><a href="{{ index_address_link }}">Addresses</a></li>
		<li><a href="{{ index_category_link }}">Categories</a></li>
		<li><a href="{{ index_city_link }}">Cities</a></li>
		<li><a href="{{ index_country_link }}">Countries</a></li>
		<li><a href="{{ index_customer_link }}">Customers</a></li>
		<li><a href="{{ index_film_link }}">Films</a></li>
		<li><a href="{{ index_filmtext_link }}">Film Texts</a></li>
		<li><a href="{{ index_inventory_link }}">Inventories</a></li>
		<li><a href="{{ index_language_link }}">Languages</a></li>
		<li><a href="{{ index_payment_link }}">Payments</a></li>
		<li><a href="{{ index_rental_link }}">Rentals</a></li>
		<li><a href="{{ index_staff_link }}">Staff</a></li>
		<li><a href="{{ index_store_link }}">Stores</a></li>
	</ul>
{% endblock %}
```

##Actors Templates

**index.html**

>vim templates/actors/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>First Name</th>
				<th>Last Name</th>
				<th>Last Update</th>
			</tr>
		</thead>

		<tbody>
			{% for id, first, last, last_update in actors %}
				<tr>
					<td>{{ first }}</td>
					<td>{{ last }}</td>
					<td>{{ last_update }} UTC</td>
					<td><form action="{{ show_actor_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_actor_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_actor_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_actor_link }}">New Actor</a>
{% endblock %}
```

**create.html**

>vim templates/actors/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_actor_link }}" method="post">
		<p>
			First Name:<br>
			<input name="first_name">
		</p>
		<p>
			Last Name:<br>
			<input name="last_name">
		</p>
		<p>
			<input type="submit" value="Create Actor"/>
		</p>
	</form>
	<a href="{{ index_actor_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/actors/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, first, last, last_update in actor %}
		<p>
			<strong>First Name:</strong>
			{{ first }}
		</p>
		<p>
			<strong>Last Name</strong>
			{{ last }}
		</p>
		<p>
			<strong>Last update:</strong>
			{{ last_update }} UTC
		</p>

		<form action="{{ update_actor_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_actor_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/actors/update.html

```
{% extends "base.html" %}

{% block body %}

	{% for id, first_name, last_name, last_update in actor %}
		<form action="{{ update_actor_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				First name<br>
				<input name="first_name" value="{{ first_name }}">
			</p>
			<p>
				Last name<br>
				<input name="last_name" value="{{ last_name }}">
			</p>
			<p>
				<input type="submit" value="Update Actor">
			</p>
		</form>
		<form action="{{ show_actor_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_actor_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

##Addresses Templates

**index.html**

>vim templates/addresses/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>Address</th>
				<th>District</th>
				<th>City id</th>
				<th>Postal code</th>
				<th>Phone</th>
				<th>Last updated</th>
			</tr>
		</thead>

		<tbody>
			{% for id, address, district, city_id, postal_code, phone, last_updated in addresses %}
				<tr>
					<td>{{ address }}</td>
					<td>{{ district }}</td>
					<td>{{ city_id }}</td>
					<td>{{ postal_code }}</td>
					<td>{{ phone }}</td>
					<td>{{ last_updated }} UTC</td>
					<td><form action="{{ show_address_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_address_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_address_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_address_link }}">New Address</a>
{% endblock %}
```

**create.html**

>vim templates/addresses/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_address_link }}" method="post">
		<p>
				Address:<br>
				<input name="address">
		</p>
		<p>
			District:<br>
			<input name="district">
		</p>
		<p>
			City Id:<br>
			<input name="city_id">
		</p>
		<p>
			Postal Code:<br>
			<input name="postal_code">
		</p>
		<p>
			Phone:<br>
			<input name="phone">
		</p>
		<p>
			<input type="submit" value="Create Address">
		</p>
	</form>
	<a href="{{ index_address_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/addresses/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, address, district, city_id, postal_code, phone, last_updated in address %}

		<p>
			<strong>Address:</strong>
			{{ address }}
		</p>
		<p>
			<strong>District</strong>
			{{ district }}
		</p>
		<p>
			<strong>City id</strong>
			{{ city_id }}
		</p>
		<p>
			<strong>Postal code</strong>
			{{ postal_code }}
		</p>
		<p>
			<strong>Phone</strong>
			{{ phone }}
		</p>
		<p>
			<strong>Last update:</strong>
			{{ address.last_update }} UTC
		</p>
		
		<form action="{{ update_address_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_address_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/addresses/update.html

```
{% extends "base.html" %}

{% block body %}

	{% for id, address, district, city_id, postal_code, phone, last_updated in address %}
	<form action="{{ update_address_save_link }}" method="post">
			<input type="hidden" value="{{ id }}" name="id">
			<p>
				Address<br>
				<input name="address" value="{{ address }}">
			</p>
			<p>
				District<br>
				<input name="district" value="{{ district }}">
			</p>
			<p>
				City id<br>
				<input name="city_id" value="{{ city_id }}">
			</p>
			<p>
				Postal code<br>
				<input name="postal_code" value="{{ postal_code }}">
			</p>
			<p>
				Phone<br>
				<input name="phone" value="{{ phone }}">
			</p>
			<p>
				<input type="submit" value="Update Address">
			</p>
		</form>

		<form action="{{ show_address_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_address_link }}" method="POST"><button type="submit">Back</button></form>
	
	{% endfor %}		
{% endblock %}
```

##Categories Templates

**index.html**

>vim templates/categories/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>Name</th>
				<th>Last Update</th>
			</tr>
		</thead>

		<tbody>
			{% for id, name, last_update in categories %}
				<tr>
					<td>{{ name }}</td>
					<td>{{ last_update }} UTC</td>
					<td><form action="{{ show_category_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_category_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_category_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_category_link }}">New Category</a>
{% endblock %}
```

**create.html**

>vim templates/categories/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_category_link }}" method="post">
		<p>
			Name:<br>
			<input name="name">
		</p>
		<p>
			<input type="submit" value="Create Category"/>
		</p>
	</form>
	<a href="{{ index_category_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/categories/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, name, last_update in category %}
		<p>
			<strong>Name:</strong>
			{{ name }}
		</p>
		<p>
			<strong>Last update:</strong>
			{{ last_update }} UTC
		</p>

		<form action="{{ update_category_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_category_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/categories/update.html

```
{% extends "base.html" %}

{% block body %}

	{% for id, name, last_update in category %}
		<form action="{{ update_category_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				Name<br>
				<input name="name" value="{{ name }}">
			</p>
			<p>
				<input type="submit" value="Update Category">
			</p>
		</form>
		<form action="{{ show_category_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_category_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

##Cities Templates

**index.html**

>vim templates/cities/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>City</th>
				<th>Country Id</th>
				<th>Last Update</th>
			</tr>
		</thead>

		<tbody>
			{% for id, city, country_id, last_update in cities %}
				<tr>
					<td>{{ city }}</td>
					<td>{{ country_id }}</td>
					<td>{{ last_update }} UTC</td>
					<td><form action="{{ show_city_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_city_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_city_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_city_link }}">New City</a>
{% endblock %}
```

**create.html**

>vim templates/cities/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_city_link }}" method="post">
		<p>
			City:<br>
			<input name="city">
		</p>
		<p>
			Country Id:<br>
			<input name="country_id">
		</p>
		<p>
			<input type="submit" value="Create City"/>
		</p>
	</form>
	<a href="{{ index_city_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/cities/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, city, country_id, last_update in city %}
		<p>
			<strong>City:</strong>
			{{ city }}
		</p>
		<p>
			<strong>Country Id:</strong>
			{{ country_id }}
		</p>
		<p>
			<strong>Last update:</strong>
			{{ last_update }} UTC
		</p>

		<form action="{{ update_city_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_city_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/cities/update.html

```
{% extends "base.html" %}

{% block body %}

	{% for id, city, country_id, last_update in city %}
		<form action="{{ update_city_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				City<br>
				<input name="city" value="{{ city }}">
			</p>
			<p>
				Country Id<br>
				<input name="country_id" value="{{ country_id }}">
			</p>
			<p>
				<input type="submit" value="Update City">
			</p>
		</form>
		<form action="{{ show_city_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_city_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

##Countries Templates

**index.html**

>vim templates/countries/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>Country</th>
				<th>Last Update</th>
			</tr>
		</thead>

		<tbody>
			{% for id, country, last_update in countries %}
				<tr>
					<td>{{ country }}</td>
					<td>{{ last_update }} UTC</td>
					<td><form action="{{ show_country_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_country_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_country_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_country_link }}">New Country</a>
{% endblock %}
```

**create.html**

>vim templates/countries/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_country_link }}" method="post">
		<p>
			Country:<br>
			<input name="country">
		</p>
		<p>
			<input type="submit" value="Create Country">
		</p>
	</form>
	<a href="{{ index_country_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/countries/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, country, last_update in country %}
		<p>
			<strong>Country:</strong>
			{{ country }}
		</p>
		<p>
			<strong>Last update:</strong>
			{{ last_update }} UTC
		</p>

		<form action="{{ update_country_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_country_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/countries/update.html

```
{% extends "base.html" %}

{% block body %}

	{% for id, country, last_update in country %}
		<form action="{{ update_country_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				Country<br>
				<input name="country" value="{{ country }}">
			</p>
			<p>
				<input type="submit" value="Update Country">
			</p>
		</form>
		<form action="{{ show_country_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_country_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

##Customers Templates

**index.html**

>vim templates/customers/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>Store Id</th>
				<th>First Name</th>
				<th>Last Name</th>
				<th>Email</th>
				<th>Address Id</th>
				<th>Active</th>
				<th>Create Date</th>
				<th>Last Update</th>
			</tr>
		</thead>

		<tbody>
			{% for id, store_id, first_name, last_name, email, address_id, active, create_date, last_update in customers %}
				<tr>
					<td>{{ store_id }}</td>
					<td>{{ first_name }}</td>
					<td>{{ last_name }}</td>
					<td>{{ email }}</td>
					<td>{{ address_id }}</td>
					<td>{{ active }}</td>
					<td>{{ create_date }} UTC</td>
					<td>{{ last_update }} UTC</td>
					<td><form action="{{ show_customer_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_customer_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_customer_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_customer_link }}">New Customer</a>
{% endblock %}
```

**create.html**

>vim templates/customers/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_customer_link }}" method="post">
		<p>
			Store id:<br>
			<input name="store_id">
		</p>
		<p>
			First name:<br>
			<input name="first_name">
		</p>
		<p>
			Last name:<br>
			<input name="last_name">
		</p>
		<p>
			Email:<br>
			<input name="email">
		</p>
		<p>
			Address id:<br>
			<input name="address_id">
		</p>
		<p>
			Active:<br>
			<input type="checkbox" name="active">
		</p>
		<p>
			<input type="submit" value="Create Customer"/>
		</p>

	</form>
	<a href="{{ index_customer_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/customers/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, store_id, first_name, last_name, email, address_id, active, create_date, last_update in customer %}
		<p>
			<strong>Store id</strong>
			{{ store_id }}
		</p>
		<p>
			<strong>First name</strong>
			{{ first_name }}
		</p>
		<p>
			<strong>Last name</strong>
			{{ last_name }}
		</p>
		<p>
			<strong>Email</strong>
			{{ email }}
		</p>
		<p>
			<strong>Address id</strong>
			{{ address_id }}
		</p>
		<p>
			<strong>Active</strong>
			{{ active }}
		</p>
		<p>
			<strong>Creation date</strong>
			{{ create_date }} UTC
		</p>
		<p>
			<strong>Last update</strong>
			{{ last_update }} UTC
		</p>
		<form action="{{ update_customer_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_customer_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/customers/update.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, store_id, first_name, last_name, email, address_id, active, create_date, last_update in customer %}
		<form action="{{ update_customer_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				Store id<br>
				<input name="store_id" value="{{ store_id }}"/>
			</p>
			<p>
				First name<br>
				<input name="first_name" value="{{ first_name }}"/>
			</p>
			<p>
				Last name<br>
				<input name="last_name" value="{{ last_name }}"/>
			</p>
			<p>
				Email<br>
				<input name="email" value="{{ email }}"/>
			</p>
			<p>
				Address id<br>
				<input name="address_id" value="{{ address_id }}"/>
			</p>
			<p>
				Active<br>
				<input name="active" value="{{ active }}"/>
			</p>
			<p>
				<input type="submit" value="Update Customer"/>
			</p>
																																																																				
		</form>
		<form action="{{ show_customer_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_customer_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

##Films Templates

**index.html**

>vim templates/films/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>Title</th>
				<th>Description</th>
				<th>Release year</th>
				<th>Language id</th>
				<th>Rental duration</th>
				<th>Rental rate</th>
				<th>Length</th>
				<th>Replacement cost</th>
				<th>Rating</th>
				<th>Special features</th>
				<th>Last updated</th>
			</tr>
		</thead>

		<tbody>
			{% for id, title, description, release_year, language_id, rental_duration, rental_rate, length, replacement_cost, rating, special_features, last_update in films %}
				<tr>
					<td>{{ title }}</td>
					<td>{{ description }}</td>
					<td>{{ release_year }}</td>
					<td>{{ language_id }}</td>
					<td>{{ rental_duration }}</td>
					<td>{{ rental_rate }}</td>
					<td>{{ length }}</td>
					<td>{{ replacement_cost }}</td>
					<td>{{ rating }}</td>
					<td>{{ special_features }}</td>
					<td>{{ last_update }} UTC</td>
					<td><form action="{{ show_film_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_film_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_film_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_film_link }}">New Film</a>
{% endblock %}
```

**create.html**

>vim templates/films/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_film_link }}" method="post">
		<p>
			Title:<br>
			<input name="title">
		</p>
		<p>
			Description:<br>
			<input name="description">
		</p>
		<p>
			Release year:<br>
			<input name="release_year">
		</p>
		<p>
			Language id:<br>
			<input name="language_id">
		</p>
		<p>
			Rental duration:<br>
			<input name="rental_duration">
		</p>
		<p>
			Rental rate:<br>
			<input name="rental_rate">
		</p>
		<p>
			Length:<br>
			<input name="length">
		</p>
		<p>
			Replacement cost:<br>
			<input name="replacement_cost">
		</p>
		<p>
			Rating:<br>
			<input name="rating">
		</p>
		<p>
			Special features:<br>
			<input name="special_features">
		</p>
		<p>
			<input type="submit" value="Create Film"/>
		</p>
	</form>
	<a href="{{ index_film_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/films/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, title, description, release_year, language_id, rental_duration, rental_rate, length, replacement_cost, rating, special_features, last_update in film %}	
		<p>
			<strong>Title</strong>
			{{ title }}
		</p>
		<p>
			<strong>Description</strong>
			{{ description }}
		</p>
		<p>
			<strong>Release year</strong>
			{{ release_year }}
		</p>
		<p>
			<strong>Language id</strong>
			{{ language_id }}
		</p>
		<p>
			<strong>Rental duration</strong>
			{{ rental_duration }}
		</p>
		<p>
			<strong>Rental rate</strong>
			{{ rental_rate }}
		</p>
		<p>
			<strong>Length</strong>
			{{ length }}
		</p>
		<p>
			<strong>Replacement cost</strong>
			{{ replacement_cost }}
		</p>
		<p>
			<strong>Rating</strong>
			{{ rating }}
		</p>
		<p>
			<strong>Special features</strong>
			{{ special_features }}
		</p>
		<p>
			<strong>Last update:</strong>
			{{ last_update }} UTC
		</p>

		<form action="{{ update_film_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_film_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/films/update.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, title, description, release_year, language_id, rental_duration, rental_rate, length, replacement_cost, rating, special_features, last_update in film %}
		<form action="{{ update_film_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				Title<br>
				<input name="title" value="{{ title }}">
			</p>
			<p>
				Description<br>
				<input name="description" value="{{ description }}">
			</p>
			<p>
				Release year<br>
				<input name="release_year" value="{{ release_year }}">
			</p>
			<p>
				Language id<br>
				<input name="language_id" value="{{ language_id }}">
			</p>
			<p>
				Rental duration<br>
				<input name="rental_duration" value="{{ rental_duration }}">
			</p>
			<p>
				Rental rate<br>
				<input name="rental_rate" value="{{ rental_rate }}">
			</p>
			<p>
				Length<br>
				<input name="length" value="{{ length }}">
			</p>
			<p>
				Replacement cost<br>
				<input name="replacement_cost" value="{{ replacement_cost }}">
			</p>
			<p>
				Rating<br>
				<input name="rating" value="{{ rating }}">
			</p>
			<p>
				Special features<br>
				<input name="special_features" value="{{ special_features }}">
			</p>
			<p>
				<input type="submit" value="Update Film">
			</p>
		</form>
		<form action="{{ show_film_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_film_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

##Filmtexts Templates

**index.html**

>vim templates/filmtexts/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>Title</th>
				<th>Description</th>
			</tr>
		</thead>

		<tbody>
			{% for id, title, description in filmtexts %}
				<tr>
					<td>{{ title }}</td>
					<td>{{ description }}</td>
					<td><form action="{{ show_filmtext_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_filmtext_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_filmtext_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_filmtext_link }}">New Film Text</a>
{% endblock %}
```

**create.html**

>vim templates/filmtexts/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_filmtext_link }}" method="post">
		<p>
			Title:<br>
			<input name="title">
		</p>
		<p>
			Description:<br>
			<input name="description">
		</p>
		<p>
			<input type="submit" value="Create Film Text"/>
		</p>
	</form>
	<a href="{{ index_filmtext_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/filmtexts/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, title, description in filmtext %}
		<p>
			<strong>Title:</strong>
			{{ title }}
		</p>
		<p>
			<strong>Description:</strong>
			{{ description }}
		</p>

		<form action="{{ update_filmtext_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_filmtext_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/filmtexts/update.html

```
{% extends "base.html" %}

{% block body %}

	{% for id, title, description in filmtext %}
		<form action="{{ update_filmtext_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				Title<br>
				<input name="title" value="{{ title }}">
			</p>
			<p>
				Description<br>
				<input name="description" value="{{ description }}">
			</p>
			<p>
				<input type="submit" value="Update Film Text">
			</p>
		</form>
		<form action="{{ show_filmtext_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_filmtext_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

##Inventories Templates

**index.html**

>vim templates/inventories/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>Film Id</th>
				<th>Store Id</th>
				<th>Last Update</th>
			</tr>
		</thead>

		<tbody>
			{% for id, film_id, store_id, last_update in inventories %}
				<tr>
					<td>{{ film_id }}</td>
					<td>{{ store_id }}</td>
					<td>{{ last_update }} UTC</td>
					<td><form action="{{ show_inventory_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_inventory_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_inventory_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_inventory_link }}">New Inventory</a>
{% endblock %}
```

**create.html**

>vim templates/inventories/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_inventory_link }}" method="post">
		<p>
			Film Id:<br>
			<input name="film_id">
		</p>
		<p>
			Store Id:<br>
			<input name="store_id">
		</p>
		<p>
			<input type="submit" value="Create Inventory"/>
		</p>
	</form>
	<a href="{{ index_inventory_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/inventories/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, film_id, store_id, last_update in inventory %}
		<p>
			<strong>Film Id:</strong>
			{{ film_id }}
		</p>
		<p>
			<strong>Store Id:</strong>
			{{ store_id }}
		</p>
		<p>
			<strong>Last update:</strong>
			{{ last_update }} UTC
		</p>

		<form action="{{ update_inventory_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_inventory_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/inventories/update.html

```
{% extends "base.html" %}

{% block body %}

	{% for id, film_id, store_id, last_update in inventory %}
		<form action="{{ update_inventory_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				Film Id<br>
				<input name="film_id" value="{{ film_id }}">
			</p>
			<p>
				Store Id<br>
				<input name="store_id" value="{{ store_id }}">
			</p>
			<p>
				<input type="submit" value="Update Inventory">
			</p>
		</form>
		<form action="{{ show_inventory_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_inventory_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

##Languages Templates

**index.html**

>vim templates/languages/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>Name</th>
				<th>Last Update</th>
			</tr>
		</thead>

		<tbody>
			{% for id, name, last_update in languages %}
				<tr>
					<td>{{ name }}</td>
					<td>{{ last_update }} UTC</td>
					<td><form action="{{ show_language_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_language_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_language_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_language_link }}">New Language</a>
{% endblock %}
```

**create.html**

>vim templates/languages/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_language_link }}" method="post">
		<p>
			Name:<br>
			<input name="name">
		</p>
		<p>
			<input type="submit" value="Create Language">
		</p>
	</form>
	<a href="{{ index_language_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/languages/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, name, last_update in language %}
		<p>
			<strong>Name:</strong>
			{{ name }}
		</p>
		<p>
			<strong>Last update:</strong>
			{{ last_update }} UTC
		</p>

		<form action="{{ update_language_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_language_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/languages/update.html

```
{% extends "base.html" %}

{% block body %}

	{% for id, name, last_update in language %}
		<form action="{{ update_language_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				Name<br>
				<input name="name" value="{{ name }}">
			</p>
			<p>
				<input type="submit" value="Update Language">
			</p>
		</form>
		<form action="{{ show_language_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_language_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

##Payments Templates

**index.html**

>vim templates/payments/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>Customer Id</th>
				<th>Staff Id</th>
				<th>Rental Id</th>
				<th>Amount</th>
				<th>Payment Date</th>
				<th>Last Update</th>
			</tr>
		</thead>

		<tbody>
			{% for id, customer_id, staff_id, rental_id, amount, payment_date, last_update in payments %}
				<tr>
					<td>{{ customer_id }}</td>
					<td>{{ staff_id }}</td>
					<td>{{ rental_id }}</td>
					<td>{{ amount }}</td>
					<td>{{ payment_date }} UTC</td>
					<td>{{ last_update }} UTC</td>
					<td><form action="{{ show_payment_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_payment_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_payment_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_payment_link }}">New Payment</a>
{% endblock %}
```

**create.html**

>vim templates/payments/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_payment_link }}" method="post">
		<p>
			Customer id:<br>
			<input name="customer_id">
		</p>
		<p>
			Staff id:<br>
			<input name="staff_id">
		</p>
		<p>
			Rental id:<br>
			<input name="rental_id">
		</p>
		<p>
			Amount:<br>
			<input name="amount">
		</p>
		<p>
			<input type="submit" value="Create Payment"/>
		</p>
	</form>
	<a href="{{ index_payment_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/payments/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, customer_id, staff_id, rental_id, amount, payment_date, last_update in payment %}
		<p>
			<strong>Customer id</strong>
			{{ customer_id }}
		</p>
		<p>
			<strong>Staff id</strong>
			{{ staff_id }}
		</p>
		<p>
			<strong>Rental id</strong>
			{{ rental_id }}
		</p>
		<p>
			<strong>Amount</strong>
			{{ amount }}
		</p>
		<p>
			<strong>Last update:</strong>
			{{ payment_date }} UTC
		</p>
		<p>
			<strong>Last update:</strong>
			{{ last_update }} UTC
		</p>

		<form action="{{ update_payment_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_payment_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/payments/update.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, customer_id, staff_id, rental_id, amount, payment_date, last_update in payment %}
		<form action="{{ update_payment_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				Customer id<br>
				<input name="customer_id" value="{{ customer_id }}">
			</p>
			<p>
				Staff id<br>
				<input name="staff_id" value="{{ staff_id }}">
			</p>
			<p>
				Rental id<br>
				<input name="rental_id" value="{{ rental_id }}">
			</p>
			<p>
				Amount<br>
				<input name="amount" value="{{ amount }}">
			</p>
			<p>
				<input type="submit" value="Update Payment">
			</p>
		</form>
		<form action="{{ show_payment_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_payment_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

##Rentals Templates

**index.html**

>vim templates/rentals/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>Rental date</th>
				<th>Inventory id</th>
				<th>Customer id</th>
				<th>Return date</th>
				<th>Staff id</th>
				<th>Last updated</th>
			</tr>
		</thead>

		<tbody>
			{% for id, rental_date, inventory_id, customer_id, return_date, staff_id, last_update in rentals %}
				<tr>
					<td>{{ rental_date }} UTC</td>
					<td>{{ inventory_id }}</td>
					<td>{{ customer_id }}</td>
					<td>{{ return_date }} UTC</td>
					<td>{{ staff_id }}</td>
					<td>{{ last_update }} UTC</td>
					<td><form action="{{ show_rental_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_rental_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_rental_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_rental_link }}">New Rental</a>
{% endblock %}
```

**create.html**

>vim templates/rentals/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_rental_link }}" method="post">
		<p>
			Inventory id:<br>
			<input name="inventory_id">
		</p>
		<p>
			Customer id:<br>
			<input name="customer_id">
		</p>
		<p>
			Staff Id:<br>
			<input name="staff_id">
		</p>
		<p>
			<input type="submit" value="Create Rental"/>
		</p>

	</form>
	<a href="{{ index_rental_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/rentals/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, rental_date, inventory_id, customer_id, return_date, staff_id, last_update in rental %}
		<p>
			<strong>Rental date</strong>
			{{ rental_date }} UTC
		</p>
		<p>
			<strong>Inventory id</strong>
			{{ inventory_id }}
		</p>
		<p>
			<strong>Customer id</strong>
			{{ customer_id }}
		</p>
		<p>
			<strong>Return date</strong>
			{{ return_date }} UTC
		</p>
		<p>
			<strong>Staff id</strong>
			{{ staff_id }}
		</p>
		<p>
			<strong>Last update:</strong>
			{{ last_update }} UTC
		</p>

		<form action="{{ update_rental_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_rental_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/rentals/update.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, rental_date, inventory_id, customer_id, return_date, staff_id, last_update in rental %}
		<form action="{{ update_rental_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				Inventory id<br>
				<input name="inventory_id" value="{{ inventory_id }}">
			</p>
			<p>
				Customer id<br>
				<input name="customer_id" value="{{ customer_id }}">
			</p>
			<p>
				Staff id<br>
				<input name="staff_id" value="{{ staff_id }}">
			</p>
			<p>
				<input type="submit" value="Update Rental">
			</p>
		</form>
		<form action="{{ show_rental_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_rental_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

##Staffs Templates

**index.html**

>vim templates/staffs/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>First Name</th>
				<th>Last Name</th>
				<th>Address id</th>
				<th>Email</th>
				<th>Store id</th>
				<th>Active</th>
				<th>Username</th>
				<th>Password</th>
				<th>Last updated</th>
			</tr>
		</thead>

		<tbody>
			{% for id, first_name, last_name, address_id, email, store_id, active, username, password, last_update in staffs %}
				<tr>
					<td>{{ first_name }}</td>
					<td>{{ last_name }}</td>
					<td>{{ address_id }}</td>
					<td>{{ email }}</td>
					<td>{{ store_id }}</td>
					<td>{{ active }}</td>
					<td>{{ username }}</td>
					<td>{{ password }}</td>
					<td>{{ last_update }} UTC</td>
					<td><form action="{{ show_staff_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_staff_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_staff_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_staff_link }}">New Staff</a>
{% endblock %}
```

**create.html**

>vim templates/staffs/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_staff_link }}" method="post">
		<p>
			First Name:<br>
			<input name="first_name">
		</p>
		<p>
			Last Name:<br>
			<input name="last_name">
		</p>
		<p>
			Address id:<br>
			<input name="address_id">
		</p>
		<p>
			Email:<br>
			<input name="email">
		</p>
		<p>
			Store id:<br>
			<input name="store_id">
		</p>
		<p>
			Active:<br>
			<input type="checkbox" name="active">
		</p>
		<p>
			Username:<br>
			<input name="username">
		</p>
		<p>
			Password:<br>
			<input name="password">
		</p>
		<p>
			<input type="submit" value="Create Staff">
		</p>

	</form>
	<a href="{{ index_staff_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/staffs/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, first_name, last_name, address_id, email, store_id, active, username, password, last_update in staff %}
		<p>
			<strong>First Name:</strong>
			{{ first_name }}
		</p>
		<p>
			<strong>Last Name:</strong>
			{{ last_name }}
		</p>
		<p>
			<strong>Address id:</strong>
			{{ address_id }}
		</p>
		<p>
			<strong>Email:</strong>
			{{ email }}
		</p>
		<p>
			<strong>Store id:</strong>
			{{ store_id }}
		</p>
		<p>
			<strong>Active:</strong>
			{{ active }}
		</p>
		<p>
			<strong>Username:</strong>
			{{ username }}
		</p>
		<p>
			<strong>Password:</strong>
			{{ password }}
		</p>
		<p>
			<strong>Last update:</strong>
			{{ last_update }} UTC
		</p>
		<form action="{{ update_staff_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_staff_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/staffs/update.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, first_name, last_name, address_id, email, store_id, active, username, password, last_update in staff %}
		<form action="{{ update_staff_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				First name<br>
				<input name="first_name" value="{{ first_name }}">
			</p>
			<p>
				Last name<br>
				<input name="last_name" value="{{ last_name }}">
			</p>
			<p>
				Address id<br>
				<input name="address_id" value="{{ address_id }}">
			</p>
			<p>
				Email<br>
				<input name="email" value="{{ email }}">
			</p>
			<p>
				Store id<br>
				<input name="store_id" value="{{ store_id }}">
			</p>
			<p>
				Active<br>
				<input type="checkbox" name="active" value="{{ active }}">
			</p>
			<p>
				Username<br>
				<input name="username" value="{{ username }}">
			</p>
			<p>
				Password<br>
				<input name="password" value="{{ password }}">
			</p>
			<p>
				<input type="submit" value="Update Staff">
			</p>
		</form>
		<form action="{{ show_staff_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_staff_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

##Stores Templates

**index.html**

>vim templates/stores/index.html

```
{% extends "base.html" %}

{% block body %}
	<table>
		<thead>
			<tr>
				<th>Address Id</th>
				<th>Last Update</th>
			</tr>
		</thead>

		<tbody>
			{% for id, address_id, last_update in stores %}
				<tr>
					<td>{{ address_id }}</td>
					<td>{{ last_update }} UTC</td>
					<td><form action="{{ show_store_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Show"></form></td>
					<td><form action="{{ update_store_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Edit"></form></td>
					<td><form action="{{ delete_store_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><input type="submit" value="Destroy"></form></td>
				</tr>
			{% endfor %}
		</tbody>
	</table>
	<br>
	<a href="{{ create_store_link }}">New Store</a>
{% endblock %}
```

**create.html**

>vim templates/stores/create.html

```
{% extends "base.html" %}

{% block body %}

	{% with messages = get_flashed_messages() %}
		{% if messages %}
			<ul>
				{% for message in messages %}
					<li>{{ message }}
				{% endfor %}
			</ul>
		{% endif %}
	{% endwith %}

	<form action="{{ save_store_link }}" method="post">
		<p>
			Address Id:<br>
			<input name="address_id">
		</p>
		<p>
			<input type="submit" value="Create Store">
		</p>
	</form>
	<a href="{{ index_store_link }}">Back</a>
{% endblock %}
```

**show.html**

>vim templates/stores/show.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, address_id, last_update in store %}
		<p>
			<strong>Address Id:</strong>
			{{ address_id }}
		</p>
		<p>
			<strong>Last update:</strong>
			{{ last_update }} UTC
		</p>

		<form action="{{ update_store_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Edit</button></form>
		<form action="{{ index_store_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

**update.html**

>vim templates/stores/update.html

```
{% extends "base.html" %}

{% block body %}
	{% for id, address_id, last_update in store %}
		<form action="{{ update_store_save_link }}" method="post">
			<input type="hidden" name="id" value="{{ id }}">
			<p>
				Address Id<br>
				<input name="address_id" value="{{ address_id }}">
			</p>
			<p>
				<input type="submit" value="Update Store">
			</p>
		</form>
		<form action="{{ show_store_link }}" method="POST"><input type="hidden" name="id" value="{{ id }}"><button type="submit">Show</button></form>
		<form action="{{ index_store_link }}" method="POST"><button type="submit">Back</button></form>
	{% endfor %}		
{% endblock %}
```

=
###Getting production ready

Remove debug = True from the app.run() line at the end of the __init__.py file.
If you were to host this on a platform you would also need to move the if __name__ == "__main__" statement to below the database settings but still above the app.run() line. This means that the app can still get the database settings if it is not run locally.

=
###The End

That's all there is to it.

Thanks for reading and hopefully you learned something. :)

Darren.


