# Peer_Learning_Document


# Atul's Approach

### Q1 Create roles as per the below-mentioned hierarchy. Accountadmin already exists in Snowflake (10).

<img width="215" alt="Screenshot 2023-03-28 at 11 47 14 PM" src="https://user-images.githubusercontent.com/122472996/228331069-066a0dcc-36a0-4e4b-98c5-9ecaf2a73b63.png">

```
CREATE ROLE DEVELOPER; -- creating the `DEVELOPER` role 

CREATE ROLE ADMIN;     -- creating the `Admin` role

CREATE ROLE PII;       -- creating the `PII` role



GRANT ROLE DEVELOPER TO ROLE ADMIN;

GRANT ROLE ADMIN TO ROLE ACCOUNTADMIN;

GRANT ROLE PII TO ROLE ACCOUNTADMIN;


```

`THIS IS HOW THE OUTPUT COMES AFTER RUNNING FROM ACCOUNTADMIN` 

OUTPUT ->  
<img width="521" alt="Screenshot 2023-03-28 at 11 51 40 PM" src="https://user-images.githubusercontent.com/122472996/228331935-12e3e658-b950-4a05-a162-3ffddcd8c86f.png">

#### Approach :
+ First creating all roles as mentioned in the question.
+ Then For creating heirarchy just granting the role to other role as mentioned.

###  Q2 Create an M-sized warehouse using the accountadmin role, name -> assignment_wh and use it for all the queries ( 5 ).

```
CREATE or replace WAREHOUSE assignment_wh
WITH 
WAREHOUSE_SIZE = 'MEDIUM';

GRANT ALL PRIVILEGES ON WAREHOUSE assignment_wh TO ROLE ADMIN;  -- Because the assigment_wh is created from ACCOUNTADMIN role so we have to give the                                                                          permission.


```

#### Approach :
+ First creating the warehouse from accountadmin role and then granting the permission to the `ADMIN` role.

###  Q3 Switch to the admin role ( 5 ).

```
USE ROLE ADMIN ;

```

### Q4 Create a database assignment_db ( 5 ).

```
grant create database on account to role admin; -- Atul ran this command from accountadmin  

CREATE DATABASE assignment_db;
```
#### Approach : 
+ First granting the permission so that `ADMIN` and create the database.
+ Then Creating the database with name `assignment_db`

### Q5 Create a schema my_schema ( 5 ).

```
CREATE SCHEMA my_schema;

```


### Q6 Create a table using any sample csv. You can get 1 by googling for sample csv’s. Preferably search for a sample employee dataset so that you have PII related columns else you can consider any column as PII ( 5).

 `Dataset description  : `

id NUMBER,
firstname VARCHAR(255),
lastname VARCHAR(255),
email VARCHAR(255),
ssn VARCHAR(255),
dept VARCHAR(255),
contact_no NUMBER,
city VARCHAR(255)

Total Record = 1000 



<img width="1049" alt="Screenshot 2023-03-29 at 12 01 35 AM" src="https://user-images.githubusercontent.com/122472996/228334301-3f89edbb-15d6-405d-bcad-acfd1cd4bf8d.png">


Final Schema of the table as per requirements: 
```
CREATE TABLE employee (
  id NUMBER,
  firstname VARCHAR(255),
  lastname VARCHAR(255),
  email VARCHAR(255),
  ssn VARCHAR(255),
  dept VARCHAR(255),
  contact_no NUMBER,
  city VARCHAR(255),
  etl_ts timestamp default current_timestamp(),
  etl_by varchar default 'snowsql',
  file_name varchar 
);
```
#### Approach : 
+ Atul have use [CSV GENERATOR](https://extendsclass.com/csv-generator.html) to create the data .

###  Q7 Also, create a variant version of this dataset ( 5 ).

```
CREATE OR REPLACE FILE FORMAT my_json_format -- This command created using snowsight and this is the file format .
  TYPE = JSON;


create table raw_table (                     -- This command created using snowsight
    raw_data variant
);

-- After we have SNOWSQL to put this json format file to the snowflake and the file has been pushed to table stage. @%[table-name] this is table stage.

put file://~/Downloads/employee2.json @%raw_table;   -- This command is applied through snowsql

-- Then command is used snowsight  and copying the data to raw_table ;
copy into raw_table from @%raw_table file_format = my_json_format;  


-- checking that file is properly loaded or not.
select * from raw_table;

``` 
#### Approach :
+ First created the File Format for the JSON
+ Then created a table with name `raw_table` with one column and it store variant data.
+ Then loaded the file from local to table stage `@%raw_table`
+ Finally Copy into the main table .

Reference - [Variant Data Type](https://docs.snowflake.com/en/sql-reference/data-types-semistructured)

### Q8 Load the file into an external and internal stage ( 5 ).

```
 CREATE OR REPLACE FILE FORMAT my_csv_format             -- creating the file format 
 TYPE = 'CSV'
 FIELD_DELIMITER = ','
 SKIP_HEADER = 1;
 
 -- internal stage
 
 create stage my_internal_stage    
 file_format = my_csv_format;
 
 GRANT ALL ON INTEGRATION s3_integration TO ROLE admin; -- Atul ran this command from ACCOUNTADMIN to provide all the privileges to `ADMIN` role.
 
 -- storage integration with aws 
 
create  storage integration s3_integration
type = external_stage
storage_provider = s3
enabled = true
storage_aws_role_arn = 'arn:aws:iam::444024112261:role/sigmoid'            -- This is copied from the iam role arn 
storage_allowed_locations = ('s3://snowflakes3aws/csv/employee1.csv');     -- This is url from s3 bucket where data is uploaded 


DESC INTEGRATION s3_integration;                                           -- This is for the making the trust relationship 

create or replace stage EXTERNAL_CSV_STAGE
URL = 's3://snowflakes3aws/csv/employee1.csv'
STORAGE_INTEGRATION = s3_integration
file_format = my_csv_format;


```
OUTPUT -> While making trust relationship between aws and snowflake 

<img width="801" alt="Screenshot 2023-03-29 at 1 54 16 PM" src="https://user-images.githubusercontent.com/122472996/228473289-d73e215e-704f-4cf0-8adb-ff668a67b31e.png">

#### Approach :
+ First Atul have created csv file format with name `my_csv_format`
+ Then created the internal stage with name `my_internal_stage`
+ Then created the s3_integration in accountadmin and then granted all privileges to the `ADMIN`
+ Then created trust realtion between aws and snowflake by updating the `Trust Relationship` in the role from where we copied the arn into the               s3_integration.
+ Then created the external_stage.

### Q9 Load data into the tables using copy into statements. In one table load from the internal stage and in another from the external ( 10 ).

```

CREATE TABLE internal_employee (
  id NUMBER,
  firstname VARCHAR(255),
  lastname VARCHAR(255),
  email VARCHAR(255),
  ssn VARCHAR(255),
  dept VARCHAR(255),
  contact_no NUMBER,
  city VARCHAR(255),
  etl_ts timestamp default current_timestamp(),    
  etl_by varchar default 'snowsql',
  file_name varchar 
);

CREATE TABLE external_employee (
  id NUMBER,
  firstname VARCHAR(255),
  lastname VARCHAR(255),
  email VARCHAR(255),
  ssn VARCHAR(255),
  dept VARCHAR(255),
  contact_no NUMBER,
  city VARCHAR(255),
  etl_ts timestamp default current_timestamp(),
  etl_by varchar default 'snowsql',
  file_name varchar 
);


put file://~/Downloads/employee1.csv @my_internal_stage;

-- copying into internal_employee from internal stage 

COPY INTO internal_employee(
id, firstname,lastname,email,ssn,dept,contact_no ,city,file_name)
  FROM (
  SELECT emp.$1, emp.$2, emp.$3, emp.$4, emp.$5, emp.$6, emp.$7,emp.$8, METADATA$FILENAME FROM @my_internal_stage/employee1.csv.gz (file_format => my_csv_format) emp);
  
-- This Metadata is store in the internal/external stage.
  

-- copying into external_employee from external stage

COPY INTO external_employee(
id, firstname,lastname,email,ssn,dept,contact_no ,city,file_name)
  FROM (
  SELECT emp.$1, emp.$2, emp.$3, emp.$4, emp.$5, emp.$6, emp.$7,emp.$8, METADATA$FILENAME FROM @EXTERNAL_CSV_STAGE (file_format => my_csv_format) emp);



```


-- checking the data is loaded perfectly or not .... 

Using  `select * from internal_employee limit 10;` 

<img width="827" alt="Screenshot 2023-03-29 at 11 20 00 AM" src="https://user-images.githubusercontent.com/122472996/228438643-9019abf3-9378-450d-9481-23f4f8c41143.png">

-- checking the data is loaded perfectly or not .... 

Using  `select * from external_employee limit 10;` 

<img width="832" alt="Screenshot 2023-03-29 at 11 22 24 AM" src="https://user-images.githubusercontent.com/122472996/228439036-ad07d12b-3f33-4752-968a-8019b7140194.png">

#### Approach:
+ Firstly, Created the table with `internal_employee` and `external_employee`.
+ Then loading the file from local to internal stage  and also load the data into the external stage.
+ Then copy into table name `internal_employee` and same with the `external_employee`
+ Then checking the data is loaded or not.



 Reference -  [For Loading the filename while inserting the data](https://docs.snowflake.com/en/user-guide/querying-metadata)



### Q10 Upload any parquet file to the stage location and infer the schema of the file ( 5 ).

```
CREATE FILE FORMAT my_parquet_format
  TYPE = parquet;
  
create stage my_parquet_stage file_format = my_parquet_format;

put file://~Downloads/data.parquet @my_parquet_stage;


SELECT *
  FROM TABLE(
    INFER_SCHEMA(
      LOCATION=>'@my_parquet_stage'
      , FILE_FORMAT=>'my_parquet_format'
      )
    );

```

Reference - [Infer Schema](https://docs.snowflake.com/en/sql-reference/functions/infer_schema)

#### Approach:
+ Firstly, Created the parquet format.
+ Then created the internal stage. 
+ Load the file in the table .
+ Then Infer the schema.

### Q11 Run a select query on the staged parquet file without loading it to a snowflake table ( 5 ).

```
select * from @my_parquet_stage/data.parquet

```

### Q12 Add masking policy to the PII columns such that fields like email, phone number, etc. show as **masked** to a user with the developer role. If the role is PII the value of these columns should be visible ( 15 ).


```
create or replace masking policy email_mask as (val string) returns string ->
    case
    when current_role() in ('ADMIN','PII') then val
    else '*********'
    end;

create or replace masking policy contact_Mask as (val integer) returns number ->
    case
    when current_role() in ('ADMIN','PII') then val
    else 99999999999
    end;


alter table if exists internal_employee 
modify column email set masking policy email_mask;

alter table if exists external_employee 
modify column email set masking policy email_mask;

alter table if exists internal_employee 
modify column contact_no set masking policy contact_Mask;

alter table if exists external_employee 
modify column contact_no set masking policy contact_Mask;

select * from internal_employee limit 10;
select * from external_employee limit 10;



```
output -> 
<img width="1110" alt="snowflake_question12" src="https://user-images.githubusercontent.com/122472996/228472192-d7980423-f71e-42bb-81e6-dd151f7479cd.png">

#### Approach : 
+ First created all the masking policy contact_mask and email_mask.
+ then adding it to the column.


Reference - [Masking Policy](https://docs.snowflake.com/en/sql-reference/sql/create-masking-policy)


### Major Insights 

+ The storage integration can only be created by an account admin but creating stages can be done by other roles. The role needs CREATE STAGE privilege     for the schema as well as the USAGE privilege on the integration.





# Mahesh's Approach

### Q  1 :- Create roles as per the below-mentioned hierarchy. Accountadmin already exists in Snowflake (10).

<img width="215" alt="Screenshot 2023-03-28 at 11 47 14 PM" src="https://user-images.githubusercontent.com/122472996/228331069-066a0dcc-36a0-4e4b-98c5-9ecaf2a73b63.png">

```
CREATE ROLE DEVELOPER; -- Creating the 'DEVELOPER' role 

CREATE ROLE ADMIN;     -- Creating the 'Admin' role

CREATE ROLE PII;       -- Creating the 'PII' role  



GRANT ROLE DEVELOPER TO ROLE ADMIN;   

GRANT ROLE ADMIN TO ROLE ACCOUNTADMIN;     

GRANT ROLE PII TO ROLE ACCOUNTADMIN;     


```




###   Explanation:
      1. Mahesh created all roles as given in the question.  
      2. After that mahesh created heirarchy just granting the role to other role as given in the question.  

 

### Output :- 
<img width="754" alt="Screenshot 2023-03-29 at 1 57 20 PM" src="https://user-images.githubusercontent.com/122514232/228473931-1faabcf3-6af1-4fec-88f9-5d2b8c6e8a43.png">

###  Q 2 :- Create an M-sized warehouse using the accountadmin role, name -> assignment_wh and use it for all the queries ( 5 ).

```
CREATE OR REPLACE WAREHOUSE assignment_wh
WITH 
WAREHOUSE_SIZE = 'MEDIUM';

GRANT ALL PRIVILEGES ON WAREHOUSE assignment_wh TO ROLE ADMIN;  -- the assigment_wh is created from ACCOUNTADMIN role so we have to give the permission.   

```

<img width="1440" alt="assignment_wh" src="https://user-images.githubusercontent.com/122514232/228787177-30a9c41a-e78f-40cd-a493-cc2c63c5bd52.png">


###   Explanation:
      1.  Mahesh created the warehouse from ACCOUNTADMIN role.  
      2.  Granted the permission to the ADMIN role.

###  Q 3 :-  Switch to the admin role ( 5 ).

```

USE ROLE ADMIN ;

```

<img width="865" alt="Screenshot 2023-03-29 at 10 57 32 PM" src="https://user-images.githubusercontent.com/122514232/228619776-ca3ece8f-1cc8-4397-9847-1d417c301fe5.png">


### Q 4 :- Create a database assignment_db ( 5 ).

```
GRANT CREATE DATABASE ON ACCOUNT TO ADMIN;    
CREATE DATABASE assignment_db;
```

###   Approach :
      1. Granted the permission so that ADMIN and create the database.
      2. Created the database with name assignment_db

### Q 5 :- Create a schema my_schema ( 5 ).

```
CREATE SCHEMA my_schema;
```


### Q 6 :- Create a table using any sample csv. You can get 1 by googling for sample csv’s. Preferably search for a sample employee dataset so that you have PII related columns else you can consider any column as PII ( 5).

 `Dataset description  : `

id NUMBER,
firstname VARCHAR(255),
lastname VARCHAR(255),
email VARCHAR(255),
ssn VARCHAR(255),
dept VARCHAR(255),
contact_no NUMBER,
city VARCHAR(255)

Total Record = 1000 



<img width="1049" alt="Screenshot 2023-03-29 at 12 01 35 AM" src="https://user-images.githubusercontent.com/122472996/228334301-3f89edbb-15d6-405d-bcad-acfd1cd4bf8d.png">


Final Schema of the table as per requirements: 
```
CREATE TABLE employee (
  id NUMBER,
  firstname VARCHAR(255),
  lastname VARCHAR(255),
  email VARCHAR(255),
  ssn VARCHAR(255),
  dept VARCHAR(255),
  contact_no NUMBER,
  city VARCHAR(255),
  etl_ts timestamp default current_timestamp(),
  etl_by varchar default 'snowsql',
  file_name varchar 
);
```

### Explanation:
               1. Downloaded the employee.csv file from the online resource and created the employee schema as per the dataset.  
               2. Additionally added three more columns name  
                     a.) etl_ts for getting the time at which the record is getting inserted.  
                     b.) etl_by for getting application name from which the record was inserted.  
                     c.) file_name for getting the name of the file used to insert data into the table.  

###  Q 7 :- Also, create a variant version of this dataset ( 5 ).

```
CREATE OR REPLACE FILE FORMAT my_json_format 
  TYPE = JSON;   

create table raw_table (                     -- This command created using snowsight
    raw_data variant  
);  

-- After we have SNOWSQL to put this json format file to the snowflake and the file has been pushed to table stage. @%[table-name] this is table stage.

put file://~/Downloads/employee2.json @%raw_table;

-- Then command is used snowsight  and copying the data to raw_table ;
copy into raw_table from @%raw_table file_format = my_json_format;


-- checking that file is properly loaded or not.
select * from raw_table;

``` 




###   Explanation:
                 1. Creating the file format.
                 2. Now creating the variant version of table with raw_data as column name of variant data type which holds the raw data in JSON format.
                 3. Now putting the contents of employee2.json file from local system to the table stage of raw_table through Snowsql cli.
                 4. Now copying the content from the table staging area to the raw_table table by specifying the file format as my_json_format                                   created above .
                 5. Querying the employee_variant table to check for data.


### Q 8 :- Load the file into an external and internal stage ( 5 ).

```
 CREATE OR REPLACE FILE FORMAT my_csv_format             -- creating the file format 
 TYPE = 'CSV'
 FIELD_DELIMITER = ','
 SKIP_HEADER = 1;
 
 -- internal stage
 
 create stage my_internal_stage 
 file_format = my_csv_format;
 
 -- storage integration with aws 
 
create  storage integration s3_integration
type = external_stage
storage_provider = s3
enabled = true
storage_aws_role_arn = 'arn:aws:iam::444024112261:role/sigmoid'
storage_allowed_locations = ('s3://snowflakes3aws/csv/employee1.csv');


DESC INTEGRATION s3_integration;

create or replace stage EXTERNAL_CSV_STAGE
URL = 's3://snowflakes3aws/csv/employee1.csv'
STORAGE_INTEGRATION = s3_integration
file_format = my_csv_format;
```
###   Explanation:
####                1. Below are the steps to load the file into an internal stage
                       a. First Creating the file format of type CSV.
                       b. Creating the named internal stage name internal_stage using CREATE STAGE SQL command with the 
                          file format of type CSV created above.  
                       c. After creating the named internal stage called internal_stage will put the 
                          file present in the local system to the internal_stage created through Snowsql cli.
                 
####                2. Below are the steps to load the file into an external stage
                       a. The above (storage_aws_role_arn = 'arn:aws:iam::444024112261:role/sigmoid')  
                          is the arn for role created in the aws via AWS MANGMENT CONSOLE .  
                       b. And the (storage_allowed_locations = ('s3://myawssnowflakeassignkaran/csv') is   
                          the path to the AWS S3 Bucket created via AWS MANGMENT CONSOLE .  

                       c. The above aws_storage_intergration`` object is created with the ROLE ACCOUNTADMIN,  
                          So we need to give all the privileges on this   integration object to ROLE ADMIN```.  
       



  
### Q 9 :- Load data into the tables using copy into statements. In one table load from the internal stage and in another from the external ( 10 ).

```

CREATE TABLE internal_employee (
  id NUMBER,
  firstname VARCHAR(255),
  lastname VARCHAR(255),
  email VARCHAR(255),
  ssn VARCHAR(255),
  dept VARCHAR(255),
  contact_no NUMBER,
  city VARCHAR(255),
  etl_ts timestamp default current_timestamp(),
  etl_by varchar default 'snowsql',
  file_name varchar 
);
CREATE TABLE external_employee (
  id NUMBER,
  firstname VARCHAR(255),
  lastname VARCHAR(255),
  email VARCHAR(255),
  ssn VARCHAR(255),
  dept VARCHAR(255),
  contact_no NUMBER,
  city VARCHAR(255),
  etl_ts timestamp default current_timestamp(),
  etl_by varchar default 'snowsql',
  file_name varchar 
);


put file://~/Downloads/employee1.csv @my_internal_stage;

-- copying into internal_employee from internal stage 

COPY INTO internal_employee(
id, firstname,lastname,email,ssn,dept,contact_no ,city,file_name)
  FROM (
  SELECT emp.$1, emp.$2, emp.$3, emp.$4, emp.$5, emp.$6, emp.$7,emp.$8, METADATA$FILENAME FROM @my_internal_stage/employee1.csv.gz (file_format => my_csv_format) emp);
  

-- copying into external_employee from external stage
COPY INTO external_employee(
id, firstname,lastname,email,ssn,dept,contact_no ,city,file_name)
  FROM (
  SELECT emp.$1, emp.$2, emp.$3, emp.$4, emp.$5, emp.$6, emp.$7,emp.$8, METADATA$FILENAME FROM @EXTERNAL_CSV_STAGE (file_format => my_csv_format) emp);

   SELECT * FROM internal_employee;

```




###   Explanation:
      1. To load the data into the tables from internal stage first we will create the table called internal_employee.     
      2. The schema is according to the csv files that is loaded into the staging area along with the additional three columns as discussed above.    
      3. Now after creating the table, we can copy the contents of the file stored in internal_staging to the table internal_employee.  
      4. Here we are directly quering from the internal stage, so we need to follow some predefined convention of using $1,$2 and so on to    
         fetch the column values and also to get the name of the file through which the data is loaded we are quering the metadata which the   
         snowflake maintains internally for the staging areas through the command METADATA$FILENAME.    
      5. Now Quering the internal_employee table to check for data.    

### Q 10 :- Upload any parquet file to the stage location and infer the schema of the file ( 5 ).

```
CREATE FILE FORMAT my_parquet_format
  TYPE = parquet;
  
create stage my_parquet_stage file_format = my_parquet_format;

put file://~Downloads/data.parquet @my_parquet_stage;


SELECT *
  FROM TABLE(
    INFER_SCHEMA(
      LOCATION=>'@my_parquet_stage'
      , FILE_FORMAT=>'my_parquet_format'
      )
    );

```

<img width="1439" alt="Screenshot 2023-03-30 at 2 22 58 PM" src="https://user-images.githubusercontent.com/122514232/228783592-4fc14bac-fa38-409f-9c88-2f21b34578f7.png">

###   Explanation:-
                1. First Mahesh created the file format for parquet type.
                2. Now we will create the stage called parquet_stage with file format of the type created above.
                3. Now we will put the file called data.parquet present in the local system to the parquet_stage created above through Snowsql cli.
                4. Now we will query the parquet file stage location which would infer the schema from the file.



### Q 11 :- Run a select query on the staged parquet file without loading it to a snowflake table ( 5 ).

```
select * from @my_parquet_stage/data.parquet

```

<img width="1439" alt="Screenshot 2023-03-30 at 2 21 49 PM" src="https://user-images.githubusercontent.com/122514232/228782966-8349843e-217a-46cc-b75e-a4471999e1e8.png">


### Q 12 :- Add masking policy to the PII columns such that fields like email, phone number, etc. show as **masked** to a user with the developer role. If the role is PII the value of these columns should be visible ( 15 ).


```


GRANT ALL PRIVILEGES ON WAREHOUSE assignment_wh TO ROLE PII;
GRANT USAGE on DATABASE assignment_db to role PII
GRANT ALL ON ALL schemas in database ASSIGNMENT_DB TO ROLE PII; 
GRANT ALL ON ALL TABLES IN SCHEMA assignment_db.my_schema TO ROLE PII;


create or replace masking policy email_mask as (val string) returns string ->
    case
    when current_role() in ('ADMIN','PII') then val
    else '*********'
    end;

create or replace masking policy contact_Mask as (val integer) returns number ->
    case
    when current_role() in ('ADMIN','PII') then val
    else 99999999999
    end;


alter table if exists internal_employee 
modify column email set masking policy email_mask;

alter table if exists external_employee 
modify column email set masking policy email_mask;

alter table if exists internal_employee 
modify column contact_no set masking policy contact_Mask;

alter table if exists external_employee 
modify column contact_no set masking policy contact_Mask;

select * from internal_employee limit 10;
select * from external_employee limit 10;



```

### Output after masking for role 'ACCOUNTADMIN' (attribute email and contanct_no are masked. )
<img width="1433" alt="Screenshot 2023-03-29 at 4 33 40 PM" src="https://user-images.githubusercontent.com/122514232/228514374-4a838356-092a-43cd-8158-435549a94be5.png">

### Output after masking for role 'ADMIN'  
<img width="1433" alt="Screenshot 2023-03-29 at 4 35 06 PM" src="https://user-images.githubusercontent.com/122514232/228514690-e5e7d83c-9f16-449e-b0d2-96543ee8199f.png">



###       Explanation:
             1. Granting all privileges to Role PII . 
             2. Here mahesh have applied masking policy in three columns i.e., email, contact_no and ssn for both internal_employee and external_employee table
             3. The agenda is whenever the ROLE is 'ADMIN' or 'PII', the three columns that has masking policy applied should be visible
             4. Otherwise for any other ROLE it should be masked and not visible
