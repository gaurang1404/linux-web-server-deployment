# Troubleshooting NGINX Errors and Warnings

This document contains issues encountered while deploying the application along with their causes and solutions.

---

# Issue 1: Nginx Warning - Conflicting Server Name

## Error

```text
[warn] conflicting server name "_" on 0.0.0.0:80, ignored
```

## Cause

Nginx ships with a default server configuration in `nginx.conf` that listens on port `80` with:

```nginx
server_name _;
```

My custom configuration also used:

```nginx
server {
    listen 80;
    server_name _;
    ...
}
```

Since two virtual hosts were configured with the same `server_name` on the same port, Nginx ignored one of them and displayed the warning.

---

## Attempted Solution

I modified my virtual host configuration to use a custom hostname.

```nginx
server {

    listen 80;

    server_name planet-travel.local;

    location / {

        proxy_pass http://127.0.0.1:3000;

        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

    }

}
```

---

## New Problem

When testing using

```bash
curl http://localhost/api/planets
```

I received

```html
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.26.3</center>
</body>
</html>
```

---

## Why This Happened

Although the request was reaching Nginx, the HTTP request contained the header

```text
Host: localhost
```

Nginx uses the `Host` header to determine which `server` block should process the request.

Since my configuration only matched

```nginx
server_name planet-travel.local;
```

the request did not match my virtual host.

Instead, Nginx served the default server configuration, which tried to find

```text
/usr/share/nginx/html/api/planets
```

Since that file did not exist, Nginx returned a **404 Not Found**.

---

## Solution

There are two possible solutions.

### Option 1

Change the virtual host to

```nginx
server_name localhost;
```

This allows requests made to

```bash
curl http://localhost
```

to match the virtual host.

---

### Option 2 (Chosen)

Keep a more realistic hostname.

Add an entry in `/etc/hosts`.

```text
127.0.0.1    planet-travel.local
```

Now requests can be made using

```bash
curl http://planet-travel.local/api/planets
```

This more closely resembles how applications are accessed in production using DNS hostnames.

---

# Issue 2: 502 Bad Gateway

## Error

```html
<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.26.3</center>
</body>
</html>
```

Nginx error log showed

```text
connect() to 127.0.0.1:3000 failed (13: Permission denied)
while connecting to upstream
```

---

## Cause

At first glance, this looked like a networking issue.

However, Express was already running correctly and could be accessed directly using

```bash
curl http://127.0.0.1:3000/api/planets
```

The actual issue was **SELinux**.

---

## What is SELinux?

**SELinux (Security-Enhanced Linux)** is an additional security layer available on Red Hat Enterprise Linux and CentOS systems.

Unlike traditional Linux permissions, which only control file ownership and read/write/execute permissions, SELinux enforces mandatory access control (MAC).

This means every process on the system is assigned a security context and is only allowed to perform actions explicitly permitted by the SELinux policy.

Even though Nginx was running correctly, SELinux prevented it from opening a TCP connection to another local process (the Node.js application).

From the operating system's perspective:

```
Nginx
   │
   │ TCP Connection
   ▼
127.0.0.1:3000
   │
Node.js
```

is still considered a network connection.

By default, SELinux does **not** allow web servers to initiate arbitrary outbound network connections.

Instead of silently allowing the request, SELinux blocked it, causing Nginx to return a **502 Bad Gateway**.

---

## Solution

Enable the SELinux boolean:

```bash
sudo setsebool -P httpd_can_network_connect 1
```

---

## Understanding the Command

### setsebool

Changes an SELinux boolean.

Booleans enable or disable optional SELinux policies without modifying the entire security policy.

---

### -P

Makes the change **persistent**.

Without `-P`, the setting would be lost after the next reboot.

---

### httpd_can_network_connect

This boolean controls whether web servers (such as Nginx and Apache) are allowed to initiate outbound network connections.

Setting it to `1` allows Nginx to communicate with backend services such as:

- Node.js
- Spring Boot
- Python Flask
- Django
- Go applications
- Redis
- Elasticsearch

---

## Why This Is Better Than Disabling SELinux

A common but poor practice is to disable SELinux completely.

```bash
sudo setenforce 0
```

or permanently edit

```text
/etc/selinux/config
```

and set

```text
SELINUX=disabled
```

While this removes the restriction, it also removes one of the most important security features provided by Enterprise Linux.

Using

```bash
sudo setsebool -P httpd_can_network_connect 1
```

is the recommended solution because it grants **only the required permission** while keeping all other SELinux protections enabled.

This follows the principle of **least privilege**, allowing Nginx to communicate with the backend application without weakening the overall security of the server.

---

## Lessons Learned

- Always verify the application works before introducing Nginx.
- The HTTP `Host` header determines which Nginx virtual host processes a request.
- Using custom hostnames may require configuring `/etc/hosts` or a DNS server.
- A **502 Bad Gateway** does not always indicate that the backend application is down.
- On CentOS and RHEL, SELinux is often the cause of `Permission denied` errors when configuring reverse proxies.
- Prefer enabling the appropriate SELinux policy rather than disabling SELinux entirely.