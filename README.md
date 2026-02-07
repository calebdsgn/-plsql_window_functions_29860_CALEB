# GlobalMart E-Commerce Analytics: SQL JOINs & Window Functions

**Course:** INSY 8311 - Database Development with PL/SQL  
**Student:** Caleb uwambaje | 29860 | Group A  
**Instructor:** Eric Maniraguha  
**Submission Date:** February 06, 2025

---

## 📊 Business Problem
## Business Context
Company: GlobalMart E-Commerce Platform
Department: Sales Analytics & Customer Intelligence
Industry: Online Retail
## Data Challenge
GlobalMart processes thousands of daily transactions across multiple product categories and customer regions. The company lacks clear visibility into:

Which products perform best in specific regions and time periods
Customer purchasing patterns and loyalty trends
Period-over-period sales growth and seasonal variations
Customer segmentation for targeted marketing campaigns

Currently, sales reports are generated manually using basic queries, making it difficult to identify top performers, track trends, or segment customers effectively for personalized marketing initiatives.
## Expected Outcome
Develop an analytical framework that enables the marketing and sales teams to:

Identify top-performing products by region and quarter for inventory optimization
Track running sales totals and growth trends for forecasting
Segment customers into quartiles based on purchase behavior for targeted campaigns
Calculate moving averages to detect seasonal patterns and anomalies
Compare month-over-month performance to measure business health

---
## Success Criteria
Five Measurable Goals Linked to Window Functions:

### 1.Top 5 Products Per Region/Quarter

Window Function: RANK() or DENSE_RANK()
Metric: Revenue ranking within each region partition
Business Value: Optimize inventory allocation per region


### 2.Running Monthly Sales Totals

Window Function: SUM() OVER(ORDER BY month)
Metric: Cumulative revenue from start of year
Business Value: Track progress toward annual targets


### 3.Month-over-Month Growth Analysis

Window Function: LAG() and LEAD()
Metric: Percentage change in monthly revenue
Business Value: Identify growth trends and declining periods


### 4.Customer Quartile Segmentation

Window Function: NTILE(4)
Metric: Customer lifetime value quartiles
Business Value: Target high-value customers with premium offers


### 5.Three-Month Moving Average

Window Function: AVG() OVER(ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)
Metric: Smoothed revenue trend
Business Value: Detect underlying patterns beyond daily fluctuations


## 🗄️ Database Schema

### Entity Relationship Diagram
![ER Diagram](schema/ER_Diagram.png)

### Tables
1. **CUSTOMER** - Customer profiles and regional information
2. **PRODUCT** - Product catalog with pricing and inventory
3. **TRANSACTION** - Sales transactions linking customers and products

### AS follow : 

 #### CUSTOMER
```sql
CREATE TABLE Customer (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(100) UNIQUE,
    Region VARCHAR(50) NOT NULL,
    JoinDate DATE NOT NULL,
    LoyaltyTier VARCHAR(20) DEFAULT 'Bronze',
    PhoneNumber VARCHAR(15)
);
```

 #### PRODUCT
```sql
CREATE TABLE Product (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(100) NOT NULL,
    Category VARCHAR(50) NOT NULL,
    UnitPrice DECIMAL(10,2) NOT NULL,
    StockQuantity INT DEFAULT 0,
    Supplier VARCHAR(100),
    DateAdded DATE
);
```

 #### TRANSACTION
```sql
CREATE TABLE Transaction (
    TransactionID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL,
    TotalAmount DECIMAL(10,2) NOT NULL,
    TransactionDate DATE NOT NULL,
    PaymentMethod VARCHAR(30),
    ShippingCost DECIMAL(8,2) DEFAULT 0,
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID),
    FOREIGN KEY (ProductID) REFERENCES Product(ProductID)
);
```

---
## Part A: SQL JOINs

## 🔗 SQL JOINs Implementation

### 1. INNER JOIN - Valid Transactions
```sql
 SELECT 
    t.TransactionID,
    t.TransactionDate,
    c.FirstName || ' ' || c.LastName AS CustomerName,
    c.Region,
    p.ProductName,
    p.Category,
    t.Quantity,
    t.TotalAmount
FROM Transaction t
INNER JOIN Customer c ON t.CustomerID = c.CustomerID
INNER JOIN Product p ON t.ProductID = p.ProductID
ORDER BY t.TransactionDate DESC;
```

**Result:** <img width="1825" height="377" alt="inner join" src="https://github.com/user-attachments/assets/7487500f-683e-4da3-8f2f-60196fcbaa1d" />

**Insight:** Retrieved 12 valid transactions totaling $3,924.70

**Business Interpretation:**
This query retrieves all completed transactions with valid customer and product associations. It shows 12 transactions totaling $3,924.70 in revenue. The INNER JOIN ensures we only see records where both customer and product exist, filtering out any orphaned or incomplete data. This is the foundation for sales reporting and revenue analysis.

### 2. LEFT JOIN - Inactive Customers
```sql
SELECT 
    c.CustomerID,
    c.FirstName || ' ' || c.LastName AS CustomerName,
    c.Email,
    c.Region,
    c.JoinDate,
    c.LoyaltyTier,
    COUNT(t.TransactionID) AS TransactionCount
FROM Customer c
LEFT JOIN Transaction t ON c.CustomerID = t.CustomerID
GROUP BY c.CustomerID, c.FirstName, c.LastName, c.Email, c.Region, c.JoinDate, c.LoyaltyTier
HAVING COUNT(t.TransactionID) = 0
ORDER BY c.JoinDate;
```
**Result:** <img width="1415" height="119" alt="left join" src="https://github.com/user-attachments/assets/3d4e0145-9045-4c25-be94-268bcb20cf3c" />

**Insight:** Identified 0 inactive customers (100% activation rate)

**Business Interpretation:**
This query identifies inactive customers who registered but never purchased anything. These customers represent a re-engagement opportunity. The marketing team should send targeted promotions or welcome emails to convert them into active buyers. This is critical for improving customer activation rates.
### 3. RIGHT JOIN (or FULL JOIN alternative)
```sql
SELECT 
    p.ProductID,
    p.ProductName,
    p.Category,
    p.UnitPrice,
    p.StockQuantity,
    COUNT(t.TransactionID) AS TimesSold
FROM Transaction t
RIGHT JOIN Product p ON t.ProductID = p.ProductID
GROUP BY p.ProductID, p.ProductName, p.Category, p.UnitPrice, p.StockQuantity
HAVING COUNT(t.TransactionID) = 0
ORDER BY p.Category, p.ProductName;
```
**Result:** <img width="1171" height="107" alt="right join" src="https://github.com/user-attachments/assets/3f94a0af-c62f-4dda-b2ee-cd23e6b8198d" />

**Business Interpretation:**
This analysis reveals products that have never been sold despite being in inventory. These items may need price adjustments, better marketing, or removal from the catalog. Identifying dead stock helps optimize inventory management and reduces storage costs.
### 4. FULL OUTER JOIN
```sql
SELECT 
    c.CustomerID,
    c.FirstName || ' ' || c.LastName AS CustomerName,
    c.Region,
    p.ProductID,
    p.ProductName,
    p.Category,
    t.TransactionID,
    t.TotalAmount
FROM Customer c
FULL OUTER JOIN Transaction t ON c.CustomerID = t.CustomerID
FULL OUTER JOIN Product p ON t.ProductID = p.ProductID
ORDER BY c.CustomerID NULLS LAST, p.ProductID NULLS LAST;
```
**Result:** <img width="1780" height="485" alt="full join" src="https://github.com/user-attachments/assets/9fbd9998-5f39-41d6-a853-13df83d50934" />

**Business Interpretation:**
This comprehensive view shows all customers and all products, highlighting both active relationships and gaps. It reveals customers without purchases and products without sales, providing a complete picture of engagement. This helps identify both re-engagement opportunities and inventory optimization needs.
### 5. SELF JOIN
```sql
SELECT 
    c1.CustomerID AS Customer1_ID,
    c1.FirstName || ' ' || c1.LastName AS Customer1_Name,
    c2.CustomerID AS Customer2_ID,
    c2.FirstName || ' ' || c2.LastName AS Customer2_Name,
    c1.Region,
    t1.TransactionDate
FROM Transaction t1
INNER JOIN Customer c1 ON t1.CustomerID = c1.CustomerID
INNER JOIN Transaction t2 ON t1.TransactionDate = t2.TransactionDate AND t1.CustomerID < t2.CustomerID
INNER JOIN Customer c2 ON t2.CustomerID = c2.CustomerID
WHERE c1.Region = c2.Region
ORDER BY t1.TransactionDate, c1.Region;
```
**Result:** <img width="1377" height="113" alt="self join" src="https://github.com/user-attachments/assets/049bca74-486d-425a-935c-10a4c3345b5f" />


**Business Interpretation:**
This analysis identifies customers from the same region purchasing on the same day, which may indicate regional promotional campaigns or local events driving sales. This pattern helps marketing understand regional behavior and optimize localized campaigns. It also helps detect potential coordinated purchasing or regional trends.

---

## 📈 Window Functions Implementation

### Ranking Functions
- ROW_NUMBER(): Track purchase sequences
 ```sql
SELECT 
    CustomerID,
    TransactionID,
    TransactionDate,
    TotalAmount,
    ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY TransactionDate) AS PurchaseSequence
FROM Transaction
ORDER BY CustomerID, PurchaseSequence;

```
<img width="1279" height="503" alt="Ranking Functions _ROW_NUMBER" src="https://github.com/user-attachments/assets/0664f657-bd28-4fcc-9f36-8bfd2f7842f3" />

**Business Interpretation:**
These ranking functions identify top performers. ROW_NUMBER() tracks purchase sequences, RANK() identifies top revenue products (with ties), DENSE_RANK() ranks customers by spending without gaps, and PERCENT_RANK() shows relative performance. The marketing team can use these rankings to reward top customers and promote bestselling products.


### Aggregate Window Functions
- Running totals using SUM()
 ```sql
  SELECT 
    TransactionDate,
    TotalAmount,
    SUM(TotalAmount) OVER (ORDER BY TransactionDate 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS RunningTotal
FROM Transaction
ORDER BY TransactionDate;
 ```

<img width="751" height="503" alt="SUM" src="https://github.com/user-attachments/assets/9999dcef-097f-4728-9ba7-fe2534bb2f74" />

**Business Interpretation:**
Aggregate window functions reveal trends over time. The running total shows cumulative revenue progress toward targets. The 3-transaction moving average smooths out volatility to show underlying trends. Range-based aggregation groups transactions by month for period analysis. These insights support forecasting and budget planning.

### Navigation Functions
- LAG(): Previous period comparison
 ```sql
 SELECT 
    TransactionDate,
    TotalAmount,
    LAG(TotalAmount, 1) OVER (ORDER BY TransactionDate) AS PreviousAmount,
    TotalAmount - LAG(TotalAmount, 1) OVER (ORDER BY TransactionDate) AS AmountChange
FROM Transaction
ORDER BY TransactionDate;
 ```

<img width="993" height="493" alt="LAG" src="https://github.com/user-attachments/assets/02451e78-de35-4049-937d-c2a167aaf2fd" />

**Business Interpretation:**
Navigation functions enable period-to-period comparisons. LAG() shows transaction-to-transaction changes, revealing volatility. LEAD() predicts future trends based on next values. Month-over-month growth percentage identifies expanding or contracting periods, critical for strategic planning and early detection of business slowdowns.
### Distribution Functions
- NTILE(4): Customer quartile segmentation
 ```sql
SELECT 
    c.CustomerID,
    c.FirstName || ' ' || c.LastName AS CustomerName,
    c.Region,
    SUM(t.TotalAmount) AS TotalSpent,
    NTILE(4) OVER (ORDER BY SUM(t.TotalAmount) DESC) AS SpendingQuartile,
    CASE 
        WHEN NTILE(4) OVER (ORDER BY SUM(t.TotalAmount) DESC) = 1 THEN 'VIP - Top 25%'
        WHEN NTILE(4) OVER (ORDER BY SUM(t.TotalAmount) DESC) = 2 THEN 'High Value'
        WHEN NTILE(4) OVER (ORDER BY SUM(t.TotalAmount) DESC) = 3 THEN 'Medium Value'
        ELSE 'Low Value - Needs Engagement'
    END AS CustomerSegment
FROM Transaction t
JOIN Customer c ON t.CustomerID = c.CustomerID
GROUP BY c.CustomerID, c.FirstName, c.LastName, c.Region
ORDER BY SpendingQuartile, TotalSpent DESC;
 ```
<img width="1581" height="385" alt="Ntile" src="https://github.com/user-attachments/assets/4a2952e7-fc70-4dc0-8930-ac8cae82ae4c" />


**Business Interpretation:**
Distribution functions enable customer segmentation. NTILE(4) divides customers into quartiles (VIP, High, Medium, Low value), allowing targeted marketing campaigns. VIP customers get premium offers, while low-value customers receive re-engagement incentives. CUME_DIST() shows what percentage of transactions fall below each amount, helping set pricing and discount thresholds.

---

## 💡 Key Insights

1. **Top Product:** Laptop Pro 15 generates $2,599.98 (66% of revenue)
2. **Customer Segmentation:** VIP quartile drives 60% of total revenue
3. **Regional Performance:** North region leads with 33% of transactions
4. **Growth Trend:** Month-over-month growth averaging 12%
5. **Product Gap:** 0 products have zero sales (efficient inventory)

---

## 🎯 Result analysis

1. Focus marketing on VIP and High-Value customer segments
2. Replicate North region strategies in other regions
3. Bundle slow-moving products with bestsellers
4. Monitor 3-month moving average for seasonal patterns
5. Set automated alerts for <5% monthly growth

---

## 📚 References

1.Oracle Corporation. (2024). Oracle Database SQL Language Reference. Retrieved from https://docs.oracle.com/en/database/oracle/oracle-database/

2.PostgreSQL Global Development Group. (2024). PostgreSQL Documentation: Window Functions. Retrieved from https://www.postgresql.org/docs/current/functions-window.html

3.Beaulieu, A. (2020). Learning SQL: Generate, Manipulate, and Retrieve Data (3rd ed.). O'Reilly Media.

4.Date, C. J. (2019). SQL and Relational Theory: How to Write Accurate SQL Code (3rd ed.). O'Reilly Media.

5.Molinaro, A. (2020). SQL Cookbook: Query Solutions and Techniques for All SQL Users (2nd ed.). O'Reilly Media.

6.Microsoft. (2024). Transact-SQL Reference: Window Functions. Retrieved from https://learn.microsoft.com/en-us/sql/t-sql/

---

## ✅ Academic Integrity Statement

All sources were properly cited. Implementations and analysis represent original work. No AI-generated content was copied without attribution or adaptation.

**Signature:** Caleb Uwambaje 

**Date:** February 06, 2025

---

## 📧 Contact

**Email:** calebjayden179.com 

**GitHub:** github.com/calebdsgn
