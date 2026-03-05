1. Title

SQL Injection – Retrieving Database Version (Oracle) – PortSwigger Lab

2. Summary / Overview

This lab demonstrates a SQL Injection vulnerability that allows an attacker to retrieve the type and version of the backend database.

The vulnerable web application is hosted in the PortSwigger Web Security Academy environment. The objective of the lab is to perform an SQL Injection attack that identifies the database engine and its version.

During the reconnaissance phase of SQL injection testing, determining the database type is very important because each database system (Oracle, MySQL, PostgreSQL, Microsoft SQL Server) has slightly different SQL syntax.

In this lab, the backend database was identified as Oracle, and the database version was successfully extracted using a UNION-based SQL Injection attack.

  3. Tools / Environment

The following tools were used during this lab:

OWASP ZAP Proxy

Web Browser

PortSwigger Web Security Academy Lab

SQL Injection Cheat Sheet

4. Step-by-Step Process / Methodology
Step 1 – Access the vulnerable parameter

The application contains a category parameter in the URL when selecting product categories.

Example request:

/filter?category=Accessories

This parameter was suspected to be vulnerable to SQL Injection.

Step 2 – Intercept the request

The request was intercepted using OWASP ZAP Proxy to analyze and modify the HTTP request.

Example intercepted request:

GET /filter?category=Accessories HTTP/1.1
Step 3 – Test SQL Injection using UNION

To test if the parameter is injectable, a UNION SELECT payload was injected.

Payload used:

' UNION SELECT 'ABC','DEF' FROM dual--

Encoded payload:

' UNION SELECT 'ABC','DEF' FROM dual--

Important notes:

Oracle databases require selecting from a table

The built‑in Oracle table DUAL was used

Step 4 – Send the request to the server

After sending the request, the response was rendered in the browser.

The application returned a product with the following values:

Product Name: ABC
Description: DEF

This confirmed that the SQL Injection attack was successful.

Step 5 – Extract the database version

Next, the payload was modified to retrieve the database version.

Payload used:

' UNION SELECT banner,'test' FROM v$version--

Explanation:

banner → contains version information

v$version → Oracle system view containing database version details

Step 6 – Send the payload

After sending the payload, the response was rendered in the browser.

The following output appeared:

CORE 11.2.0.2.0 Production

This reveals the Oracle database version.

5. Results / Observations

The SQL Injection attack successfully:

Confirmed that the application is vulnerable to UNION-based SQL Injection

Identified the database type as Oracle

Extracted the database version

Returned database version:

CORE 11.2.0.2.0 Production

The lab was successfully solved after retrieving the database version.

6. Analysis / Explanation

The vulnerability exists because the application directly includes user input inside an SQL query without proper validation or sanitization.

Example vulnerable query (conceptual example):

SELECT name, description FROM products
WHERE category = 'Accessories'

When the attacker injects malicious SQL code:

' UNION SELECT banner,'test' FROM v$version--

The final query becomes something like:

SELECT name, description FROM products
WHERE category = ''
UNION SELECT banner,'test' FROM v$version--

This causes the database to return version information from the Oracle system table.

The reason we know the database is Oracle is because:

The payload uses FROM dual

The table v$version is specific to Oracle databases

If another database were used (MySQL, PostgreSQL, MSSQL), this syntax would likely fail.

7. Mitigation / Recommendations

To prevent SQL Injection vulnerabilities, developers should implement the following security measures:

1. Use Prepared Statements

Parameterized queries prevent attackers from injecting SQL code.

Example (secure approach):

SELECT name, description FROM products WHERE category = ?
2. Input Validation

All user input should be validated and sanitized before processing.

3. Use ORM Frameworks

ORM frameworks often handle query parameterization automatically.

4. Apply Least Privilege

Database accounts used by the application should have minimal privileges to reduce impact if an attack occurs.

5. Implement Web Application Firewall (WAF)

A WAF can detect and block common SQL Injection patterns.

8. Appendix / References

Lab Source

PortSwigger Web Security Academy – SQL Injection Labs

References

PortSwigger SQL Injection Cheat Sheet

OWASP Top 10 – Injection

Oracle Database Documentation

✅ Keywords

SQL Injection
Pentesting
Oracle Database
Web Security
PortSwigger Lab
Bug Bounty
