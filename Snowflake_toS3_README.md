
---

# Snowflake to S3 Integration and Data Unloading  

This File contains SQL scripts for integrating Snowflake with Amazon S3, setting up a file format for data unloading, creating a stage, and unloading data from Snowflake to S3.  

## Overview  

The SQL script performs the following operations:  
1. Creates a **Storage Integration** between Snowflake and S3 using an IAM Role.  
2. Defines a **File Format** for unloading data in CSV format.  
3. Sets up a **Stage** pointing to an S3 bucket for unloading data.  
4. **Unloads data** from a Snowflake table to the S3 bucket.  

---

## Script Details  

### 1. Storage Integration  
The `STORAGE INTEGRATION` establishes a secure connection between Snowflake and an S3 bucket using the IAM Role.  

```sql  
CREATE OR REPLACE STORAGE INTEGRATION snowflake_s3  
TYPE = EXTERNAL_STAGE  
STORAGE_PROVIDER = 'S3'  
ENABLED = TRUE  
STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::982534391219:role/snowflake-s3-role'  -- "Replace with your role arn"
STORAGE_ALLOWED_LOCATIONS = ('s3://remorph-reconcile-snowflake/'); -- "Replace with your bucket path"
```  

- **STORAGE_PROVIDER**: Specifies S3 as the provider.  
- **STORAGE_AWS_ROLE_ARN**: IAM role that grants access to the S3 bucket.  
- **STORAGE_ALLOWED_LOCATIONS**: Restricts access to a specific S3 bucket.  

---

### 2. File Format  
The `FILE FORMAT` defines how data will be exported.  

```sql  
CREATE OR REPLACE FILE FORMAT unloading__ff  
TYPE = "csv"  
EMPTY_FIELD_AS_NULL = TRUE  
SKIP_HEADER = 0  
COMPRESSION = NONE;  
```  

- **TYPE**: Specifies CSV as the file type.  
- **EMPTY_FIELD_AS_NULL**: Treats empty fields as NULL values.  
- **COMPRESSION**: No compression is applied.  

---

### 3. Stage  
The `STAGE` connects Snowflake to the S3 bucket for data transfer.  

```sql  
CREATE OR REPLACE STAGE unload_stage  
URL = 's3://remorph-reconcile-snowflake/'  
STORAGE_INTEGRATION = snowflake_s3  
FILE_FORMAT = unloading__ff;  
```  

- **STORAGE_INTEGRATION**: Links the stage to the S3 integration.  
- **FILE_FORMAT**: Specifies the previously defined file format.  

---

### 4. Data Unloading  
The `COPY INTO` command exports data from the Snowflake table `product_sale` to the S3 bucket.  

```sql  
COPY INTO @unload_stage/product  
FROM product_sale  
HEADER = TRUE  
OVERWRITE = TRUE;  
```  

- **HEADER**: Adds a header row to the output file.  
- **OVERWRITE**: Replaces existing files with the same name in the target location.  

---

## Prerequisites  

- **Snowflake Account** with permissions to create storage integrations and stages.  
- **AWS IAM Role** configured with access to the S3 bucket.  
- **S3 Bucket** with permissions to allow Snowflake access.  

---

## Steps to Run  

1. **Configure the IAM Role**: Ensure the `STORAGE_AWS_ROLE_ARN` is correctly set up with permissions for the S3 bucket.  
2. **Execute the Script**: Run the SQL script in the Snowflake worksheet.  
3. **Verify Data in S3**: Check the S3 bucket to ensure the data is successfully unloaded.  

---
