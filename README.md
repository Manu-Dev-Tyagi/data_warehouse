# Data Warehouse Learning Project - Sales Analytics

## Overview
This project demonstrates a complete data warehouse implementation for sales analytics using Databricks. It showcases the fundamental concepts of data warehousing including staging, transformation, dimensional modeling, and fact table creation using a real-world sales dataset.

## 🏗️ Project Architecture

### Data Flow
```
Source Data → Staging → Transformation → Dimensional Model → Fact Table
   ↓           ↓           ↓              ↓              ↓
sales_orders → stg_sales → trans_sales → Dimensions → FactSales
```

### Database Structure
- **Source Database**: `sales_new` - Contains raw sales order data
- **Data Warehouse**: `orderDWH` - Contains processed and modeled data

## 📊 Data Model

### Star Schema Design
This project implements a classic star schema with:
- **1 Fact Table**: `FactSales` - Contains business metrics and foreign keys
- **4 Dimension Tables**: Customers, Products, Regions, and Dates

```
                    FactSales
                       |
        ┌──────────────┼──────────────┐
        |              |              |
   dim_customers  dim_products  dim_regions
        |              |              |
   dim_dates
```

## 🗄️ Table Definitions

### Source Table: `sales_new.sales_orders`
Contains raw sales data with the following structure:
```sql
CREATE TABLE sales_new.sales_orders (
    order_id INT,
    order_date DATE,
    customer_id INT,
    customer_name STRING,
    customer_email STRING,
    product_id INT,
    product_name STRING,
    region_id INT,
    region_name STRING,
    country STRING,
    quantity INT,
    unit_price DECIMAL(10,2),
    total_amount DECIMAL(12,2)
);
```

### Staging Layer: `orderdwh.stg_sales`
- **Purpose**: Raw data copy from source system
- **Implementation**: Direct copy of source data
- **Benefits**: Preserves original data, enables reprocessing

### Transformation Layer: `orderdwh.trans_sales`
- **Purpose**: Cleaned and validated data
- **Implementation**: View with data quality filters
- **Filters**: Removes records with NULL quantities

### Dimension Tables

#### 1. `orderdwh.dim_customers`
```sql
- DimCustomers_key (Surrogate Key)
- customer_id (Business Key)
- customer_name
- customer_email
```

#### 2. `orderdwh.dim_products`
```sql
- DimProducts_key (Surrogate Key)
- product_key (Business Key)
- product_name
```

#### 3. `orderdwh.dim_regions`
```sql
- DimRegions_key (Surrogate Key)
- region_id (Business Key)
- region_name
- country
```

#### 4. `orderdwh.dim_dates`
```sql
- DimDates_key (Surrogate Key)
- order_date (Business Key)
```

### Fact Table: `FactSales`
```sql
- order_id (Business Key)
- quantity (Measure)
- unit_price (Measure)
- total (Measure)
- DimCustomers_key (Foreign Key)
- DimProducts_key (Foreign Key)
- DimDates_key (Foreign Key)
- DimRegions_key (Foreign Key)
```

## 🔄 ETL Process

### 1. Data Extraction
- Source: `sales_new.sales_orders` table
- Sample data includes 10 sales records across different regions and products

### 2. Data Staging
```sql
CREATE OR REPLACE TABLE orderdwh.stg_sales AS
SELECT * FROM sales_new.sales_orders
```

### 3. Data Transformation
```sql
CREATE OR REPLACE VIEW orderdwh.trans_sales AS
SELECT * FROM orderDWH.STG_SALES WHERE quantity IS NOT NULL
```

### 4. Dimension Loading
- **Customer Dimension**: Extracts unique customer information
- **Product Dimension**: Extracts unique product information
- **Region Dimension**: Extracts unique region and country information
- **Date Dimension**: Extracts unique order dates

### 5. Fact Table Population
```sql
INSERT INTO FactSales (
  order_id, quantity, unit_price, total,
  DimCustomers_key, DimProducts_key, DimDates_key, DimRegions_key
)
SELECT
  t.order_id, t.quantity, t.unit_price, t.total_amount,
  c.DimCustomers_key, p.DimProducts_key, d.DimDates_key, r.DimRegions_key
FROM orderdwh.trans_sales t
LEFT JOIN orderdwh.dim_customers c ON t.customer_id = c.customer_id
LEFT JOIN orderdwh.dim_products p ON t.product_id = p.product_key
LEFT JOIN orderdwh.dim_dates d ON t.order_date = d.order_date
LEFT JOIN orderdwh.dim_regions r ON t.region_id = r.region_id
```

## 📈 Sample Data Analysis

### Available Metrics
- **Sales Volume**: Quantity sold per order
- **Revenue**: Total amount per order
- **Pricing**: Unit price analysis
- **Geographic Distribution**: Sales by region and country
- **Customer Behavior**: Purchase patterns by customer
- **Product Performance**: Sales by product

### Sample Queries

#### Total Sales by Region
```sql
SELECT 
    r.region_name,
    r.country,
    SUM(f.total) as total_revenue,
    COUNT(f.order_id) as total_orders
FROM FactSales f
JOIN orderdwh.dim_regions r ON f.DimRegions_key = r.DimRegions_key
GROUP BY r.region_name, r.country
ORDER BY total_revenue DESC;
```

#### Top Customers by Revenue
```sql
SELECT 
    c.customer_name,
    c.customer_email,
    SUM(f.total) as total_spent,
    COUNT(f.order_id) as total_orders
FROM FactSales f
JOIN orderdwh.dim_customers c ON f.DimCustomers_key = c.DimCustomers_key
GROUP BY c.customer_name, c.customer_email
ORDER BY total_spent DESC;
```

## 🎯 Learning Objectives Achieved

### 1. **Data Warehouse Design**
- ✅ Implemented star schema architecture
- ✅ Created proper dimension and fact tables
- ✅ Used surrogate keys for dimensions
- ✅ Established proper foreign key relationships

### 2. **ETL Development**
- ✅ Built staging layer for data ingestion
- ✅ Implemented transformation logic
- ✅ Created dimensional model
- ✅ Populated fact table with proper joins

### 3. **Data Modeling Best Practices**
- ✅ Separated concerns (staging, transformation, presentation)
- ✅ Used appropriate data types
- ✅ Implemented data quality checks
- ✅ Created normalized dimension tables

### 4. **SQL Skills**
- ✅ Complex JOIN operations
- ✅ Window functions (ROW_NUMBER)
- ✅ Data aggregation and grouping
- ✅ View and table creation

## 🚀 Getting Started

### Prerequisites
- Access to Databricks workspace
- Basic SQL knowledge
- Understanding of relational databases

### Setup Instructions
1. **Create Source Data**: Execute the `sales_new.sales_orders` creation and insertion scripts
2. **Build Data Warehouse**: Run the database creation script
3. **Implement Staging**: Execute the staging table creation
4. **Create Transformations**: Build the transformation view
5. **Build Dimensions**: Execute dimension table creation scripts
6. **Populate Fact Table**: Run the fact table population script

### Execution Order
```sql
-- 1. Source data
CREATE DATABASE IF NOT EXISTS sales_new;
CREATE TABLE sales_new.sales_orders (...);
INSERT INTO sales_new.sales_orders VALUES (...);

-- 2. Data warehouse
CREATE DATABASE orderDWH;

-- 3. Staging
CREATE OR REPLACE TABLE orderdwh.stg_sales AS SELECT * FROM sales_new.sales_orders;

-- 4. Transformation
CREATE OR REPLACE VIEW orderdwh.trans_sales AS SELECT * FROM orderDWH.STG_SALES WHERE quantity IS NOT NULL;

-- 5. Dimensions
CREATE OR REPLACE TABLE orderdwh.dim_customers AS (...);
CREATE TABLE orderdwh.dim_products AS (...);
CREATE TABLE orderdwh.dim_regions AS (...);
CREATE OR REPLACE TABLE orderdwh.dim_dates AS (...);

-- 6. Fact table
CREATE TABLE FactSales (...);
INSERT INTO FactSales (...);
```

## 🔍 Data Quality Features

### Implemented Checks
- **NULL Validation**: Filters out records with NULL quantities
- **Data Deduplication**: Uses DISTINCT in dimension creation
- **Referential Integrity**: Proper foreign key relationships
- **Data Consistency**: Standardized data types and formats

### Monitoring Points
- Record counts at each stage
- Data completeness checks
- Join success rates
- Performance metrics

## 📊 Business Value

### Analytics Capabilities
- **Sales Performance**: Track revenue and volume trends
- **Customer Insights**: Analyze buying patterns and preferences
- **Geographic Analysis**: Understand regional performance
- **Product Analytics**: Identify top-performing products
- **Temporal Analysis**: Track sales over time

### Reporting Use Cases
- Daily/Weekly/Monthly sales reports
- Customer segmentation analysis
- Regional performance dashboards
- Product performance tracking
- Revenue forecasting

## 🧪 Hands-On Exercises

### Exercise 1: Data Exploration
```sql
-- Explore the data at each stage
SELECT COUNT(*) FROM sales_new.sales_orders;
SELECT COUNT(*) FROM orderdwh.stg_sales;
SELECT COUNT(*) FROM orderdwh.trans_sales;
```

### Exercise 2: Dimension Analysis
```sql
-- Analyze customer distribution
SELECT country, COUNT(*) as customer_count
FROM orderdwh.dim_regions
GROUP BY country;
```

### Exercise 3: Fact Table Analysis
```sql
-- Calculate total revenue
SELECT SUM(total) as total_revenue
FROM FactSales;
```

## 📈 Performance Considerations

### Optimization Techniques Used
- **Efficient Joins**: Proper indexing on business keys
- **Data Partitioning**: Consider partitioning by date for large datasets
- **View Usage**: Transformation layer as view for flexibility
- **Surrogate Keys**: Efficient integer-based relationships

### Scalability Features
- Modular design allows easy expansion
- Separate layers enable independent scaling
- View-based transformations reduce storage
- Standardized naming conventions

## 🔮 Future Enhancements

### Potential Improvements
1. **Incremental Loading**: Implement change data capture
2. **Data Lineage**: Track data flow and transformations
3. **Audit Trail**: Log all data modifications
4. **Performance Monitoring**: Add query performance tracking
5. **Data Governance**: Implement data quality rules

### Advanced Features
- **Slowly Changing Dimensions (SCD)**: Handle customer/product changes
- **Real-time Updates**: Stream data processing
- **Advanced Analytics**: Machine learning integration
- **Data Catalog**: Metadata management

## 📖 Additional Resources

### Databricks-Specific
- [Databricks Documentation](https://docs.databricks.com/)
- [Delta Lake Guide](https://docs.databricks.com/delta/index.html)
- [Databricks Academy](https://academy.databricks.com/)

### Data Warehousing Concepts
- "The Data Warehouse Toolkit" by Ralph Kimball
- "Building the Data Warehouse" by W.H. Inmon
- "Star Schema: The Complete Reference" by Christopher Adamson

## 🤝 Contributing

This project demonstrates fundamental data warehousing concepts. Feel free to:
- Extend the dimensional model
- Add more data quality checks
- Implement additional transformations
- Create new analytical views

## 📄 License

This project is for educational purposes. Use it to learn data warehousing fundamentals and build upon the concepts demonstrated.

---

**Happy Data Warehousing! 🚀**

*This project successfully demonstrates the complete data warehouse lifecycle from source data to analytical insights using modern cloud technologies and best practices.* 
