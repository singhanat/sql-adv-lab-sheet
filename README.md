# Advanced Data Analysis and Query Optimization Using SQL

**แบบฝึกหัด จำนวน 30 ข้อ**
> Sections: Advanced Joins | Set Operations | Subqueries | Performance | CTE | Window Functions

## Data set
![ER](ER.png)

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
<summary>💡 Solution</summary>

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
<summary>💡 Solution</summary>

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
<summary>💡 Solution</summary>

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
<summary>💡 Solution</summary>

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
