# Global Superstore Sales Analysis

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


### Exploratory Data Analysis in PostgreSQL
EDA involved exploring the Global Superstore dataset to answer Key questions.The key questions were grouped into two:

  - Primary Questions
  - Which region incurs the highest shipping costs, and how does it compare to other regions?
  - What is the average shipping cost for different product categories, and are there any significant variations?
  - How are our orders distributed among different priorities, and is there any impact on profitability?
  - What is the average profit associated with different order priorities, and are there trends or patterns?
  - What percentage of our orders are being returned, and is there a specific product category with a high return rate?
  - Which products have the highest return rates, and what impact do returns have on overall sales and profit?

1. What is the total sales amount for each category and sub-category?
2. Which product has the highest and lowest sales?
3. What is the overall profit for each region?
4. Identify the products with the highest and lowest profits.
5. Which category has the highest and lowest quantity sold?
6. What is the average quantity sold for each sub-category?
7. Analyse the impact of discounts on sales and profit.
8. Identify products with the highest and lowest discounts.
9. What is the total shipping cost for each region?
10. Calculate the average shipping cost for different categories.
11. Analyze the distribution of orders based on priority.
12. Calculate the average profit for different order priorities.
13. Determine the percentage of returned orders.
14. Identify the products with the highest return rates.
15. Compare sales and profit across different regions.
16. Identify regions with the highest and lowest sales.
17. Analyse the sales and profit trends over different years.
18. Compare the performance of different product categories
19. Identify the most popular product categories.
20. Which region incurs the highest shipping costs, and how does it compare to other regions?
21. What is the average shipping cost for different product categories, and are there any significant variations?
22. How are our orders distributed among different priorities, and is there any impact on profitability?
23. What is the average profit associated with different order priorities, and are there trends or patterns?
24. What percentage of our orders are being returned, and is there a specific product category with a high return rate?
25. Which products have the highest return rates, and what impact do returns have on overall sales and profit?

#### Data Cleaning

After my Exploratory data analysis in PostgreSQL,I saved the results in .xlsx and performed the following tasks in Excel:
1. Removig duplicates.
2. Removing Incomplete Values.
3. Coverting data to different datatypes.















