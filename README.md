   # **SUPPLY CHAIN ANALYSIS**

## *TABLE OF CONTENTS*

- *[Project Summary](#project-summary)*
- *[Data Source](#data-source)*
- *[Tools Used](#tools-used)*
- *[Database Normalization](#database-normalization)*
- *[Importing csv Files into PostgreSQL Using Python Script ](#importing-csv-files-into-postgresql-using-python-script)*
- *[Setbacks of the Python Script](#setbacks-of-the-python-script)*
- *[Data Cleaning](#data-cleaning)*
- *[Exploratory Data Analysis in PostgreSQL](#exploratory-data-analysis-in-postgresql)*
- *[Findings](#findings)*
- *[Recommendations](#recommendations)*
- *[Limitations](#limitations)*

### **Project Summary**
---

 This project aims to extract actionable insights from this [Global Superstore dataset](https://www.kaggle.com/datasets/endofnight17j03/global-superstore) and generate reports for stakeholders. This process will assist decision-makers in 
 optimizing supply chain processes and gaining a more profound understanding of the company's performance.

### **Data Source**

 The project utilises datasets, namely "customers.csv," "orders.csv," "products.csv," and "returns.csv," to present a thorough analysis of sales transactions across different markets, covering a wide range of product categories and sub- 
 categories. These datasets contain essential information, including product details (Product ID, Category, Sub-Category, and Product Name), sales metrics (Sales, Quantity, Discount, Profit), logistical details (Shipping Cost, Order Priority), 
 order specifics (Order ID, Region), and return information (Returned, Order ID, Customer ID, Product ID).

### **Tools Used** 

- Excel - Database Normalization,Data Cleaning 
- PostgreSQL - Exploratory Data Analysis
- Tableau - Creating reports
- Python - Developing a Jupyter Notebook to Automatically import .csv files to PostgreSQL database

### **Database Normalization**

 *Converting the dataset to 1NF,2NF,3NF*

 To minimize redundancy, dependency and optimize performance, I normalized the dataset.I started with the 1NF which involved making sure the dataset has no Multivalued Attributes in any cell. Achieving the First Normal Form (1NF) involved 
 ensuring that each table's column contained single record and all rows were unique. 

 In 2NF, I ensured each non-prime attribute is fully functionally dependent on the entire primary key. In the Customers, Products, and Orders tables, there were no partial dependency, and all columns were fully functionally dependent on the 
 primary key. In the Returns table, the composite key (Order ID, Customer ID, Product ID) ensures that each column is fully functionally dependent on this key, avoiding partial dependencies. The rationale for using a composite key in the 
 Returns table was to Identify the specific order associated with each return, Identify the customer who made the order and subsequently returned a product and the specific product that was returned. Together, these three columns create a 
 composite key that ensures each return record is uniquely identified. This is important because a single order ID or customer ID alone may not be sufficient to uniquely identify a return, especially if a customer has placed multiple orders, 
 and some products may be part of multiple orders.The 2NF already satisfies the 3NF requirement which ensures that there are no transitive dependencies in each table, and each non-prime attribute is directly dependent on the primary key.The 
 overall restructuring process involved transitioning the database from a Denormalized form to a Normalized form, and the finalized structure was saved as a .csv file for further use.

![Normalization](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/1e23b4f9-d49e-438d-b840-cdfc05a1f023)

### **Importing csv Files into PostgreSQL Using Python Script** 

 Being a data analyst involves more than just creating dashboards; efficiency in task implementation is crucial. As a PostgreSQL user I face a lot challenges when importing large datasets into my database. In difference to some other 
 Relational Database Management System (RDBMS), where importing data is relatively straightforward, PostgreSQL present hurdles. One notable challenge is the manual creation of schemas, especially when dealing with extensive CSV files that 
 encompass numerous columns. In PostgreSQL, each schema needs to be accurately created, and the conversion of data types to PostgreSQL-compatible formats adds an additional layer of complexity.
 To overcome this, I utilised Jupyter Notebook to develop a python script using Python libraries (os, shutil, NumPy, pandas and psycopg2) to automate schema creation and data import tasks, streamlining the workflow. This approach not only 
 saved me time but also ensured accuracy, reducing errors associated with manual processes.

*Features and Workflow:*
1. File Organisation:
The script begins by identifying all CSV files in the current directory. It then creates a new directory named 'datasets' (or uses an existing one) and moves the CSV files into this directory.
This step ensures a well-organised and centralised location for dataset management.
3. CSV to Pandas DataFrames:
The script utilises the Pandas library to read each CSV file into a Pandas DataFrame. It handles potential encoding issues, using ISO-8859-1 encoding when necessary, showcasing attention to data integrity.
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

# Find CSV files in my current working directory
csv_files = [file for file in os.listdir(os.getcwd()) if file.endswith('.csv')]

# Create a new directory
dataset_dir = 'datasets'
try:
    os.mkdir(dataset_dir)
except FileExistsError:
    pass

# Move the CSV files to the new directory
for csv in csv_files:
    src_path = os.path.abspath(csv)
    dest_path = os.path.join(dataset_dir, csv)
    try:
        shutil.move(src_path, dest_path)
        print(f"Moved {csv} to {dataset_dir}")
    except FileNotFoundError:
        print(f"File not found: {csv}")
    except shutil.Error as e:
        print(f"Error moving file {csv}: {e}")

# Read CSV files into Pandas DataFrames
df = {}
for file in csv_files:
    file_path = os.path.join(dataset_dir, file)
    try:
        df[file] = pd.read_csv(file_path, encoding="ISO-8859-1")
        print(f"Using ISO-8859-1 encoding for {file}")
    except UnicodeDecodeError:
        print(f"Error decoding file {file}")

# Clean table names and column names
for file, dataframe in df.items():
    clean_table_name = file.lower().replace(" ", "_").replace("?", "") \
        .replace("-", "_").replace("/", "_").replace("\\", "_").replace("%", "") \
        .replace(")", "").replace("(", "").replace("$", "")

# Remove .csv extension from clean_table_name
    table_name = clean_table_name.split('.')[0]

    dataframe.columns = [col.lower().replace(" ", "_").replace("?", "").replace("¢", "") \
                         .replace("-", "_").replace("/", "_").replace("\\", "_").replace("%", "") \
                         .replace(")", "").replace("(", "").replace("$", "").replace(".", "")
                         for col in dataframe.columns]

   # Replacement dictionary that maps pandas datatypes to PostgreSQL datatypes
    replacements = {
    'object': 'varchar',
    'float64': 'double precision',
    'float32': 'real',
    'int64': 'bigint',
    'int32': 'integer',
    'int16': 'smallint',
    'timedelta64[ns]': 'varchar',
    'timedelta64': 'interval',
    'datetime64[ns]': 'timestamp',
    'bool': 'boolean',
    'datetime64': 'timestamp'}
    
    # Table Schema
    col_str = ", ".join("{} {}".format(n, replacements.get(str(d), 'varchar')) for (n, d) in
                        zip(dataframe.columns, dataframe.dtypes))

    # Open a database connection
    hostname = 'localhost'
    database = 'globalsuperstore'
    username = 'postgres'
    password = 'password'
    port_id = 5432
    
    conn = psycopg2.connect(
        host=hostname,
        dbname=database,
        user=username,
        password=password,
        port=port_id
    )
    cursor = conn.cursor()
    print(f'Opened the database successfully for {table_name}')

    # Drop table with same name
    cursor.execute(f"DROP TABLE IF EXISTS {table_name};")
    
    # Create SQL table
    cursor.execute(f"CREATE TABLE {table_name} ({col_str});")
    print(f'{table_name} was created successfully')

    # Save dataframe to csv file
    dataframe.to_csv(file, header=dataframe.columns, index=False, encoding='utf-8')

    # Open csv file,save it as an object
    with open(file) as my_file:
        print(f'File {file} opened in memory')
        
    # Upload to database    
        cursor.copy_expert(sql=f"COPY {table_name} FROM STDIN WITH CSV HEADER DELIMITER AS ',';", file=my_file)
        print(f'File {file} copied to database')

    # Open permission for any user that access the database
    cursor.execute(f"GRANT SELECT ON TABLE {table_name} TO public;")
    conn.commit()
    cursor.close()
    conn.close()

# For loop end message
print('All tables have been successfully imported into the database.')

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
```
### **Setbacks of the Python Script**
 After successfully importing the datasets into the database, i noticed the script lacked ability to assign primary and foreign keys to tables, complicating referencing of primary keys by foreign keys in other tables and resulting in slow 
 execution of queries with JOIN statments. Additionally, it neglected the creation of indexes for the tables, causing queries with WHERE and JOIN statements to execute slowly. Constraints were also not added to tables making the database lose 
 its integrity and reliability. Although I plan to address these issues given more time, for now, I created the database manually.
 I started with designing an Entity Relationship Diagram to aid in establishing relationships between the different tables before i proceeded in the creation of the actual tables.

![ERD](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/956e6b6f-80dd-4fa4-ad10-a13f15d038ef)

### **Data Cleaning**

 *Performed the following tasks in Excel:*
1. While creating the Entity-Relationship Diagram (ERD), I encountered duplicate entries in the customer_id, product_id, and order_id columns. These duplications caused numerous errors when attempting to import the dataset into the database. 
   Since these IDs were intended as primary keys, I opted to remove them and generate new unique IDs for all tables. To do this, I employed functions such as CONCATENATE, LEFT, UPPER, and RAND(). Additionally, I utilized the Text to Columns 
   feature to separate the customer_name into first_name and last_name. This facilitated the concatenation of the initial letters of both names with the random numbers I generated. Subsequently, I employed the COUNTIF function to ensure that 
   these newly generated IDs were devoid of duplicates and remained unique

2. I eliminated currency symbols from columns containing currency values to prevent potential errors in future database queries which will require me to use the REGEXP_REPLACE function to remove non-numeric characters and CAST datatype.I did 
   this to save time.
   
```sql

CREATE TABLE products (
  product_id varchar(30) PRIMARY KEY,
  category varchar(15),
  sub_category varchar(15),
  product_name varchar(200)
);
CREATE INDEX ON products(product_id)

CREATE TABLE customers (
  customer_id varchar(30) PRIMARY KEY,
  first_name varchar(15),
  last_name varchar(15),
  segment varchar(15),
  postal_code varchar(10),
  city varchar(50),
  state varchar(50),
  country varchar(50),
  region varchar(50),
  market varchar(20)
);
CREATE INDEX ON customers(customer_id)

CREATE TABLE orders (
  order_id varchar(30) PRIMARY KEY,
  order_date date,
  ship_date date,
  ship_mode varchar(15),
  customer_id varchar(30),
  product_id varchar(30),
  sales numeric,
  quantity smallint,
  discount double precision,
  profit numeric,
  shipping_cost double precision,
  order_priority varchar(10),
  CONSTRAINT FK_orders.product_id
  FOREIGN KEY (product_id)
  REFERENCES products(product_id),
  CONSTRAINT FK_orders.customer_id
  FOREIGN KEY (customer_id)
  REFERENCES customers(customer_id)
);
CREATE INDEX ON orders(order_id)

CREATE TABLE returns (
  returned varchar(10),
  order_id varchar(30),
  region varchar(50),
  customer_id varchar(30),
  product_id varchar(30),
  CONSTRAINT FK_returns.product_id
  FOREIGN KEY (product_id)
  REFERENCES products(product_id),
  CONSTRAINT FK_returns.customer_id
  FOREIGN KEY (customer_id)
  REFERENCES customers(customer_id),
  CONSTRAINT FK_returns.order_id
  FOREIGN KEY (order_id)
  REFERENCES orders(order_id)
);
CREATE INDEX CK ON  returns (order_id, region, customer_id, product_id);
/* Due to extended query execution times, I implemented indexes on select columns to expedite query processing.
   These indexes enable the database engine to swiftly pinpoint relevant rows based on the indexed columns, notably reducing query execution times, especially within tables housing substantial datasets.
   However, I kept in mind that while indexes bolster read performance, they may modestly hinder write operations like INSERT, UPDATE, and DELETE due to index maintenance overhead.
   Consequently, I carefully balanced index creation, ensuring they are selectively employed to enhance query performance where necessary.*/
```

### **Exploratory Data Analysis in PostgreSQL**
 I chose to use an SQL database for exploring the data because the dataset was really big. When I tried to delete duplicate values in Excel, it froze because of the size of the dataset. That is why I decided to use an SQL database, as it can handle large amounts of data easily.
 
 *The EDA involved exploring the Global Superstore dataset to answer these Key questions:*

1. WHICH REGION INCURRED THE HIGHEST SHIPPING COSTS, AND HOW DOES IT COMPARE TO OTHER REGIONS?
 ```sql
 --THE QUERY AIMS TO IDENTIFY THE REGION THAT INCURRED THE HIGHEST SHIPPING COSTS
 /* The SQL query retrieves the total shipping costs for each region by combining information from the 'customers' and 'orders' tables. 
    The results are then grouped by region and ordered in descending order based on the total shipping cost. 
    If the optional --LIMIT 1; line is uncommented and executed, 
    it will return only the top result, indicating the region with the highest shipping cost.*/
SELECT region,ROUND(SUM(shipping_cost)::numeric,2) AS total_shipping_cost
FROM customers
INNER JOIN orders
ON customers.customer_id = orders.customer_id
GROUP BY region
ORDER BY total_shipping_cost DESC
--LIMIT 1;
```
 ![Region incur the highest shipping costs](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/aa573efe-06a3-4216-ac0a-64f683b3a115)

  2. WHAT IS THE AVERAGE SHIPPING COST FOR DIFFERENT PRODUCT CATEGORIES, AND ARE THERE ANY SIGNIFICANT VARIATIONS?
```sql
--AVERAGE SHIPPING COST FOR DIFFERENT PRODUCT CATEGORIES, AND SIGNIFICANT VARIATIONS
/* This SQL query calculates and presents the average shipping cost and standard deviation for different product categories.
   By joining the 'orders' and 'products' tables, the query groups the results by product category.
   The output is then sorted in descending order based on the average shipping cost and standard deviation,
   allowing for the identification of product categories with higher average shipping costs and significant variations in shipping costs.*/
SELECT category,ROUND(AVG(shipping_cost)::INT,2) AS Avg_shipping_cost,ROUND(STDDEV(shipping_cost)::INT,2) AS shipping_cost_STDDEV
FROM(
SELECT order_id,products.product_id,category,shipping_cost
FROM orders
INNER JOIN products
ON orders.product_id = products.product_id)
GROUP BY category
ORDER BY avg_shipping_cost DESC,shipping_cost_stddev DESC;
```
![What is the average shipping cost for different product categories](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/be236b96-e65b-4220-a8c0-8c72c4b6c76f)

  3. HOW ARE OUR ORDERS DISTRIBUTED AMONG DIFFERENT PRIORITIES, AND IS THERE ANY IMPACT ON PROFITABILITY?
  ```sql
  --ORDER PRIORITY AND IMPACT ON PROFITABILITY
  /* This query analyse the impact of order priority on profitability in the 'orders' table. 
     It retrieves information such as the count of orders, total profit, average profit, 
     and profit margin for each unique order priority. 
     The results are grouped by order priority and ordered in ascending order.*/
SELECT order_priority,
COUNT(*) AS order_count,
SUM(profit) AS total_profit,
ROUND(AVG(profit),2) AS average_profit,
ROUND(AVG(profit / sales),2) AS profit_margin
FROM orders
GROUP BY order_priority
ORDER BY order_priority ASC;
```
 ![How are our orders distributed among different priorities](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/cac496c6-3da0-4220-8548-b04c28be7920) 

  4. WHICH PRODUCT CATEGORIES AND SUB-CATEGORIES ARE DRIVING THE MAJORITY OF OUR PROFITS, AND HOW CAN WE OPTIMISE OUR PRODUCT MIX OR MARKETING EFFORTS TO ENHANCE PROFITABILITY IN UNDERPERFORMING CATEGORIES?
 ```sql
 --SUB_CATEGORIES DRIVING THE MAJORITY OF PROFITS
 /* This query identifies the sub-categories that contribute the most to overall profits.
    It combines information from the 'products' and 'orders' tables, summing up the profits for each unique combination of category and sub-category. 
    The results are then ordered in descending order based on the total profit,
    providing insights into which sub-categories are driving the majority of profits in the dataset.*/
SELECT category,sub_category,SUM(profit) AS total_profit
FROM products
INNER JOIN orders 
ON products.product_id = orders.product_id
GROUP BY category,sub_category
ORDER BY total_profit DESC;
```
![Which product categories and sub-categories are driving the majority of our profits](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/7bb1bdff-c774-4ca7-8059-5640ca756e49)
  
  5. WHICH PRODUCTS HAVE THE HIGHEST RETURN RATES, AND WHAT IMPACT DO RETURNS HAVE ON OVERALL SALES AND PROFIT?
```sql
--PRODUCTS WITH THE HIGHEST RETURN RATES
/* This query Joins the Returns table with the Products table using the common column Product_id to identify the products that have been returned,
   it then gives a list of products with an indication of whether they were returned or not.*/
SELECT products.product_id, products.Product_name, returns.returned 
FROM products 
JOIN returns
ON products.product_id = returns.product_id; 

/* The query calculates the return rate for each product, displaying the total returns, total sales, and return rate percentage.
   It includes information about total sales amount and total profit for each product.*/
SELECT Products.Product_id, Products.Product_name,COUNT(Returns.Returned) AS TotalReturns,
COUNT(*) AS TotalSales,ROUND((COUNT(Returns.Returned) * 100.0 / COUNT(*)),0) AS ReturnRate,
SUM(orders.Sales) AS TotalSalesAmount,
SUM(orders.Profit) AS TotalProfit
FROM Products
LEFT JOIN Returns
ON Products.Product_id = Returns.Product_id
LEFT JOIN Orders
ON Products.Product_id = orders.Product_id
GROUP BY Products.Product_id, Products.Product_name
ORDER BY ReturnRate DESC;
```
![Which products have the highest return rates](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/a5249736-9788-4fd3-9708-c69b36ecc443)

### **Findings**


1. ![Highest Shipping Costs](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/939cb413-7052-4fe4-bfec-f0530981d92c)


- Western Europe had the highest total shipping costs, followed by Central America and Oceania.

2. ![Different Product Categories](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/90009492-4e2e-4a48-88bc-286f1aef5a85)


 - Technology products had the highest average shipping cost among the categories, and there is a significant variation reflected by the high standard deviation. This may 
   be due to factors like the fragility or higher value of technology items, leading 
   to specialized packaging and shipping requirements.

 - The average shipping cost for Furniture was relatively high, and there was a significant variation as indicated by the high standard deviation.This suggests that 
   shipping costs for Furniture items varies widely.

 - The average shipping cost for Office Supplies were lower compared to Furniture, and there was a moderate level of variation as indicated by the standard deviation. 
   Office Supplies might have standard shipping requirements, resulting in lower overall costs.

3. ![Different Priorities](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/f83e1fd9-d325-4c53-9740-0961396601b5)


 - While Critical priority orders had the second lowest order count, they contributed significantly to total profit.They had the highest average profit per order at $31.59, indicating that each order of Critical priority tends to bring in a high profit.The profit margin for Critical priority orders is the highest among all priority levels at 0.06%, suggesting that a significant portion of the revenue from these orders translates into profit.
 - High priority orders had a relatively high order count, but their individual contribution to profit is lower compared to Critical priority.The average profit per order for High priority is lower than Critical at $27.12.
 - Low priority orders also had a higher order count, but their individual contribution to profit is relatively lower.The average profit per order for Low priority is the lowest among all priority levels at $24.20.
 - Medium priority orders had the highest order count among all priority levels, contributing significantly to total profit.However, the average profit per order for Medium priority is slightly lower compared to Critical priority, at $29.36.
 - Across all priority levels, the profit margins were consistent at around 0.05-0.06%, indicating a relatively stable profitability across different priority levels.

4. ![Product Categories Driving profit](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/7f69c182-c223-4e9a-842f-fc886ed2291b)


  - "Technology" and specifically "Copiers" were the top contributors to profits, followed by "Phones". These products seem to be performing exceptionally well.
  - "Furniture" as a category had mixed performance. While "Bookcases" and "Chairs" are profitable, "Tables" show a negative profit, indicating potential issues in this sub-category.
  - "Office Supplies" also contribute significantly, with "Appliances","Storage", and "Binders" being the top performers.

5. ![Products with the Highest Return Rates](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/4b113f63-b0c4-4f76-b79f-0c65a347d48b)


- Products like "Harbour Creations Executive Leather Armchair, Black" and "Novimex Executive Leather Armchair, Adjustable" had a 100% return rate.
- Products with high return rates also exhibited significant negative profit values, such as "Harbour Creations Executive Leather Armchair, Black" with a profit of -189.46 but a return rate of 100%.
  NOTE:

### **Recommendations**

Based on the insights derived from this analysis, The company should:

1. Consider establishing partnerships with local distributors or logistics firms in Western Europe. This collaborative approach will aim to harness their regional expertise, potentially leading to a reduction in shipping costs.
2. Investigate the factors contributing to the high variation in shipping costs for Technology products. it should explore potential partnerships or negotiated rates with carriers to reduce shipping costs without compromising on the safety of 
   the products.
3. Consider optimizing the shipping process for Furniture items. Exploring potential partnerships with logistics companies for bulk shipping discounts or negotiating better rates based on shipping volume.
4. Continue monitoring shipping costs for Office Supplies and explore opportunities for further cost reduction. it should also consider negotiating better rates with shipping carriers based on the consistent shipping patterns of these items.
5. Consider strategies to increase Critical priority orders as they have a higher average profit. This may involve targeted marketing campaigns, promotions, or enhancing the service for critical items to attract more customers to prioritize their orders as Critical.
6. Analyse high-priority items to identify opportunities for increasing the average profit per order. This could involve negotiating better supplier deals, optimizing shipping costs, or offering complementary products that can enhance the value of high-priority orders.
7. Explore ways to increase efficiency in handling low and medium-priority orders. While low and medium-priority orders contribute significantly to total profit due to their higher order counts, finding ways to increase efficiency in handling these orders can further improve profitability. This might include optimizing supply chain processes, reducing overhead costs associated with processing these orders, or exploring partnerships for cost-effective solutions in fulfillment and delivery.
8. Keep a close eye on profit margins and implement measures to maintain or improve them. This might involve monitoring costs, revising pricing strategies, or negotiating better terms with suppliers to improve margins without compromising on quality or service.
9. Consider expanding product offerings or enhancing marketing efforts in the "Technology" category since it has a high profitability .
10. Analyse factors contributing to the negative profit in the "Tables" sub-category. It should identify whether it's due to pricing, demand, or other issues and develop strategies to address these.
11. Focus marketing efforts on high-profit products like "Copiers" and "Phones" to maximize returns.
12. Implement stringent quality control measures for products with high return rates. Ensure that product descriptions are accurate and provide comprehensive information to set realistic customer expectations.
13. Implement a system to continuously monitor return rates and customer feedback. Regularly analyze the data to identify trends and make informed decisions about product offerings and improvements

### **Limitations** 

-  The datasets were not normalized,i had to manauelly normalize it to make my analysis accurate.
-  I had to remove all non-numeric characters from the sales,profit and discount columns because they woukd have affected my queries which will require me to clean and convert these columns before i would have gotten my results.
-  The Python script i wrote had its own setbacks which required me to manually create each table,create relationships with each table and add constraints.

*[TABLE OF CONTENTS](#table-of-contents)*
