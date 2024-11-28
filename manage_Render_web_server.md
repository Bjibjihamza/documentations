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

#### Create the Server

Create a New Project Directory Create a directory for your server project
```bash
mkdir /var/www/server
cd /var/www/server
```
Initialize a Node.js Project Initialize a package.json file
```bash
npm init -y
```
Install Express.js Install Express.js, which simplifies server creation:
```bash
npm install express
```
Create the Server Code Create a file named server.js
```bash
touch server.js
```
Open server.js in your favorite editor

```bash
nano server.js
```
add the following code:
```bash
const express = require('express');
const app = express();
const PORT = 4000;

app.get('/', (req, res) => {
    res.send('Hello, World! Your server is running on localhost:4000');
});

app.listen(PORT, () => {
    console.log(`Server is running on http://localhost:${PORT}`);
});

```
Run the Server Start the server using Node.js:
```bash
node server.js
```
Access the Server Open your browser or a tool like Postman and visit:
```bash
http://localhost:4000
```
##### What Happens?
 - The server will respond with "Hello, World! Your server is running on localhost:4000" whenever you visit `/`.

#### To Stop the Server
- Press `Ctrl + C` in your terminal.

### 2. deploy your Node.js server code to GitHub
#### 2.1 If Git is not installed on your Linux system, you can install it by following these steps:
Update Your System
```bash
sudo apt update
```
Install Git
```bash
sudo apt install git -y
```
Verify the Installation
```bash
git --version
```
You should see a version number, like git version 2.x.x.

Configure Git
```bash
git config --global user.name "Your Name"
```
```bash
git config --global user.email "your-email@example.com"
```
To verify your configuration, run:
```bash
git config --list
```

#### 2.2 Prepare Your Project for GitHub
Initialize a Git Repository
Navigate to your project folder and run:
```bash
git init   
```
Create a .gitignore File
Exclude unnecessary files like node_modules. Create a .gitignore file:
```bash
touch .gitignore
```
```bash
nano .gitignore
```
Add the following to .gitignore:
```bash
node_modules
.env
```

#### 2.3 Commit Your Code
Stage Your Changes
```bash
git add .
```
Commit Your Code
```bash
git commit -m "Initial commit: Node.js server"
```

#### 2.4 Create a Repository on GitHub
   - Go to [GitHub]([https://github.com/]) and log in.
   - Click the + button in the top-right corner and select New repository.
   - Fill in the repository name and other details, then click Create repository.

#### 2.5 Link Your Local Repository to GitHub
```bash
git remote add origin https://github.com/your-username/your-repository.git
```
#### 2.6 Verify Your Code on GitHub
Go to your repository URL (e.g., `https://github.com/your-username/your-repository`). Your code should now be visible.

### 3. manage your Render web server
 #### Create a Render Account
 - If you don't have a Render account, go to [Render]([https://render.com/])  and sign up.

#### Install the Render CLI 
Install Render CLI
```bash
curl -fsSL https://render.com/install-cli | bash
```
Verify Installation
```bash
render --version
```
Login to Render , This will open a browser window to authenticate your account.
```bash
render login
```
#### Prepare Your GitHub Repository
```bash
https://github.com/your-username/your-repository.git
```
#### Deploy Your Application from GitHub
*Option 1: Using Render Dashboard (GUI)*
 - Go to the Render Dashboard: [Render Dashboard]([https://dashboard.render.com/])
 - Click on "New Web Service"
 - Select GitHub as your source.
 - Connect your GitHub account and select the repository you want to deploy.
 - Configure the service (e.g., environment, branch, etc.).
 - Deploy.

*Option 2: Deploy Using Render CLI (Manual)*
Create a Web Service on Render Using CLI: First, navigate to the directory where your GitHub repository is located (if you haven't cloned it yet):
```bash
git clone https://github.com/your-username/your-repository.git
cd your-repository
```
Create a Render Web Service: Use the following render command to create a web service:
```bash
render web create --name your-service-name --branch main --env node --build-command "npm install" --start-command "npm start" --repo https://github.com/your-username/your-repository.git
```
 `your-service-name`: Your desired service name on Render.
 `main`: The branch name you want to deploy (replace with another branch if needed).
 `node`: The environment if you're deploying a Node.js app. Modify it according to your stack.
 `npm install` and `npm start`: Your build and start commands (adjust these if using other tools like Yarn, Python, etc.).

 Deploy: Render will automatically deploy your app once you create the web service.

 Push Changes: If you make changes to your repository, you can push them directly to GitHub and Render will automatically redeploy
 ```bash
git push origin main
```

### 3. Manage and Monitor the Service
  Once your app is deployed, you can use the Render CLI to interact with it.
  - List your services:
      ```bash
      render services:list
      ```
  - Access logs:
      ```bash
      render logs <your-service-name>
      ```
  - Deploy changes (after modifying your repository locally):
      ```bash
      render deploy <your-service-name>
      ```

### 4. Use SSH (If Enabled)
If you have enabled SSH access to your Render web service, you can modify files directly on the server.
#### Add an SSH key to Render 
Once your app is deployed, you can use the Render CLI to interact with it.
  - Generate an SSH key (if you don’t already have one):
      ```bash
      ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
      ```
      Save the key (default location is ~/.ssh/id_rsa).
    
  - Add your public key to Render:
     - Go to the Render Dashboard.
     - Navigate to Account Settings > SSH Keys.
     - Add the contents of your ~/.ssh/id_rsa.pub file.
#### Connect to the Server
  - Obtain the SSH command for your web service
     - In the Render Dashboard, navigate to your service.
     - Look for the "SSH" or "Shell" option (if available).
    
  - Connect via SSH:
     - ssh username@your-service.onrender.com



 
 





