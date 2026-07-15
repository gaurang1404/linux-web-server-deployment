# Install and Configure NGINX

So far, we've been accessing our application directly through the Express server on **port 3000**.

While this is perfectly acceptable during development, it is **not** the recommended approach for production deployments.

Instead, production environments typically place a **reverse proxy** such as **NGINX** in front of the application.

---

## Why Use NGINX?

**NGINX** is a high-performance web server and reverse proxy that sits between clients and your application.

Instead of exposing the Express application directly to the network, incoming requests are first received by NGINX. NGINX then forwards (or **proxies**) those requests to the Express server.

The request flow looks like this:

```text
Client
   │
   ▼
NGINX (Port 80)
   │
   ▼
Express Application (Port 3000)
```

Using NGINX provides several advantages:

- **Uses the default HTTP port (80)**, so users don't need to specify a custom port such as `:3000`.
- **Adds an additional security layer** by preventing direct access to the application server.
- **Handles incoming client connections efficiently**, reducing the workload on the application.
- Supports features such as **SSL/TLS termination**, **load balancing**, **compression**, **caching**, and **rate limiting**.
- Makes it easier to scale applications as your infrastructure grows.

For these reasons, NGINX is one of the most widely used reverse proxies in production environments.

---

## Install NGINX

Install the NGINX package using YUM:

```bash
sudo yum install nginx -y
```

This downloads and installs NGINX along with any required dependencies.

---

## Verify the Installation

Check the installed version:

```bash
nginx -v
```

Example:

```text
nginx version: nginx/1.26.3
```

---

## Enable NGINX at Boot

Configure NGINX to start automatically whenever the server boots.

```bash
sudo systemctl enable nginx
```

---

## Start the NGINX Service

Start the service:

```bash
sudo systemctl start nginx
```

---

## Verify That NGINX Is Running

Check the service status:

```bash
sudo systemctl status nginx
```

If everything has been configured correctly, you should see:

```text
Active: active (running)
```

This confirms that NGINX has started successfully and is ready to accept incoming requests.

---

## Next Step

Great! NGINX is now installed and running.

At this point, NGINX is listening on **port 80**, the default port used for HTTP traffic.

In the next section, we'll configure the firewall to allow HTTP traffic and then set up **NGINX as a reverse proxy**, forwarding incoming requests to our Express application running on **port 3000**.