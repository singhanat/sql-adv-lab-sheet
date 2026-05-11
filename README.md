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
       ROUND(ABS(a.unit_price - b.unit_price)::NUMERIC, 2) AS price_diff
FROM products a
JOIN products b ON a.category_id = b.category_id
               AND a.product_id < b.product_id
WHERE ABS(a.unit_price - b.unit_price) <= 5
ORDER BY price_diff;
```

</details>

---

### Q3 · รายละเอียด Order พร้อมลูกค้าและพนักงาน (3-table JOIN)
**Section:** I &nbsp;|&nbsp; **Topic:** `MULTIPLE JOINS: JOIN & JOIN`

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
       e.first_name || ' ' || e.last_name AS employee_name,
       o.order_date
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN employees e ON o.employee_id = e.employee_id
WHERE EXTRACT(YEAR FROM o.order_date) = 1997
ORDER BY o.order_date;
```

</details>

---
