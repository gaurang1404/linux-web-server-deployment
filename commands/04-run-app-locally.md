## Install the Project Dependencies

From inside the project directory, install all the required Node.js packages:

```bash
sudo npm install
```

> **Note:** `npm install` (or `npm i`) reads the project's `package.json` file and installs all the dependencies required to run the application.

---

### Create the Environment File

Next, create the environment variables file:

```bash
touch .env
```

Open the file using **vi** (or **vim**):

```bash
sudo vi .env
```

Add the following configuration:

```env
ENV=development
DEV_HOST=localhost
PROD_HOST=0.0.0.0
PORT=3000
```

Save the file and exit the editor.

---

### Start the Application

Now that the dependencies have been installed and the environment file has been configured, start the application:

```bash
npm start
```

If everything has been configured correctly, the application should start successfully and begin listening on **port 3000**.

At this stage, the application is running **locally** on the server. In the next section, we'll verify that it's working as expected before making it accessible over the network.