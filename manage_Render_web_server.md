##  Manage  Render web server directly from your Linux terminal

### 1. create a simple server
####  Install Node.js Using NodeSource

Update the System Packages
```bash
 sudo apt update
sudo apt upgrade -y
```

Install Required Dependencies

```bash
 sudo apt install curl -y
```

Add the Node.js Repository Use NodeSource to get the latest stable version:

```bash
 curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```
Install Node.js
```bash
 sudo apt install nodejs -y
```

Verify the Installation Check the installed version of Node.js and npm:
```bash
 node -v
npm -v
```

