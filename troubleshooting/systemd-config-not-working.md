# Troubleshooting: systemd Service Configuration Not Working

This guide covers some of the most common mistakes developers make while creating and configuring **systemd service files**.

If your service fails to start or keeps crashing, don't panic, most issues can be identified quickly by checking the service status and logs.

---

# Step 1: Check the Service Status

The first thing you should do is check whether systemd was able to start the service.

```bash
systemctl status <service-name>
```

Example:

```bash
systemctl status planet-travel
```

This command usually tells you whether:

- The service is running.
- The service exited unexpectedly.
- The configuration file contains syntax errors.
- The executable could not be found.

---

# Step 2: Check the Logs

If the status isn't obvious, inspect the service logs.

```bash
journalctl -u <service-name>
```

Example:

```bash
journalctl -u planet-travel
```

Most startup problems can be diagnosed from these logs.

---

# Common Mistakes

## 1. Forgot to Reload systemd

After creating or editing a service file, systemd doesn't automatically detect the changes.

Always run:

```bash
sudo systemctl daemon-reload
```

Otherwise, systemd continues using the old configuration.

---

## 2. Incorrect WorkingDirectory

Example:

```ini
WorkingDirectory=/opt/planet-travel
```

If this directory does not exist, the service will fail before it even starts.

Verify it:

```bash
ls /opt/planet-travel
```

---

## 3. Wrong ExecStart Path

A very common mistake is using an incorrect executable path.

Example:

```ini
ExecStart=/usr/bin/npm start
```

Verify that the executable exists:

```bash
which npm
```

Example output:

```text
/usr/bin/npm
```

If `which npm` returns a different path, update `ExecStart` accordingly.

---

## 4. Wrong User or Group

Example:

```ini
User=planet-travel-srvacc
Group=planet-travel-srvacc
```

If the user does not exist, systemd cannot start the service.

Verify:

```bash
id planet-travel-srvacc
```

---

## 5. Application Files Owned by the Wrong User

The service account must have permission to access the application files.

Check ownership:

```bash
ls -l /opt
```

Example:

```text
drwxr-xr-x 7 planet-travel-srvacc planet-travel-srvacc planet-travel
```

If ownership is incorrect:

```bash
sudo chown -R planet-travel-srvacc:planet-travel-srvacc /opt/planet-travel
```

---

## 6. Missing Environment File

Example:

```ini
EnvironmentFile=/opt/planet-travel/.env
```

Verify that it exists:

```bash
ls -la /opt/planet-travel
```

If the file is missing, recreate it.

---

## 7. Incorrect Environment Variables

Even if `.env` exists, incorrect values can prevent the application from starting.

For example:

```env
PORT=
```

or

```env
ENV=dev
```

when the application expects:

```env
ENV=development
```

Always verify the contents of your environment file.

---

## 8. Application Doesn't Start Manually

Before blaming systemd, verify that the application itself works.

Run:

```bash
cd /opt/planet-travel
npm start
```

If this command fails, fix the application first.

Systemd cannot fix application errors.

---

## 9. Forgot to Open the Firewall Port

The service may be running correctly but still be inaccessible.

Check:

```bash
sudo firewall-cmd --list-all
```

Verify that your application port appears under:

```text
ports:
```

Example:

```text
ports: 3000/tcp
```

---

## 10. Application Listening Only on localhost

If Express is listening on:

```text
localhost
```

or

```text
127.0.0.1
```

only the local machine can access it.

For external access, configure the application to listen on:

```text
0.0.0.0
```

---

## 11. Forgot to Restart the Service

If you modify:

- the application code,
- the `.env` file,
- or the service file,

the running service won't automatically pick up the changes.

Restart it:

```bash
sudo systemctl restart planet-travel
```

If you modified the service file itself, remember to reload systemd first:

```bash
sudo systemctl daemon-reload
sudo systemctl restart planet-travel
```

---

## 12. Port Already in Use

Sometimes another application is already using the configured port.

Check:

```bash
sudo ss -tulpn | grep 3000
```

If another process is listening on the port, either stop it or configure your application to use a different port.

