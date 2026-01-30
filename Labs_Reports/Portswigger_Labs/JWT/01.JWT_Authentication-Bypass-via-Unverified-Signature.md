# JWT Authentication Bypass via Unverified Signature

## Overview
This issue occurs when a server **fails to verify the JWT signature**, allowing attackers to modify token payloads while reusing the original signature. As a result, privilege escalation and full account takeover become possible.

---

## Root Cause (What’s Broken)

The application **does not validate the JWT signature**.

This means:
- The payload can be modified
- The signature can be left untouched
- The server will still trust the token

---

## Recon & Initial Findings

- JWT identified in `/my-account`
- Token payload decoded
- `sub` claim changed from `wiener` → `administrator`
- Confirmed `/admin` endpoint is admin-only

---

## Exploitation Steps

### Step 1: Modify the JWT Payload

Change the payload to:

    {
      "sub": "administrator"
    }

### Step 2: Critical Rule — Do NOT Re-sign the Token
Leave the signature exactly as it is.

Do NOT:

- Generate a new signature
- Change the secret
- Change the alg value

In Burp Suite:

- Inspector → JWT → Payload
- Edit claim
- Click Apply changes

### Step 3: Send the Modified Request
Using Burp Repeater:

- Path: /admin
- Cookie: Modified JWT (payload edited, signature unchanged)
- Send the request

✅ Admin panel becomes accessible

### Step 4: Complete the Objective
- Navigate to /admin
- Delete the required user (e.g., carlos)
- 🎯 Lab solved

## Why This Works
### Vulnerability
- JWT authentication bypass due to missing signature verification

### Impact
- Privilege escalation for any authenticated user
- Full account takeover (including admin access)

### Attack Technique
- Modify JWT claims (sub, role, etc.)
- Reuse the original signature
- Server blindly trusts token contents

### Remediation / Correct Fix

- Always verify JWT signatures
- Reject tokens with:
  - Invalid signatures
  - Tampered payloads
- Use well-maintained and trusted JWT libraries correctly

### Pro Tip (Burp Workflow)
For JWT labs like this:
1. Inspector → JWT → Payload
2. Edit claim
3. Apply changes
4. Never touch the signature unless the lab explicitly mentions:
  - alg: none
  - Key confusion attacks

