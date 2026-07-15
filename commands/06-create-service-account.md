# Create a Service Account

So far, we've been running the application using a regular user account. While this is perfectly fine for testing, it is **not** considered a best practice for production environments.

Before deploying the application as a system service, we'll create a **dedicated service account** that will be responsible for running the application.

---

## Why Use a Service Account?

A **service account** is a user account created specifically to run an application or service.

Unlike regular users, service accounts are given **only the permissions they need** to perform their intended task.

This follows the **Principle of Least Privilege**, which states that every user or service should have the minimum permissions required to function.

### Why is this important?

Imagine an attacker manages to compromise the application.

- If the application is running as an **administrator** (or another highly privileged user), the attacker could gain extensive access to the system, potentially compromising other applications or the operating system itself.
- If the application is running under a **restricted service account**, the attacker's capabilities are significantly limited because the account has very few permissions.

For this reason, production applications are almost always run using dedicated service accounts rather than administrator accounts.

---

## Create the Service Account

Run the following command:

```bash
sudo useradd --system --create-home --shell /sbin/nologin planet-travel-srvacc
```

---

## Understanding the Command

| Option | Description |
|---------|-------------|
| `--system` | Creates a system account intended for running services instead of interactive users. |
| `--create-home` | Creates a home directory for the service account. |
| `--shell /sbin/nologin` | Prevents interactive logins, ensuring the account cannot be used to log in through SSH or a terminal. |
| `planet-travel-srvacc` | The name of the service account. |

---

## Verify the Service Account

Check the account details:

```bash
id planet-travel-srvacc
```

You'll see information similar to:

```text
uid=995(planet-travel-srvacc)
gid=993(planet-travel-srvacc)
groups=993(planet-travel-srvacc)
```

This confirms that the service account has been created successfully.

---

## Transfer Ownership of the Application

Since the application will eventually run as the service account, it should also own the application files.

Change the ownership of the project directory:

```bash
sudo chown -R planet-travel-srvacc:planet-travel-srvacc /opt/planet-travel
```

The `-R` flag applies the ownership change recursively to all files and subdirectories.

---

## Verify the Ownership

Navigate to the parent directory if needed:

```bash
cd /opt
```

List the directory contents:

```bash
ls -l
```

You should see the `planet-travel` directory owned by the new service account.

Example:

```text
drwxr-xr-x  8 planet-travel-srvacc planet-travel-srvacc 4096 Jul 15 21:10 planet-travel
```

This confirms that the service account now owns the application files.

---

## Next Step

Awesome! The service account has been created, and the application files have been assigned to it.

In the next section, we'll create a **systemd service** to run the application using this service account, ensuring it starts automatically when the server boots and continues running reliably in the background.