## Домашння работа №14

1. Для выполнения этого домашнего задания были созданы 2 таблицы exmployees и departments, на которых и будут выполняться запросы. Их структры выглядят следующим образом: 
```sql
staff=# select * from employees;
 id |  name   | department_id
----+---------+---------------
  1 | Alex    |          1012
  2 | Henry   |          1020
  3 | Maria   |          1012
  4 | Albert  |          1041
  5 | Mark    |          1020
  6 | Jenifer |
(6 rows)
```

```sql
staff=# select * from departments ;
  id  | department_name
------+-----------------
 1012 | Accounting
 1020 | Marketing
 1041 | Logistics
(3 rows)
```

2. Реализуем прямое соединение двух этих таблиц. Оно возвращает только те строки, которые имеют совпадения в обеих таблицах
```sql
staff=# SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
  name  | department_name
--------+-----------------
 Alex   | Accounting
 Henry  | Marketing
 Maria  | Accounting
 Albert | Logistics
 Mark   | Marketing
(5 rows)
```

3. Далее выполним левостороннее соединение. Оно возвращает все строки из левой таблицы и совпадающие строки из правой таблицы.
```sql
staff=# SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
  name   | department_name
---------+-----------------
 Alex    | Accounting
 Henry   | Marketing
 Maria   | Accounting
 Albert  | Logistics
 Mark    | Marketing
 Jenifer |
(6 rows)
```

4. Кросс соединение возвращает декартово произведение двух таблиц, то есть все возможные комбинации строк из обеих таблиц.
```sql
staff=# SELECT e.name, d.department_name
FROM employees e
CROSS JOIN departments d;
  name   | department_name
---------+-----------------
 Alex    | Accounting
 Alex    | Marketing
 Alex    | Logistics
 Henry   | Accounting
 Henry   | Marketing
 Henry   | Logistics
 Maria   | Accounting
 Maria   | Marketing
 Maria   | Logistics
 Albert  | Accounting
 Albert  | Marketing
 Albert  | Logistics
 Mark    | Accounting
 Mark    | Marketing
 Mark    | Logistics
 Jenifer | Accounting
 Jenifer | Marketing
 Jenifer | Logistics
(18 rows)
```

5. Полное соединение возвращает все строки из обеих таблиц, где строки без совпадений заполняются NULL.
```sql
staff=# SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;
  name   | department_name
---------+-----------------
 Alex    | Accounting
 Henry   | Marketing
 Maria   | Accounting
 Albert  | Logistics
 Mark    | Marketing
 Jenifer |
(6 rows)
```

6. Далее попытался объединить LEFT JOIN и FULL OUTER JOIN в один запрос. В результате мы получим всех сотрудников и их отделы (если они существуют), а также всех сотрудников и отделы, включая строки, которые не имеют совпадений.
```sql
staff=# SELECT
    e.name AS employee_name,
    d.department_name AS department_name,
    e2.name AS another_employee_name,
    d2.department_name AS another_department_name
FROM
    employees e
LEFT JOIN departments d ON e.department_id = d.id
FULL OUTER JOIN employees e2 ON e.department_id = e2.department_id
FULL OUTER JOIN departments d2 ON e.department_id = d2.id;
 employee_name | department_name | another_employee_name | another_department_name
---------------+-----------------+-----------------------+-------------------------
 Alex          | Accounting      | Maria                 | Accounting
 Alex          | Accounting      | Alex                  | Accounting
 Henry         | Marketing       | Mark                  | Marketing
 Henry         | Marketing       | Henry                 | Marketing
 Maria         | Accounting      | Maria                 | Accounting
 Maria         | Accounting      | Alex                  | Accounting
 Albert        | Logistics       | Albert                | Logistics
 Mark          | Marketing       | Mark                  | Marketing
 Mark          | Marketing       | Henry                 | Marketing
 Jenifer       |                 |                       |
               |                 | Jenifer               |
(11 rows)
```
