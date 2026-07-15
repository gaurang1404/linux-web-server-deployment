# Run the Application as a systemd Service

So far, we've been starting the application manually using:

```bash
npm start
```

While this works well during development and testing, it isn't the recommended approach for running applications in a production environment.

If you close the terminal or your SSH session ends, the application will stop running. Additionally, the application won't automatically restart if it crashes or when the server reboots.

To solve these problems, Linux provides **systemd**.

---

## What is systemd?

**systemd** is the default service manager used by most modern Linux distributions.

It is responsible for starting, stopping, monitoring, and automatically restarting background services on the system.

Using a systemd service provides several benefits:

- Automatically starts the application when the server boots.
- Restarts the application if it crashes.
- Allows services to be managed using a consistent interface.
- Maintains application logs through the system journal.
- Makes applications easier to monitor and maintain.

For production deployments, running applications as a **systemd service** is considered a best practice.

---

## What is `systemctl`?

`systemctl` is the command-line utility used to interact with **systemd**.

It allows you to manage services on the system, including starting, stopping, restarting, enabling, disabling, and checking their status.

Throughout the remainder of this guide, we'll use `systemctl` to manage our application instead of running `npm start` manually.

---

## Create the Service File

Systemd stores custom service definitions under:

```text
/etc/systemd/system
```

Create a new service file:

```bash
sudo vi /etc/systemd/system/planet-travel.service
```

Paste the following configuration into the file:

```ini
[Unit]
Description=Ticket booking platform to book tickets to any planet out in space!
After=network.target

[Service]
Type=simple

User=planet-travel-srvacc
Group=planet-travel-srvacc

WorkingDirectory=/opt/planet-travel

ExecStart=/usr/bin/npm start

Restart=always
RestartSec=5

EnvironmentFile=/opt/planet-travel/.env

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Save the file and exit the editor.

---

## Understanding the Service File

The service file tells **systemd** how the application should be started and managed.

Some important directives include:

| Directive | Purpose |
|-----------|---------|
| `Description` | A short description of the service. |
| `After=network.target` | Ensures the service starts only after networking is available. |
| `User` / `Group` | Specifies the service account that will run the application. |
| `WorkingDirectory` | Sets the application's working directory before starting it. |
| `ExecStart` | The command used to start the application. |
| `Restart=always` | Automatically restarts the application if it exits unexpectedly. |
| `RestartSec=5` | Waits five seconds before attempting a restart. |
| `EnvironmentFile` | Loads environment variables from the specified `.env` file. |
| `StandardOutput` / `StandardError` | Sends application logs to the system journal. |
| `WantedBy=multi-user.target` | Makes the service available during the normal multi-user boot process. |

---

## Reload systemd

Whenever a new service file is created or an existing one is modified, systemd needs to reload its configuration.

Run:

```bash
sudo systemctl daemon-reload
```

---

## Enable the Service

Enable the application using systectl:

```bash
sudo systemctl enable planet-travel
```

This will make the service start automatically on boot.

---

## Start the Service

Start the application using systemctl:

```bash
sudo systemctl start planet-travel
```

Unlike `npm start`, the application is now managed by systemd.

---

## Verify the Service Status

Check whether the service started successfully:

```bash
systemctl status planet-travel
```

If everything has been configured correctly, you should see:

```text
Active: active (running)
```

---

## View Service Logs

If the service fails to start or behaves unexpectedly, you can inspect its logs using:

```bash
journalctl -u planet-travel
```

This displays the logs collected by systemd and is often the quickest way to identify configuration errors or application startup issues.

---

## Common systemd Issues

If your service isn't starting as expected, don't worry—systemd configuration mistakes are very common, even for experienced developers.

I've documented some of the most common issues and their solutions in the troubleshooting guide: [Troubleshooting systemd Configuration](../troubleshooting/systemd-config-not-working.md)

---

## Next Step

There we go, your application is now running as a **systemd service**.

In the next section, we'll configure NGINX to handle our web server instead of our Express app doing it on its own.