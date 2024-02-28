#  E-Commerce Sales Analysis

### PROJECT SUMMARY

This project aims to extract actionable insights from this Global Superstore dataset and visualize the results through dashboards. This process will assist decision-makers in optimizing supply chain processes and gaining a more profound understanding of the company's performance.

### Data Source

The project utilizes datasets, namely "customers.csv," "orders.csv," "products.csv," and "returns.csv," to present a thorough analysis of sales transactions across different markets, covering a wide range of product categories and sub-categories. These datasets contain essential information, including product details (Product ID, Category, Sub-Category, and Product Name), sales metrics (Sales, Quantity, Discount, Profit), logistical details (Shipping Cost, Order Priority), order specifics (Order ID, Region), and return information (Returned, Order ID, Customer ID, Product ID.

### Tools

- Excel - Database Normalization,Data Cleaning 
- PostgreSQL - Exploratory Data Analysis
- Tableau - Creating reports [Download here](https://tableau.com)
- Python - Developed a Jupyter Notebook that Automatically import .csv file to PostgreSQL

  
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

Returns Table (1NF):
Columns:
•	Returned
•	Order ID
•	Region
•	Customer ID
•	Product ID
1.	All columns/attribute had single records and all the rows were Unique.

Customers Table (2NF):
Columns:

•	Customer ID (Primary Key)
•	Customer Name
•	Segment
•	Postal Code
•	City
•	State
•	Country
•	Region
•	Market



Products Table (2NF):
Columns:
•	Product ID (Primary Key)
•	Product Name
•	Category
•	Sub-Category

Orders Table (2NF):
Columns:
•	Order ID (Primary Key)
•	Order Date
•	Ship Date
•	Ship Mode
•	Customer ID (Foreign Key)
•	Product ID (Foreign Key)
•	Sales
•	Quantity
•	Discount
•	Profit
•	Shipping Cost
•	Order Priority

Returns Table (2NF):
Columns:
•	Returned
•	Order ID (Composite Key)
•	Region
•	Customer ID (Composite Key)
•	Product ID (Composite Key)

In 2NF, I ensured each non-prime attribute is fully functionally dependent on the entire primary key. In the Customers, Products, and Orders tables, there were no partial dependency, and all columns were fully functionally dependent on the primary key. In the Returns table, the composite key (Order ID, Customer ID, Product ID) ensures that each column is fully functionally dependent on this key, avoiding partial dependencies. The rationale for using a composite key in the Returns table was to Identify the specific order associated with each return, Identify the customer who made the order and subsequently returned a product and the specific product that was returned. Together, these three columns create a composite key that ensures each return record is uniquely identified. This is important because a single order ID or customer ID alone may not be sufficient to uniquely identify a return, especially if a customer has placed multiple orders, and some products may be part of multiple orders.
The 2NF already satisfies the 3NF requirement which ensures that there are no transitive dependencies in each table, and each non-prime attribute is directly dependent on the primary key.





