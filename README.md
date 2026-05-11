# Advanced Data Analysis and Query Optimization Using SQL

**แบบฝึกหัด จำนวน 30 ข้อ**
> Sections: Advanced Joins | Set Operations | Subqueries | Performance | CTE | Window Functions

## Data set
![ER](ER.png)

### Q1 · พนักงานและหัวหน้าตัวเอง (SELF JOIN แบบ equi)
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
