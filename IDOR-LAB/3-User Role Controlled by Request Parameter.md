# 🔐 Access Control Vulnerability – IDOR / User Role Controlled by Request Parameter

---

## 🧩 1. Lab Overview

**Lab Name:**  
User Role Controlled by Request Parameter  

### 🧠 Key Insight
The user role (Admin / User) is controlled via a **request parameter**, such as:
- Cookie  
- Header  
- URL parameter  

### 🎯 Objective
- Access `/admin`  
- Delete the user **Carlos**  
- Without having real admin privileges  

---

## 🚨 2. Critical Observation

**Lab Description:**
> "Admin panel at /admin which identifies administrators using a cookie"

⚠️ This statement alone indicates a serious vulnerability.

---

## ⚠️ 3. Root Cause

The developer implemented insecure logic like:

```php
if ($_COOKIE['admin'] == "true") {
    allow_admin_access();
}
❌ Why This is Wrong
Cookies are controlled by the user
They are not secure by default
They are not validated server-side
They can be modified بسهولة

📌 This is called:

Client-Side Role Control

🧠 4. Why a User Account is Provided

Because we need to:

Observe how cookies are generated
Understand how roles are assigned
Analyze the system behavior

📌 This is part of Reconnaissance

🔍 5. Login & Observation Phase

After logging in, intercept the request using Burp Suite.

Example Response:
Set-Cookie: admin=false
Set-Cookie: session=abc123
🧠 Analysis:
session → Normal
admin=false → 🚨 Critical issue
🧠 6. Attacker Thinking

Ask yourself:

What happens if I change admin=false to admin=true?

If:

Behavior changes
New privileges appear

➡️ This confirms Broken Access Control

🧪 7. Testing with Burp Suite (Professional Method)
🔹 Step 1: Send request to Repeater
GET /my-account
Cookie: admin=false

➡️ No admin access

🔹 Step 2: Modify the Cookie
GET /my-account
Cookie: admin=true

🔥 Result:

Admin panel becomes visible
New links appear
Elevated privileges granted

📌 This confirms:

The server blindly trusts user-controlled cookies

🌐 8. Exploitation via Browser

Go to:

Inspect → Application → Cookies

Modify:

admin = true

🔄 Refresh the page

🔥 Result:

Admin panel appears
🎯 9. Goal Execution
Navigate to /admin
Locate user list
Click Delete Carlos

✔️ User deleted
✔️ Lab solved

💣 10. Real-World Impact

This vulnerability can lead to:

Privilege escalation (User → Admin)
Data deletion
Role manipulation
Sensitive data exposure
Full system compromise

💥 All by changing:

admin=false → admin=true
🔎 11. Detection Strategy
🧠 Mindset:

After every login:

➡️ Inspect cookies

Look for:
admin
role
isAdmin
Then:
Modify values
Observe behavior changes
✅ Quick Checklist
Role stored in cookie
Boolean values (true/false)
No JWT or signature
No server-side validation
Response changes after modification

✔️ If all true → Vulnerability confirmed

🔐 12. Proper Mitigation
✔️ Secure Implementation
Store roles on the server
Use cookies only for session IDs
if ($_SESSION['role'] !== 'admin') {
    http_response_code(403);
    exit;
}
❌ Critical Mistake
if ($_COOKIE['admin'] == true)
🧠 13. Key Takeaways

Any privilege coming from the user = Fake privilege

or

Cookies must never define user roles

🎓 14. Lessons Learned
Access Control is more critical than Authentication
Cookies are not a trusted source
Burp Repeater is a powerful testing tool
Simple mistakes can lead to full compromise
