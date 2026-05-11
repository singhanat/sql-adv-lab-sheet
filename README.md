# Advanced Data Analysis and Query Optimization Using SQL

**แบบฝึกหัด จำนวน 30 ข้อ**
- Section I — Advanced Joins & Set Operations (Q1–Q8)
- Section II — Advanced Subqueries (Q9–Q14)
- Section III — Performance Tuning & Query Rewriting (Q15–Q17)
- Section IV — Common Table Expressions (CTEs) (Q18–Q21)
- Section V — Window Functions (Q22–Q30)

## Data set
![ER](img/ER.png)

---
## Section I — Advanced Joins & Set Operations (Q1–Q8)
---

### Q1 : พนักงานและหัวหน้าตัวเอง (SELF JOIN แบบ equi)
**Topic:** `SELF JOIN`

**Scenario:**
HR ต้องการรายงานแสดงชื่อพนักงานคู่กับชื่อ Manager ของแต่ละคน โดยใช้ SELF JOIN บน employees table ที่มี ReportsTo อ้างอิงถึง EmployeeID ตัวเอง

**Task:**
แสดง EmployeeName, ManagerName เรียงตาม EmployeeName (เฉพาะพนักงานที่มีหัวหน้า)

**Sample Data:**

*Table: `employees`*

| EmployeeID | FirstName | LastName | ReportsTo |
| --- | --- | --- | --- |
| 1 | Nancy | Davolio | 2 |
| 2 | Andrew | Fuller |  |
| 3 | Janet | Leverling | 2 |
| 5 | Steven | Buchanan | 2 |
| 6 | Michael | Suyama | 5 |

**Expected Output:**

| EmployeeName | ManagerName |
| --- | --- |
| Janet Leverling | Andrew Fuller |
| Michael Suyama | Steven Buchanan |
| Nancy Davolio | Andrew Fuller |
| Steven Buchanan | Andrew Fuller |

<details>
<summary>Solution</summary>

```sql
SELECT CONCAT(e.first_name, ' ', e.last_name) AS employee_name,
       CONCAT(m.first_name, ' ', m.last_name) AS manager_name
FROM employees e
JOIN employees m ON e.reports_to = m.employee_id
ORDER BY employee_name;
```

</details>

---

### Q2 : หาคู่สินค้าที่ราคาห่างกันไม่เกิน 5 บาท (ในหมวดเดียวกัน)
**Topic:** `NON-EQUI SELF JOIN`

**Scenario:**
ทีม Pricing ต้องการหาคู่สินค้าที่อยู่ใน Category เดียวกันและมีราคาใกล้เคียงกัน (ต่างกันไม่เกิน 5) เพื่อวิเคราะห์ competitive pricing โดยไม่ให้จับคู่กับตัวเอง และให้ ProductID ของ a น้อยกว่า b เสมอ

**Task:**
แสดง ProductA, ProductB, PriceA, PriceB, PriceDiff เรียงตาม PriceDiff

**Sample Data:**

*Table: `products (sample)`*

| ProductID | ProductName | CategoryID | UnitPrice |
| --- | --- | --- | --- |
| 1 | Chai | 1 | 18.00 |
| 2 | Chang | 1 | 19.00 |
| 24 | Guaraná Fantástica | 1 | 4.50 |
| 35 | Steeleye Stout | 1 | 18.00 |
| 38 | Côte de Blaye | 1 | 263.50 |

**Expected Output:**

| ProductA | ProductB | PriceA | PriceB | PriceDiff |
| --- | --- | --- | --- | --- |
| Chai | Chang | 18.00 | 19.00 | 1.00 |
| Chai | Steeleye Stout | 18.00 | 18.00 | 0.00 |
| Chang | Steeleye Stout | 19.00 | 18.00 | 1.00 |

<details>
<summary>Solution</summary>

```sql
SELECT a.product_name AS product_a,
       b.product_name AS product_b,
       a.unit_price AS price_a,
       b.unit_price AS price_b,
       ROUND(CAST(ABS(a.unit_price - b.unit_price) AS DECIMAL(10,2)), 2) AS price_diff
FROM products a
JOIN products b ON a.category_id = b.category_id
               AND a.product_id < b.product_id
WHERE ABS(a.unit_price - b.unit_price) <= 5
ORDER BY price_diff;
```

</details>

---

### Q3 : รายละเอียด Order พร้อมลูกค้าและพนักงาน (3-table JOIN)
**Topic:** `MULTIPLE JOINS: JOIN & JOIN`

**Scenario:**
ทีม Sales ต้องการ report ที่รวมข้อมูล Order, ชื่อลูกค้า และชื่อพนักงานที่รับออเดอร์ไว้ในตารางเดียว เฉพาะปี 1997

**Task:**
แสดง OrderID, CustomerName, EmployeeName, OrderDate เรียงตาม OrderDate (เฉพาะพนักงานที่มีคำสั่งซื้อเท่านั้น ใช้ INNER JOIN ทุกตาราง)

**Sample Data:**

*Table: `orders`*

| order_id | customer_id | employee_id | order_date |
| --- | --- | --- | --- |
| 10400 | EASTC | 1 | 1997-01-01 |
| 10401 | HANAR | 1 | 1997-01-01 |
| 10402 | ERNSH | 8 | 1997-01-02 |

*Table: `customers`*

| customer_id | company_name |
| --- | --- |
| EASTC | Eastern Connection |
| HANAR | Hanari Carnes |
| ERNSH | Ernst Handel |

*Table: `employees`*

| employee_id | first_name | last_name |
| --- | --- | --- |
| 1 | Nancy | Davolio |
| 8 | Laura | Callahan |

**Expected Output:**

| order_id | customer_name | employee_name | order_date |
| --- | --- | --- | --- |
| 10400 | Eastern Connection | Nancy Davolio | 1997-01-01 |
| 10401 | Hanari Carnes | Nancy Davolio | 1997-01-01 |
| 10402 | Ernst Handel | Laura Callahan | 1997-01-02 |

<details>
<summary>Solution</summary>

```sql
SELECT o.order_id,
       c.company_name AS customer_name,
       CONCAT(e.first_name, ' ', e.last_name) AS employee_name,
       o.order_date
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN employees e ON o.employee_id = e.employee_id
WHERE EXTRACT(YEAR FROM o.order_date) = 1997
ORDER BY o.order_date;
```

</details>

---

### Q4 : สินค้าทุกชิ้นพร้อมชื่อ Category (บางชิ้นอาจไม่มี Category)
**Topic:** `MULTIPLE JOINS: JOIN & LEFT JOIN`

**Scenario:**
ทีม Data ต้องการ list สินค้าทุกชิ้น แม้บางชิ้นยังไม่ได้จัด Category โดยให้ JOIN กับ order_details เพื่อดูว่าเคยขายแล้วหรือยัง และ LEFT JOIN กับ categories เพื่อดูชื่อหมวด

**Task:**
แสดง ProductName, CategoryName (NULL = 'Uncategorized'), OrderCount เรียงตาม OrderCount DESC

**Sample Data:**

*Table: `products`*

| product_id | product_name | category_id |
| --- | --- | --- |
| 1 | Chai | 1 |
| 2 | Chang | 1 |
| 3 | Aniseed Syrup | 2 |
| 99 | Mystery Item |  |

*Table: `categories`*

| category_id | category_name |
| --- | --- |
| 1 | Beverages |
| 2 | Condiments |

*Table: `order_details`*

| order_id | product_id |
| --- | --- |
| 10248 | 1 |
| 10249 | 1 |
| 10250 | 2 |
| 10251 | 3 |

**Expected Output:**

| product_name | category_name | order_count |
| --- | --- | --- |
| Chai | Beverages | 2 |
| Aniseed Syrup | Condiments | 1 |
| Chang | Beverages | 1 |
| Mystery Item | Uncategorized | 0 |

<details>
<summary>Solution</summary>

```sql
SELECT p.product_name,
       COALESCE(c.category_name, 'Uncategorized') AS category_name,
       COUNT(od.order_id) AS order_count
FROM products p
LEFT JOIN categories c ON p.category_id = c.category_id
LEFT JOIN order_details od ON p.product_id = od.product_id
GROUP BY p.product_name, c.category_name
ORDER BY order_count DESC;
```

</details>

---

### Q5 : Order ที่ Freight สูงกว่าค่าเฉลี่ย Freight ของ Employee คนนั้น
**Topic:** `MULTIPLE JOINS: Non-equi JOIN condition`

**Scenario:**
ต้องการหา Order ที่มี Freight สูงกว่าค่าเฉลี่ย Freight ของพนักงานคนเดียวกัน โดยใช้ JOIN กับ derived table และ non-equi condition

**Task:**
แสดง OrderID, EmployeeID, Freight, AvgFreight (ทศนิยม 2) เรียงตาม Freight DESC

**Sample Data:**

*Table: `orders (sample)`*

| order_id | employee_id | freight |
| --- | --- | --- |
| 10248 | 1 | 32.38 |
| 10249 | 1 | 11.61 |
| 10250 | 1 | 65.83 |
| 10251 | 2 | 41.34 |
| 10252 | 2 | 51.30 |

**Expected Output:**

| order_id | employee_id | freight | avg_freight |
| --- | --- | --- | --- |
| 10250 | 1 | 65.83 | 36.61 |
| 10252 | 2 | 51.30 | 46.32 |

<details>
<summary>Solution</summary>

```sql
SELECT o.order_id, o.employee_id, o.freight,
       ROUND(CAST(avg_e.avg_freight AS DECIMAL(10,2)), 2) AS avg_freight
FROM orders o
JOIN (
    SELECT employee_id, AVG(freight) AS avg_freight
    FROM orders
    GROUP BY employee_id
) avg_e ON o.employee_id = avg_e.employee_id
        AND o.freight > avg_e.avg_freight
ORDER BY o.freight DESC;
```

</details>

---

### Q6 : เมืองที่มีทั้ง Customer และ Supplier (INTERSECT)
**Topic:** `UNION / INTERSECT / EXCEPT`

**Scenario:**
ทีม Logistics ต้องการรู้ว่ามีเมืองใดบ้างที่มีทั้งลูกค้าและ Supplier อยู่ด้วยกัน เพื่อวางแผนจุดรับ-ส่งสินค้า

**Task:**
แสดงรายชื่อ City ที่ปรากฏในทั้ง customers และ suppliers เรียง A-Z

**Sample Data:**

*Table: `customers (sample)`*

| customer_id | city |
| --- | --- |
| ALFKI | Berlin |
| ANATR | Mexico City |
| AROUT | London |

*Table: `suppliers (sample)`*

| supplier_id | city |
| --- | --- |
| 1 | London |
| 2 | New Orleans |
| 3 | Berlin |

**Expected Output:**

| city |
| --- |
| Berlin |
| London |

<details>
<summary>Solution</summary>

```sql
SELECT city FROM customers
INTERSECT
SELECT city FROM suppliers
ORDER BY city;
```

</details>

---

### Q7 · ประเทศที่มี Customer แต่ไม่มี Supplier
**Topic:** `EXCEPT`

**Scenario:**
ทีม Procurement ต้องการรู้ว่าประเทศใดมีลูกค้าแต่ยังไม่มี Supplier อยู่ เพื่อวางแผนหา Supplier ใหม่

**Task:**
แสดงรายชื่อ Country ที่อยู่ใน customers แต่ไม่อยู่ใน suppliers เรียง A-Z

**Sample Data:**

*Table: `customers (sample)`*

| customer_id | country |
| --- | --- |
| ALFKI | Germany |
| ANATR | Mexico |
| AROUT | UK |
| BERGS | Sweden |

*Table: `suppliers (sample)`*

| supplier_id | country |
| --- | --- |
| 1 | UK |
| 2 | USA |
| 3 | Germany |

**Expected Output:**

| country |
| --- |
| Mexico |
| Sweden |

<details>
<summary>Solution</summary>

```sql
SELECT country FROM customers
EXCEPT
SELECT country FROM suppliers
ORDER BY country;
```

</details>

---

### Q8 · รวม log การสั่งซื้อจาก 2 ปี (UNION ALL รักษา duplicate)
**Section:** I &nbsp;|&nbsp; **Topic:** `UNION ALL`

**Scenario:**
ทีม Data Eng ต้องการ combine คำสั่งซื้อจากปี 1996 และ 1997 ไว้ในชุดเดียวกัน โดยเก็บซ้ำไว้หมด (ไม่ตัด) และเพิ่ม column Year บอกที่มา

**Task:**
แสดง OrderID, CustomerID, OrderDate, Source ('1996' หรือ '1997') เรียงตาม OrderDate

**Sample Data:**

*Table: `orders (sample)`*

| order_id | customer_id | order_date |
| --- | --- | --- |
| 10248 | VINET | 1996-07-04 |
| 10249 | TOMSP | 1996-07-05 |
| 10400 | EASTC | 1997-01-01 |
| 10401 | HANAR | 1997-01-01 |

**Expected Output:**

| order_id | customer_id | order_date | source |
| --- | --- | --- | --- |
| 10248 | VINET | 1996-07-04 | 1996 |
| 10249 | TOMSP | 1996-07-05 | 1996 |
| 10400 | EASTC | 1997-01-01 | 1997 |
| 10401 | HANAR | 1997-01-01 | 1997 |

<details>
<summary>Solution</summary>

```sql
SELECT order_id, customer_id, order_date, '1996' AS source
FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 1996
UNION ALL
SELECT order_id, customer_id, order_date, '1997' AS source
FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 1997
ORDER BY order_date;
```

</details>

---

## Section II — Advanced Subqueries (Q9–Q14)

---

### Q9 : ลูกค้าที่มีคำสั่งซื้ออย่างน้อย 1 รายการ (EXISTS)
**Topic:** `EXISTS`

**Scenario:**
ทีม CRM ต้องการดึงเฉพาะลูกค้าที่เคยสั่งซื้อจริงอย่างน้อย 1 ครั้ง โดยใช้ EXISTS ซึ่งเร็วกว่า IN สำหรับ subquery ขนาดใหญ่

**Task:**
แสดง CustomerID, CompanyName เรียง CompanyName

**Sample Data:**

*Table: `customers`*

| customer_id | company_name |
| --- | --- |
| ALFKI | Alfreds Futterkiste |
| ANATR | Ana Trujillo |
| BOLID | Bólido Comidas |

*Table: `orders`*

| order_id | customer_id |
| --- | --- |
| 10248 | ALFKI |
| 10249 | ALFKI |
| 10250 | ANATR |

**Expected Output:**

| customer_id | company_name |
| --- | --- |
| ALFKI | Alfreds Futterkiste |
| ANATR | Ana Trujillo |

<details>
<summary>Solution</summary>

```sql
SELECT c.customer_id, c.company_name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id
)
ORDER BY c.company_name;
```

</details>

---

### Q10 : ลูกค้าที่ไม่เคยสั่งซื้อเลย (NOT EXISTS)
**Topic:** `NOT EXISTS`

**Scenario:**
ทีม Sales ต้องการ list ลูกค้าที่ยังไม่เคยมี Order เพื่อทำ re-engagement campaign

**Task:**
แสดง CustomerID, CompanyName, Country เรียง CompanyName

**Sample Data:**

*Table: `customers`*

| customer_id | company_name | country |
| --- | --- | --- |
| ALFKI | Alfreds Futterkiste | Germany |
| ANATR | Ana Trujillo | Mexico |
| BOLID | Bólido Comidas | Spain |

*Table: `orders`*

| order_id | customer_id |
| --- | --- |
| 10248 | ALFKI |
| 10249 | ANATR |

**Expected Output:**

| customer_id | company_name | country |
| --- | --- | --- |
| BOLID | Bólido Comidas | Spain |

<details>
<summary>Solution</summary>

```sql
SELECT c.customer_id, c.company_name, c.country
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id
)
ORDER BY c.company_name;
```

</details>

---

### Q11 : สินค้าที่แพงกว่าทุกชิ้นใน Category Seafood (ALL)
**Topic:** `ALL`

**Scenario:**
ทีม Pricing ต้องการหาสินค้าที่มีราคาสูงกว่าทุกสินค้าใน Category Seafood เพื่อระบุ premium products

**Task:**
แสดง ProductName, UnitPrice, CategoryID เรียงตาม UnitPrice DESC

**Sample Data:**

*Table: `products (sample)`*

| product_id | product_name | category_id | unit_price |
| --- | --- | --- | --- |
| 10 | Ikura | 8 | 31.00 |
| 13 | Konbu | 8 | 6.00 |
| 18 | Carnarvon Tigers | 8 | 62.50 |
| 38 | Côte de Blaye | 1 | 263.50 |
| 29 | Thüringer Rostbratwurst | 6 | 123.79 |

*Table: `categories (sample)`*

| category_id | category_name |
| --- | --- |
| 1 | Beverages |
| 6 | Meat/Poultry |
| 8 | Seafood |

**Expected Output:**

| product_name | unit_price | category_id |
| --- | --- | --- |
| Côte de Blaye | 263.50 | 1 |
| Thüringer Rostbratwurst | 123.79 | 6 |

<details>
<summary>Solution</summary>

```sql
SELECT product_name, unit_price, category_id
FROM products
WHERE unit_price > ALL (
    SELECT unit_price FROM products
    WHERE category_id = (
        SELECT category_id FROM categories
        WHERE category_name = 'Seafood'
    )
)
ORDER BY unit_price DESC;
```

</details>

---

### Q12 : พนักงานที่มี Freight Order อย่างน้อย 1 รายการสูงกว่า 100 (ANY)
**Topic:** `ANY / SOME`

**Scenario:**
ทีม Ops ต้องการหาพนักงานที่เคยรับออเดอร์ที่มี Freight สูงกว่า 100 อย่างน้อย 1 ครั้ง

**Task:**
แสดง EmployeeID, FullName เรียงตาม EmployeeID

**Sample Data:**

*Table: `employees`*

| employee_id | first_name | last_name |
| --- | --- | --- |
| 1 | Nancy | Davolio |
| 2 | Andrew | Fuller |
| 3 | Janet | Leverling |

*Table: `orders`*

| order_id | employee_id | freight |
| --- | --- | --- |
| 10248 | 1 | 32.38 |
| 10249 | 1 | 11.61 |
| 10250 | 2 | 65.83 |
| 10251 | 2 | 151.30 |
| 10252 | 3 | 41.34 |

**Expected Output:**

| employee_id | full_name |
| --- | --- |
| 2 | Andrew Fuller |

<details>
<summary>Solution</summary>

```sql
SELECT e.employee_id,
       CONCAT(e.first_name, ' ', e.last_name) AS full_name
FROM employees e
WHERE e.employee_id = ANY (
    SELECT employee_id FROM orders
    WHERE freight > 100
)
ORDER BY e.employee_id;
```

</details>

---

### Q13 : สินค้าที่ราคาสูงกว่าค่าเฉลี่ยของ Category ตัวเอง (Derived Table)
**Topic:** `Derived Table (Subquery in FROM)`

**Scenario:**
ต้องการหาสินค้าที่ราคาสูงกว่าค่าเฉลี่ยของ Category ตัวเอง โดยใช้ Derived Table (subquery ใน FROM) แทน correlated subquery เพื่อ performance ที่ดีกว่า

**Task:**
แสดง ProductName, CategoryName, UnitPrice, AvgCategoryPrice (ทศนิยม 2) เรียงตาม CategoryName, UnitPrice DESC

**Sample Data:**

*Table: `categories`*

| category_id | category_name |
| --- | --- |
| 1 | Beverages |
| 2 | Condiments |

*Table: `products`*

| product_id | product_name | category_id | unit_price |
| --- | --- | --- | --- |
| 1 | Chai | 1 | 18.00 |
| 2 | Chang | 1 | 19.00 |
| 38 | Côte de Blaye | 1 | 263.50 |
| 3 | Aniseed Syrup | 2 | 10.00 |
| 4 | Chef Anton's | 2 | 22.00 |

**Expected Output:**

| product_name | category_name | unit_price | avg_category_price |
| --- | --- | --- | --- |
| Côte de Blaye | Beverages | 263.50 | 100.17 |
| Chef Anton's | Condiments | 22.00 | 16.00 |

<details>
<summary>Solution</summary>

```sql
SELECT p.product_name, c.category_name, p.unit_price,
       ROUND(CAST(ca.avg_price AS DECIMAL(10,2)), 2) AS avg_category_price
FROM products p
JOIN categories c ON p.category_id = c.category_id
JOIN (
    SELECT category_id, AVG(unit_price) AS avg_price
    FROM products
    GROUP BY category_id
) ca ON p.category_id = ca.category_id
     AND p.unit_price > ca.avg_price
ORDER BY c.category_name, p.unit_price DESC;
```

</details>

---

### Q14 : เปรียบเทียบ: หาพนักงานที่มียอดสูงกว่าค่าเฉลี่ย (Rewrite correlated → JOIN)
**Topic:** `Correlated Subquery vs EXISTS Performance`

**Scenario:**
Query เดิมใช้ correlated subquery ซึ่งช้า ต้องการ rewrite เป็น JOIN กับ Derived Table แทน (ตามแนวทางใน slide Index Strategy)

**Task:**
แสดง EmployeeName, TotalFreight, AvgAllFreight (ทศนิยม 2) เฉพาะพนักงานที่มี TotalFreight เกินค่าเฉลี่ยรวม เรียงตาม TotalFreight DESC

**Sample Data:**

*Table: `employees`*

| employee_id | first_name | last_name |
| --- | --- | --- |
| 1 | Nancy | Davolio |
| 2 | Andrew | Fuller |
| 3 | Janet | Leverling |

*Table: `orders`*

| order_id | employee_id | freight |
| --- | --- | --- |
| 10248 | 1 | 32.38 |
| 10249 | 1 | 11.61 |
| 10250 | 2 | 65.83 |
| 10251 | 2 | 51.30 |
| 10252 | 3 | 5.00 |

**Expected Output:**

| employee_name | total_freight | avg_all_freight |
| --- | --- | --- |
| Andrew Fuller | 117.13 | 33.22 |
| Nancy Davolio | 43.99 | 33.22 |

<details>
<summary>Solution</summary>

```sql
-- ช้า: correlated subquery
-- WHERE SUM(freight) > (SELECT AVG(...) FROM ...)

-- เร็ว: JOIN กับ derived table
SELECT CONCAT(e.first_name, ' ', e.last_name) AS employee_name,
       ROUND(CAST(emp_total.total_freight AS DECIMAL(10,2)), 2) AS total_freight,
       ROUND(CAST(global_avg.avg_all_freight AS DECIMAL(10,2)), 2) AS avg_all_freight
FROM employees e
JOIN (
    SELECT employee_id, SUM(freight) AS total_freight
    FROM orders
    GROUP BY employee_id
) emp_total ON e.employee_id = emp_total.employee_id
JOIN (
    SELECT AVG(total) AS avg_all_freight
    FROM (SELECT employee_id, SUM(freight) AS total FROM orders GROUP BY employee_id) t
) global_avg ON emp_total.total_freight > global_avg.avg_all_freight
ORDER BY total_freight DESC;
```

</details>

---

## Section III — Performance Tuning & Query Rewriting (Q15–Q17)

---

### Q15 : Rewrite: หา Order ในปี 1997 โดยไม่ใช้ Function บน indexed column
**Topic:** `Query Rewriting: BETWEEN แทน YEAR()`

**Scenario:**
Query เดิม: WHERE EXTRACT(YEAR FROM OrderDate) = 1997 ซึ่งทำให้ DB ไม่สามารถใช้ Index บน OrderDate ได้ ให้ rewrite เป็น range predicate แทนตามแนวทางใน slide

**Task:**
แสดง OrderID, CustomerID, OrderDate เรียงตาม OrderDate โดยเขียน WHERE แบบ BETWEEN เพื่อ index-friendly

**Sample Data:**

*Table: `orders (sample)`*

| order_id | customer_id | order_date |
| --- | --- | --- |
| 10399 | VAFFE | 1996-12-31 |
| 10400 | EASTC | 1997-01-01 |
| 10401 | HANAR | 1997-06-15 |
| 10402 | ERNSH | 1997-12-31 |
| 10403 | MEREP | 1998-01-01 |

**Expected Output:**

| order_id | customer_id | order_date |
| --- | --- | --- |
| 10400 | EASTC | 1997-01-01 |
| 10401 | HANAR | 1997-06-15 |
| 10402 | ERNSH | 1997-12-31 |

<details>
<summary>Solution</summary>

```sql
-- ช้า: function บน indexed column
-- WHERE EXTRACT(YEAR FROM order_date) = 1997

-- เร็ว: range predicate — index ใช้ได้
SELECT order_id, customer_id, order_date
FROM orders
WHERE order_date BETWEEN '1997-01-01' AND '1997-12-31'
ORDER BY order_date;
```

</details>

---

### Q16 : หา Category ที่มีสินค้า Discontinued (ใช้ EXISTS แทน COUNT)
**Topic:** `EXISTS แทน COUNT(*) > 0`

**Scenario:**
Query เดิมใช้ COUNT(*) > 0 ซึ่งต้องนับทุก row ก่อน ให้ rewrite ด้วย EXISTS ซึ่งหยุดทันทีที่เจอ row แรก

**Task:**
แสดง CategoryID, CategoryName เฉพาะ Category ที่มีสินค้า Discontinued = 1 เรียง CategoryName

**Sample Data:**

*Table: `categories`*

| category_id | category_name |
| --- | --- |
| 1 | Beverages |
| 2 | Condiments |
| 3 | Confections |

*Table: `products`*

| product_id | category_id | product_name | discontinued |
| --- | --- | --- | --- |
| 1 | 1 | Chai | 0 |
| 5 | 2 | Chef Anton's Gumbo Mix | 1 |
| 9 | 3 | Mishi Kobe Niku | 1 |
| 10 | 3 | Ikura | 0 |

**Expected Output:**

| category_id | category_name |
| --- | --- |
| 2 | Condiments |
| 3 | Confections |

<details>
<summary>Solution</summary>

```sql
-- ช้า: COUNT(*) > 0
-- WHERE (SELECT COUNT(*) FROM products WHERE ...) > 0

-- เร็ว: EXISTS หยุดทันทีที่เจอ row แรก
SELECT c.category_id, c.category_name
FROM categories c
WHERE EXISTS (
    SELECT 1 FROM products p
    WHERE p.category_id = c.category_id
      AND p.discontinued = 1
)
ORDER BY c.category_name;
```

</details>

---

### Q17 : Top 5 Order ที่มี Freight สูงสุด (ใช้ LIMIT ให้เร็ว)
**Topic:** `LIMIT เพื่อลด work`

**Scenario:**
Dashboard ต้องแสดงแค่ Top 5 Orders ที่มี Freight สูงที่สุด ควรใส่ LIMIT เพื่อลด data ที่ต้องส่งกลับ

**Task:**
แสดง OrderID, CustomerID, EmployeeID, Freight เรียงตาม Freight DESC LIMIT 5

**Sample Data:**

*Table: `orders (sample)`*

| order_id | customer_id | employee_id | freight |
| --- | --- | --- | --- |
| 10372 | QUEEN | 5 | 890.78 |
| 10514 | ERNSH | 3 | 789.95 |
| 10897 | HUNGO | 3 | 603.54 |
| 10817 | RATTC | 5 | 306.07 |
| 10430 | ERNSH | 4 | 458.78 |
| 10359 | SEVES | 5 | 288.43 |

**Expected Output:**

| order_id | customer_id | employee_id | freight |
| --- | --- | --- | --- |
| 10372 | QUEEN | 5 | 890.78 |
| 10514 | ERNSH | 3 | 789.95 |
| 10897 | HUNGO | 3 | 603.54 |
| 10430 | ERNSH | 4 | 458.78 |
| 10817 | RATTC | 5 | 306.07 |

<details>
<summary>Solution</summary>

```sql
SELECT order_id, customer_id, employee_id, freight
FROM orders
ORDER BY freight DESC
LIMIT 5;
```

</details>

---

## Section IV — Common Table Expressions (CTEs) (Q18–Q21)

---

### Q18 : ยอดขายรวมต่อลูกค้า โดยใช้ CTE แทน Subquery ซ้อน
**Topic:** `Basic CTE (WITH)`

**Scenario:**
ทีม BI ต้องการ query ที่อ่านง่าย แสดงยอดขายรวมต่อลูกค้าและเปรียบเทียบกับค่าเฉลี่ยรวม โดยใช้ CTE แทน nested subquery

**Task:**
แสดง CustomerID, TotalRevenue, AvgRevenue (ทศนิยม 2) และ AbovAvg (TRUE/FALSE) เรียงตาม TotalRevenue DESC

**Sample Data:**

*Table: `orders`*

| order_id | customer_id |
| --- | --- |
| 10248 | ALFKI |
| 10249 | ANATR |
| 10250 | ALFKI |
| 10251 | HANAR |

*Table: `order_details`*

| order_id | unit_price | quantity |
| --- | --- | --- |
| 10248 | 14.00 | 12 |
| 10249 | 9.80 | 10 |
| 10250 | 19.00 | 5 |
| 10251 | 7.70 | 15 |

**Expected Output:**

| customer_id | total_revenue | avg_revenue | above_avg |
| --- | --- | --- | --- |
| ALFKI | 263.00 | 131.50 | TRUE |
| HANAR | 115.50 | 131.50 | FALSE |
| ANATR | 98.00 | 131.50 | FALSE |

<details>
<summary>Solution</summary>

```sql
WITH customer_revenue AS (
    SELECT o.customer_id,
           ROUND(CAST(SUM(od.quantity * od.unit_price) AS DECIMAL(10,2)), 2) AS total_revenue
    FROM orders o
    JOIN order_details od ON o.order_id = od.order_id
    GROUP BY o.customer_id
),
avg_revenue AS (
    SELECT ROUND(CAST(AVG(total_revenue) AS DECIMAL(10,2)), 2) AS avg_revenue
    FROM customer_revenue
)
SELECT cr.customer_id, cr.total_revenue, ar.avg_revenue,
       (cr.total_revenue > ar.avg_revenue) AS above_avg
FROM customer_revenue cr, avg_revenue ar
ORDER BY cr.total_revenue DESC;
```

</details>

---

### Q19 : วิเคราะห์ยอดขายพนักงานเทียบ Department avg (Multi-CTE)
**Topic:** `Multi-CTE`

**Scenario:**
ต้องการ Multi-CTE วิเคราะห์พนักงานโดย CTE แรกคำนวณยอดขายรวมต่อพนักงาน CTE ที่สองดึง Top performer แล้ว JOIN กัน

**Task:**
แสดง EmployeeName, TotalRevenue, Rank (ตาม TotalRevenue DESC) โดยใช้อย่างน้อย 2 CTE

**Sample Data:**

*Table: `employees`*

| employee_id | first_name | last_name |
| --- | --- | --- |
| 1 | Nancy | Davolio |
| 2 | Andrew | Fuller |
| 3 | Janet | Leverling |

*Table: `orders`*

| order_id | employee_id |
| --- | --- |
| 10248 | 1 |
| 10249 | 2 |
| 10250 | 2 |
| 10251 | 3 |
| 10252 | 1 |

*Table: `order_details`*

| order_id | unit_price | quantity |
| --- | --- | --- |
| 10248 | 14.00 | 12 |
| 10249 | 9.80 | 10 |
| 10250 | 9.80 | 10 |
| 10251 | 7.70 | 15 |
| 10252 | 5.00 | 10 |

**Expected Output:**

| employee_name | total_revenue | rank |
| --- | --- | --- |
| Nancy Davolio | 218.00 | 1 |
| Andrew Fuller | 196.00 | 2 |
| Janet Leverling | 115.50 | 3 |

<details>
<summary>Solution</summary>

```sql
WITH emp_sales AS (
    SELECT o.employee_id,
           ROUND(CAST(SUM(od.quantity * od.unit_price) AS DECIMAL(10,2)), 2) AS total_revenue
    FROM orders o
    JOIN order_details od ON o.order_id = od.order_id
    GROUP BY o.employee_id
),
emp_ranked AS (
    SELECT employee_id, total_revenue,
           RANK() OVER (ORDER BY total_revenue DESC) AS rank
    FROM emp_sales
)
SELECT CONCAT(e.first_name, ' ', e.last_name) AS employee_name,
       er.total_revenue, er.rank
FROM emp_ranked er
JOIN employees e ON er.employee_id = e.employee_id
ORDER BY er.rank;
```

</details>

---

### Q20 : แสดง Org Chart พนักงานทุกระดับ (Recursive CTE)
**Topic:** `Recursive CTE`

**Scenario:**
HR ต้องการ Org Chart แบบ recursive แสดงพนักงานทุกคนพร้อม Level (0 = CEO, 1 = direct report ฯลฯ) และ Path ชื่อ (เช่น Andrew Fuller > Nancy Davolio)

**Task:**
แสดง EmployeeID, FullName, Level, Path เรียงตาม Level, EmployeeID

**Sample Data:**

*Table: `employees`*

| employee_id | first_name | last_name | reports_to |
| --- | --- | --- | --- |
| 1 | Nancy | Davolio | 2 |
| 2 | Andrew | Fuller |  |
| 3 | Janet | Leverling | 2 |
| 5 | Steven | Buchanan | 2 |
| 6 | Michael | Suyama | 5 |

**Expected Output:**

| employee_id | full_name | level | path |
| --- | --- | --- | --- |
| 2 | Andrew Fuller | 0 | Andrew Fuller |
| 1 | Nancy Davolio | 1 | Andrew Fuller > Nancy Davolio |
| 3 | Janet Leverling | 1 | Andrew Fuller > Janet Leverling |
| 5 | Steven Buchanan | 1 | Andrew Fuller > Steven Buchanan |
| 6 | Michael Suyama | 2 | Andrew Fuller > Steven Buchanan > Michael Suyama |

<details>
<summary>Solution</summary>

```sql
WITH RECURSIVE org AS (
    -- Anchor: CEO (ไม่มีหัวหน้า)
    SELECT employee_id,
           CONCAT(first_name, ' ', last_name) AS full_name,
           reports_to,
           0 AS level,
           CONCAT(first_name, ' ', last_name) AS path
    FROM employees
    WHERE reports_to IS NULL

    UNION ALL

    -- Recursive: หา subordinates
    SELECT e.employee_id,
           CONCAT(e.first_name, ' ', e.last_name),
           e.reports_to,
           org.level + 1,
           CONCAT(org.path, ' > ', e.first_name, ' ', e.last_name)
    FROM employees e
    JOIN org ON e.reports_to = org.employee_id
)
SELECT employee_id, full_name, level, path
FROM org
ORDER BY level, employee_id;
```

</details>

---

### Q21 : Top 2 สินค้าขายดีสุดของแต่ละ Category (CTE + ROW_NUMBER)
**Topic:** `CTE + Window Function`

**Scenario:**
ทีม Product ต้องการ Top 2 สินค้าที่ขายดีที่สุด (by revenue) ในแต่ละ Category โดยใช้ CTE คำนวณ revenue แล้วใช้ Window Function จัดอันดับ

**Task:**
แสดง CategoryName, ProductName, TotalRevenue, Rank เฉพาะ Rank 1-2 เรียงตาม CategoryName, Rank

**Sample Data:**

*Table: `categories`*

| category_id | category_name |
| --- | --- |
| 1 | Beverages |
| 2 | Condiments |

*Table: `products`*

| product_id | product_name | category_id |
| --- | --- | --- |
| 1 | Chai | 1 |
| 2 | Chang | 1 |
| 38 | Côte de Blaye | 1 |
| 3 | Aniseed Syrup | 2 |
| 4 | Chef Anton's | 2 |
| 5 | Gumbo Mix | 2 |

*Table: `order_details`*

| order_id | product_id | unit_price | quantity |
| --- | --- | --- | --- |
| 10248 | 1 | 14.00 | 12 |
| 10249 | 2 | 19.00 | 10 |
| 10250 | 38 | 263.50 | 5 |
| 10251 | 3 | 10.00 | 8 |
| 10252 | 4 | 22.00 | 12 |
| 10253 | 5 | 21.35 | 6 |

**Expected Output:**

| category_name | product_name | total_revenue | rank |
| --- | --- | --- | --- |
| Beverages | Côte de Blaye | 1317.50 | 1 |
| Beverages | Chang | 190.00 | 2 |
| Condiments | Chef Anton's | 264.00 | 1 |
| Condiments | Gumbo Mix | 128.10 | 2 |

<details>
<summary>Solution</summary>

```sql
WITH product_revenue AS (
    SELECT p.product_id, p.product_name, p.category_id,
           ROUND(CAST(SUM(od.quantity * od.unit_price) AS DECIMAL(10,2)), 2) AS total_revenue
    FROM products p
    JOIN order_details od ON p.product_id = od.product_id
    GROUP BY p.product_id, p.product_name, p.category_id
),
ranked AS (
    SELECT pr.product_name, c.category_name, pr.total_revenue,
           ROW_NUMBER() OVER (PARTITION BY pr.category_id ORDER BY pr.total_revenue DESC) AS rank
    FROM product_revenue pr
    JOIN categories c ON pr.category_id = c.category_id
)
SELECT category_name, product_name, total_revenue, rank
FROM ranked
WHERE rank <= 2
ORDER BY category_name, rank;
```

</details>

---

## Section V — Window Functions (Q22–Q30)

![Window Functions](img/window-functions.png)

---

### Q22 : จำนวน Order ทั้งหมดของแต่ละ Customer แสดงทุก row
**Topic:** `OVER(PARTITION BY)`

**Scenario:**
ต้องการเห็น OrderID ทุก row พร้อมจำนวน Order รวมของ Customer นั้น โดยไม่ collapse (ต่างจาก GROUP BY) เพื่อใช้ใน downstream processing

**Task:**
แสดง CustomerID, OrderID, OrderDate และ TotalOrdersForCustomer เรียงตาม CustomerID, OrderDate

**Sample Data:**

*Table: `orders (sample)`*

| order_id | customer_id | order_date |
| --- | --- | --- |
| 10248 | VINET | 1996-07-04 |
| 10249 | VINET | 1996-07-05 |
| 10250 | TOMSP | 1996-07-08 |
| 10251 | VINET | 1996-07-08 |

**Expected Output:**

| customer_id | order_id | order_date | total_orders_for_customer |
| --- | --- | --- | --- |
| TOMSP | 10250 | 1996-07-08 | 1 |
| VINET | 10248 | 1996-07-04 | 3 |
| VINET | 10249 | 1996-07-05 | 3 |
| VINET | 10251 | 1996-07-08 | 3 |

<details>
<summary>Solution</summary>

```sql
SELECT customer_id, order_id, order_date,
       COUNT(*) OVER (PARTITION BY customer_id) AS total_orders_for_customer
FROM orders
ORDER BY customer_id, order_date;
```

</details>

---

### Q23 : จัดอันดับ Order ของแต่ละ Customer ตามวันที่ (ROW_NUMBER)
**Topic:** `OVER(PARTITION BY ... ORDER BY)`

**Scenario:**
CRM ต้องการทราบว่า Order แต่ละใบเป็น Order ที่เท่าไรของลูกค้าคนนั้น เพื่อวิเคราะห์ order frequency

**Task:**
แสดง CustomerID, OrderID, OrderDate, OrderSequence เรียงตาม CustomerID, OrderDate

**Sample Data:**

*Table: `orders (sample)`*

| order_id | customer_id | order_date |
| --- | --- | --- |
| 10248 | VINET | 1996-07-04 |
| 10274 | VINET | 1996-08-12 |
| 10295 | VINET | 1996-09-02 |
| 10249 | TOMSP | 1996-07-05 |
| 10356 | TOMSP | 1996-11-18 |

**Expected Output:**

| customer_id | order_id | order_date | order_sequence |
| --- | --- | --- | --- |
| TOMSP | 10249 | 1996-07-05 | 1 |
| TOMSP | 10356 | 1996-11-18 | 2 |
| VINET | 10248 | 1996-07-04 | 1 |
| VINET | 10274 | 1996-08-12 | 2 |
| VINET | 10295 | 1996-09-02 | 3 |

<details>
<summary>Solution</summary>

```sql
SELECT customer_id, order_id, order_date,
       ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS order_sequence
FROM orders
ORDER BY customer_id, order_date;
```

</details>

---

![img](img/window-frame-row-vs-range.png)

![img](img/aggregate.png)

---

### Q24 : ยอดสะสม Freight รายออเดอร์ของแต่ละพนักงาน (Running Total)
**Topic:** `Window Frame ROWS BETWEEN (Cumulative)`

**Scenario:**
ต้องการคำนวณ Running Total ของ Freight สะสมตามลำดับ OrderDate ของแต่ละพนักงาน โดยใช้ ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

**Task:**
แสดง EmployeeID, OrderID, OrderDate, Freight, CumulativeFreight เรียงตาม EmployeeID, OrderDate

**Sample Data:**

*Table: `orders (sample)`*

| order_id | employee_id | order_date | freight |
| --- | --- | --- | --- |
| 10248 | 1 | 1996-07-04 | 32.38 |
| 10252 | 1 | 1996-07-09 | 51.30 |
| 10255 | 1 | 1996-07-12 | 148.33 |
| 10249 | 2 | 1996-07-05 | 11.61 |
| 10251 | 2 | 1996-07-08 | 41.34 |

**Expected Output:**

| employee_id | order_id | order_date | freight | cumulative_freight |
| --- | --- | --- | --- | --- |
| 1 | 10248 | 1996-07-04 | 32.38 | 32.38 |
| 1 | 10252 | 1996-07-09 | 51.30 | 83.68 |
| 1 | 10255 | 1996-07-12 | 148.33 | 232.01 |
| 2 | 10249 | 1996-07-05 | 11.61 | 11.61 |
| 2 | 10251 | 1996-07-08 | 41.34 | 52.95 |

<details>
<summary>Solution</summary>

```sql
SELECT employee_id, order_id, order_date, freight,
       ROUND(CAST(SUM(freight) OVER (
           PARTITION BY employee_id
           ORDER BY order_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS DECIMAL(10,2)), 2) AS cumulative_freight
FROM orders
ORDER BY employee_id, order_date;
```

</details>

---

### Q25 : Moving Average 3 Order ของ Freight แต่ละพนักงาน
**Topic:** `Window Frame ROWS BETWEEN (Moving Avg)`

**Scenario:**
Finance ต้องการ smooth ค่า Freight ด้วย 3-order moving average (รวม current + 2 ก่อนหน้า) ของแต่ละพนักงาน ตาม OrderDate

**Task:**
แสดง EmployeeID, OrderID, OrderDate, Freight, MovingAvg3 (ทศนิยม 2) เรียงตาม EmployeeID, OrderDate

**Sample Data:**

*Table: `orders (sample)`*

| order_id | employee_id | order_date | freight |
| --- | --- | --- | --- |
| 10248 | 1 | 1996-07-04 | 32.38 |
| 10252 | 1 | 1996-07-09 | 51.30 |
| 10255 | 1 | 1996-07-12 | 148.33 |
| 10265 | 1 | 1996-07-25 | 55.28 |
| 10249 | 2 | 1996-07-05 | 11.61 |
| 10251 | 2 | 1996-07-08 | 41.34 |

**Expected Output:**

| employee_id | order_id | order_date | freight | moving_avg_3 |
| --- | --- | --- | --- | --- |
| 1 | 10248 | 1996-07-04 | 32.38 | 32.38 |
| 1 | 10252 | 1996-07-09 | 51.30 | 41.84 |
| 1 | 10255 | 1996-07-12 | 148.33 | 77.34 |
| 1 | 10265 | 1996-07-25 | 55.28 | 84.97 |
| 2 | 10249 | 1996-07-05 | 11.61 | 11.61 |
| 2 | 10251 | 1996-07-08 | 41.34 | 26.48 |

<details>
<summary>Solution</summary>

```sql
SELECT employee_id, order_id, order_date, freight,
       ROUND(CAST(AVG(freight) OVER (
           PARTITION BY employee_id
           ORDER BY order_date
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS DECIMAL(10,2)), 2) AS moving_avg_3
FROM orders
ORDER BY employee_id, order_date;
```

</details>

---

![img](img/ranking.png)

---

### Q26 : จัดอันดับพนักงานตามยอดขาย (RANK vs DENSE_RANK)
**Topic:** `RANK() / DENSE_RANK()`

**Scenario:**
HR ต้องการ rank พนักงานตามยอด Freight รวม โดยแสดงทั้ง RANK (ข้ามเลขถ้าเท่ากัน) และ DENSE_RANK (ไม่ข้ามเลข) เพื่อเปรียบเทียบ

**Task:**
แสดง EmployeeName, TotalFreight, RANK, DENSE_RANK เรียงตาม TotalFreight DESC

**Sample Data:**

*Table: `employees`*

| employee_id | first_name | last_name |
| --- | --- | --- |
| 1 | Nancy | Davolio |
| 2 | Andrew | Fuller |
| 3 | Janet | Leverling |
| 4 | Margaret | Peacock |

*Table: `orders`*

| order_id | employee_id | freight |
| --- | --- | --- |
| 10248 | 1 | 32.38 |
| 10249 | 2 | 11.61 |
| 10250 | 3 | 32.38 |
| 10251 | 4 | 41.34 |
| 10252 | 1 | 11.00 |

**Expected Output:**

| employee_name | total_freight | RANK | DENSE_RANK |
| --- | --- | --- | --- |
| Margaret Peacock | 41.34 | 1 | 1 |
| Nancy Davolio | 43.38 | 2 | 2 |
| Janet Leverling | 32.38 | 3 | 3 |
| Andrew Fuller | 11.61 | 4 | 4 |

<details>
<summary>Solution</summary>

```sql
SELECT CONCAT(e.first_name, ' ', e.last_name) AS employee_name,
       ROUND(CAST(SUM(o.freight) AS DECIMAL(10,2)), 2) AS total_freight,
       RANK() OVER (ORDER BY SUM(o.freight) DESC) AS "RANK",
       DENSE_RANK() OVER (ORDER BY SUM(o.freight) DESC) AS "DENSE_RANK"
FROM employees e
JOIN orders o ON e.employee_id = o.employee_id
GROUP BY e.employee_id, e.first_name, e.last_name
ORDER BY total_freight DESC;
```

</details>

---

![img](img/distribution.png)

---

### Q27 : Percentile ของแต่ละสินค้าตาม UnitPrice ใน Category ตัวเอง
**Topic:** `PERCENT_RANK() / CUME_DIST()`

**Scenario:**
ทีม Pricing ต้องการรู้ว่าสินค้าแต่ละชิ้นอยู่ใน percentile ที่เท่าไรของ Category ตัวเอง โดยใช้ทั้ง PERCENT_RANK และ CUME_DIST

**Task:**
แสดง CategoryName, ProductName, UnitPrice, PercentRank (ทศนิยม 2), CumeDist (ทศนิยม 2) เรียงตาม CategoryName, UnitPrice

**Sample Data:**

*Table: `categories`*

| category_id | category_name |
| --- | --- |
| 1 | Beverages |

*Table: `products`*

| product_id | product_name | category_id | unit_price |
| --- | --- | --- | --- |
| 24 | Guaraná | 1 | 4.50 |
| 1 | Chai | 1 | 18.00 |
| 2 | Chang | 1 | 19.00 |
| 35 | Steeleye | 1 | 18.00 |
| 38 | Côte de Blaye | 1 | 263.50 |

**Expected Output:**

| category_name | product_name | unit_price | percent_rank | cume_dist |
| --- | --- | --- | --- | --- |
| Beverages | Guaraná | 4.50 | 0.00 | 0.20 |
| Beverages | Chai | 18.00 | 0.25 | 0.60 |
| Beverages | Steeleye | 18.00 | 0.25 | 0.60 |
| Beverages | Chang | 19.00 | 0.75 | 0.80 |
| Beverages | Côte de Blaye | 263.50 | 1.00 | 1.00 |

<details>
<summary>Solution</summary>

```sql
SELECT c.category_name, p.product_name, p.unit_price,
       ROUND(CAST(PERCENT_RANK() OVER (
           PARTITION BY p.category_id ORDER BY p.unit_price
       ) AS DECIMAL(10,2)), 2) AS percent_rank,
       ROUND(CAST(CUME_DIST() OVER (
           PARTITION BY p.category_id ORDER BY p.unit_price
       ) AS DECIMAL(10,2)), 2) AS cume_dist
FROM products p
JOIN categories c ON p.category_id = c.category_id
ORDER BY c.category_name, p.unit_price;
```

</details>

---

![img](img/analytical.png)

---

### Q28 : เปรียบ Freight Order ปัจจุบันกับ Order ก่อนและหลัง (LAG + LEAD)
**Topic:** `LAG() / LEAD()`

**Scenario:**
Finance ต้องการดู Freight ของแต่ละ Order เทียบกับ Order ก่อนหน้าและถัดไปของลูกค้าเดียวกัน เพื่อวิเคราะห์ trend

**Task:**
แสดง CustomerID, OrderID, OrderDate, Freight, PrevFreight, NextFreight เรียงตาม CustomerID, OrderDate

**Sample Data:**

*Table: `orders (sample)`*

| order_id | customer_id | order_date | freight |
| --- | --- | --- | --- |
| 10248 | VINET | 1996-07-04 | 32.38 |
| 10274 | VINET | 1996-08-12 | 6.01 |
| 10295 | VINET | 1996-09-02 | 1.15 |
| 10737 | VINET | 1997-11-11 | 7.79 |

**Expected Output:**

| customer_id | order_id | order_date | freight | prev_freight | next_freight |
| --- | --- | --- | --- | --- | --- |
| VINET | 10248 | 1996-07-04 | 32.38 | NULL | 6.01 |
| VINET | 10274 | 1996-08-12 | 6.01 | 32.38 | 1.15 |
| VINET | 10295 | 1996-09-02 | 1.15 | 6.01 | 7.79 |
| VINET | 10737 | 1997-11-11 | 7.79 | 1.15 | NULL |

<details>
<summary>Solution</summary>

```sql
SELECT customer_id, order_id, order_date, freight,
       LAG(freight) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_freight,
       LEAD(freight) OVER (PARTITION BY customer_id ORDER BY order_date) AS next_freight
FROM orders
ORDER BY customer_id, order_date;
```

</details>

---

![img](img/analytical-2.png)

---

### Q29 : Freight ครั้งแรกและล่าสุดของแต่ละลูกค้า (FIRST/LAST VALUE)
**Topic:** `FIRST_VALUE() / LAST_VALUE()`

**Scenario:**
CRM ต้องการเปรียบเทียบ Freight ของ Order แรกกับ Order ล่าสุดของลูกค้าแต่ละราย เพื่อดูว่า Freight เปลี่ยนแปลงอย่างไร

**Task:**
แสดง CustomerID, OrderID, OrderDate, Freight, FirstOrderFreight, LastOrderFreight เรียงตาม CustomerID, OrderDate

**Sample Data:**

*Table: `orders (sample)`*

| order_id | customer_id | order_date | freight |
| --- | --- | --- | --- |
| 10248 | VINET | 1996-07-04 | 32.38 |
| 10274 | VINET | 1996-08-12 | 6.01 |
| 10295 | VINET | 1996-09-02 | 1.15 |
| 10737 | VINET | 1997-11-11 | 7.79 |

**Expected Output:**

| customer_id | order_id | order_date | freight | first_order_freight | last_order_freight |
| --- | --- | --- | --- | --- | --- |
| VINET | 10248 | 1996-07-04 | 32.38 | 32.38 | 7.79 |
| VINET | 10274 | 1996-08-12 | 6.01 | 32.38 | 7.79 |
| VINET | 10295 | 1996-09-02 | 1.15 | 32.38 | 7.79 |
| VINET | 10737 | 1997-11-11 | 7.79 | 32.38 | 7.79 |

<details>
<summary>Solution</summary>

```sql
SELECT customer_id, order_id, order_date, freight,
       FIRST_VALUE(freight) OVER (
           PARTITION BY customer_id ORDER BY order_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS first_order_freight,
       LAST_VALUE(freight) OVER (
           PARTITION BY customer_id ORDER BY order_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS last_order_freight
FROM orders
ORDER BY customer_id, order_date;
```

</details>

---

![img](img/analytical-3.png)

---

### Q30 : Freight Order ที่ 2 ของแต่ละลูกค้า (NTH_VALUE)
**Topic:** `NTH_VALUE()`

**Scenario:**
ทีม Analytics ต้องการดึง Freight ของ Order ที่ 2 (เรียงตาม OrderDate) ของลูกค้าแต่ละราย เพื่อวิเคราะห์ repeat purchase behavior

**Task:**
แสดง CustomerID, OrderID, OrderDate, Freight, SecondOrderFreight เรียงตาม CustomerID, OrderDate

**Sample Data:**

*Table: `orders (sample)`*

| order_id | customer_id | order_date | freight |
| --- | --- | --- | --- |
| 10248 | VINET | 1996-07-04 | 32.38 |
| 10274 | VINET | 1996-08-12 | 6.01 |
| 10295 | VINET | 1996-09-02 | 1.15 |
| 10249 | TOMSP | 1996-07-05 | 11.61 |
| 10356 | TOMSP | 1996-11-18 | 36.71 |
| 10822 | TOMSP | 1998-01-08 | 9.88 |

**Expected Output:**

| customer_id | order_id | order_date | freight | second_order_freight |
| --- | --- | --- | --- | --- |
| TOMSP | 10249 | 1996-07-05 | 11.61 | 36.71 |
| TOMSP | 10356 | 1996-11-18 | 36.71 | 36.71 |
| TOMSP | 10822 | 1998-01-08 | 9.88 | 36.71 |
| VINET | 10248 | 1996-07-04 | 32.38 | 6.01 |
| VINET | 10274 | 1996-08-12 | 6.01 | 6.01 |
| VINET | 10295 | 1996-09-02 | 1.15 | 6.01 |

<details>
<summary>Solution</summary>

```sql
SELECT customer_id, order_id, order_date, freight,
       NTH_VALUE(freight, 2) OVER (
           PARTITION BY customer_id ORDER BY order_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS second_order_freight
FROM orders
ORDER BY customer_id, order_date;
```

</details>

