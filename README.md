# Net Market (EN)

## Development Setup

### Prerequisites 
- Github account
- Docker for Mac(https://docs.docker.com/docker-for-mac/install/) or Docker for Windows(https://www.docker.com/docker-windows)　

### Clone repository

```console
$ cd
$ mkdir projects
$ cd projects
$ git clone https://github.com/THitokuse/net-market.git
```

### Creating a Database
```console
// build docker image
$ docker-compose build

// create and migrate db
$ docker-compose run web bundle exec rake db:create
$ docker-compose run web bundle exec rake db:migrate
$ docker-compose run web bundle exec rake db:seed
```

### Introducing reCAPTCHA

It is necessary to install reCAPTCHA in each environment because it is used for new user registration and login function in the development environment. 
https://www.google.com/recaptcha/intro/v3.html

- Create reCAPTCHA above
- Add SITE_KEY and PRIVATE_KEY generated when creating to shell (ex: ~ /. Bash_profile) 
- 
```
export RECAPTCHA_SITE_KEY="xxxxxxxxx"
export RECAPTCHA_PRIVATE_KEY="xxxxxxxx"
```

### Launching local server
```console
$ docker-compose up
```
アクセス　http://localhost:3000

### Stop the development server
```console
$ docker-compose down
```

### bundle install
```console
$ docker-compose run web bundle install
```

### Login to Docker bash
```console
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
4e5622b8f25f        mercari_web         "bundle exec rails s…"   43 hours ago        Up 23 seconds       0.0.0.0:3000->3000/tcp   mercari_web_1
d47e7254918d        mysql:5.6           "docker-entrypoint.s…"   43 hours ago        Up 24 seconds       0.0.0.0:3306->3306/tcp   mercari_db_1
$ docker exec -it mercari_web_1 /bin/bash
```
Here you can perform operations such as debugging. 

### Logging in to net market db
```console
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
4e5622b8f25f        mercari_web         "bundle exec rails s…"   43 hours ago        Up 23 seconds       0.0.0.0:3000->3000/tcp   mercari_web_1
d47e7254918d        mysql:5.6           "docker-entrypoint.s…"   43 hours ago        Up 24 seconds       0.0.0.0:3306->3306/tcp   mercari_db_1
$ docker exec -it mercari_db_1 /bin/bash
$ mysql -u root -p
Enter password: mercari_umeda
mysql> show databases;
+---------------------+
| Database            |
+---------------------+
| information_schema  |
| mercari_development |
| mercari_test        |
| mysql               |
| performance_schema  |
+---------------------+
```

It is possible to operate the DB of net-market here.

You can also browse with Sequel pro (docker-compose up)
Set the following in sequel pro. 
```
Select Standards

Name: mercari_localhost
Localhost: 127.0.0.1
Username: root
Password: mercari_umeda
Port: 4306 
```
Connect with.

### Run RSpec in docker environment.
```
1. docker-compose run web bundle exec rake db: migrate RAILS_ENV = test
→ You will migrate in the test environment
2. docker-compose run web bundle exec rspec spec / controllers / ●●●● _controller_spec.rb
→ Execute the specified test file 
```

### Add Japanese support to MYSQL on docker 

Normally, it is latin compatible, so even if you enter Japanese on the table, it will not be displayed.
Therefore, you can set it to Japanese by writing it in /etc/mysql/my.cnf and overwriting it.

I want to edit my.cnf on the terminal, but vim is not included, so I will include vim. 
```
RUN ["apt-get", "update"]
RUN ["apt-get", "install", "-y", "vim"]
```
After Build, execute the following code 
```
$ docker exec -it mercari_db_1 /bin/bash
# apt-get update
# apt-get install vim
# vim etc/mysql/my.cnf
```
Write and save the following code in my.cnf
```
[mysqld]
character-set-server=utf8
collation-server=utf8_general_ci

[client]
default-character-set=utf8
```
Build again and check MYSQL

### Docker command memo

```
# Check the Ruby version
$ docker-compose run --rm web ruby -v

# Check Rails version
$ docker-compose run --rm workspace php artisan -v

# Rail routes
$ docker-compose run --rm web rails routes
```

## Net market DB design

### Diagram
https://www.draw.io/#G1OzJugJEFpL-U19UEEycdfbTGCAxYNAYY

![Net market ER diagram ](https://user-images.githubusercontent.com/45042275/55183070-5f939400-51d2-11e9-9950-6fe0605e4792.png)

## Users table
--Table for net market users

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|nickname|varchar(255)|null: false|
|first_name|varchar(255)|null: false|
|last_name|varchar(255)|null: false|
|first_name_kana|varchar(255)|null: false|
|last_name_kana|varchar(255)|null: false|
|email|varchar(255)|null: false|
|tel|varchar(32)|null: false|
|password|varchar(255)|null: false|
|prefecture_code|integer(11)|null: false|
|zip|varchar(16)|null: false|
|city_name|varchar(255)|null: false|
|street|varchar(255)|-------|
|birth_day|integer(8)|null: false|
|birth_month|integer(8)|null: false|
|birth_year|integer(8)|null: false|
|self_introduce|text|-------|

- prefecture_code
jp_prefecture導入 / jp_prefecture(import)

### Association
- has_many items
- has_many likes
- has_many evalutes
- has_many todos
- has_many notice


## Items table
--Product listing table 

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|name|varchar(255)|null: false|
|content|text|-------|
|price|int(16)|null: false|
|prefecture_code|integer(11)|null: false|
|status|integer(2)|null:false, default: 0|
|deliverymethod|reference|null: false, foreign_key: true|
|deliveryburden|reference|null: false, foreign_key: true|
|deliverydate|reference|null: false, foreign_key: true|
|brand|reference|null: false, foreign_key: true|
|upper_category|reference|null: false, foreign_key: true|
|middle_category|reference|null: false, foreign_key: true|
|lower_category|reference|null: false, foreign_key: true|
|size|reference|null: false, foreign_key: true|
|seller|reference|null: false, foreign_key: true|

- prefecture_code
jp_prefecture導入 / jp_prefecture(import)

### enum
--Product status enum
--Status: {Selling: 1, Trading: 2, Sold: 3} 


### Association
- has_many salers
- has_many likes
- has_many comments
- has_many status
- has_many item_images
- belongs_to brand
- belongs_to delivery_date
- belongs_to delivery_method
- belongs_to delivery_burden
- belongs_to upper_category
- belongs_to middle_category
- belongs_to lower_category
- belongs_to size
- belongs_to user

## Item_images table
--Product image table 

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|item|reference|null: false, foreign_key: true|
|name|varchar(255)|null: false|
|image|varchar(255)|null: false|

### Association
- belongs_to item


## Salers table
--Buyer intermediate table 

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|user|reference|null: false, foreign_key: true|
|item|reference|null: false, foreign_key: true|

### Association
- belongs_to user
- belongs_to item

## Comments table
--Comment table for items on sale 

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|comment|text|null: false|
|item|reference|null: false, foreign_key: true|
|user|reference|null: false, foreign_key: true|

### Association
- belongs_to user
- belongs_to item
- 
## Evalutes table
--User evaluation table 

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|user|reference|null: false, foreign_key: true|
|evalute_type|reference|null: false, foreign_key: true|

### Association
- belongs_to user
- belongs_to evalute_types


## Evalute_types table
--Evaluation type master table
1. Good
2. Normal
3. Bad 

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|name|varchar(255)|null: false|

### Association
- has_many evalutes


## Todos table
--To-do list table 

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|user|reference|null: false, foreign_key: true|
|to_do|varchar(255)|null: false|

### Association
- belongs_to user


## Brands table
--Brand table 

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|name|varchar(255)|null: false|

### Association
- has_many items

## Upper_categories table
--Category table 

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|name|varchar(255)|null: false|

### Association
- has_many items
- has_many middle_categories

## Middle_categories table
--Category table 

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|name|varchar(255)|null: false|
|upper_category|reference|null: false, foreign_key: true|
|size_type|reference|null: false, foreign_key: true|

### Association
- has_many items
- has_many lower_categories
- belongs_to upper_category
- belongs_to size_type

## Lower_categories table
--Category table

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|name|varchar(255)|null: false|
|middle_category|reference|null: false, foreign_key: true|

### Association
- has_many items
- belongs_to middle_category

## Size_type table
--Size type table

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|size_type|varchar(255)|null: false|

### Association
- has_many sizes
- has_many middle_categories

## Size table
--Size (unit) table

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|name|varchar(255)|null: false|
|size_type|reference|null: false, foreign_key: true|

### Association
- has_many items
- belongs_to size_type

## Like table
--Like table

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|item|reference|null: false, foreign_key: true|
|user|reference|null: false, foreign_key: true|

### Association
- belongs_to item
- belongs_to user

## Delivery_method table
--Delivery method master table

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|name|varchar(255)|null: false|

### Association
- has_many items

## Delivery_burden table
--Shipping fee burden master table

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|name|varchar(255)|null: false|

### Association
- has_many items


## Delivery_date table
-Estimated shipping date Master table

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|name|varchar(255)|null: false|

### Association
- has_many items

## News table
--News table for all Mercari

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|news|text|null: false|


## Notice table
--News for storing individual news

|Column|Type|Options|
|------|----|-------|
|id|integer(11)|AI, PRIMARY_KEY|
|notice|text|null: false|
|user|reference|null: false, foreign_key: true|

### Association
- belongs_to user
