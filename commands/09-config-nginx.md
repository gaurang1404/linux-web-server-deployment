# Configure NGINX as a Reverse Proxy

Now that NGINX is installed and running, it's time to configure it as a **reverse proxy** for our Express application.

Before doing that, let's first ensure that NGINX is accessible over HTTP.

---

## Allow HTTP Traffic Through the Firewall

View the current firewall configuration:

```bash
sudo firewall-cmd --list-all
```

Look for the `services` field.

Example:

```text
services: cockpit dhcpv6-client ssh
```

If the `http` service is **not** listed, add it permanently:

```bash
sudo firewall-cmd --permanent --add-service=http
```

Expected output:

```text
success
```

Reload the firewall configuration to apply the changes:

```bash
sudo firewall-cmd --reload
```

Verify the configuration again:

```bash
sudo firewall-cmd --list-all
```

You should now see:

```text
services: cockpit dhcpv6-client http ssh
```

---

## Verify NGINX

Open a browser and visit:

```text
http://localhost
```

If you're accessing the server from another machine, replace `localhost` with your VM's IP address:

```text
http://<VM-IP>
```

You should see the default **NGINX Welcome Page**.

This confirms that:

- NGINX is running.
- The firewall allows HTTP traffic.
- NGINX is successfully listening on **port 80**.

> **Notice:** We didn't specify a port number in the URL because **80** is the default port for HTTP.

---

## Configure NGINX

Currently, NGINX serves its default `index.html` page.

Instead, we want it to forward incoming requests to our Express application.

Create a new NGINX configuration file:

```bash
sudo vi /etc/nginx/conf.d/planet-travel.conf
```

Paste the following configuration:

```nginx
server {

    listen 80;

    server_name _;

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

Save the file and exit the editor.

---

## Understanding the Configuration

Let's briefly look at the important directives.

| Directive | Purpose |
|-----------|---------|
| `listen 80` | Listens for incoming HTTP requests on port 80. |
| `server_name _` | Matches all incoming hostnames. This is useful for simple deployments and demonstrations. |
| `location /` | Applies the configuration to every incoming request. |
| `proxy_pass` | Forwards requests to the Express application running on port 3000. |
| `proxy_set_header` | Preserves important client information such as the original host, client IP address, and protocol. This allows the backend application to know where requests originated. |

---

## Why Does `proxy_pass` Use `127.0.0.1`?

You may have noticed that the proxy points to:

```text
http://127.0.0.1:3000
```

instead of the server's network IP address.

This is intentional.

Once NGINX is acting as the public-facing web server, **only NGINX needs to communicate with the Express application**.

Since both NGINX and Express are running on the **same machine**, it's more efficient and secure for NGINX to forward requests using the **loopback interface (`127.0.0.1`)** rather than sending them back out through the network interface.

This approach offers several advantages:

- Reduces unnecessary network traffic.
- Prevents direct access to the application from outside the server.
- Ensures all external requests pass through NGINX, where additional security, logging, and performance features can be applied.

---

## Update the Application Configuration

Since Express will now only receive requests from NGINX running on the same machine, update the `.env` file.

Open:

```bash
sudo vi .env
```

Update it to:

```env
ENV=production
DEV_HOST=localhost
PROD_HOST=127.0.0.1
PORT=3000
```

Save and exit the editor.

---

## Test the NGINX Configuration

Before restarting NGINX, always validate the configuration.

Run:

```bash
sudo nginx -t
```

Expected output:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

If you see this output, your configuration is valid.

---

## About This Warning

While testing the configuration, you may encounter the following warning:

```text
[warn] conflicting server name "_" on 0.0.0.0:80, ignored
```

If you're following this guide for learning or demonstration purposes, you can safely ignore this warning.

However, for production deployments, it is recommended to resolve it.

I've documented the solution here:

- **[NGINX troubleshooting](../troubleshooting/nginx-warning-fix.md)**

After resolving the warning (or choosing to ignore it for this tutorial), continue to the next section.

---

## Next Step

Excellent! NGINX has now been configured to act as a reverse proxy for your Express application.

In the next section, we'll verify the deployment.