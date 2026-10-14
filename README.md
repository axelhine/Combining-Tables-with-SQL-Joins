# Combining Tables with SQL Joins

> **Disclaimer:** This project was completed as part of the Google Cybersecurity Certificate (Coursera). The scenario and database come from the course. The queries and explanations are my own documentation of the work I performed.

## Project description
I was investigating a recent security incident that compromised some machines, and the information I needed was spread across several tables. Using SQL in a MariaDB shell, I used `INNER JOIN`, `LEFT JOIN`, and `RIGHT JOIN` to connect the `machines`, `employees`, and `log_in_attempts` tables through shared columns. Joins let an analyst connect who is using a machine, which machines have no owner, and who has made login attempts, without searching each table separately.

## Database Tables
| Table | Contents | Shared column |
|---|---|---|
| `machines` | Hardware information (device ID, operating system, patch date) | `device_id` |
| `employees` | Employee information (username, department, office) | `device_id`, `username` |
| `log_in_attempts` | Login activity (username, date, time, country, success) | `username` |

The `machines` table has no employee names, and the `employees` table has no hardware details. A shared column, called a key, is what lets SQL connect the two. `device_id` links machines to employees, and `username` links employees to their login attempts.

## Join Types Used
| Join | What it returns |
|---|---|
| `INNER JOIN` | Only rows that have a match in **both** tables |
| `LEFT JOIN` | **All** rows from the left table (listed after `FROM`), plus matching rows from the right table. Unmatched rows show `NULL` |
| `RIGHT JOIN` | **All** rows from the right table (listed after `JOIN`), plus matching rows from the left table. Unmatched rows show `NULL` |

Because both tables can have a column with the same name, I use dot notation (`table.column`) so SQL knows exactly which column I mean.

## Match employees to their machines
To find out which employees are using which machines, I joined `machines` and `employees` on their shared `device_id` column. An inner join keeps only the machines that matched an employee:

```sql
SELECT *
FROM machines
INNER JOIN employees ON machines.device_id = employees.device_id;
```
Result: **185** rows returned.

## Return more data
The inner join hides anything without a match, so I used outer joins to see the gaps.

**Left join.** Keeps every machine, including machines not assigned to any employee. Unassigned machines show `NULL` in the employee columns:

```sql
SELECT *
FROM machines
LEFT JOIN employees ON machines.device_id = employees.device_id;
```
Result: the `username` value in the last record returned is `NULL`, which means that machine has no employee assigned.

**Right join.** Keeps every employee, including employees with no machine assigned. Missing machines show `NULL` in the machine columns:

```sql
SELECT *
FROM machines
RIGHT JOIN employees ON machines.device_id = employees.device_id;
```
Result: the `username` value in the last record returned is `areyes`.

Swapping the table order turns a right join into a left join, so `FROM machines LEFT JOIN employees` returns the same result as `FROM employees RIGHT JOIN machines`. Most analysts use `LEFT JOIN` almost exclusively because it reads naturally: start with the main list and pull in whatever matches.

## Retrieve login attempt data
To continue the investigation, I needed every login attempt along with the employee who made it. I joined `employees` and `log_in_attempts` on the shared `username` column:

```sql
SELECT *
FROM employees
INNER JOIN log_in_attempts ON employees.username = log_in_attempts.username;
```
Result: **200** records returned.

## Findings
| Query | Result |
|---|---|
| Inner join: `machines` and `employees` on `device_id` | 185 rows |
| Left join: last record's `username` | `NULL` (machine with no assigned employee) |
| Right join: last record's `username` | `areyes` |
| Inner join: `employees` and `log_in_attempts` on `username` | 200 records |

- The `NULL` at the end of the left join shows a machine that isn't tied to any employee. In an incident investigation, unassigned machines are worth a closer look, since no one is responsible for keeping them updated or monitored.
- Inner joins only show complete matches, while left and right joins keep unmatched records visible. Choosing the right join type decides whether gaps in the data show up or stay hidden.

## Full Query Sequence
```sql
SELECT * FROM machines INNER JOIN employees ON machines.device_id = employees.device_id;
SELECT * FROM machines LEFT JOIN employees ON machines.device_id = employees.device_id;
SELECT * FROM machines RIGHT JOIN employees ON machines.device_id = employees.device_id;
SELECT * FROM employees INNER JOIN log_in_attempts ON employees.username = log_in_attempts.username;
```

## Summary
I used `INNER JOIN` to match 185 machine records to their employees, then used `LEFT JOIN` and `RIGHT JOIN` to find machines with no assigned user and employees with no assigned machine. I finished by joining `employees` with `log_in_attempts` on `username` to connect 200 login records to the people who made them. Combining tables this way lets an analyst trace an incident across hardware, people, and login activity from a single query.

## Key Takeaways
- Joins connect tables through a shared column, such as `device_id` or `username`.
- `INNER JOIN` keeps only matching rows. Unmatched rows disappear, so gaps in the data stay hidden.
- `LEFT JOIN` keeps everything from the first table, and `RIGHT JOIN` keeps everything from the second. Unmatched rows show `NULL`.
- Swapping the table order lets a `LEFT JOIN` do the work of a `RIGHT JOIN`, which is why most analysts stick with `LEFT JOIN`.
- Use dot notation (`table.column`) when both tables share a column name, to avoid ambiguity errors.
- `SELECT *` on a join returns duplicate columns such as two `device_id` columns. In real reports, name the specific columns you need, for example `machines.device_id, employees.username, machines.operating_system`.
- `NULL` values in a join result point to missing relationships, like an unassigned machine, which are often what an investigation needs to find.

## Tools
- MariaDB shell
- SQL
