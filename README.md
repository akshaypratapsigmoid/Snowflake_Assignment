# Snowflake_Assignment
Github repository for snowflake assignment

### 1. Create roles as per the below-mentioned hierarchy. Accountadmin already exists in Snowflake.

<img src=https://user-images.githubusercontent.com/122514456/228508708-c6e058ff-8c4f-4318-a368-9a442a94a2b1.png width="250" height="250">

### Solution:

```
CREATE ROLE ADMIN;
CREATE ROLE DEVELOPER;
CREATE ROLE PII;
SHOW ROLES;
GRANT ROLE DEVELOPER TO ROLE ADMIN;
GRANT ROLE ADMIN TO ROLE ACCOUNTADMIN;
GRANT ROLE PII TO ROLE ACCOUNTADMIN;
```

### Output:
#### Role hierarchy:
<img width="1061" alt="Screenshot 2023-03-29 at 10 42 50 PM" src="https://user-images.githubusercontent.com/122514456/228616893-4e978034-8c46-4623-a6d3-4831df1018db.png">


### 2. Create an M-sized warehouse using the accountadmin role, name -> assignment_wh and use it for all the queries.

### Solution:
```
CREATE OR REPLACE WAREHOUSE assignment_wh
    WAREHOUSE_SIZE = MEDIUM
    AUTO_SUSPEND = 60
    AUTO_RESUME = TRUE
    COMMENT = 'This is a virtual warehouse of size Medium that will be used to process queries in the assignment.';

GRANT ALL PRIVILEGES ON WAREHOUSE assignment_wh TO ROLE ADMIN;
```

### Output:
<img width="1121" alt="Screenshot 2023-03-29 at 10 50 19 PM" src="https://user-images.githubusercontent.com/122514456/228618273-0d733ce6-4f10-4b25-a9f6-ec71e846c264.png">


### 3. Switch to the admin role.

### Solution:

```
USE ROLE ADMIN;
```

### Output:
<img width="1121" alt="Screenshot 2023-03-29 at 10 52 39 PM" src="https://user-images.githubusercontent.com/122514456/228618935-cc0bf531-1b77-498d-af2c-05d321b5cdc4.png">


### 4. Create a database assignment_db.

### Solution:
```
USE ROLE ACCOUNTADMIN;
GRANT CREATE DATABASE ON ACCOUNT TO ROLE ADMIN;
USE ROLE ADMIN;
CREATE OR REPLACE DATABASE assignment_db;
```

### Output:
<img width="1121" alt="Screenshot 2023-03-29 at 10 58 39 PM" src="https://user-images.githubusercontent.com/122514456/228620160-2b7e6be1-2f9f-4c21-9316-70f65917aa85.png">


### 5. Create a schema my_schema.

### Solution:

```
CREATE SCHEMA my_schema;
```

### Output:
<img width="1118" alt="Screenshot 2023-03-29 at 11 02 49 PM" src="https://user-images.githubusercontent.com/122514456/228621017-03c586e1-3d3a-4b35-8536-46bb1fdb47ad.png">


### 6. Create a table using any sample csv. You can get 1 by googling for sample csv’s. Preferably search for a sample employee dataset so that you have PII related columns else you can consider any column as PII.

### Solution:

```
CREATE OR REPLACE TABLE EMPLOYEE(
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

### Output:
<img width="1118" alt="Screenshot 2023-03-29 at 11 05 33 PM" src="https://user-images.githubusercontent.com/122514456/228621726-bb9d1363-5abe-472b-a05e-40a64d173a36.png">


### 7. Also, create a variant version of this dataset.

### Solution:

```
CREATE OR REPLACE FILE FORMAT my_json_format
    TYPE = JSON;

CREATE OR REPLACE TABLE RAW_VARIANT (
    RAW_VARIANT variant
);

put file:///Users/akshay/Documents/employee.json @%RAW_VARIANT;
COPY INTO RAW_VARIANT FROM @%RAW_VARIANT FILE_FORMAT=MY_JSON_FORMAT;
SELECT * FROM RAW_VARIANT;
```

### Output:
<img width="1118" alt="Screenshot 2023-03-29 at 11 09 09 PM" src="https://user-images.githubusercontent.com/122514456/228622543-b76bd592-9047-4640-908c-2d528bf0f70e.png">
<img width="1280" alt="7 ---2" src="https://user-images.githubusercontent.com/123646244/228774632-628cad0f-a882-4b37-8ea3-d97fa8a6d573.png">

<img width="1117" alt="Screenshot 2023-03-29 at 11 38 25 PM" src="https://user-images.githubusercontent.com/122514456/228629258-85c0efb9-0aa8-404d-8ce0-d6b1ec98613d.png">


### 8. Load the file into an external and internal stage.

### Solution:

```
USE ROLE ACCOUNTADMIN;
CREATE OR REPLACE STORAGE INTEGRATION s3_int
    type = external_stage
    storage_provider = s3
    enabled = true
    storage_aws_role_arn = 'arn:aws:iam::444024112261:role/snowflake_assignment_role'
    storage_allowed_locations = ('s3://bucketsnowflakeassignment/snowflake_assignment_folder/');
DESC INTEGRATION s3_int;

GRANT ALL ON INTEGRATION s3_int TO ROLE admin;

USE ROLE ADMIN;

CREATE OR REPLACE FILE FORMAT my_csv_format
   TYPE = 'CSV'
   FIELD_DELIMITER = ','
   FIELD_OPTIONALLY_ENCLOSED_BY = '"'
   SKIP_HEADER = 1;

CREATE STAGE my_s3_stage
  STORAGE_INTEGRATION = s3_int
  URL = 's3://bucketsnowflakeassignment/snowflake_assignment_folder/'
  FILE_FORMAT = my_csv_format;

CREATE STAGE my_internal_stage
    FILE_FORMAT = my_csv_format;
    
put file:///Users/akshay/Documents/employee1.csv @my_internal_stage;
```

### Output:
<img width="1162" alt="8--- 1" src="https://user-images.githubusercontent.com/123646244/228774718-9aeb2e18-73c4-44a3-b833-99934fb9bb4e.png">

<img width="1159" alt="Screenshot 2023-03-30 at 12 04 45 AM" src="https://user-images.githubusercontent.com/122514456/228635285-46110b8b-6211-4ca5-8597-213322f8989e.png">


### 9. Load data into the tables using copy into statements. In one table load from the internal stage and in another from the external.

### Solution:

```
CREATE OR REPLACE TABLE internal_staging_employee (
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

CREATE OR REPLACE TABLE external_s3_staging_employee (
  id NUMBER,
  firstname VARCHAR(255),
  lastname VARCHAR(255),
  email VARCHAR(255),
  ssn VARCHAR(255),
  dept VARCHAR(255),
  contact_no NUMBER,
  city VARCHAR(255),
  etl_ts timestamp default current_timestamp(),
  etl_by varchar default 'aws_s3',
  file_name varchar 
);

copy into INTERNAL_STAGING_EMPLOYEE(id, firstname, lastname, email, ssn, dept, contact_no, city, file_name)
    from (select emp.$1, emp.$2,emp.$3,emp.$4,emp.$5,emp.$6,emp.$7,emp.$8,METADATA$FILENAME FROM             
          @MY_INTERNAL_STAGE/employee1.csv.gz (file_format => my_csv_format) emp);
         
copy into EXTERNAL_S3_STAGING_EMPLOYEE(id, firstname, lastname, email, ssn, dept, contact_no, city, file_name)
    from (SELECT emp.$1, emp.$2, emp.$3, emp.$4, emp.$5, emp.$6, emp.$7,emp.$8, METADATA$FILENAME FROM
          @my_s3_stage/employee1.csv (file_format => my_csv_format) emp);
```
### Output

<img width="1280" alt="9--1" src="https://user-images.githubusercontent.com/123646244/228774782-0a7daa2b-4363-4bef-a01e-62ee120970b5.png">
<img width="1280" alt="9--2" src="https://user-images.githubusercontent.com/123646244/228774834-132aba0a-591f-4f1e-bbbc-ce60f251b7d8.png">


### 10. Upload any parquet file to the stage location and infer the schema of the file.

### Solution:

```
CREATE FILE FORMAT my_parquet_format
  TYPE = parquet;

create stage my_parquet_stage file_format = my_parquet_format;

put file:///Users/akshay/Documents/data.parquet @my_parquet_stage;

SELECT * FROM TABLE( 
        INFER_SCHEMA( 
            LOCATION=>'@my_parquet_stage',
            FILE_FORMAT=>'my_parquet_format'));
```

### Output:

<img width="1280" alt="10---1" src="https://user-images.githubusercontent.com/123646244/228774942-888b16f1-4c6f-45da-84df-6835ce05d9d2.png">

<img width="1119" alt="Screenshot 2023-03-30 at 12 24 58 AM" src="https://user-images.githubusercontent.com/122514456/228639810-0e6f941c-69fb-4258-9514-b51a1abe4e1a.png">


### 11. Run a select query on the staged parquet file without loading it to a snowflake table.

### Solution:

```
select * from @my_parquet_stage/data.parquet;
```

### Output:
<img width="1120" alt="Screenshot 2023-03-30 at 12 32 07 AM" src="https://user-images.githubusercontent.com/122514456/228641912-28d39006-6e01-4e0b-ad28-34e6e75a7afa.png">


### 12. Add masking policy to the PII columns such that fields like email, phone number, etc. show as *masked* to a user with the developer role. If the role is PII the value of these columns should be visible.

### Solution:

```
create or replace masking policy email_mask as (val string) returns string ->
    case
    when current_role() in ('ADMIN','PII') then val
    else '**masked**'
    end;

create or replace masking policy contact_Mask as (val integer) returns number ->
    case
    when current_role() in ('ADMIN','PII') then val
    else 9999999999
    end;


alter table if exists internal_staging_employee 
modify column email set masking policy email_mask;

alter table if exists external_s3_staging_employee 
modify column email set masking policy email_mask;

alter table if exists internal_staging_employee 
modify column contact_no set masking policy contact_Mask;

alter table if exists external_s3_staging_employee 
modify column contact_no set masking policy contact_Mask;

GRANT ALL PRIVILEGES ON WAREHOUSE assignment_wh TO ROLE PII;
GRANT USAGE on DATABASE assignment_db to role PII;
GRANT ALL ON ALL schemas in database ASSIGNMENT_DB TO ROLE PII; 
GRANT ALL ON ALL TABLES IN SCHEMA assignment_db.my_schema TO ROLE PII;

GRANT ALL PRIVILEGES ON WAREHOUSE assignment_wh TO ROLE DEVELOPER;
GRANT USAGE on DATABASE assignment_db to role DEVELOPER;
GRANT ALL ON ALL schemas in database ASSIGNMENT_DB TO ROLE DEVELOPER; 
GRANT ALL ON ALL TABLES IN SCHEMA assignment_db.my_schema TO ROLE DEVELOPER;

select * from internal_staging_employee limit 10;
select * from external_s3_staging_employee limit 10;
```

### Output:
#### Both tables, ASSIGNMENT_DB.MY_SCHEMA.EXTERNAL_S3_STAGING_EMPLOYEE and ASSIGNMENT_DB.MY_SCHEMA.INTERNAL_STAGING_EMPLOYEE, for PII role:
<img width="1120" alt="Screenshot 2023-03-30 at 12 56 14 AM" src="https://user-images.githubusercontent.com/122514456/228647393-a82c6d99-3ae0-4190-aa7d-5dd37bdbdf1a.png">
<img width="1120" alt="Screenshot 2023-03-30 at 1 01 13 AM" src="https://user-images.githubusercontent.com/122514456/228647772-7d09c816-4a13-4f0c-b772-3a3ab9498062.png">

#### Both tables, ASSIGNMENT_DB.MY_SCHEMA.EXTERNAL_S3_STAGING_EMPLOYEE and ASSIGNMENT_DB.MY_SCHEMA.INTERNAL_STAGING_EMPLOYEE, for DEVELOPER role:
<img width="1120" alt="Screenshot 2023-03-30 at 1 04 29 AM" src="https://user-images.githubusercontent.com/122514456/228648896-0115a810-82d7-42df-9576-34df3d053038.png">
<img width="1120" alt="Screenshot 2023-03-30 at 1 05 58 AM" src="https://user-images.githubusercontent.com/122514456/228648954-3a06fd2e-5707-4fcb-b669-6b0545e0a7df.png">
