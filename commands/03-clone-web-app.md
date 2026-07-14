# Clone the Application

Now that you've successfully connected to the **Application Server**, the next step is to clone the application repository and verify that it runs correctly on the server.

Before deploying an application as a service, it's always a good practice to ensure it builds and runs successfully in a normal user environment.

## About the Demo Application

For this guide, I'll be using a sample application that I developed called **Planet Travel**. It's a simple Node.js application that lets users book tickets to planets in space.

If you'd like to explore the source code or understand how the application is built, you can find it here: [Planet Travel GitHub Repository](https://github.com/gaurang1404/planet-travel)

> **Note:** You don't have to use the same application to follow this guide. Feel free to use your own application instead. Just ensure that you adapt the installation and configuration steps accordingly, as the remainder of this guide will use **Planet Travel** as the reference application.
---

## Ensure You Have Sudo Privileges

You'll need **sudo** privileges to install the required software packages.

### Check the Current User

Run:

```bash
whoami
```

Example:

```text
gaurang
```

### Verify User Privileges

To view the groups and permissions assigned to a user, run:

```bash
id <username>
```

Example:

```bash
id gaurang
```

If your current user does not have the necessary privileges, you have two options:

- Switch to another user that has sudo access.

```bash
su <username>
```

- Alternatively, update the `/etc/sudoers` file and grant your user sudo privileges.

> **Note:** For this guide, we'll use a regular user account to clone and test the application. Later, we'll create a dedicated **service account** to run the application securely as a background service.

---

## Verify the Package Manager

Since this guide uses **CentOS** (a Red Hat-based Linux distribution), we'll use the **YUM** package manager.

Verify that YUM is installed:

```bash
which yum
```

Example output:

```text
/usr/bin/yum
```

---

## Install Git and Node.js

Install the required packages:

```bash
sudo yum install git nodejs -y
```

---

## Verify the Installation

### Git

```bash
git --version
```

Example:

```text
git version 2.52.0
```

### Node.js

```bash
node --version
```

Example:

```text
v22.23.1
```

### npm

```bash
npm --version
```

Example:

```text
10.9.8
```

> **Note:** These are the versions used while creating this guide. Using the same versions is recommended if possible, as it minimizes the chances of running into version-specific issues.

---

## Having Trouble Installing Node.js?

If the Node.js installation fails, refer to the troubleshooting guide:

- [`Cannot Install NodeJs`](../troubleshooting/cannot-install-nodejs.md)

Resolve the issue before continuing.

---

## Clone the Repository

By convention, software that is **manually installed** (i.e., not managed by the operating system's default package repositories) is often stored under the `/opt` directory.

We'll follow the same convention and clone the project there.

Navigate to `/opt`:

```bash
cd /opt
```

Clone the repository:

```bash
git clone https://github.com/gaurang1404/planet-travel.git
```

Move into the project directory:

```bash
cd planet-travel
```

---

## Verify the Project Structure

Display the directory structure using:

```bash
tree
```

This allows you to verify that the repository has been cloned successfully and gives you a quick overview of the application's folder structure.

---

## Next Step

The application source code is now available on your server.

Great! In the next section, we'll install the project dependencies and verify that the application runs successfully before deploying it as a production service.