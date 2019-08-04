# 样例

以销售为例子的数据库表以及数据内容

## 数据库

```

# 数据库名sales

create database sales;

use sales;

```

## 表

1. Customer

客户信息表

| id | name | mail | birth | phone | city | join_time |
| - | - | - | - | - | - | - | - |

```

CREATE TABLE Customer (
    id int NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name varchar(256) NOT NULL,
    mail varchar(256) NOT NULL,
    birth DATE,
    phone varchar(256),
    city varchar(256),
    join_time DATETIME
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

`NOTE:` engine和charset可以不加，默认引擎是InnoDB，字符集utf8可以使用中文，兼容性也较好。

2. Product

商品信息表

| id | name | qgp | category |
| - | - | - | - |

```

CREATE TABLE Product (
    id int NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name varchar(256) NOT NULL,
    qgp int NOT NULL,
    category varchar(256)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

`NOTE:` qgp是保质期，单位为天

3. Stock

商品表

| id | price | create_time | pid | import_time | import_order_id | export_order_id |
| - | - | - | - | - | - | - | 

```

CREATE TABLE Stock (
    id int NOT NULL AUTO_INCREMENT PRIMARY KEY,
    price int NOT NULL,
    create_time DateTime,
    pid int,
    import_order_id int,
    export_order_id int,
    FOREIGN KEY (pid) REFERENCES Product (id),
    FOREIGN KEY (import_order_id) REFERENCES ImportOrder (id),
    FOREIGN KEY (export_order_id) REFERENCES ExportOrder (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

`NOTE:` pid是Product的主键, import_order_id是Import order主键, export_order_id是export order的主键

4. Import order

进货单表

| id | create_time | cid |

```

CREATE TABLE ImportOrder (
    id int NOT NULL AUTO_INCREMENT  PRIMARY KEY,
    create_time DateTime,
    cid int,
    FOREIGN KEY (cid) REFERENCES Customer (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

5. Export order

```

CREATE TABLE ExportOrder (
    id int NOT NULL AUTO_INCREMENT  PRIMARY KEY,
    create_time DateTime,
    cid int,
    FOREIGN KEY (cid) REFERENCES Customer (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8; 

```

出货单表

| id | create_time | cid |


## 角色

1. boss

```

CREATE USER boss@localhost IDENTIFIED BY "boss";

GRANT All PRIVILEGES ON Sales.* TO boss@localhost;

```

有所有权限，不会直接写Sql，但会使用别人提供的Sql(存储过程，视图等)

2. employee

```

CREATE USER employeeForExporting"@"localhost IDENTIFIED BY "exporting";

GRANT SELECT ON Sales.Product TO employee@localhost;

GRANT SELECT ON Sales.Customer TO employee@localhost;

GRANT SELECT, INSERT, DELETE, UPDATE ON Sales.Stock TO employee@localhost;

GRANT SELECT, INSERT, DELETE, UPDATE ON Sales.ExportOrder TO employee@localhost;

GRANT SELECT, INSERT, DELETE, UPDATE ON Sales.ImportOrder TO employee@localhost;

```

会使用和进出货相关的Sql，用Sql来负责出货，有Product，Customer读权限，Stock,Export order，Import order读写权限。

4. manager

```

CREATE USER manager@localhost IDENTIFIED BY "manager";

GRANT SELECT ON Sales.* TO manager@localhost;

GRANT INSERT, DELETE ON Sales.Customer TO manager@localhost;

GRANT Create ROUTINE, CREATE view ON Sales.* TO manager@localhost;

```

对销售和进货情况进行分析，有所有表读权限。

对用户信息进行录入。有Customer, Product的写权限。

给boss提供视图和存储过程。

## 例子

1. root(admin) 建表 见上

```

mysql -u root -p

use sales;

```

2. 切换到manager

```

quit;

mysql -u manager -p

use sales;

```

3. 添加索引
    * Customer索引：
        * birth
        * name
        * city
        * join_time

    * Product索引：
        * qgp
        * name
        * category

    * Stock
        * price
        * create_time

    * ImportOrder/ExportOrder
        * create_time

```

CREATE INDEX birth ON Customer (birth);
CREATE INDEX name ON Customer (name);
CREATE INDEX city ON Customer (city);
CREATE INDEX join_time ON Customer (join_time);

CREATE INDEX qgp ON Product (qgp);
CREATE INDEX name ON Product (name);
CREATE INDEX CATEGORY ON Product (category);

CREATE INDEX price ON Stock (price);
CREATE INDEX create_time ON Stock (create_time);

CREATE INDEX create_time ON ImportOrder (create_time);
CREATE INDEX create_time ON ExportOrder (create_time);

```

4. 切换到Employee

```

quit;

mysql -u employee -p

use sales;

```

5. 添加客户

```

INSERT INTO Customer (name, mail, birth, phone, city, join_time) VALUES
("CustomerA", "a@customer.com", "1990-01-01T00:00:00", "10000000", "Nanjing", "2000-01-01T00:00:00"),
("CustomerA", "b@customer.com", "1985-01-01T00:00:00", "10000001", "Beijing", "2005-01-01T00:00:00"),
("CustomerC", "c@customer.com", "1980-01-01T00:00:00", "10000002", "Shanghai", "2010-01-01T00:00:00"),
("CustomerD", "d@customer.com", "1975-01-01T00:00:00", "10000003", "Nanjing", "2015-01-01T00:00:00"),
("CustomerE", "e@customer.com", "1970-01-01T00:00:00", "10000004", "Beijing", "2015-01-01T00:00:00");

```

6. 添加商品信息

```

INSERT INTO Product (name, qgp, category) VALUES
('noodle', 30, 'food'),
('peach', 3, 'fruit'),
('milk', 3, 'food'),
('tissue', 365, 'daily'),
('umbrella', 365, 'daily');

```

7. 切换到employee

```

quit;

mysql -u employee -p

use sales;

```

8. 第一批进货

三个月前向CustomerA进了5个noodle;
去年向CustomerB进了3个tissue;
今天向CustomerC进了3个umbrella;

```

# 使用事务是为了

START TRANSACTION;

INSERT INTO ImportOrder (create_time, cid) VALUES
(date_add(now(), interval -3 month), 1);

INSERT INTO Stock (price, create_time, pid, import_order_id) VALUES 
(1, date_add(now(), interval -7 day), 1, 1),
(1, date_add(now(), interval -7 day), 1, 1),
(1, date_add(now(), interval -7 day), 1, 1),
(1, date_add(now(), interval -7 day), 1, 1),
(1, date_add(now(), interval -7 day), 1, 1)
;

COMMIT;

```

```

START TRANSACTION;

INSERT INTO ImportOrder (create_time, cid) VALUES
(date_add(now(), interval -1 year), 2);

INSERT INTO Stock (price, create_time, pid, import_order_id) VALUES 
(1, date_add(now(), interval -1 year), 4, 2),
(1, date_add(now(), interval -1 year), 4, 2),
(1, date_add(now(), interval -1 year), 4, 2)
;

COMMIT;

```

```

START TRANSACTION;

INSERT INTO ImportOrder (create_time, cid) VALUES
(now(), 3);

INSERT INTO Stock (price, create_time, pid, import_order_id) VALUES 
(15, date_add(now(), interval -1 day), 5, 3),
(15, date_add(now(), interval -1 day), 5, 3),
(15, date_add(now(), interval -1 day), 5, 3)
;

COMMIT;

```

9. 第二批进货

今天向CustomerD进了2个noodle

```

START TRANSACTION;

INSERT INTO ImportOrder (create_time, cid) VALUES
(now(), 4);

INSERT INTO Stock (price, create_time, pid, import_order_id) VALUES 
(3, date_add(now(), interval -1 day), 1, 4),
(1, date_add(now(), interval -2 day), 1, 4)
;

COMMIT;

```

10. employee按照进货时间(倒叙)查看库存情况(商品名，进货时间)

```

# JOIN 三张表(Stock, import_order, product), 这里使用LEFT JOIN是因为不希望出现结果里出现没有库存的商品, 如果希望出现应该使用JOIN或者颠倒左右的位置

# 也可以不用join用子查询

SELECT p.name, i.create_time FROM Stock AS s LEFT JOIN ImportOrder AS i ON s.import_order_id = i.id LEFT JOIN Product as p ON p.id = s.pid order by create_time desc;

```

11. employee统计库存数量

```

SELECT p.name, Count(*) AS stock_count FROM Product AS p LEFT JOIN Stock AS s ON p.id = s.pid GROUP BY p.name;

```

12. employee按照库存量(倒叙)排序查看库存数量

```

SELECT p.name, Count(*) AS stock_count FROM Product AS p LEFT JOIN Stock AS s ON p.id = s.pid GROUP BY p.name ORDER BY stock_count desc;

```

13. employee按照库存量(倒叙)排序查看库存数量并且过滤掉商品类型是食物的商品

# 用where来过滤是因为这是在groupby之前做的

```

SELECT p.name, Count(*) AS stock_count FROM Product AS p LEFT JOIN Stock AS s ON p.id = s.pid WHERE p.category <> "food" GROUP BY p.name ORDER BY stock_count desc;

```

14. employee按照库存量(倒叙)排序查看库存数量并且过滤掉商品类型是食物的商品并且库存数量小于等于1

# 用having来过滤是因为这是在groupby之后做的

```

SELECT p.name, Count(*) AS stock_count FROM Product AS p LEFT JOIN Stock AS s ON p.id = s.pid WHERE p.category <> "food" GROUP BY p.name HAVING stock_count <= 1 ORDER BY stock_count desc;

```

14. employee查看过期的商品

```

SELECT s.id, p.name, s.create_time, CONCAT(p.qgp, ' days') AS QGP from Stock AS s LEFT JOIN Product AS p ON s.pid = p.id LEFT JOIN ImportOrder as i ON s.import_order_id = i.id where date_add(i.create_time, interval p.qgp day) < now();

```

15. employee删除这些过期的商品

```

DELETE s FROM Stock AS s LEFT JOIN Product AS p ON s.pid = p.id LEFT JOIN ImportOrder as i ON s.import_order_id = i.id where date_add(i.create_time, interval p.qgp day) < now();

```

16. employee售货

客户D昨天买了三个noodle
客户E昨天买了1个umbrella

```

START TRANSACTION;

INSERT INTO ExportOrder (create_time, cid) VALUES
(date_add(now(), interval -1 day), 4);

UPDATE Stock SET export_order_id = 1 WHERE pid = 1 LIMIT 3;

COMMIT;

```

```

START TRANSACTION;

INSERT INTO ExportOrder (create_time, cid) VALUES
(date_add(now(), interval -1 day), 5);

UPDATE Stock SET export_order_id = 2 WHERE pid = 5 LIMIT 1;

COMMIT;

```

17. 退出employee,以manager身份登入

```

quit;

mysql -u manager -p

use sales;

```

18. manager查看当天过生日的客户

```

SELECT * FROM Customer WHERE date(birth) = date(now());

```

19. manager给从CustomerC买来的umbrella打八折

```

UPDATE Stock AS s LEFT JOIN ImportOrder AS i ON s.import_order_id = i.id LEFT JOIN Customer AS c ON c.id = i.cid LEFT JOIN Product AS p ON s.pid = p.id SET price = 0.8 * price where c.name = "CustomerC" and p.name = "umbrella";

```

20. manager统计购买消费最多的用户

```

SELECT c.name, Sum(s.price) AS SumOfMoney FROM ExportOrder AS e LEFT JOIN Customer AS c ON c.id = e.cid LEFT JOIN Stock AS s ON s.export_order_id = e.id GROUP BY c.name ORDER BY SumOfMoney DESC LIMIT 1;

```

21. 将18，20做成视图

```

CREATE VIEW customer_whos_on_birthday AS SELECT * FROM Customer WHERE date(birth) = date(now());

```

```

CREATE VIEW customer_whos_big_customer AS SELECT c.name, Sum(s.price) AS SumOfMoney FROM ExportOrder AS e LEFT JOIN Customer AS c ON c.id = e.cid LEFT JOIN Stock AS s ON s.export_order_id = e.id GROUP BY c.name ORDER BY SumOfMoney DESC LIMIT 1;

```

22. manager统计某一个商品某一年购买消费最多的用户，并做成存储过程

```

# delimiter被用来修改句尾结束的分隔符，因为我们需要多个语句，所以使用;会使存储过程结束，需要替换

DELIMITER $$

CREATE PROCEDURE big_customer (IN product_name varchar(256), IN which_year varchar(16), OUT custom_name varchar(256), OUT payment int)
BEGIN
    SELECT c.name, Sum(s.price) AS SumOfMoney INTO custom_name, payment FROM ExportOrder AS e LEFT JOIN Customer AS c ON c.id = e.cid LEFT JOIN Stock AS s ON s.export_order_id = e.id LEFT JOIN Product AS p on p.id = s.pid WHERE year(e.create_time) = which_year and p.name = product_name GROUP BY c.name ORDER BY SumOfMoney DESC LIMIT 1;
END $$

DELIMITER ;

```

23. 退出manager,以boss登入

```

quit;

mysql -u boss -p

use sales;

```

24. 使用 customer_whos_big_customer 视图

```

select * from customer_whos_big_customer;

```

25. 使用 big_customer 视图

```

CALL big_customer('noodle', '2019', @customer, @payment);

SELECT @customer AS Customer, @payment;

```

26. 开除employee

```

drop user employee;

```