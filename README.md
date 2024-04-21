   # **SUPPLY CHAIN ANALYSIS**

## *TABLE OF CONTENTS*

- *[Project Summary](#project-summary)*
- *[Data Source](#data-source)*
- *[Tools Used](#tools-used)*
- *[Importing csv Files into PostgreSQL Using Python Script ](#importing-csv-files-into-postgresql-using-python-script)*
- *[Setbacks of the Python Script](#setbacks-of-the-python-script)*
- *[Entity Relationship Diagram](#entity-relationship-diagram)*
- *[Creating Database Tables](#creating-database-tables)*
- *[Exploratory Data Analysis in PostgreSQL](#exploratory-data-analysis-in-postgresql)*
- *[Limitations](#limitations)*

### **Project Summary**
---

 This project aims to extract actionable insights from this [Global Superstore dataset](https://www.kaggle.com/datasets/endofnight17j03/global-superstore) and generate reports for stakeholders. This process will assist decision-makers in 
 optimizing supply chain processes and gaining a more profound understanding of the company's performance.

### **Data Source**

 The project utilises datasets, namely "customers.csv," "orders.csv," "products.csv," and "returns.csv," to present a thorough analysis of sales transactions across different markets, covering a wide range of product categories and sub- 
 categories. These datasets contain essential information, including product details (Product ID, Category, Sub-Category, and Product Name), sales metrics (Sales, Quantity, Discount, Profit), logistical details (Shipping Cost, Order Priority), 
 order specifics (Order ID, Region), and return information (Returned, Order ID, region).

### **Tools Used** 

- Excel - I cleaned currency symbols from columns containing currency values to prevent potential errors in future database queries which will require me to use the REGEXP_REPLACE function in PostgreSQL to remove non-numeric characters and CAST datatype.
  I did this to save time.
- PostgreSQL - Exploratory Data Analysis.
- Tableau - Creating reports.
- Python - Developing a Jupyter Notebook to Automatically import .csv files to PostgreSQL database.

### **Importing csv Files into PostgreSQL Using Python Script** 

 Being a data analyst involves more than just creating dashboards; efficiency in task implementation is crucial. As a PostgreSQL user I face a lot challenges when importing large datasets into my database. In difference to some other 
 Relational Database Management System (RDBMS), where importing data is relatively straightforward, PostgreSQL present hurdles. One notable challenge is the manual creation of schemas, especially when dealing with extensive CSV files that 
 encompass numerous columns. In PostgreSQL, each schema needs to be accurately created, and the conversion of data types to PostgreSQL-compatible formats adds an additional layer of complexity.
 To overcome this, I utilized Jupyter Notebook to develop a python script using Python libraries (os, shutil, NumPy, pandas and psycopg2) to automate schema creation and data import tasks, streamlining the workflow. This approach not only 
 saved me time but also ensured accuracy, reducing errors associated with manual processes.

*Features and Workflow:*
1. File Organisation:
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
    database = 'global_superstore'
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

### **Entity Relationship Diagram**

![ERD](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/219041bc-c4a3-40a5-8750-9a8ba6c7e099)

### **Creating Database Tables**

```sql

CREATE TABLE customers (
row_id INTEGER PRIMARY KEY,
customer_id VARCHAR(30),
customer_name VARCHAR(30),
city TEXT,
state VARCHAR(50),
country VARCHAR(50),
region VARCHAR(50),
market VARCHAR(20)
);
CREATE INDEX customer_id ON customers(customer_id);


CREATE TABLE products(
row_id INTEGER REFERENCES customers(row_id),
category VARCHAR(30),
sub_category VARCHAR(30),
product_id VARCHAR(30),
product_name VARCHAR(200),
sales NUMERIC,
quantity INTEGER,
discount DECIMAL,
profit NUMERIC
);
CREATE INDEX product_id ON products(product_id)


CREATE TABLE orders(
row_id INTEGER REFERENCES customers(row_id),
order_id VARCHAR(30),
order_date DATE,
shipping_date DATE,
shipping_mode VARCHAR(15),
shipping_cost NUMERIC,
order_priority TEXT
);
CREATE INDEX order_id ON orders(order_id)


CREATE TABLE returns(
row_id INTEGER REFERENCES customers(row_id),
returned TEXT,
order_id VARCHAR(30),
region VARCHAR(50)	
);
CREATE INDEX returned ON returns(returned)

/* Due to extended query execution times, I implemented indexes on selected columns to expedite query processing.
   These indexes enabled the database engine to swiftly pinpoint relevant rows based on the indexed columns, notably reducing query execution times, especially within tables housing substantial datasets.
   However, I kept in mind that while indexes bolster read performance, they might modestly hinder write operations like INSERT, UPDATE, and DELETE due to index maintenance overhead.
   Consequently, I carefully balanced index creation, ensuring they are selectively employed to enhance query performance where necessary.*/
```

### **Exploratory Data Analysis in PostgreSQL**
 Due to how huge the dataset was, I used SQL database for exploring the dataset as it can handle large amounts of data easily.
 
 *The EDA involved exploring the Global Superstore dataset to answer these Key questions:*

1. *WHICH REGION INCURRED THE HIGHEST SHIPPING COSTS, AND HOW DOES IT COMPARE TO OTHER REGIONS?*
 ```sql
 /* This query calculates the total shipping cost and the shipment cost-profit ratio for each region, 
    providing insights into the shipping expenses and their relationship with profitability. 
    The results are grouped by region and ordered based on total shipping cost in descending order.*/
SELECT region,ROUND(SUM(shipping_cost),0) AS total_shipping_cost,ROUND(SUM(shipping_cost) / SUM(profit),2) AS shipment_cost_profit_ratio
FROM customers
INNER JOIN orders 
ON customers.row_id = orders.row_id
INNER JOIN products 
ON customers.row_id = products.row_id
GROUP BY region
ORDER BY total_shipping_cost DESC;
```
![WHICH REGION INCURRED THE HIGHEST SHIPPING COSTS, AND HOW DOES IT COMPARE TO OTHER REGIONS](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/48ff995c-9fb8-4627-9a1b-12ae18ced404)

![Total Shipment Cost Profit Ratio](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/85dd6525-3bc9-489a-b278-669e469750c7)
 [Please click here for a full understanding of this report](https://public.tableau.com/views/SupplyChainAnalysis_17133848919640/TotalShipmentCostProfitRatio?:language=en-US&publish=yes&:sid=&:display_count=n&:origin=viz_share_link)

#### *Findings*
- Western Europe: With a total shipping cost of $185,841, it's our highest spender in terms of shipping. However, when we look at the ratio of shipping cost to profit, it's 0.85, indicating a relatively balanced expenditure considering the profits generated from this 
  region.

- Central America: Followed closely by Central America, which incurred a total shipping cost of $131,820. Its shipment cost-profit ratio is 0.83, also reflecting a relatively efficient shipping operation.

- Oceania: Surprisingly, Oceania comes in next with a total shipping cost of $120,802. However, it has the highest shipment cost-profit ratio among all regions, standing at 1.01. This suggests that while shipping costs are high, they are well-aligned with the profits 
  generated from this region.

- Southeastern Asia: Despite a total shipping cost of $93,879, this region stands out with a considerably high shipment cost-profit ratio of 5.26. This indicates that our shipping costs here may be disproportionately high compared to the profits generated.

- Western Asia: On the other hand, Western Asia has a total shipping cost of $34,480, but it shows a negative shipment cost-profit ratio of -0.64. This suggests that our shipping costs exceed the profits generated from this region, warranting further investigation into 
  cost optimization strategies.

#### *Recommendations*
Based on the insights gleaned from the analysis of shipping costs and profitability by region, here are some recommendations for optimizing our shipping operations:
- Evaluate the shipping strategies currently employed in each region to ensure they are aligned with profitability goals. For regions with high shipping costs relative to profits (e.g., Southeastern Asia), consider alternative s 
  shipping methods or renegotiating shipping contracts to reduce costs.

- Streamline supply chain processes to reduce shipping costs and improve overall efficiency. This could involve optimizing inventory management, minimizing shipping distances, and consolidating shipments to reduce transportation 
  expenses.

- Consider adjusting product pricing or introducing shipping surcharges for regions where shipping costs exceed profitability. This can help offset high shipping expenses and ensure that shipping operations remain financially sustainable.

- Explore partnerships with local distributors or logistics providers in high-cost regions to leverage their expertise and infrastructure. Collaborating with local partners can help reduce shipping costs and improve delivery efficiency.

- Invest in technology solutions such as route optimization software, inventory management systems, and real-time tracking tools to enhance visibility and control over shipping operations. These tools can help identify cost-saving opportunities 
  and improve overall logistics performance.

2. *WHAT IS THE AVERAGE SHIPPING COST FOR DIFFERENT PRODUCT CATEGORIES, AND ARE THERE ANY SIGNIFICANT VARIATIONS?*
```sql
/* The query starts with a CTE named shipping_costs.
   It joins the orders, customers, and products tables to gather information about the product category and corresponding 
   shipping costs.
   The ON clauses establish the relationships between the tables based on the row_id column.
   The main query selects data from the shipping_costs CTE.It calculates the average (AVG()), 
   standard deviation (STDDEV()), maximum (MAX()), and minimum (MIN()) shipping costs for each product category.
   The ROUND() function is used to round the average and standard deviation values to two decimal places.
   The results are grouped by product category using the GROUP BY clause.*/
 
WITH shipping_costs AS 
(SELECT category,shipping_cost FROM orders
JOIN customers 
ON orders.row_id = customers.row_id
JOIN products 
ON customers.row_id = products.row_id)

SELECT 
category,
ROUND(AVG(shipping_cost),2) AS average_shipping_cost,
ROUND(STDDEV(shipping_cost),2) AS standard_deviation,
MAX(shipping_cost) AS max_shipping_cost,
MIN(shipping_cost) AS min_shipping_cost
FROM shipping_costs
GROUP BY category;
```
 ![WHAT IS THE AVERAGE SHIPPING COST FOR DIFFERENT PRODUCT CATEGORIES, AND ARE THERE ANY SIGNIFICANT VARIATIONS](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/8eb1fd4e-ad3f-43fd-930f-68e0dd60af58)

 ![Average Shipping Cost For different product](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/f49319fd-3553-4d47-9809-99afef46495a)
 
 [Please click here for a full understanding of this report](https://public.tableau.com/views/AVERAGESHIPPINGCOSTFORDIFFERENTPRODUCTCATEGORIES/AverageShippingCostFordifferentproduct?:language=en-US&publish=yes&:sid=&:display_count=n&:origin=viz_share_link)

  #### *Findings*
 - For furniture items, the average shipping cost is approximately $44.68, but there's quite a bit of variability, with a standard deviation of $72.58. This means that 
   shipping costs for furniture can vary significantly. The most expensive shipping cost observed was $923.63, while the lowest was $1.04.

 - Moving on to office supplies, the average shipping cost is notably lower at $13.11. However, there's still some variability, with a standard deviation of $35.03. The 
   highest shipping cost observed for office supplies was $867.69, while the lowest was $1.002.
 
 - Lastly, for technology products, the average shipping cost is the highest among the categories at $50.01. Similar to furniture, there's a considerable amount of 
   variability, with a standard deviation of $79.01. The highest shipping cost observed for technology items was $933.57, and the lowest was $1.04.

#### *Recommendations*
Pricing Adjustments:

- Furniture: Given the higher average shipping costs and significant variability, consider adjusting pricing strategies to incorporate shipping costs effectively. This may involve either absorbing some shipping costs into product prices or offering tiered pricing 
  options based on shipping distances.

- Office Supplies: With lower average shipping costs compared to other categories, there may be opportunities to offer competitive pricing to attract more customers. Consider leveraging this advantage in marketing campaigns to emphasize affordability.

- Technology: Since technology products have the highest average shipping costs, explore strategies to optimize shipping logistics and negotiate better rates with carriers. Additionally, consider adjusting product pricing to reflect these higher shipping costs, 
  ensuring profitability while remaining competitive in the market.

Logistics Optimization:

- Furniture and Technology: Given the significant variability in shipping costs within these categories, focus on optimizing logistics to minimize costs. This could involve consolidating shipments, optimizing shipping routes, and negotiating volume discounts with 
  carriers. Implementing efficient inventory management practices can also help reduce shipping distances and associated costs.

- Office Supplies: While shipping costs for office supplies are comparatively lower, there's still room for optimization. Consider implementing efficient packaging practices to reduce dimensional weight charges and exploring alternative shipping methods for cost 
  savings.

Customer Communication:

- Clearly communicate shipping policies and costs to customers upfront to manage expectations and avoid surprises at checkout. Transparency regarding shipping costs can help build trust and loyalty among customers.

- Consider offering incentives such as free shipping thresholds or discounted shipping rates for bulk orders to incentivize larger purchases and increase customer satisfaction.

Continuous Monitoring:

- Regularly monitor shipping costs and analyse trends over time to identify opportunities for further optimization. Keep abreast of changes in carrier rates, fuel prices, and shipping regulations to adapt strategies accordingly and maintain competitiveness in the 
  market.

3. HOW ARE OUR ORDERS DISTRIBUTED AMONG DIFFERENT PRIORITIES, AND IS THERE ANY IMPACT ON PROFITABILITY?
  ```sql
  --ORDER PRIORITY AND IMPACT ON PROFITABILITY
  /* This query analyze the impact of order priority on profitability in the 'orders' and 'products table. 
     It retrieves information such as the count of orders, total profit, average profit, 
     and profit margin for each unique order priority. 
     The results are grouped by order priority and ordered in ascending order.*/

SELECT order_priority,COUNT(*) AS order_count,SUM(profit) AS total_profit,
ROUND(AVG(profit),2) AS average_profit,ROUND(AVG(profit / sales),2) AS profit_margin
FROM orders
JOIN products
ON orders.row_id = products.row_id
GROUP BY order_priority
ORDER BY order_priority ASC;
```

![Orders Priority](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/13ea50ad-ae87-437e-a95e-093da07bf3ac)

![Different Priorities](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/4b5b7b4a-62fc-4212-998a-330d86dbb92a)

[Please click here for a full understanding of this report](https://public.tableau.com/views/OrdersDistributedAmongDifferentPriorities/DifferentPriorities?:language=en-US&publish=yes&:sid=&:display_count=n&:origin=viz_share_link)

#### *Findings*
 - While Critical priority orders had the second lowest order count, they contributed significantly to total profit.They had the highest average profit per order at $31.59, indicating that each order of Critical priority tends to bring in a high profit.The profit 
   margin for Critical priority orders is the highest among all priority levels at 0.06%, suggesting that a significant portion of the revenue from these orders translates into profit.
 
 - High priority orders had a relatively high order count, but their individual contribution to profit is lower compared to Critical priority.The average profit per order for High priority is lower than Critical at $27.12.
 
 - Low priority orders also had a higher order count, but their individual contribution to profit is relatively lower.The average profit per order for Low priority is the lowest among all priority levels at $24.20.
 
 - Medium priority orders had the highest order count among all priority levels, contributing significantly to total profit.However, the average profit per order for Medium priority is slightly lower compared to Critical priority, at $29.36.

 - Across all priority levels, the profit margins were consistent at around 0.05-0.06%, indicating a relatively stable profitability across different priority levels.

#### *Recommendations*
Based on the insights derived from this analysis, The company should:
- Consider strategies to increase Critical priority orders as they have a higher average profit. This may involve targeted marketing campaigns, promotions, or enhancing the service for critical items to attract more customers to prioritize their orders as Critical.

- Analyze high-priority items to identify opportunities for increasing the average profit per order. This could involve negotiating better supplier deals, optimizing shipping costs, or offering complementary products that can enhance the value of high-priority orders.

- Explore ways to increase efficiency in handling low and medium-priority orders. While low and medium-priority orders contribute significantly to total profit due to their higher order counts, finding ways to increase efficiency in handling these orders can further 
  improve profitability. This might include optimizing supply chain processes, reducing overhead costs associated with processing these orders, or exploring partnerships for cost-effective solutions in fulfillment and delivery.

- Keep a close eye on profit margins and implement measures to maintain or improve them. This might involve monitoring costs, revising pricing strategies, or negotiating better terms with suppliers to improve margins without compromising on quality or service.

4.  WHICH PRODUCT CATEGORIES AND SUB-CATEGORIES ARE DRIVING THE MAJORITY OF OUR PROFITS, AND HOW CAN WE OPTIMISE OUR PRODUCT MIX OR MARKETING EFFORTS TO ENHANCE PROFITABILITY IN UNDERPERFORMING CATEGORIES?
 ```sql
/* This query uses Common Table Expressions (CTEs) to calculate the total profit for each product category
   and sub-category (category_profit) and ranks them based on profitability (category_rank). 
   Then, it selects the category, sub-category, total profit, 
   and categorizes them as "High Profit" or "Underperforming" based on their rank.*/

WITH category_profit AS 
(SELECT category,sub_category,SUM(profit) AS total_profit
FROM products 
GROUP BY category,sub_category),

category_rank AS 
(SELECT category,sub_category,total_profit,RANK() OVER (ORDER BY total_profit DESC) AS profit_rank
FROM category_profit)

SELECT category,sub_category,total_profit,
CASE
WHEN profit_rank <= 3 THEN 'High Profit'
ELSE 'Underperforming'
END AS profitability_status
FROM
category_rank;
```
![PRODUCT CATEGORIES AND SUB-CATEGORIES ARE DRIVING THE MAJORITY OF  PROFITS](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/81434a94-932a-49e6-84d4-7f354c3e8578)

![Products Categories and Sub-Categories Driving the Majority Profit](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/d7226af2-2d4e-48b8-bee5-0a0d861d59c7)

[Please click here for a full understanding of this report](https://public.tableau.com/views/ProductsCategoriesandSub-CategoriesDrivingtheMajorityProfit/ProductsCategoriesandSub-Categories?:language=en-US&publish=yes&:sid=&:display_count=n&:origin=viz_share_link)


#### *Findings*
- High-Profit Categories and Sub-Categories:
  
  Technology: Copiers and Phones: These categories are our shining stars in terms of profitability. We've made significant profits from sales in these areas.
  It suggests that our Copiers and Phones are in high demand or have excellent profit margins, indicating that customers are willing to pay a premium for these products.
  This is fantastic news as it shows we're doing something right here, whether it's our product quality, pricing strategy, or marketing efforts.

  Furniture: Bookcases: Bookcases are another area where we're doing exceptionally well. It seems that our Bookcases are highly sought after, contributing significantly to our overall profitability in the furniture category.
  This could be due to their design, functionality, or perhaps they're meeting a particular need in the market that our competitors aren't addressing as effectively.

- Underperforming Categories and Sub-Categories:
  
  Office Supplies: Appliances, Storage, Binders, Paper, Art, Envelopes, Supplies, Labels, Fasteners: Unfortunately, these categories are not performing as well as we'd hoped.
  While they still generate revenue, their profitability is lower compared to our high-profit categories. It's crucial for us to analyse why these categories are underperforming.
  Are there issues with product quality, pricing, or perhaps we're not effectively reaching our target audience with our marketing efforts? These are areas we need to investigate further to turn things around.

  Technology: Accessories and Machines: Within the technology category, Accessories and Machines are also not meeting our profitability expectations.
  This could indicate that either the demand for these products is lower than anticipated, or our profit margins are not as high as we'd like them to be.We need to reassess our approach to selling these products and explore ways to increase their profitability.

  Furniture: Chairs, Furnishings, Tables: Similarly, in the furniture category, Chairs, Furnishings, and Tables are also underperforming.
  It's essential for us to understand why these products are not resonating with customers as much as our high-profit products. Are there design issues, pricing concerns, or perhaps we're not effectively targeting the right audience? Addressing these questions will 
  be crucial in improving profitability in these areas.

#### *Recommendations*
- Technology (Copiers and Phones) and Furniture (Bookcases): Since these categories are driving the majority of our profits, it's essential to continue investing in them.
  We should allocate resources to further improve product quality, expand product lines, and enhance marketing efforts to maintain and potentially increase our market share in these lucrative segments.

- Office Supplies (Appliances, Storage, Binders, Paper, Art, Envelopes, Supplies, Labels, Fasteners): For these underperforming categories, we need to conduct a thorough analysis to identify the root causes of their lacklustre profitability.
  This might involve gathering customer feedback, evaluating product quality and pricing, and assessing our marketing strategies. Once we understand the reasons behind their underperformance, we can develop targeted initiatives to address these issues.
  This could include product redesign, pricing adjustments, targeted marketing campaigns, or even discontinuation of low-performing products.

- Technology (Accessories and Machines) and Furniture (Chairs, Furnishings, Tables): Similarly, we need to reassess our approach to selling products in these categories.It's essential to understand customer preferences and market trends to identify areas forimprovement.
  This might involve product innovation, pricing optimization, or refining our marketing messaging to better resonate with our target audience.

- Segmenting our customer base and personalizing our marketing efforts can help us better target our products to specific customer needs and preferences. By understanding the unique requirements of different customer segments,
  we can tailor our product offerings and promotions to maximize profitability. 

- Profitability is not static, and market dynamics can change rapidly. Therefore, it's crucial to continuously monitor sales performance, customer feedback, and market trends. Regularly evaluating our product mix and making adjustments in response to changing market 
  conditions will be essential to maintaining and improving profitability over the long term.

- Leveraging data analytics tools and techniques can provide valuable insights into customer behaviour, product performance, and market trends. By investing in advanced analytics capabilities, we can make more informed decisions and develop targeted strategies to 
  optimize profitability across all product categories.

5.  WHICH PRODUCTS HAVE THE HIGHEST RETURN RATES, AND WHAT IMPACT DO RETURNS HAVE ON OVERALL SALES AND PROFIT?

    I faced a challenge in accurately quantifying the returned items, as they were recorded simply as 'YES' without any specific numerical values attached to them in the dataset.
    To fix this, I wrote an SQL script that first counted all the 'YES' entries in the returned column, associated them with the corresponding order_ids, and then linked those order_ids to the particular product names.
    This helped me determine the quantitative form of orders that were returned for each product.

 ![returns_sample](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/163d2197-3f75-4665-95b4-f24d67f38a36)
 
```sql
-- Quantifying the returned items

WITH returncounts AS (
SELECT order_id,COUNT(*) AS return_count
FROM returns 
WHERE returned = 'Yes'
GROUP BY order_id
),

productreturns AS (
SELECT product_id,product_name,return_count
FROM returncounts
JOIN orders 
ON returncounts.order_id = orders.order_id
JOIN products 
ON orders.row_id = products.row_id
)

SELECT product_id,product_name,COALESCE(SUM(return_count), 0) AS total_return_count
FROM productreturns
GROUP BY product_id, product_name
ORDER BY total_return_count DESC;
```
![Quantitative form](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/e8f9c238-2b48-4cb1-af9c-f7ca0cf792b6)

  I then used the quantitative form of the result to find the products with the hightest return rate and the overall impact on sales and profit

```sql
/* This SQL query first calculates the count of returned orders for each product, 
   associating them with their respective product names and IDs, and then determines the total sales and profit for each product. 
   It further calculates the return rate as the percentage of returned items compared to total sales, rounding it to two decimal places. */

WITH returncounts AS (
SELECT orders.row_id,COUNT(*) AS return_count
FROM returns 
JOIN orders
ON returns.order_id = orders.order_id
WHERE returns.returned = 'Yes'
GROUP BY orders.row_id
),

productreturns AS (
SELECT products.product_id,products.product_name,COALESCE(returncounts.return_count, 0) AS return_count
FROM products
LEFT JOIN returncounts
ON products.row_id = returncounts.row_id
),

totalsalesprofit AS (
SELECT product_id,SUM(sales) AS total_sales,SUM(profit) AS total_profit
FROM products
GROUP BY product_id
)

SELECT pr.product_id,pr.product_name,SUM(pr.return_count) AS total_return_count,
COALESCE(ts.total_sales, 0) AS total_sales,COALESCE(ts.total_profit, 0) AS total_profit,
ROUND((SUM(pr.return_count)::NUMERIC / NULLIF(COALESCE(ts.total_sales, 0), 0) * 100)::NUMERIC,2) AS return_rate
FROM ProductReturns AS pr
LEFT JOIN totalsalesprofit AS ts 
ON pr.product_id = ts.product_id
GROUP BY pr.product_id, pr.product_name, ts.total_sales, ts.total_profit
ORDER BY total_return_count DESC
LIMIT 25;
```
![product_returned](https://github.com/beshungh/Sales-Supply-Chain-Analysis/assets/135900689/d52e80f7-a9ad-405a-b818-86938b5b2e3e)

#### *Findings*
 The product return analysis unveils a spectrum of return counts and rates across various products. Notably, Staples stands out with 12 returns and a 17% return rate, yet maintaining robust sales of $7,008.19 and profit of $2,611.04. 
 Conversely, Smead File Cart, Single Width, and Smead File Cart, Blue, exhibit 9 and 6 returns respectively, alongside meager return rates of 4% and 5%, coupled with negative profit margins of -$237.69 and -$1,381.01. 
 Additionally, products like Tenex Folders, Single Width, reveal 6 returns, with a noteworthy 25% return rate, albeit generating only modest profit of $123.60, indicating potential avenues for enhancing customer satisfaction.

#### *Recommendations*
Based on the findings of the product return analysis, here are some recommendations:
-  Conduct a thorough investigation into the reasons behind the high return rates for specific products, such as Staples and Smead File Cart, to identify any underlying issues with product quality, design, or customer satisfaction.

-  Collaborate with product development teams to implement improvements or modifications to products with high return rates, focusing on addressing customer complaints or enhancing product features to meet market expectations.

-  trengthen quality control processes throughout the product lifecycle, including manufacturing, packaging, and shipping, to minimize defects and ensure products meet customer expectations upon delivery.

-  Analyse customer feedback and reviews associated with products exhibiting high return rates to gain insights into specific pain points or areas for improvement, guiding product development and marketing strategies.

-  Enhance customer support services to streamline the return process and provide timely assistance to customers experiencing issues with products, aiming to improve overall satisfaction and reduce return rates.

-  Optimize inventory management practices to mitigate the impact of returns on overall profitability, including monitoring stock levels, forecasting demand accurately, and implementing efficient return handling procedures.

-  Establish a system for ongoing monitoring and evaluation of product return data, allowing for real-time insights into emerging trends or issues, and enabling proactive interventions to address them effectively.














### **Limitations** 

-  I had to remove all non-numeric characters from the sales,profit and discount columns because they would have affected my queries which will require me to clean and convert these columns before i would have gotten my results.
-  The Python script i wrote had its own setbacks which required me to manually create each table,create relationships with each table and add constraints.

*[TABLE OF CONTENTS](#table-of-contents)*
