Part2

#Problem1
Write a query to compare each active account’s balance to the average balance of all closed
accounts of the same type.

Create a difference column holding the difference between the account balance
(amount) and the average balance of closed accounts for accounts of the same type
to the right of all the other columns


select * from account limit 10;
select type, avg(amount) from problem1.account group by type;

select A.id, A.type, A.status, A.amount, (A.amount - B.avg_amt) as difference
from account A 
join (select type, avg(amount) as avg_amt from problem1.account group by type) B
On A.type = B.type;


#Problem2
## Create an employee table in the metastore that contains the employee records stored in HDFS

create database problem2;

create EXTERNAL TABLE employee
(
   id INT,
   fname   STRING,
   lname   STRING,
   address STRING,
   city    STRING,
   state   STRING,
   zip     STRING,
   birthday  STRING,
   hireday STRING,
 )
  STORED AS PARQUET
  LOCATION '/user/training/problem2/data/employee'
  
  
#Problem3
Output Requirements
 Create a metastore table named solution stored in the problem3 database that
contains the customer records that match the criteria
 Each solution record should contain the Customer ID, first name, last name, and
home phone of the customerf
 Maintain the same column names and datatypes in the new table as the fields from
the customer table


#Problem5
Output Requirements
 Write the report query in the local file /home/training/problem5/solution.sql
 Executing the solution.sql script from the hive command-line tools should generate
the report output
 The report should contain first name, last name, zip with a comma as the delimiter
between fields
 Only output records that have a city value of ‘Palo Alto’ and state value of ‘CA’
 Duplicate records (if any) should be included


#Problem6
Output Requirements
 Create a new table named solution stored in the problem6 database with the same
file format as the employee table
 Maintain the same column names and datatypes in the new table except for the
the birthday field. Rename that column to “birthyear”
 The birthday field in the solution table should be truncated to only contain year
instead of the current month/day/year data that is in the employee table

create table solution
(
	  id       INT,
	  fname  STRING,
	  lname    STRING,
	  address STRING,
	  city     STRING,
	  state   STRING,
	  zip      STRING,
	  birthyear STRING
)
insert into table solution
select id, fname, lname, address, city, state, zip, substring(birthday,7,9)
from employee;

select * from solution limit 10;


#Problem7
Output Requirements
 Write the report query in the local file /home/training/problem7/solution.sql
 Executing the solution.sql script from the hive command-line tools should generate
the report output
 The employee names should be printed out as the last name, a comma, and the
first name
 The output should be sorted by last name and then by first name and should only
contain employees whose city is ‘Seattle’
 Duplicate names should be included (if any)

select * from employee limit 10;

select concat(lname, ',', fname) as fullname from employee 
where city ='Seattle'  limit 10;



#Problem8
Data Description
The data files are in the HDFS directory /user/training/problem8/data/customer/.
MySQL database information:
 Installation: localhost
 Database name: problem8
 Table name: solution
 Username: cloudera
 Password: cloudera
Output Requirements
 Export all of the customer data from HDFS into the MySQL solution table
 The solution table already exists in the MySQL database but currently has no rows

create database problem8;

sqoop export --connect jdbc:mysql://localhost/problem8 --username cloudera --password cloudera --export-dir /user/training/problem8/data/customer --table solution --input-fields-terminated-by '\t'


#Problem9
Output Requirements
 Create a new table named solution stored in the problem9 database
 Maintain the same column names and datatypes as the customer table, except
store the id as a string
 The solution table should have all of the data from the customer table, with the
addition of the letter ‘A’ to the existing id values to make them unique

select * from customer limit 10;

create table solution
(
	  id     STRING,
	  fname  STRING,
	  lname    STRING,
	  address STRING,
	  city     STRING,
	  state   STRING,
	  zip      STRING,
	  birthyear STRING
)

insert into solution
select concat('A', id),
	     fname,
	     lname,
	     address,
	     city,
	     state,
	     zip,
	     birthday
  FROM customer;
  
alter table customer change id id string;

select * from solution limit 10;


 
   

   
