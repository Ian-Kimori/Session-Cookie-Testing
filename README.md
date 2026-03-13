# Session-Cookie-Testing

***

# **1. What To Test**

You want to determine:

1.  Can session tokens be **guessed**, **predicted**, or **forged**?

2.  Can attackers **modify cookies** to impersonate users?

3.  Does the session token have **enough randomness**?

4.  Do session tokens change when:
    *   user logs in
    *   user logs out and back in
    *   privileges change

5.  Can you access someone else’s session using:
    *   stolen token
    *   modified cookie
    *   weakly signed token

***

# **2. Tools Needed**

*   Burp Suite (Community OK)
*   Browser (Chrome/Firefox)
*   Two user accounts:
    *   **User A** (normal)
    *   **User B** (another user)

***

# **3. Step 1 — Identify the Session Token**

Log in as any user.

### Go to:

`F12 → Application → Cookies`

Look for session cookies:

*   `session_id`
*   `JSESSIONID`
*   `PHPSESSID`
*   `auth_token`
*   `jwt`
*   `token`
*   `zbx_session` (Zabbix apps)
*   `access_token`

**Write down the full value.**

***

# **4. Step 2 — Gather Multiple Session Tokens**

You must collect tokens for:

### A) Same user — multiple logins

1.  Log out
2.  Log in
3.  Capture token
4.  Repeat 5 times

**Why?**  
To check if tokens are predictable or similar.

***

### B) Different users

Login as:

*   User A → capture token
*   User B → capture token

Compare token patterns.

***

# **PASS Conditions**

✔ Tokens look random  
✔ Tokens change on every login  
✔ No pattern, no structure  
✔ No readable user info  
✔ Long (32–64+ characters)

Example of a strong token:

    EYJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJBcHA

OR

    dfba98c724cf76c93a0d5b71c76e3f0c8ef21a9df232

***

# **FAIL Conditions (VERY IMPORTANT)**

❌ Tokens look similar  
❌ Tokens share patterns  
❌ Tokens increase like counters  
❌ Tokens start with user ID  
❌ Tokens contain readable info (e.g. base64)

Examples of bad tokens:

    token=12345
    token=abc123
    token=user_001
    token=admin:true
    token=UID=3&role=admin
    token=MTIzNDU2Nzg5

These are **predictable or forgeable**.

***

# **5. Step 3 — Test Session Token Predictability**

### Method:

*   Open **Burp → Decoder**
*   Try decoding the token as:
    *   Base64
    *   Base64URL
    *   Hex
    *   ASCII

### ❌ FAIL:

If decoding reveals readable info:

    {"user_id": 5, "role": "admin"}

Attackers can tamper with it → **Session forging risk**.

***

# **6. Step 4 — Modify the Session Token**

Using Burp Suite:

1.  Capture an authenticated request (User A)
2.  Send to Repeater
3.  Modify the cookie value slightly:

Examples:

    Original = 4f2fcbd1ac9c89e02a0f30
    Modified = 4f2fcbd1ac9c89e02a0f31

Or truncate:

    abcd1234 → abcd12

### ✔ PASS

Server returns:

*   401 Unauthorized
*   403 Forbidden
*   Redirect to login

### ❌ FAIL

Server still accepts the modified token → **token validation weak**.

***

# **7. Step 5 — Attempt Session Fixation**

Test if token changes after login.

1.  Record token BEFORE login (unauthenticated)
2.  Record token AFTER login

### ✔ PASS

Tokens are different.

### ❌ FAIL

If token **stays the same**, attacker can fix your session.

***

# **8. Step 6 — Test Cookie Signature (Important)**

Look for two cookies:

Example:

    session=abc123
    session.sig=834923af

Modify **session** only.

### ✔ PASS

Server rejects the cookie (signature mismatch)

### ❌ FAIL

Server accepts the cookie after modification → **session forging possible**

***

# **9. Step 7 — Attempt Using Another User’s Token**

### How to test:

1.  Log in as User A
2.  Copy User A's session cookie
3.  Paste it into browser logged in as **User B**
    *   Use Chrome extension "Cookie Editor"
    *   Or in Burp: modify cookie

### ✔ PASS

User B gets logged out or blocked.

### ❌ FAIL (Critical)

User B becomes User A  
→ Session hijacking possible  
→ PCI DSS violation  
→ Major vulnerability

***

# **10. Step 8 — Test Token Expiry & Logout**

### A) Does token expire after logout?

After logging out:

*   Try using the token in Burp

PASS:

*   Token invalid

FAIL:

*   Token still works → session not invalidated

***

### B) Does token expire automatically?

Let session sit idle for 10–20 minutes.

PASS:

*   Token expires

FAIL:

*   Token never expires → session poisoning possible

***

# **11. FINAL PASS / FAIL CHECKLIST**

### ✔ PASS if:

*   Tokens random & unpredictable
*   Tokens contain no readable info
*   Token changes on every login
*   Modifying token causes invalidation
*   Token signature enforced
*   Logout invalidates session
*   User A token cannot be used for User B
*   Tokens expire after inactivity
*   No token reuse across logins

### ❌ FAIL if:

*   Tokens predictable
*   Tokens contain user ID or role
*   Tokens do not change after login
*   Modified tokens still work
*   Token signature missing or broken
*   Cross-user token reuse possible
*   Logout does not invalidate
*   No session timeout

***

***

If you want, I can generate a **Burp Suite step‑by‑step attack plan**, a **session security checklist**, or a **sample evidence sheet** you can paste into your audit.
