---

# Snowflake S3 Integration and Data Loading

This file contains SQL scripts for integrating Snowflake with Amazon S3, creating an external stage, and loading data from S3 into a Snowflake table. The script supports two methods of copying data: using a pre-defined Snowflake stage or directly referencing the S3 bucket URL.

---

## **How It Works**

---

### **Step 1: Create a Storage Integration**
The `STORAGE INTEGRATION` securely connects Snowflake to the S3 bucket by leveraging an AWS IAM role with the necessary permissions.

This integration ensures that only authorized access is granted to the specified S3 bucket location.

```sql
CREATE STORAGE INTEGRATION s3_int
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = 'S3'
ENABLED = TRUE
STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::001234567890:role/myrole'
STORAGE_ALLOWED_LOCATIONS = ('s3://snowflake-datademo/load/');
```

- **`STORAGE_AWS_ROLE_ARN`**: The ARN of the AWS IAM role with permissions to access the S3 bucket.
- **`STORAGE_ALLOWED_LOCATIONS`**: Specifies the S3 bucket or prefix that Snowflake can access.

---

### **Step 2: Create a Stage**
A Snowflake stage acts as a pointer to the S3 bucket and uses the storage integration for access. Stages simplify the process of loading data by allowing users to reference the stage instead of the full S3 bucket URL.

```sql
CREATE STAGE my_s3_stage
STORAGE_INTEGRATION = s3_int
URL = 's3://snowflake-datademo/load/';
```

- **`STORAGE_INTEGRATION`**: The previously created storage integration.
- **`URL`**: The S3 bucket URL where the data files are stored.

---

### **Step 3: Copy Files from S3 to a Snowflake Table**

#### **Method A: Using the Stage**
This method references the stage (`my_s3_stage`) to load data into the Snowflake table (`mycsvtable`). The stage automatically connects to the S3 bucket specified in its configuration.

```sql
COPY INTO mycsvtable
FROM @my_s3_stage
FILE_FORMAT = (TYPE = CSV SKIP_HEADER = 1 FIELD_DELIMITER = '|');
```

---

#### **Method B: Using the S3 Bucket URL**
This method directly references the S3 bucket URL without using a stage. The `STORAGE_INTEGRATION` ensures secure access.

```sql
COPY INTO mycsvtable
FROM 's3://snowflake-datademo/load/'
STORAGE_INTEGRATION = s3_int
FILE_FORMAT = (TYPE = CSV SKIP_HEADER = 1 FIELD_DELIMITER = '|');
```

- **`FILE_FORMAT`**:
  - `TYPE = CSV`: Indicates the file format is CSV.
  - `SKIP_HEADER = 1`: Skips the first row (header) in the file.
  - `FIELD_DELIMITER = '|'`: Uses a pipe (`|`) as the column separator.

---

### **Prerequisites**

Before running this script, ensure the following:

1. **AWS S3 Bucket**
   - The bucket `s3://snowflake-datademo/load/` should exist and contain the files to be loaded.

2. **AWS IAM Role**
   - The IAM role `arn:aws:iam::001234567890:role/myrole` should have these permissions:
     - `s3:GetObject`
     - `s3:ListBucket`
     - `s3:PutObject` (if unloading data back to S3).

3. **Snowflake Permissions**
   - The user must have permissions to create storage integrations, stages, and tables.

---

### **Usage Instructions**

#### Step 1: Clone the Repository
Clone this repository to your local machine:
```bash
git clone https://github.com/your-repo-name/snowflake-s3-integration.git
```

#### Step 2: Modify the Script
Update the following placeholders in the SQL script:
- Replace `arn:aws:iam::001234567890:role/myrole` with your AWS IAM Role ARN.
- Replace `s3://snowflake-datademo/load/` with your S3 bucket URL.

#### Step 3: Execute the Script
Run the SQL script in your Snowflake worksheet or via the Snowflake CLI.

#### Step 4: Verify Data
After execution, confirm that the data has been loaded by querying the table:
```sql
SELECT * FROM mycsvtable;
```

---

### **Notes**
1. **File Format Settings**: Customize the `FILE_FORMAT` based on your data structure.
2. **Error Handling**: Use Snowflake's `COPY_HISTORY` view for troubleshooting.
3. **Data Security**: Follow the principle of least privilege when configuring IAM roles.

---