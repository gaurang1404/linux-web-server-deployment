# Troubleshooting Node.js Installation on CentOS

If you encounter errors after installing Node.js using `yum`, it is often caused by package dependency or version mismatches. For example, the version of Node.js available in the repository may expect a newer version of another library (such as SQLite) than what is currently installed on your system.

In such cases, it is recommended to remove the existing installation and install Node.js from the official NodeSource repository.

## Step 1: Remove the existing Node.js installation

```bash
sudo yum remove nodejs nodejs-libs nodejs-npm nodejs-docs nodejs-full-i18n
```

This removes all packages that were installed from the CentOS repositories.

## Step 2: Add the official NodeSource repository

```bash
curl -fsSL https://rpm.nodesource.com/setup_22.x | sudo bash -
```

This script:

- Adds the official NodeSource repository.
- Imports the required GPG signing key.
- Configures YUM to install Node.js from NodeSource instead of the CentOS repositories.

## Step 3: Install Node.js

```bash
sudo yum install -y nodejs
```

## Step 4: Verify the installation

Check the installed versions of Node.js and npm:

```bash
node -v
npm -v
```

Example output:

```text
v22.x.x
10.x.x
```

If both commands execute successfully, Node.js has been installed correctly.