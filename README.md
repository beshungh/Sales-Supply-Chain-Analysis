# Global Superstore Sales Analysis

### PROJECT SUMMARY

This project aims to extract actionable insights from this [Global Superstore dataset](https://www.kaggle.com/datasets/endofnight17j03/global-superstore) and visualize the results through dashboards. This process will assist decision-makers in optimizing supply chain processes and gaining a more profound understanding of the company's performance.

### Data Source

The project utilizes datasets, namely "customers.csv," "orders.csv," "products.csv," and "returns.csv," to present a thorough analysis of sales transactions across different markets, covering a wide range of product categories and sub-categories. These datasets contain essential information, including product details (Product ID, Category, Sub-Category, and Product Name), sales metrics (Sales, Quantity, Discount, Profit), logistical details (Shipping Cost, Order Priority), order specifics (Order ID, Region), and return information (Returned, Order ID, Customer ID, Product ID.

### Tools Used 

- Excel - Database Normalization,Data Cleaning 
- PostgreSQL - Exploratory Data Analysis
- Tableau - Creating reports
- Python - Developed a Jupyter Notebook that Automatically import .csv files to PostgreSQL

  
### Preparation of Data

#### Database Normalization

- Converting the dataset to 1NF,2NF,3NF

To minimize redundancy, dependency and optimize performance, I normalized the dataset.I started with the 1NF which involved making sure the dataset has no Multivalued Attributes in any cell. Achieving the First Normal Form (1NF) involved ensuring that each table's column contained single record and all rows were unique. 

- Customers Table (1NF):
   - Columns: 
   - Customer ID
   - Customer Name
   - Segment
   - Postal Code
   - City
   - State
   - Country
   -	Region
   - Market

- Products Table (1NF):
    - Columns:
    - Product ID
    - Product Name
    - Category
    - Sub-Category

- Orders Table (1NF):
    - Columns:
    - Order ID
    - Order Date
    - Ship Date
    - Ship Mode
    - Customer ID
    - Product ID
    - Sales
    - Quantity
    - Discount
    - Profit
    - Shipping Cost
    - Order Priority

- Returns Table (1NF):
    - Columns:
  	- Returned
    - Order ID
    - Region
    - Customer ID
    - Product ID
#### All columns/attribute had single records and all the rows were Unique.

- Customers Table (2NF):
  - Columns:
  - Customer ID (Primary Key)
  - Customer Name
  - Segment
  - Postal Code
  - City
  - State
  - Country
  - Region
  - Market

- Products Table (2NF):
  - Columns:
  - Product ID (Primary Key)
  - Product Name
  - Category
  - Sub-Category

- Orders Table (2NF):
  - Columns:
  - Order ID (Primary Key)
  - Order Date
  - Ship Date
  - Ship Mode
  - Customer ID (Foreign Key)
  - Product ID (Foreign Key)
  - Sales
  - Quantity
  - Discount
  - Profit
  - Shipping Cost
  - Order Priority

- Returns Table (2NF):
  - Columns:
  - Returned
  - Order ID (Composite Key)
  - Region
  - Customer ID (Composite Key)
  - Product ID (Composite Key)

In 2NF, I ensured each non-prime attribute is fully functionally dependent on the entire primary key. In the Customers, Products, and Orders tables, there were no partial dependency, and all columns were fully functionally dependent on the primary key. In the Returns table, the composite key (Order ID, Customer ID, Product ID) ensures that each column is fully functionally dependent on this key, avoiding partial dependencies. The rationale for using a composite key in the Returns table was to Identify the specific order associated with each return, Identify the customer who made the order and subsequently returned a product and the specific product that was returned. Together, these three columns create a composite key that ensures each return record is uniquely identified. This is important because a single order ID or customer ID alone may not be sufficient to uniquely identify a return, especially if a customer has placed multiple orders, and some products may be part of multiple orders.The 2NF already satisfies the 3NF requirement which ensures that there are no transitive dependencies in each table, and each non-prime attribute is directly dependent on the primary key.The overall restructuring process involved transitioning the database from a Denormalized form to a Normalized form, and the finalised structure was saved as a .csv file for further use.

#### Importing the .csv Files into PostgreSQL 

Being a data analyst involves more than just creating dashboards; efficiency in task implementation is crucial. As a PostgreSQL user I face a lot challenges when importing large datasets into my database due to the manual creation of schemas and datatype conversions before the data can be imported. In difference to some other Relational Database Management System (RDBMS), where importing data is relatively straightforward, PostgreSQL present hurdles. One notable challenge is the manual creation of schemas, especially when dealing with extensive CSV files that encompass numerous columns. In PostgreSQL, each schema needs to be accurately created, and the conversion of data types to PostgreSQL-compatible formats adds an additional layer of complexity.
To overcome this, I utilised Jupyter Notebook to develop a python script using (os, shutil, NumPy, pandas and psycopg2) to automate schema creation and data import tasks, streamlining the workflow. This approach not only saved me time but also ensured accuracy, reducing errors associated with manual processes.

Features and Workflow:
1. File Organization:
The script begins by identifying all CSV files in the current directory. It then creates a new directory named 'datasets' (or uses an existing one) and moves the CSV files into this directory.
This step ensures a well-organized and centralized location for dataset management.
3. CSV to Pandas DataFrames:
The script utilizes the Pandas library to read each CSV file into a Pandas DataFrame. It handles potential encoding issues, using ISO-8859-1 encoding when necessary, showcasing attention to data integrity.
4. Table Creation and Column Transformation:
The script dynamically generates table names based on file names and cleanses column names to remove spaces, special characters, and ensure compatibility with PostgreSQL conventions.
It then iterates over each DataFrame, determining column data types and creating corresponding tables in the PostgreSQL database.
5. PostgreSQL Database Connection:
The script establishes a connection to the PostgreSQL database using user-defined parameters such as hostname, database name, username, password, and port. This ensures flexibility for connecting to various PostgreSQL instances.
6. Data Import:
The script employs the psycopg2 library to execute SQL commands for table creation and data import. It leverages PostgreSQL's COPY command, a high-performance mechanism for loading large amounts of data.
This method significantly enhances the speed of data import compared to traditional row-by-row insertion.
8. Granting Permissions:
To ensure data accessibility, the script grants SELECT permissions on the created tables to the 'public' role, providing transparency and security in data sharing.
9. Closing Connections:
The script responsibly closes the database connection after each iteration, promoting resource efficiency and preventing potential connection issues.

```python
import os
import shutil
import numpy as np
import pandas as pd
import psycopg2

# Find CSV files in the current directory
csv_files = [file for file in os.listdir(os.getcwd()) if file.endswith('.csv')]

# Create a new directory
dataset_dir = 'datasets'
try:
    os.mkdir(dataset_dir)
except FileExistsError:
    pass

# Iterate over DataFrames, clean column names, and upload to PostgreSQL
replacements = {
    'object': 'varchar',
    'float64': 'float',
    'int64': 'bigint',
    'timedelta64[ns]': 'varchar',
    'datetime64[ns]': 'timestamp',
    'bool': 'boolean',
    'datetime64': 'timestamp'
}

hostname = 'localhost'
database = 'covid19'
username = 'postgres'
password = 'password'
port_id = 5432

for file, dataframe in df.items():
    clean_table_name = file.lower().replace(" ", "_").replace("?", "") \
        .replace("-", "_").replace("/", "_").replace("\\", "_").replace("%", "") \
        .replace(")", "").replace("(", "").replace("$", "")

    table_name = clean_table_name.split('.')[0]

    dataframe.columns = [col.lower().replace(" ", "_").replace("?", "").replace("¢", "") \
                         .replace("-", "_").replace("/", "_").replace("\\", "_").replace("%", "") \
                         .replace(")", "").replace("(", "").replace("$", "").replace(".", "")
                         for col in dataframe.columns]

    col_str = ", ".join("{} {}".format(n, replacements.get(str(d), 'varchar')) for (n, d) in
                        zip(dataframe.columns, dataframe.dtypes))

    conn = psycopg2.connect(
        host=hostname,
        dbname=database,
        user=username,
        password=password,
        port=port_id
    )

    # When all the tables are imported into the database these are the output messages

Moved Customers.csv to datasets
Moved Orders.csv to datasets
Moved Products.csv to datasets
Moved Returns.csv to datasets
Using ISO-8859-1 encoding for Customers.csv
Using ISO-8859-1 encoding for Orders.csv
Using ISO-8859-1 encoding for Products.csv
Using ISO-8859-1 encoding for Returns.csv
Opened the database successfully for customers
customers was created successfully
File Customers.csv opened in memory
File Customers.csv copied to database
Opened the database successfully for orders
orders was created successfully
File Orders.csv opened in memory
File Orders.csv copied to database
Opened the database successfully for products
products was created successfully
File Products.csv opened in memory
File Products.csv copied to database
Opened the database successfully for returns
returns was created successfully
File Returns.csv opened in memory
File Returns.csv copied to database
All tables have been successfully imported into the database.


   # This section of the code is just a snippet. If you need the entire code, please get in touch with me at beshungh@gmail.com.

```

### Exploratory Data Analysis in PostgreSQL
The EDA involved exploring the Global Superstore dataset to answer Key questions.These key questions were grouped into two part:

###### Primary Questions
  - Which region incur the highest shipping costs, and how does it compare to other regions?
  - What is the average shipping cost for different product categories, and are there any significant variations?
  - How are our orders distributed among different priorities, and is there any impact on profitability?
  - What is the average profit associated with different order priorities, and are there trends or patterns?
  - What percentage of our orders are being returned, and is there a specific product category with a high return rate?
  - Which products have the highest return rates, and what impact do returns have on overall sales and profit?

###### Secondary Questions
 - What is the total sales amount for each category and sub-category?
 - Which product has the highest and lowest sales?
 - What is the overall profit for each region?
 - Identify the products with the highest and lowest profits.
 - Which category has the highest and lowest quantity sold?
 - What is the average quantity sold for each sub-category?
 - Analyse the impact of discounts on sales and profit.
 - Identify products with the highest and lowest discounts.
 - What is the total shipping cost for each region?
 - Calculate the average shipping cost for different categories.
 - Analyse the distribution of orders based on priority.
 - Calculate the average profit for different order priorities.
 - Determine the percentage of returned orders.
 - Identify the products with the highest return rates.
 - Compare sales and profit across different regions.
 - Identify regions with the highest and lowest sales.
 - Analyse the sales and profit trends over different years.
 - Compare the performance of different product categories
 - Identify the most popular product categories.

### Data Cleaning

After my Exploratory data analysis in PostgreSQL,I saved the results in .xlsx and performed the following tasks in Excel:
1. Removig duplicates.
2. Removing Incomplete Values.
3. Coverting data to different datatypes.


### Data Analysis Queries in PostgreSQL
```sql
1. -- Region that incurs the highest shipping costs

SELECT region,ROUND(SUM(shipping_cost)::numeric,2) AS total_shipping_cost
FROM customers
INNER JOIN orders
ON customers.customer_id = orders.customer_id
GROUP BY region
ORDER BY total_shipping_cost DESC
--LIMIT 1;

2. -- Average shipping cost for different product categories, and significant variations

SELECT category,ROUND(AVG(shipping_cost)::INT,2) AS Avg_shipping_cost,ROUND(STDDEV(shipping_cost)::INT,2) AS Shipping_cost_STDDEV
FROM(
SELECT order_id,products.product_id,category,shipping_cost
FROM orders
INNER JOIN products
ON orders.product_id = products.product_id)
GROUP BY category
ORDER BY avg_shipping_cost DESC,shipping_cost_stddev DESC;

3. -- Cleaning the profit column to remove non-numeric characters like '$' and ','

UPDATE orders
SET profit = REGEXP_REPLACE(profit, '[^0-9-]', '','g');

4. -- Converting Profit column datatype From Character varying to numeric

ALTER TABLE orders
ALTER COLUMN profit TYPE NUMERIC
USING profit::NUMERIC;

5. -- Cleaning the sales column to remove non-numeric characters like '$' and ','

UPDATE sales
SET sales = REGEXP_REPLACE(sales, '[^0-9-]', '','g');

6. -- Converting sales column datatype From Character varying to numeric

ALTER TABLE orders
ALTER COLUMN sales TYPE NUMERIC
USING sales::NUMERIC;

7. -- Order priority and impact on profitability

SELECT order_priority,
COUNT(*) AS order_count,
SUM(profit) AS total_profit,
ROUND(AVG(profit),2) AS average_profit,
ROUND(AVG(profit / sales),2) AS profit_margin
FROM orders
GROUP BY order_priority
ORDER BY order_priority ASC;






















```

### Results/Findings

1. ![Highest shipping cost](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/88f76f83-5d16-48ed-a384-f05ea73891fa)

- Western Europe had the highest total shipping costs, followed by Oceania and Central America.


2. ![Average shipping cost for different product categories](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/2c73be25-2430-4b80-9caa-186486a4f066)

 - Technology products have the highest average shipping cost among the categories, and there is a significant variation reflected by the high standard deviation. This may be due to factors like the fragility or higher value of technology items, leading 
   to specialized packaging and shipping requirements.

 - The average shipping cost for Furniture was relatively high, and there was a significant variation as indicated by the high standard deviation.This suggests that shipping costs for Furniture items varies widely.

 - The average shipping cost for Office Supplies were lower compared to Furniture, and there was a moderate level of variation as indicated by the standard deviation. Office Supplies might have standard shipping requirements, resulting in lower overall costs.











### Recommendations

Based on the insights derived from this analysis, The company should:

1. Consider establishing partnerships with local distributors or logistics firms in Western Europe. This collaborative approach will aim to harness their regional expertise, potentially leading to a reduction in shipping costs.
2. Investigate the factors contributing to the high variation in shipping costs for Technology products. it should explore potential partnerships or negotiated rates with carriers to reduce shipping costs without compromising on the safety of the products.
3. Consider optimizing the shipping process for Furniture items. Exploring potential partnerships with logistics companies for bulk shipping discounts or negotiating better rates based on shipping volume.
4. Continue monitoring shipping costs for Office Supplies and explore opportunities for further cost reduction. it should also consider negotiating better rates with shipping carriers based on the consistent shipping patterns of these items.

















