# -Complete-IDOR-Guide-Beginner-Advanced

<aside>
📚

## **🎓 Complete IDOR Guide: Beginner → Advanced**

---

## **🔰 BEGINNER LEVEL: Understanding IDOR**

### **What is IDOR in Simple Terms?**

IDOR stands for **Insecure Direct Object Reference** and is a type of access control vulnerability. Web applications often use references to determine what data to return when you make a request. However, if the web server doesn't perform checks to ensure you are allowed to view that data before sending it, it can lead to serious sensitive information disclosure.

Think of it like this: You're at a hotel. Your room is #101. If you can open room #102 just by changing the number on the key card, that's an IDOR vulnerability! The hotel didn't check if you're actually allowed to enter that room.

**Web Example:**

```
Your order: [example.com/order?id=100](http://example.com/order?id=100)
Try this: [example.com/order?id=101](http://example.com/order?id=101) ← Just change the number!
```

If you can see someone else's order details, that's IDOR!

### **Why is This Dangerous?**

- 🔴 See other people's private information (emails, addresses, phone numbers)
- 🔴 Modify or delete data that isn't yours
- 🔴 Financial fraud, identity theft, data breaches

### **How to Test for IDOR (Beginner Steps)**

**Step 1:** Find URLs with IDs

```
/user/123
/document/456
/order?id=789
```

**Step 2:** Note your own ID (e.g., 123)

**Step 3:** Change it to another number (e.g., 124, 125, 122)

**Step 4:** Check the result

- ❌ Can you see someone else's data? → **IDOR Found!**
- ✅ Get error or access denied? → Properly secured

### **Common Places to Find IDOR**

- 👤 User profiles: `/profile/123`
- 📄 Documents: `/document/456`
- 🛒 Orders: `/order?id=789`
- 📧 Messages: `/inbox/101`
- 💳 Transactions: `/payment/202`
- 📷 Photos: `/photo/303`

---

## **📚 INTERMEDIATE LEVEL: Core Concepts**

### **Authentication vs Authorization (Critical Difference!)**

| Concept | Question | Example |
| --- | --- | --- |
| **Authentication** | *"Who are you?"* | Login with username/password, session cookies, JWT tokens |
| **Authorization** | *"What can you do?"* | Can you view this page? Can you delete this file? |

> **🔑 Key Rule:** Authorization CANNOT happen before authentication!
> 

> The system must know WHO you are before checking WHAT you can do.
> 

**Important:** Authentication happens on EVERY request, not just login!

- You login → Get session cookie/token
- Every request includes this cookie/token
- Server verifies it on EVERY request
- If expired → Redirect to login page

### **Privilege Escalation Types**

**⬆️ Vertical Escalation (Going UP)**

```
User → Admin
Employee → Manager
Regular → Superuser
```

**Example:** Normal user can delete any account (admin action)

**↔️ Horizontal Escalation (Going SIDEWAYS)**

```
User A → User B's data (same privilege level)
Customer 1 → Customer 2's orders
Patient X → Patient Y's records
```

**Example:** You can view your orders, but also other customers' orders

> **IDOR is typically horizontal privilege escalation**
> 

> You're using a feature you're allowed to use, but accessing data you shouldn't see.
> 

### **Why "IDOR" is a Confusing Name**

Many security experts prefer **"Authorization Bypass"** because:

**❌ The problem is NOT:**

- Using IDs in URLs (perfectly normal!)
- Sequential numbers (common practice)
- Direct object references (efficient design)

**✅ The REAL problem is:**

- Missing authorization checks
- Not verifying user owns the resource
- Trusting user input without validation

**Common Developer Mistake:**

```
/user/1 → /user/ea21f09b2 (encoded)
```

"We fixed IDOR by encoding the ID!"

**Wrong!** If the server still doesn't check permissions, it's still vulnerable. The ID format doesn't matter!

### **IDOR in Different HTTP Methods**

**GET (Reading data):**

```
GET /api/users/123
```

**POST (Creating/Updating):**

```json
POST /api/orders/update
{
  "order_id": 999,
  "shipping_address": "Attacker's address"
}
```

**PUT (Updating):**

```
PUT /api/profile/123
{"email": "[attacker@evil.com](mailto:attacker@evil.com)"}
```

**DELETE (Deleting):**

```
DELETE /api/documents/456
```

**PATCH (Partial update):**

```
PATCH /api/accounts/789
{"balance": 999999}
```

> **Pro Tip:** Always test ALL HTTP methods, not just GET!
> 

### **Testing with Browser DevTools**

**Step-by-step:**

1. Press **F12** (or Right-click → Inspect)
2. Go to **Network** tab
3. Perform an action (view profile, order, etc.)
4. Find the request that fetches your data
5. Right-click → **Copy → Copy as cURL**
6. Modify the ID in the command
7. Run in terminal to test

**Example:**

```bash
# Original request (your data)
curl 'https://example.com/api/user/123' \
  -H 'Cookie: session=abc123'

# Modified request (try to access other user)
curl 'https://example.com/api/user/124' \
  -H 'Cookie: session=abc123'
```

---

## **🚀 ADVANCED LEVEL: Deep Dive**

### **Advanced IDOR Techniques**

**1. Encoded/Hashed IDs**

IDs might be base64, hex, MD5, or other formats:

```bash
# Original: 123
# Base64: MTIz
# Hex: 7b
# MD5: 202cb962ac59075b964b07152d234b70

# Decode → Modify → Re-encode
echo "MTIz" | base64 -d  # Returns: 123
echo "124" | base64      # Returns: MTI0
```

Test with modified encoded value:

```
/user/MTI0  (base64 for 124)
```

**2. UUID/GUID References**

Even random-looking IDs can be vulnerable:

```
/api/document/550e8400-e29b-41d4-a716-446655440000
```

If the server doesn't check ownership, **it's still IDOR!** The ID format is irrelevant.

**3. Mass Assignment Vulnerabilities**

Inject unauthorized parameters:

```json
POST /api/users/update
{
  "name": "Alice",
  "email": "[alice@test.com](mailto:alice@test.com)",
  "role": "admin",          ← Try adding
  "is_premium": true,       ← Try adding
  "account_balance": 99999  ← Try adding
}
```

**4. Multi-Step IDOR Chains**

Combine vulnerabilities for maximum impact:

```
Step 1: IDOR to list users → Get all user IDs
Step 2: IDOR to view profiles → Get email addresses  
Step 3: IDOR to reset passwords → Takeover accounts
```

**5. Blind IDOR**

No visible response, but action executes:

```bash
DELETE /api/photos/999
Response: 200 OK (generic message)

# Verify by:
# - Creating test account
# - Uploading photo as test user
# - Deleting with different account
# - Check if photo is actually gone
```

**6. Time-of-Check Time-of-Use (TOCTOU)**

```
1. Load page → Auth check passes (your resource)
2. Modify ID in browser DevTools → No recheck
3. Perform action → Executes on different resource!
```

**7. Array/Object Injection**

```
# Try these variations:
?id=123
?id[]=123&id[]=124
?id[0]=123
?user_id=123
?user[id]=123
```

### **Automation Tools**

**Burp Suite Extensions:**

- **Autorize** - Auto-tests all requests with different sessions
- **AuthMatrix** - Matrix testing for multiple users/roles
- **Auto Repeater** - Automatically modifies and resends

**Command-Line Tools:**

```bash
# Fuzzing with ffuf
ffuf -w ids.txt -u "https://target.com/api/user/FUZZ" \
     -H "Cookie: session=abc123"

# Parameter discovery with Arjun
arjun -u https://target.com/api/profile

# Using wfuzz
wfuzz -z range,1-1000 \
      -H "Cookie: session=xyz" \
      https://target.com/order?id=FUZZ

# Simple bash loop
for i in {1..100}; do
  curl -H "Cookie: session=abc" \
       "https://target.com/user/$i"
done
```

### **Professional Testing Methodology**

**Phase 1: Reconnaissance**

- Map all endpoints with object references
- Identify parameter patterns (id, user_id, order_id, doc_id)
- Note authentication mechanisms
- Document API structure

**Phase 2: Account Setup**

- Create 2-3 test accounts (User A, B, C)
- Create resources for each user (orders, documents, etc.)
- Document legitimate IDs for each account

**Phase 3: Cross-Account Testing**

```
Login as User A:
  ✓ Access User A's resources (baseline)
  ✗ Try User B's resources (should fail)
  ✗ Try User C's resources (should fail)
  
Test ALL operations:
  - Read (GET)
  - Create (POST)
  - Update (PUT/PATCH)
  - Delete (DELETE)
```

**Phase 4: Privilege Testing**

- Unauthenticated access (no login)
- Expired session tokens
- Different role levels (user vs admin)
- Admin endpoints with regular user credentials

**Phase 5: Edge Cases**

```
id=-1          (negative)
id=0           (zero)
id=999999999   (very large)
id=1' OR '1'='1  (SQL injection chars)
id[]=1&id[]=2  (array)
id=../../../etc/passwd  (path traversal)
```

### **Secure Coding: Defense Guide**

**❌ INSUFFICIENT Defenses (Don't rely on these!):**

- Obfuscating IDs
- Encoding/hashing IDs
- Using UUIDs instead of integers
- Client-side validation only
- Rate limiting alone

**✅ PROPER Defenses:**

**1. Server-Side Authorization (ESSENTIAL)**

```python
# Python/Flask
@app.route('/order/<int:order_id>')
@login_required
def view_order(order_id):
    order = Order.query.get_or_404(order_id)
    
    # CRITICAL: Check ownership
    if order.user_id != current_[user.id](http://user.id):
        abort(403)
    
    return render_template('order.html', order=order)
```

```jsx
// Node.js/Express
app.get('/api/documents/:id', authenticate, async (req, res) => {
    const doc = await Document.findById([req.params.id](http://req.params.id));
    
    if (!doc) {
        return res.status(404).json({ error: 'Not found' });
    }
    
    // CRITICAL: Verify ownership
    if (doc.ownerId !== [req.user.id](http://req.user.id)) {
        return res.status(403).json({ error: 'Access denied' });
    }
    
    res.json(doc);
});
```

```php
// PHP
function getOrder($orderId) {
    global $db, $currentUser;
    
    // Query with ownership check
    $stmt = $db->prepare(
        "SELECT * FROM orders 
         WHERE order_id = ? AND user_id = ?"
    );
    $stmt->execute([$orderId, $currentUser['id']]);
    
    return $stmt->fetch();
}
```

```go
// Go
func GetOrder(w http.ResponseWriter, r *http.Request) {
    orderID := chi.URLParam(r, "id")
    userID := r.Context().Value("user_id").(int)
    
    var order Order
    err := db.QueryRow(
        "SELECT * FROM orders WHERE id = $1 AND user_id = $2",
        orderID, userID,
    ).Scan(&order)
    
    if err != nil {
        http.Error(w, "Not found or access denied", 403)
        return
    }
    
    json.NewEncoder(w).Encode(order)
}
```

**2. Indirect Object References**

```python
# Session-specific mapping
# User never sees actual database IDs
user_session = {
    'documents': {
        'ref_a1b2': 1001,  # Internal doc ID
        'ref_c3d4': 1002,
        'ref_e5f6': 1003
    }
}

# User can only access refs in their session
```

**3. Access Control Lists**

```sql
-- Always join with ownership table
SELECT o.* 
FROM orders o
INNER JOIN user_orders uo ON [o.id](http://o.id) = uo.order_id
WHERE [o.id](http://o.id) = ? AND uo.user_id = ?
```

**4. Logging & Monitoring**

```python
# Log all access attempts
[logger.info](http://logger.info)(f"Access: user={user_id}, resource={resource_id}, "
            f"owner={resource.owner_id}, "
            f"result={'allowed' if auth else 'denied'}")

# Alert on suspicious patterns
if failed_access_count > 5:
    send_alert(f"User {user_id} attempting multiple "
               f"unauthorized accesses")
```

### **Real-World Impact Examples**

**💰 Critical Severity:**

1. **Bank Statements** - View other users' financial records
2. **Medical Records** - HIPAA violations, lawsuits
3. **Account Takeover** - Full compromise via IDOR chain

**🔴 High Severity:**

1. **Private Messages** - Corporate espionage, privacy breach
2. **Order Manipulation** - Change shipping, financial loss
3. **PII Exposure** - Names, addresses, phone numbers

### **Bug Bounty Report Template**

```markdown
# IDOR in Order Management System

## Severity
High (CVSS 7.5)

## Summary
Authenticated users can view/modify orders belonging to other users 
by manipulating the order_id parameter.

## Vulnerability Details
- Endpoint: GET /api/orders/{order_id}
- Parameter: order_id
- Authentication: Required
- Authorization Check: Missing

## Steps to Reproduce
1. Create two accounts:
   - User A: [usera@test.com](mailto:usera@test.com) / pass123
   - User B: [userb@test.com](mailto:userb@test.com) / pass123

2. Login as User A, create order → Note order_id: 1001

3. Login as User B

4. Access: GET /api/orders/1001
   Header: Cookie: session=user_b_session

5. Result: User B can view User A's order

## Proof of Concept
[Screenshots showing:]
- User A's order (legitimate)
- User B viewing User A's order (unauthorized)
- Burp Suite request/response

## Impact
- Confidentiality breach (PII exposure)
- Integrity risk (order modification)
- Financial fraud potential
- GDPR/CCPA violations

## Remediation
```

# Add authorization check:

if order.user_id != current_[user.id](http://user.id):

abort(403)

```

## References
- OWASP: Broken Access Control
- CWE-639: Authorization Bypass
```

---

## **🎯 Key Takeaways Summary**

**Beginners:**

✅ IDOR = accessing other users' data by changing IDs

✅ Test by changing numbers in URLs

✅ If you see someone else's data → vulnerability!

**Intermediate:**

✅ Authentication ≠ Authorization

✅ IDOR = horizontal privilege escalation

✅ Test ALL HTTP methods (GET, POST, PUT, DELETE)

✅ Problem = missing checks, NOT ID format

**Advanced:**

✅ Test encoded/hashed IDs

✅ Look for blind IDORs

✅ Chain multiple IDORs

✅ Automate with Burp Suite extensions

**Developers:**

✅ **ALWAYS** verify ownership server-side

✅ Use parameterized queries with ownership checks

✅ Log all access attempts

✅ Obfuscation ≠ Security

---

## **📖 Additional Resources**

- [OWASP Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [PortSwigger Access Control Labs](https://portswigger.net/web-security/access-control)
- [HackerOne IDOR Reports](https://www.hackerone.com/)
- [CWE-639](https://cwe.mitre.org/data/definitions/639.html)

</aside>
