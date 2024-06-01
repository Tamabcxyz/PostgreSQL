## Learn something new about PostgreSQL

#### Install setup
sudo apt update         
sudo apt install postgresql postgresql-contrib          
sudo systemctl start postgresql             
sudo systemctl enable postgresql            
sudo systemctl restart postgresql           

#### common command https://www.postgresql.org/docs/14/sql-commands.html
sudo -i -u postgres (switch to user postgres)               
psql (access postgressql with default user is postgres)                    
\password postgres (to change password of user postgres)              
\q (to exit)           
\c mydatabase (connect to mydatabase)                       
\e (edit using editor)              

#### data type can be check with SELECT(NUMBER::type)
```
numbers
    + SMALLINT (2 bytes -32,768 to 32,767)
    + INTEGER (4 bytes -2,147,483,648 to 2,147,483,647)
    + BIGINT (8 bytes -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807)
    + SMALLSERIAL (2 bytes 1 to 32,767)
    + SERIAL (4 bytes 1 to 2,147,483,647)
    + BIGSERIAL (8 bytes 1 to 9,223,372,036,854,775,807)
    + REAL (4 bytes, 6 decimal digits precision)
    + DOUBLE precision ()
    + NUMERIC
    + DECIMAL
    + FLOAT
date/time/interval
geometric
boolean
currency
character
    + CHAR(5)       (store some characters, lenght will always be 5 even if PG has to insert spaces)
    + VARCHAR       (store any length of string)
    + VARCHAR(40)   (store a string up to 40 characters, automatically remove extra characters)
    + TEXT          (store any length of string)
range
xml
binary
json
arrays
uuid
```

##### create database
```
CREATE DATABASE name
    [ WITH ] [ OWNER [=] user_name ]
           [ TEMPLATE [=] template ]
           [ ENCODING [=] encoding ]
           [ LOCALE [=] locale ]
           [ LC_COLLATE [=] lc_collate ]
           [ LC_CTYPE [=] lc_ctype ]
           [ TABLESPACE [=] tablespace_name ]
           [ ALLOW_CONNECTIONS [=] allowconn ]
           [ CONNECTION LIMIT [=] connlimit ]
           [ IS_TEMPLATE [=] istemplate ]
Ex: CREATE DATABASE mydatabase;
```
ALTER DATABASE template1 REFRESH COLLATION VERSION;

##### create table and insert data
```
CREATE TABLE cities (
  name VARCHAR(50), 
  country VARCHAR(50),
  population INTEGER,
  area INTEGER
);

INSERT INTO cities (name, country, population, area)
VALUES ('Tokyo', 'Japan', 38505000, 8223);

INSERT INTO cities (name, country, population, area)
VALUES 
	('Delhi', 'India', 28125000, 2240),
  ('Shanghai', 'China', 22125000, 4015),
  ('Sao Paulo', 'Brazil', 20935000, 3043);

CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50)
);

INSERT INTO users (username)
VALUES
	('monahan93'),
  ('pferrer'),
  ('si93onis'),
  ('99stroman');

CREATE TABLE photos (
  id SERIAL PRIMARY KEY,
  url VARCHAR(200),
  user_id INTEGER REFERENCES users(id)
);

INSERT INTO photos (url, user_id)
VALUES
	('http://one.jpg', 4);

INSERT INTO photos (url, user_id)
VALUES
	('http://two.jpg', 1),
  ('http://25.jpg', 1),
  ('http://36.jpg', 1),
  ('http://754.jpg', 2),
  ('http://35.jpg', 3),
  ('http://256.jpg', 4);
```

##### string operators and functions
```
||              ==>join two strings
CONCAT()        ==>join two strings
LOWER()         ==>give a lower case string
LENGTH()        ==>gives number of characters in a string
UPPER()         ==>gives a upper case string
Ex:
SELECT name || country FROM cities;

SELECT name || ', ' || country FROM cities;

SELECT name || ', ' || country AS location FROM cities;

SELECT CONCAT(name, country) AS location FROM cities;

SELECT CONCAT(name, ', ', country) AS location FROM cities;

SELECT
  CONCAT(UPPER(name), ', ', UPPER(country)) AS location
FROM
  cities;

SELECT
  UPPER(CONCAT(name, ', ', country)) AS location
FROM
  cities;
```

##### comparison math operators
```
=                   =>are the values equal?
>                   =>is the value on the left greater?
<                   =>is the value on the left less?
>=                  =>is the value on the left greater or equal to?
in                  =>is the value present in a list
<=                  =>is the value on the left lesser or equal to?
<>                  =>are the values not equal?
!=                  =>are the values not equal?
BETWEEN             =>is the value between two other values?
NOT IN              =>is the value not present in a list?
```


##### update table
```
UPDATE cities
SET population = 12345
WHERE name = 'Tokyo';
```
##### update table
```
DELETE
FROM cities
WHERE name = 'Tokyo';
```

##### query
```
SELECT
FROM                        (specifies starting set of rows to work with)           
JOIN                        (merge in data from additional tables)                  
WHERE                       (filters the set of rows)                               
GROUP BY                    (groups rows by a unique set of values)                 
HAVING                      (filters the set of groups)                             


SELECT name, price * units_sold AS revenue
FROM phones

SELECT name, area FROM cities WHERE area > 4000;
SELECT name, area FROM cities WHERE area IN ('Delhi', 'Sanghai');
SELECT name, area FROM cities WHERE area NOT IN ('Tokio'); 
SELECT name, area FROM cities WHERE area NOT IN ('Tokio') AND name='Newdeli'; 

SELECT * from photos
JOIN users ON users.id = photos.user_id;

(
    SELECT *
    FROM products
    ORDER BY price DESC
    LIMIT 4
)
UNION           (this key word to join two result to gether and make sure them unique)
(
    SELECT *
    FROM products
    ORDER BY price / weight DESC
    LIMIT 4
);

subquery

SELECT name, price
FROM products
WHERE price > (
    SELECT MAX(price)
    FROM products
    WHERE department = 'Toys'
);

common table expression (CTE)

WITH tags AS (
  SELECT user_id, post_id, created_at FROM cation_tags
  UNION ALL
  SELECT user_id, post_id, created_at FROM photo_tags
)
SELLECT username, tags.created_at
FROM users;
JOIN tags ON tags.user_id = users.id
WHERE tags.created_at < '2010-01-07';


recursive CTE

WITH RECURSIVE countdown(val) AS (
  SELECT 10 as val
  UNION
  SELECT val - 1 FROM countdown WHERE val > 1
)
SELECT *
FROM countdown;
```

#### some important node
- Foreign keys can be NULL
- If you delete a record, it has been a foreign key in another table
ON DELETE NO ACTION                 throw an error
ON DELETE RESTRICT                  throw an error
ON DELETE CASCADE                   delete record of another table too
```
CREATE TABLE photos (
  id SERIAL PRIMARY KEY,
  url VARCHAR(200),
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE
);
```
ON DELETE SET NULL                  set the fogreign key to NULL
```
CREATE TABLE photos (
  id SERIAL PRIMARY KEY,
  url VARCHAR(200),
  user_id INTEGER REFERENCES users(id) ON DELETE SET NULL
);
```
ON DELETE SET DEFAULT               set default value

- Join and aggregation              
join            
+ produces values by merging together rows from different related tables                
+ use a join most time that you're asked to find data that involves mutiple resources           
aggregation
+ looks at many rows and calculates a single value                  
+ words like 'most', 'everage', 'least' are a sign that you need to an aggegation                

#### aggregate function
```
COUNT()                         =>return the number of values in a group of values
SUM()                           =>finds the sum of a group of numbers
AVG()                           =>finds the average of a group of numbers
MIN()                           =>return the minimum value from a group of numbers
MAX()                           =>returns the maximum value from a group of numbers
GREATEST()                      =>select the greatest number
LEAST()                         =>select the least number
```
#### commonalities with intersect
```
UNION                           =>join together result of two queries and remove duplicate rows
UNION ALL                       =>join together result of two queries
INTERSECT                       =>find the rows common in the result of two queries. Remove duplicates
INTERSECT ALL                   =>find the rows common in the result of two queries
EXCEPT                          =>find the rows that are present in first query but not second query. Remove duplicates
EXCEPT ALL                      =>find the rows that are present in first query but not second query
```

#### constraint in postgresql
Constraints in PostgreSQL are rules applied to columns and tables to ensure the integrity and accuracy of the data in the database.
1. NOT NULL Constraint:
```
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL
);
```
2. UNIQUE Constraint:
```
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE
);
```
3. PRIMARY KEY Constraint:
```
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
```
4. FOREIGN KEY Constraint:
```
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department_id INT,
    CONSTRAINT fk_department
        FOREIGN KEY(department_id) 
        REFERENCES departments(id)
);
```
5. CHECK Constraint:
```
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    salary NUMERIC CHECK (salary > 0)
);
```
6. EXCLUSION Constraint:
```
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    salary NUMERIC,
    CONSTRAINT salary_exclusion EXCLUDE USING gist (salary WITH =)
);
```
#### Adding Constraints to Existing Tables
Add a NOT NULL constraint:
```
ALTER TABLE employees 
ALTER COLUMN name SET NOT NULL;
```
Add a UNIQUE constraint:
```
ALTER TABLE employees 
ADD CONSTRAINT unique_email UNIQUE (email);
```
Add a PRIMARY KEY constraint:
```
ALTER TABLE employees 
ADD CONSTRAINT pk_employee PRIMARY KEY (id);
```
Add a FOREIGN KEY constraint:
```
ALTER TABLE employees 
ADD CONSTRAINT fk_department 
FOREIGN KEY (department_id) REFERENCES departments (id);
```
Add a CHECK constraint:
```
ALTER TABLE employees 
ADD CONSTRAINT check_salary CHECK (salary > 0);
```
#### Dropping Constraints
Drop a NOT NULL constraint:
```
ALTER TABLE employees 
ALTER COLUMN name DROP NOT NULL;
```
Drop a UNIQUE constraint:
```
ALTER TABLE employees 
DROP CONSTRAINT unique_email;
```
Drop a PRIMARY KEY constraint:
```
ALTER TABLE employees 
DROP CONSTRAINT pk_employee;
```
Drop a FOREIGN KEY constraint:
```
ALTER TABLE employees 
DROP CONSTRAINT fk_department;
```
Drop a CHECK constraint:
```
ALTER TABLE employees 
DROP CONSTRAINT check_salary;
```
