# 样例

以销售为例子的数据库表以及数据内容

## 数据库

sales

create database sales;

use sales;

## 表

1. Customer

客户信息表

| id | name | mail | birth | phone | city | join_time |
| - | - | - | - | - | - | - | - |

CREATE TABLE Customer (
    id int NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name varchar(256) NOT NULL,
    mail varchar(256) NOT NULL,
    birth DATE,
    phone varchar(256),
    city varchar(256),
    join_time DATETIME
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

`NOTE:` engine和charset可以不加，默认引擎是InnoDB，字符集utf8可以使用中文，兼容性也较好。

2. Product

商品信息表

| id | name | qgp | category |
| - | - | - | - |

CREATE TABLE Product (
    id int NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name varchar(256) NOT NULL,
    qgp int NOT NULL,
    category varchar(256)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

`NOTE:` qgp是保质期，单位为天

3. Stock

商品表

| id | price | create_time | pid | import_time | import_order_id | export_order_id |
| - | - | - | - | - | - | - | 

CREATE TABLE Stock (
    id int NOT NULL AUTO_INCREMENT PRIMARY KEY,
    price int NOT NULL,
    create_time DateTime,
    pid int,
    import_time DateTime,
    import_order_id int,
    export_order_id int,
    FOREIGN KEY ("pid") REFERENCES Product ("id")
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

`NOTE:` pid是Product的主键, import_order_id是Import order主键, export_order_id是export order的主键

4. Import order

进货单表

| id | create_time |

5. Export order

6. Discount

| id | discount |

出货单表

| id | create_time |


## 角色

1. boss

create user boss@localhost identified by "boss";

有所有权限，不会直接写Sql，但会使用别人提供的Sql(存储过程，视图等)

2. employee

 create user employeeForExporting"@"localhost identified by "exporting";

会使用和进出货相关的Sql，用Sql来负责出货，有Product，Custom读权限，Stock，Order，Export order，Import order读写权限。

4. manager

create user manager@localhost identified by "manager";

对销售和进货情况进行分析，有所有表读权限。

对用户信息进行录入。有Customer, Product的写权限。

给boss提供视图和存储过程。

## 例子

