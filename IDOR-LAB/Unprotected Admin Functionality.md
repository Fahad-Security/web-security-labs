# 🔐 Access Control Vulnerability – IDOR / Unprotected Admin Functionality (With Unpredictable URL)

---

## 🧩 1. Lab Overview

**Lab Name:**  
Unprotected Admin Functionality with Unpredictable URL  

### 🧠 Key Insight
- The admin panel is **not protected**
- The URL is **not guessable**
- Brute force is ineffective  

However:  
🔴 The admin URL is **disclosed داخل التطبيق (within the application)**

### 🎯 Objective
- Find the admin panel URL  
- Access it without authorization  
- Delete the user **Carlos**

---

## ⚠️ 2. Root Cause

The developer made two critical mistakes:

### ❌ Mistake #1:
Relied on hiding the URL instead of implementing proper access control  

### ❌ Mistake #2:
Exposed the admin path inside:
- JavaScript
- HTML
- Comments  

📌 This is known as:
> **Security by Obscurity** — not real security

---

## 🧠 3. Attacker Mindset

When you see:

- "Unpredictable URL"
- "Disclosed somewhere"

➡️ Immediately ask:

> Where could the developer have leaked the admin path?

### احتمالات التسريب:
- View Source  
- JavaScript files  
- HTML comments  
- `robots.txt`  
- Network requests  

---

## 🔍 4. Discovery Process

### 🔹 Step 1: Check `robots.txt`
Sometimes contains:
``` id="f7s2ka"
Disallow: /admin-secret-123

❌ Not found in this lab

🔹 Step 2: View Page Source

Open:

Right Click → View Page Source

Search for keywords:

admin
panel
dashboard
role
isAdmin
🔥 5. Key Finding

Inside a JavaScript file, the following logic was found:

if (user.isAdmin) {
    adminPath = "/admin-xyz-9283";
}
🧠 Critical Insight:
Admin logic is exposed in the frontend
The admin panel URL is directly visible
🚨 6. Why This is a Critical Vulnerability

Because:

JavaScript is sent to the client
Anyone can read it
No server-side authorization exists

❌ The server logic is effectively:

"If you know the URL… you are allowed in"

💥 7. Exploitation
Step:

Copy the exposed path:

/admin-xyz-9283

Open it in the browser:

🔥 Access granted to admin panel

⚠️ No protection:
No login
No role check
No session validation
🎯 8. Goal Execution

Inside the admin panel:

User list is visible
Delete functionality is available
Action:
Click Delete Carlos

✔️ User deleted
✔️ Lab solved

💣 9. Real-World Impact

This vulnerability can lead to:

Full system takeover
User deletion
Role escalation
Sensitive data exposure
Complete compromise without technical exploitation

💥 Only requires:

URL
View Source
🔎 10. Detection Strategy
🧠 ثابت Mindset:

When you encounter:

Hidden admin panels
Unknown URLs
No authentication

➡️ Always check:

View Source
JavaScript files
Network tab
Comments
robots.txt
🔐 11. Proper Mitigation
✔️ Secure Approach
Never expose sensitive paths in frontend code
Enforce access control on the server
✔️ Example:
if (!isAdmin($_SESSION['user'])) {
    http_response_code(403);
    exit;
}
❌ Incorrect Approach
Hiding URLs only
Assuming users won’t find them
🧠 12. Key Takeaways

Anything in the frontend = Exposed

or

Unpredictable URL ≠ Secure URL
