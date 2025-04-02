
# Office Project - Data Engineering Pipeline

## Project Overview
This project replicates a real-world enterprise data pipeline where data is ingested from PostgreSQL into Hadoop using Sqoop, processed using Hive & Impala, and automated with Apache Airflow. The project aims to build a scalable and maintainable ETL workflow. Future enhancements include integrating Azure Data Lake Storage (ADLS) and Databricks for advanced data processing and analytics.

## Tech Stack
- **Source Database:** PostgreSQL (running on Windows)
- **Data Ingestion:** Apache Sqoop
- **Big Data Storage:** Hadoop HDFS (running on CentOS 7 VM)
- **Querying & Processing:** Hive & Impala
- **Workflow Orchestration:** Apache Airflow
- **Querying Interface:** Hue
- **Future Enhancements:** ADLS, Databricks

## Architecture Diagram
(Placeholder for a diagram showing the data flow from PostgreSQL to Hadoop, with Hive/Impala processing and Airflow automation)

## Setup Instructions

### Prerequisites
- PostgreSQL installed and configured on Windows.
- Hadoop installed on CentOS 7 VM.
- Hive, Impala, and Hue set up on CentOS 7.
- Sqoop installed for data ingestion.
- Airflow installed for scheduling jobs.

### Step 1: Configure PostgreSQL
Ensure the PostgreSQL database is running and contains sample tables.
```sql
CREATE DATABASE metastore;
CREATE USER hive WITH PASSWORD 'yourpassword';
GRANT ALL PRIVILEGES ON DATABASE metastore TO hive;
```

### Step 2: Configure Hive Metastore with PostgreSQL
Modify `hive-site.xml` to point to PostgreSQL instead of Derby.
```xml
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:postgresql://localhost:5432/metastore</value>
</property>
```

Run schema initialization:
```bash
schematool -dbType postgres -initSchema
```

### Step 3: Sqoop Data Transfer from PostgreSQL to Hadoop
Example Sqoop command:
```bash
sqoop import \
--connect jdbc:postgresql://localhost:5432/mydb \
--username myuser --password mypassword \
--table employees \
--target-dir /user/hive/warehouse/employees \
--num-mappers 1
```

### Step 4: Create External Table in Hive
```sql
CREATE EXTERNAL TABLE employees (
    id INT,
    name STRING,
    department STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hive/warehouse/employees';
```

### Step 5: Automate with Apache Airflow
Example DAG definition:
```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

default_args = {
    'owner': 'airflow',
    'start_date': datetime(2024, 3, 10),
}

dag = DAG('sqoop_to_hive', default_args=default_args, schedule_interval='@daily')

sqoop_task = BashOperator(
    task_id='sqoop_import',
    bash_command='sqoop import --connect jdbc:postgresql://localhost:5432/mydb --username myuser --password mypassword --table employees --target-dir /user/hive/warehouse/employees --num-mappers 1',
    dag=dag
)
```

### Step 6: Query Data with Impala
```sql
SELECT * FROM employees LIMIT 10;
```

## Future Enhancements
- **Azure Data Lake Storage (ADLS):** Transitioning storage to ADLS for cloud-based scalability.
- **Databricks Integration:** Leveraging Databricks for advanced analytics and ML workflows.

## Repository Structure
```
/office_project_data_engineering
│── README.md
│── airflow/
│   ├── dags/
│   │   ├── sqoop_to_hive.py
│── hive/
│   ├── queries.sql
│── sqoop/
│   ├── sqoop_import.sh
│── docs/
│   ├── architecture_diagram.png
```

## Conclusion
This project provides hands-on experience with key Data Engineering tools and workflows. It simulates a real-world ETL pipeline that can be enhanced further with cloud technologies.
