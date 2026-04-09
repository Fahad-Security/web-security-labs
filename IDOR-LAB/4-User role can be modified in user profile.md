# 🔐 Access Control Vulnerability – IDOR / User Role Can Be Modified in User Profile

---

## 🧩 Lab Information

**Lab Name:**  
User Role Can Be Modified in User Profile  

**Vulnerability Type:**  
Broken Access Control (IDOR / Client-Side Role Manipulation)

### 🧠 Core Idea
The application allows modification of the user role from the **client-side**, leading to privilege escalation.

### 🎯 Objective
- Access `/admin`  
- Delete the user **Carlos**

### 📌 Key Detail
- Admin users have: `roleid = 2`  
- Your account has: `roleid = 1` (normal user)

---

## 🟢 Phase 1: Login & Traffic Observation

### 🔹 Step 1: Login

Request:
```http
POST /login
username=...&password=...

📌 Normal behavior — no modification needed yet
➡️ This request may be useful later for automation

🟢 Phase 2: Searching for Access Control Weakness
🔹 Step 2: Inspect "My Account"

Request:

GET /my-account?id=username
🔍 Initial Test:

Try modifying the id parameter:

id=admin
id=administrator
❌ Result:
302 Redirect → Login

📌 Conclusion:

This parameter is not exploitable

🟢 Phase 3: Email Change (Critical Entry Point)
🔹 Step 3: Change Email

Request:

POST /my-account/change-email
email=test@test.ca

➡️ Send this request to Burp Repeater

🟢 Phase 4: Response Analysis (Critical Step)
🔹 Step 4: Inspect Response

Response:

{
  "username": "wiener",
  "email": "test@test.ca",
  "apikey": "xxxx",
  "roleid": 1
}

🚨 Critical Issue:

The server exposes roleid
From lab description:
roleid = 2 → Admin
🧠 Key Question:

Does the server accept modification of roleid from the client?

🟢 Phase 5: Exploitation
🔹 Step 5: Modify Request

Add a new parameter:

POST /my-account/change-email
email=test@test.ca&roleid=2

📤 Send request

✅ Result:
"roleid": 2

🔥 Privilege escalation successful
➡️ You are now an Admin

🟢 Phase 6: Verify Privileges
Refresh My Account
A new link appears:

➡️ Admin Panel

📌 Confirmation: Exploit worked 100%

🟢 Phase 7: Delete Carlos
🔹 Step 6: Access Admin Panel

Go to:

/admin
🔹 Action:
Click Delete Carlos

Request:

GET /admin/delete?username=carlos

🎉 Result:

Congratulations, you solved the lab

💥 Technical Summary
🧨 Vulnerability Type:
Broken Access Control
Client-Side Role Manipulation
⚠️ Root Cause:
Server returns roleid in response
Server accepts modified roleid from client ❌
No validation or authorization checks
🔐 Proper Mitigation
✔️ Secure Approach:
Never allow client to control roles
Store roles server-side only
Ignore any role-related input from user
if ($_SESSION['role'] !== 'admin') {
    http_response_code(403);
    exit;
}
❌ Insecure Behavior:
Accepting roleid from request
Trusting client-side data
🧠 Attacker Mindset

When testing similar features:

🔍 Focus on:
Update profile
Change email
Edit account
🚩 Look for sensitive fields in responses:
role
roleid
isAdmin
accessLevel
permissions
⚡ Rule:

If you see it… try to modify it

🧠 Key Takeaways
Any privilege sent by the client is untrusted
Exposure of role-related data is dangerous
Lack of validation = full compromise
🎓 Lessons Learned
Access Control failures can be subtle but critical
Always analyze responses, not just requests
Parameter pollution is a powerful technique
Simple input manipulation can lead to full admin access
