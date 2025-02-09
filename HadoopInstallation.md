in this tutorial we will pure hands on installation from scratch , not relying upon proprietary/open source tools 

## SSH Setup

#### **Why Install `openssh-server`?**

The **SSH daemon (sshd)** is essential for Hadoop installations because:

1. **Communication Between Nodes** : 
	- Hadoop is a distributed system, meaning it runs on multiple machines (nodes) that need to communicate with each other.
	- SSH allows secure communication between the master node (NameNode/ResourceManager) and worker nodes (DataNodes/NodeManagers).
	
2. **Passwordless SSH**:
	- Hadoop requires **passwordless SSH** between the master and worker nodes to automate tasks like starting/stopping services and distributing jobs.
	- Without SSH, you would need to manually log in to each node to perform operations, which is impractical in a distributed environment.
	
3. **Security**:
    - SSH encrypts all communication between nodes, ensuring that sensitive data (like configuration files and job details) is transmitted securely.
    
4. **Hadoop Scripts**:
    - Hadoop's startup and shutdown scripts (e.g., `start-dfs.sh`, `start-yarn.sh`) use SSH to connect to the nodes and start/stop the required services.

---

### Steps to Install and Configure SSH

##### 1. Install SSH Daemon

```bash
sudo apt update
sudo apt install openssh-server
```

This command installs the OpenSSH server on your Ubuntu system. The OpenSSH server allows your machine to accept SSH connections from other machines. Here's a breakdown:
- **`openssh-server`**: Enables your machine to act as an SSH server.
- **`openssh-client`**: Installed by default on Ubuntu, allows your machine to act as an SSH client (to connect to other machines).

##### 2. Enable SSH Daemon

```bash
sudo systemctl enable ssh
```

##### 3. Activate (Start) SSH Daemon

```bash
sudo systemctl start ssh
```

##### 4. Check SSH Daemon Status

```bash
sudo systemctl status ssh
```

######  The output should tell you that the service is running and enabled to start on system boot:

```bash
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled) 
     Active: active (running) since ...
     ```


## **Step 3: Setup Password-less SSH Login**

Hadoop scripts (e.g., `start-dfs.sh`, `start-yarn.sh`) rely on password-less SSH to automate tasks like starting/stopping services and distributing jobs across the cluster. Follow these steps to set up password-less SSH login:

### **Commands to Set Up Password-less SSH**

1. **Generate SSH Key Pair**:
    
```bash
    cd ~
    ssh-keygen -t rsa
    ```
    
- Press `<Enter>` for all prompts to accept default settings.
- This creates a public-private key pair in `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`.
        ```
        
1. **Add Public Key to Authorized Keys**:
    
	```bash
	cat .ssh/id_rsa.pub >> .ssh/authorized_keys
	```
	
- This adds your public key to the `authorized_keys` file, allowing password-less login.
        
2. **Set Correct Permissions**:
    
```bash
    chmod 700 .ssh
    chmod 640 .ssh/authorized_keys
    ```
    
- Ensures only the owner can read/write the `.ssh` directory and `authorized_keys` file.
        
3. **Add SSH Key to SSH Agent**:
    
```bash
    ssh-add
```

- Adds your private key to the SSH agent for seamless authentication.
        
4. **Test SSH Login to Localhost**:
    
```bash
    ssh localhost
    ```
    
- The first time, you’ll see a message like:
        
```bash
        
        The authenticity of host 'localhost (127.0.0.1)' can't be established.
        ECDSA key fingerprint is SHA256:...
        Are you sure you want to continue connecting (yes/no)?
        
```

Type `yes` and press `<Enter>`.
        
- Once logged in, exit the session:
        
```bash
        exit
        ```
    
1. **Test Password-less SSH Again**:
    
```bash
    ssh localhost
    exit
```

- This time, you should log in without any prompts.
## **Step 4: Create a Folder for Hadoop and Related Projects**

To keep Hadoop and its ecosystem organized, we create a dedicated directory under `/usr/local/hadoop`. Since `/usr/local` is owned by `root`, we need to switch to the `root` user, create the directory, and assign ownership to the `hdadmin` user. We also create a group called `hdgroup` for better permission management.

### **Commands to Create the Hadoop Folder**

2. **Switch to Root User**:
    
```bash
    sudo su -
    ```
- This switches the current user to `root`.
        
3. **Create the Hadoop Directory**:
    
```bash
    mkdir /usr/local/hadoop
    ```
- This creates a directory at `/usr/local/hadoop`.
        
4. **Create a Group for Hadoop**:
    
```bash
    groupadd hdgroup
    ```
- This creates a new group called `hdgroup`.
        
1. **Assign Ownership to user (for me `hamzabji`) **:
    
```bash
    chown hamzabji:hdgroup /usr/local/hadoop
    ```
- This changes the ownership of `/usr/local/hadoop` to the `hamzabji` user and `hdgroup` group.
        
2. **Exit Root User**:
    
```bash
    exit
```    
- This returns you to the `hdadmin` user.
        
---

## **Step 5: Download Apache Hadoop**

The next step is to download the latest Hadoop distribution from the official Apache Hadoop website. As of this writing, the latest stable version is **Hadoop 3.3.6**. You can download it using a web browser or the command line.

### **Commands to Download Hadoop**

3. **Navigate to the Downloads Directory**:
    
```bash
    cd ~/Downloads
    ```
- This ensures the Hadoop tarball is downloaded to the `Downloads` directory.
        
4. **Download Hadoop Using `wget`**:
    
```bash
    wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
    ```
- This downloads the Hadoop 3.3.6 tarball from the Apache website.
        
---

## **Step 6: Install Apache Hadoop**

After downloading Hadoop, the next step is to install it by extracting the compressed tarball into the `/usr/local/hadoop` directory.

### **Commands to Install Hadoop**

5. **Navigate to the Hadoop Directory**:
    
```bash
    cd /usr/local/hadoop
    ```
- This ensures Hadoop is installed in the correct location.
        
6. **Extract the Hadoop Tarball**:
    
```bash
    tar -zxvf ~/Downloads/hadoop-3.3.6.tar.gz
    ```
- This extracts the contents of the tarball into the current directory (`/usr/local/hadoop`).
        
    - The `-zxvf` flags mean:
        - `z`: Decompress the gzip file.
        - `x`: Extract the files.
        - `v`: Verbose output (shows progress).
        - `f`: Specifies the file to extract.
            
7. **Verify the Extraction**:
    
```bash    
    ls
    ```
- You should see a directory named `hadoop-3.3.6`.
        

---

## Step 7: Setup HADOOP_HOME and Related Environment Variables

Now that Hadoop is installed, we need to set the `HADOOP_HOME` and `PATH` environment variables. This allows you to run Hadoop commands (e.g., `hdfs`, `yarn`, `start-dfs.sh`) without specifying the full path to the binaries.

### Why Set HADOOP_HOME and PATH?

- **HADOOP_HOME**:  
  - Points to the root directory of your Hadoop installation.  
  - Used by Hadoop scripts and commands to locate configuration files and binaries.

- **PATH**:  
  - Adds Hadoop's `bin` and `sbin` directories to the system's executable search path.  
  - Allows you to run Hadoop commands from anywhere in the terminal.

### Commands to Set HADOOP_HOME and PATH

8. **Switch to Root User**:
   ```bash
   sudo su -
   ```
9. **Set HADOOP_HOME**:
    
   ```bash
    echo 'export HADOOP_HOME=/usr/local/hadoop/hadoop-3.3.6' > /etc/profile.d/hadoop.sh
       ```
    - This creates a script in `/etc/profile.d` that sets the `HADOOP_HOME` environment variable for all users.
        
10. **Update PATH**:
    
     ```bash    
    echo 'export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin' >> /etc/profile.d/hadoop.sh
       ```
    - This appends Hadoop's `bin` and `sbin` directories to the `PATH` environment variable.
        
11. **Make the Script Executable**:
    
   ```bash
    chmod +x /etc/profile.d/hadoop.sh
   ```    
12. **Exit Root User**:
    
   ```bash
    exit
       ```
13. **Reload Environment Variables**:
    
   ```bash
    source /etc/profile
       ```

---

## Step 8: Setup JDK and JAVA_HOME Environment Variable

Hadoop requires Java to run. We will install OpenJDK 8 and set the `JAVA_HOME` environment variable to point to the JDK installation directory.

### Why Set JAVA_HOME?

- **Hadoop Dependency**:
    - Hadoop is written in Java and requires a JDK to run.
    - The `JAVA_HOME` variable tells Hadoop where to find the Java runtime.
        
- **Eclipse and Spark**:
    - Setting `JAVA_HOME` is also necessary for tools like Eclipse and Apache Spark, which rely on Java.
        

### Commands to Install OpenJDK and Set JAVA_HOME

14. **Install OpenJDK 8**:
     ```bash
    sudo apt-get install openjdk-8-jdk
   ```
    - When prompted, type `Y` and press `<Enter>` to continue.
        
15. **Set JAVA_HOME**:
    
	  ```bash
	    echo 'export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64' > /etc/profile.d/java.sh
       ```
    - This creates a script in `/etc/profile.d` that sets the `JAVA_HOME` environment variable for all users.
        

16. **Make the Script Executable**:
    
     ```bash
    chmod +x /etc/profile.d/java.sh
       ```
17. **Exit Root User**:
    
   ```bash
    exit
       ```
18. **Reload Environment Variables**:
    
   ```bash
    source /etc/profile
       ```
19. **Verify Java Installation**:
    
   ```bash
    java -version
       ```
       
- You should see output like:
        
   ```bash
        openjdk version "1.8.0_392"
        OpenJDK Runtime Environment (build 1.8.0_392-8u392-b07-0ubuntu1~20.04-b07)
        OpenJDK 64-Bit Server VM (build 25.392-b07, mixed mode)
           ```
           
20. **Verify JAVA_HOME**:
    
   ```bash
    echo $JAVA_HOME
       ```
       
- You should see the path to your JDK installation, e.g., `/usr/lib/jvm/java-1.8.0-openjdk-amd64`.
        
---




## Step 9: Logout and Log Back In

After setting up environment variables (`HADOOP_HOME`, `JAVA_HOME`, and `PATH`), you need to log out and log back in to ensure the changes take effect. This step ensures that the environment variables are properly set for all new terminal sessions.

### Steps to Logout and Verify Environment Variables

21. **Log Out**:
   - Click the downward-pointing arrow in the top-right corner of your desktop.
   - Select **Power Off / Logout > Logout**.

22. **Log Back In**:
   - Log in to your user account.

23. **Open a New Terminal**:
   - Start a new terminal session.

24. **Verify `HADOOP_HOME`**:
   ```bash
   echo $HADOOP_HOME
   ```
- You should see:
    
   ```bash
    /usr/local/hadoop/hadoop-3.3.6
       ```

25. **Verify `JAVA_HOME`**:
   ```bash
    echo $JAVA_HOME
       ```
    - You should see:
        
   ```bash
        /usr/lib/jvm/java-1.8.0-openjdk-amd64
           ```
26. **Verify `PATH`**:
    
   ```bash
    echo $PATH
       ```
- You should see `$HADOOP_HOME/bin` and `$HADOOP_HOME/sbin` in the output.
        
    - More specifically, you should see:
        
   ```bash
        /usr/local/hadoop/hadoop-3.3.6/bin:/usr/local/hadoop/hadoop-3.3.6/sbin:...
           ```

---

## Step 10: Make Data Directories

Hadoop requires specific directories to store filesystem metadata, data, and logs. These directories are essential for the proper functioning of Hadoop's distributed file system (HDFS).

### Why Create Data Directories?

- **NameNode (`nn`)**:
    
    - Stores metadata about the HDFS filesystem (e.g., file structure, permissions).
        
- **Secondary NameNode (`snn`)**:
    
    - Periodically merges the NameNode's metadata to prevent data loss.
        
- **DataNode (`dn`)**:
    
    - Stores the actual data blocks of files in HDFS.
        

### Commands to Create Data Directories

27. **Create NameNode Directory**:
    
   ```bash
    mkdir -p /usr/local/hadoop/data/nn
       ```
28. **Create Secondary NameNode Directory**:
    
   ```bash
    mkdir -p /usr/local/hadoop/data/snn
       ```
29. **Create DataNode Directory**:
    
   ```bash
    mkdir -p /usr/local/hadoop/data/dn
       ```
30. **Verify the Directories**:
    
   ```bash
    ls -ld /usr/local/hadoop/data/*
       ```
    - You should see:
        
   ```bash
        
        drwxr-xr-x 2 hdadmin hdgroup 4096 Oct 10 12:34 /usr/local/hadoop/data/dn
        drwxr-xr-x 2 hdadmin hdgroup 4096 Oct 10 12:34 /usr/local/hadoop/data/nn
        drwxr-xr-x 2 hdadmin hdgroup 4096 Oct 10 12:34 /usr/local/hadoop/data/snn
           ```


---


### **Step 11: Configure `core-site.xml`**

The `core-site.xml` file is one of the core configuration files in Hadoop. It contains settings that are common across the entire Hadoop ecosystem. In this step, you are configuring the `fs.default.name` property, which specifies the default filesystem and the location of the NameNode.

#### **Steps to Configure:**

31. **Navigate to the Hadoop Configuration Directory:**
    
    - Change to the Hadoop configuration directory using the command:
        
        ```bash
        cd /usr/local/hadoop/hadoop-3.3.6/etc/hadoop
                ```
    - This is where all Hadoop configuration files are stored.
        
32. **Edit the `core-site.xml` File:**
    
    - Open the `core-site.xml` file in a text editor (e.g., `nano`):
        
        ```bash
        nano core-site.xml
                ```
    - The file will initially contain only empty `<configuration>` tags.
        
33. **Add the `fs.default.name` Property:**
    
    - Replace the empty `<configuration>` tags with the following configuration:

    ```bash
        <configuration>
          <property>
            <name>fs.default.name</name>
            <value>hdfs://localhost:9000</value>
          </property>
        </configuration>
     ```

        
    - **Explanation:**
        
        - `<name>fs.default.name</name>`: This property defines the default filesystem for Hadoop. It specifies the protocol (`hdfs`), the host (`localhost`), and the port (`9000`) where the NameNode is running.
            
        - `<value>hdfs://localhost:9000</value>`: This is the URI for the NameNode. It tells Hadoop where to find the NameNode, which is the central metadata server for HDFS (Hadoop Distributed File System).
            
34. **Save and Exit:**
    
    - Save the changes and exit the editor. `ctl+x` ,`y` than `ENTER`
        

---

### **Step 12: Configure `hdfs-site.xml`**

The `hdfs-site.xml` file contains configuration settings specific to HDFS (Hadoop Distributed File System). In this step, you are configuring properties related to replication, NameNode, Secondary NameNode, and DataNode directories.

#### **Steps to Configure:**

35. **Navigate to the Hadoop Configuration Directory:**
    
    - Ensure you are in the Hadoop configuration directory:
        
        ```bash
         cd /usr/local/hadoop/hadoop-3.3.6/etc/hadoop
                ```
36. **Edit the `hdfs-site.xml` File:**
    
    - Open the `hdfs-site.xml` file in a text editor:
        
        ```bash
        nano hdfs-site.xml
                ```
    - The file will initially contain only empty `<configuration>` tags.
        
37. **Add the Required Properties:**
    
    - Replace the empty `<configuration>` tags with the following configuration:
        ```bash
        
        <configuration>
          <property>
            <name>dfs.replication</name>
            <value>1</value>
          </property>
          <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:/usr/local/hadoop/data/nn</value>
          </property>
          <property>
            <name>fs.checkpoint.dir</name>
            <value>file:/usr/local/hadoop/data/snn</value>
          </property>
          <property>
            <name>fs.checkpoint.edits.dir</name>
            <value>file:/usr/local/hadoop/data/snn</value>
          </property>
          <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:/usr/local/hadoop/data/dn</value>
          </property>
        </configuration>
        
           ```
        
    - **Explanation of Properties:**
        
        - **`dfs.replication`**: This property sets the replication factor for HDFS. In a single-node setup, replication is not needed, so it is set to `1`.
            
        - **`dfs.namenode.name.dir`**: This specifies the directory where the NameNode stores its metadata (e.g., file system image and edit logs).
            
        - **`fs.checkpoint.dir`**: This is the directory where the Secondary NameNode stores checkpoints of the file system metadata.
            
        - **`fs.checkpoint.edits.dir`**: This is the directory where the Secondary NameNode stores edits logs during checkpointing.
            
        - **`dfs.datanode.data.dir`**: This specifies the directory where the DataNode stores actual data blocks.
            
38. **Save and Exit:**
        

---

### **Key Points to Include in Documentation:**

39. **Purpose of Each File:**
    
    - `core-site.xml`: Contains global settings for the Hadoop cluster.
        
    - `hdfs-site.xml`: Contains HDFS-specific settings.
        
40. **Configuration Details:**
    
    - Explain the purpose of each property being configured.
        
    - Highlight the importance of setting `dfs.replication` to `1` in a single-node setup.
        
41. **Directory Structure:**
    
    - Emphasize the need to create the directories specified in `hdfs-site.xml` (e.g., `/usr/local/hadoop/data/nn`, `/usr/local/hadoop/data/snn`, `/usr/local/hadoop/data/dn`) before starting Hadoop.
        
42. **Common Mistakes:**
    
    - Warn against incorrect file paths or typos in property names.
        
    - Remind users to remove the original empty `<configuration>` tags.
        
43. **Next Steps:**
    
    - After configuring these files, the next steps typically involve formatting the NameNode and starting the Hadoop services.
        

---

### **Step 13: Configure `mapred-site.xml`**

The `mapred-site.xml` file is used to configure the MapReduce framework in Hadoop. In this step, you are configuring Hadoop to use YARN as the MapReduce engine and setting environment variables for the MapReduce Application Master, map tasks, and reduce tasks.

#### **Steps to Configure:**

44. **Navigate to the Hadoop Configuration Directory:**
    
    - Change to the Hadoop configuration directory using the command:
        
        ```bash     
        cd /usr/local/hadoop/hadoop-3.3.6/etc/hadoop
                ```
                
45. **Edit the `mapred-site.xml` File:**
    
    - Open the `mapred-site.xml` file in a text editor (e.g., `vi`):
        
        ```bash
        nano mapred-site.xml
                ```
46. **Add the Required Properties:**
    
    - Replace the empty `<configuration>` tags with the following configuration:
        
        ```bash
        <configuration>
          <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
          </property>
          <property>
            <name>yarn.app.mapreduce.am.env</name>
            <value>HADOOP_MAPRED_HOME=/usr/local/hadoop/hadoop-3.3.6</value>
          </property>
          <property>
            <name>mapreduce.map.env</name>
            <value>HADOOP_MAPRED_HOME=/usr/local/hadoop/hadoop-3.3.6</value>
          </property>
          <property>
            <name>mapreduce.reduce.env</name>
            <value>HADOOP_MAPRED_HOME=/usr/local/hadoop/hadoop-3.3.6</value>
          </property>
        </configuration>
        
                ```
                
        
    - **Explanation of Properties:**
        
        - **`mapreduce.framework.name`**: This property specifies the runtime framework for executing MapReduce jobs. Setting it to `yarn` means that YARN will be used as the MapReduce engine.
            
        - **`yarn.app.mapreduce.am.env`**: This sets the environment variable `HADOOP_MAPRED_HOME` for the MapReduce Application Master (AM). It points to the Hadoop installation directory.
            
        - **`mapreduce.map.env`**: This sets the environment variable `HADOOP_MAPRED_HOME` for map tasks.
            
        - **`mapreduce.reduce.env`**: This sets the environment variable `HADOOP_MAPRED_HOME` for reduce tasks.
            
47. **Save and Exit:**
    
    - Save the changes and exit the editor.
        

---

### **Step 14: Configure `yarn-site.xml`**

The `yarn-site.xml` file is used to configure YARN, which is the resource management layer of Hadoop. In this step, you are configuring YARN to enable the MapReduce shuffle service, which is necessary for MapReduce jobs to run properly.

#### **Steps to Configure:**

48. **Navigate to the Hadoop Configuration Directory:**
    
    - Ensure you are in the Hadoop configuration directory:
        
        ```bash
        
        cd /usr/local/hadoop/hadoop-3.3.6/etc/hadoop
                ```
49. **Edit the `yarn-site.xml` File:**
    
    - Open the `yarn-site.xml` file in a text editor:
        
        ```bash        
        nano yarn-site.xml
                ```
50. **Add the Required Properties:**
    
    - Replace the empty `<configuration>` tags with the following configuration:
        
        ```bash
        <configuration>
          <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
          </property>
          <property>
            <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
            <value>org.apache.hadoop.mapred.ShuffleHandler</value>
          </property>
        </configuration>
        
                ```
        
    - **Explanation of Properties:**
        
        - **`yarn.nodemanager.aux-services`**: This property specifies the auxiliary services that the NodeManager should implement. For MapReduce, the `mapreduce_shuffle` service is required to handle the shuffle phase of MapReduce jobs.
            
        - **`yarn.nodemanager.aux-services.mapreduce.shuffle.class`**: This property specifies the class that implements the shuffle service. The `org.apache.hadoop.mapred.ShuffleHandler` class is the default implementation.
            
51. **Save and Exit:**
    
    - Save the changes and exit the editor.
        

---

### **Key Points to Include in Documentation:**

52. **Purpose of Each File:**
    
    - `mapred-site.xml`: Configures the MapReduce framework and its integration with YARN.
        
    - `yarn-site.xml`: Configures YARN, the resource management layer of Hadoop.
        
53. **Configuration Details:**
    
    - Explain the purpose of each property being configured.
        
    - Highlight the importance of setting `mapreduce.framework.name` to `yarn` for using YARN as the MapReduce engine.
        
54. **Shuffle Service:**
    
    - Emphasize the role of the `mapreduce_shuffle` service in MapReduce jobs and why it needs to be configured in `yarn-site.xml`.
        
55. **Common Mistakes:**
    
    - Warn against incorrect property names or values.
        
    - Remind users to remove the original empty `<configuration>` tags.
        
56. **Next Steps:**
    
    - After configuring these files, the next steps typically involve starting the Hadoop services (e.g., NameNode, DataNode, ResourceManager, NodeManager).




### **Step 15: Modify Java Heap Sizes and a Few Configurations**

In this step, you configure Java heap sizes and other environment variables to optimize Hadoop for a workstation with limited resources. Additionally, you suppress a warning related to the `NativeCodeLoader`.

#### **Steps to Configure:**

57. **Navigate to the Hadoop Configuration Directory:**
    
    - Change to the Hadoop configuration directory:
        
        ```bash
        cd /usr/local/hadoop/hadoop-3.3.6/etc/hadoop
                ```
58. **Edit the `hadoop-env.sh` File:**
    
    - Open the `hadoop-env.sh` file in a text editor:
        
        ```bash
        nano hadoop-env.sh
                ```
    - Add or modify the following lines to set the Java heap sizes and Hadoop options:
        
        ```bash
        export HADOOP_HEAPSIZE_MAX="500"
        export HADOOP_HEAPSIZE_MIN="250"
        export HADOOP_OPTS="-XX:-PrintWarnings -Djava.net.preferIPv4Stack=true"
            ```
            
    - **Explanation:**
        
        - **`HADOOP_HEAPSIZE_MAX`**: Sets the maximum heap size (in MB) for Hadoop processes. Here, it is set to 500 MB.
            
        - **`HADOOP_HEAPSIZE_MIN`**: Sets the minimum heap size (in MB) for Hadoop processes. Here, it is set to 250 MB.
            
        - **`HADOOP_OPTS`**: Adds additional Java options. In this case:
            
            - `-XX:-PrintWarnings`: Disables printing of certain warnings.
                
            - `-Djava.net.preferIPv4Stack=true`: Ensures Hadoop uses IPv4 instead of IPv6.
                
59. **Edit the `log4j.properties` File:**
    
    - Open the `log4j.properties` file in a text editor:
        
        ```bash
        nano log4j.properties
                ```
    - Add the following line to suppress the `NativeCodeLoader` warning:
        
        ```bash
        log4j.logger.org.apache.hadoop.util.NativeCodeLoader=ERROR
                ```
    - **Explanation:**
        
        - This line prevents the following warning from appearing in the logs:
            
            ```bash
            WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
                    ```
                    
        - The warning occurs because Hadoop cannot find native libraries for compression. However, this does not affect functionality, as Hadoop falls back to Java-based implementations.
            
60. **Save and Exit:**
    
    - Save the changes and exit the editor.
        

---

### **Step 16: Format HDFS**

Before starting the HDFS NameNode, you must format the HDFS filesystem. This step initializes the directory specified in `dfs.namenode.name.dir` (configured in `hdfs-site.xml`) and prepares it for use.

#### **Steps to Format HDFS:**

61. **Run the Format Command:**
    
    - Execute the following command to format the NameNode:
        
        ```bash
        
        $ hdfs namenode -format
                ```
                
    - **Explanation:**
        
        - This command initializes the directory specified in `dfs.namenode.name.dir` (e.g., `/usr/local/hadoop/data/nn`).
            
        - Formatting destroys any existing data in the directory and creates a new filesystem.
            
62. **Expected Output:**
    
    - If the formatting is successful, you will see output similar to the following:
        
        ```bash
        Storage directory /usr/local/hadoop/data/nn has been successfully formatted.
                ```
                
    - If the directory is already formatted, you may see a warning. You can proceed if you are sure you want to overwrite the existing data.
        
63. **Important Notes:**
    
    - Formatting should only be done once during the initial setup or if you want to reset the HDFS filesystem.
        
    - Formatting deletes all existing data in the NameNode directory, so ensure you have backups if necessary.
        

---

### **Key Points to Include in Documentation:**

64. **Purpose of Heap Size Configuration:**
    
    - Explain that heap sizes are adjusted to optimize Hadoop for workstations with limited resources.
        
    - Highlight the importance of setting appropriate heap sizes to avoid out-of-memory errors.
        
65. **Suppressing Warnings:**
    
    - Explain the purpose of the `NativeCodeLoader` warning and why it can be safely suppressed.
        
66. **Formatting HDFS:**
    
    - Emphasize that formatting is a one-time step required to initialize the HDFS filesystem.
        
    - Warn users that formatting will delete all existing data in the NameNode directory.
        
67. **Common Mistakes:**
    
    - Remind users to uncomment lines in `hadoop-env.sh` by removing the `#` symbol.
        
    - Ensure the `dfs.namenode.name.dir` directory exists before formatting.
        
68. **Next Steps:**
    
    - After formatting HDFS, the next steps typically involve starting the Hadoop services (e.g., NameNode, DataNode, ResourceManager, NodeManager).



### **Step 17: Start the HDFS NameNode Service**

The NameNode is the central metadata server for HDFS. It manages the filesystem namespace and regulates access to files by clients.

#### **Steps to Start the NameNode:**

69. **Navigate to the Hadoop `sbin` Directory:**
    
    - Change to the `sbin` directory where Hadoop scripts are located:
        
        ```bash
        cd /usr/local/hadoop/hadoop-3.3.6/sbin
                ```
70. **Start the NameNode Service:**
    
    - Use the following command to start the NameNode:
        
        ```bash
	    hdfs --daemon start namenode
                ```
    - **Explanation:**
        
        - The `hdfs --daemon start` command is used to start HDFS daemons in Hadoop 3.x.
            
        - If the NameNode starts successfully, it will create log files in the `logs` directory.
            
71. **Verify the Logs:**
    
    - Check the NameNode log file to ensure there are no errors:
        ```bash
        view /usr/local/hadoop/hadoop-3.3.6/logs/hadoop-<username>-namenode-<hostname>.log
                ```
                
    - Replace `<username>` and `<hostname>` with your actual username and hostname.
        

---

### **Step 18: Start the HDFS SecondaryNameNode Service**

The SecondaryNameNode is responsible for periodic checkpointing of the NameNode's metadata.

#### **Steps to Start the SecondaryNameNode:**

72. **Start the SecondaryNameNode Service:**
    
    - Use the following command to start the SecondaryNameNode:
        
        ```bash
        hdfs --daemon start secondarynamenode
                ```
73. **Verify the Logs:**
    
    - Check the SecondaryNameNode log file to ensure there are no errors:
        
        ```bash
        view /usr/local/hadoop/hadoop-3.6/logs/hadoop-<username>-secondarynamenode-<hostname>.log
                ```

---

### **Step 19: Start the HDFS DataNode Service**

The DataNode stores the actual data blocks in HDFS.

#### **Steps to Start the DataNode:**

74. **Start the DataNode Service:**
    
    - Use the following command to start the DataNode:
        
        ```bash
    hdfs --daemon start datanode
                ```
76. **Verify the Logs:**
    
    - Check the DataNode log file to ensure there are no errors:
        
        ```bash
    view /usr/local/hadoop/hadoop-3.6/logs/hadoop-<username>-datanode-<hostname>.log
                ```

---

### **Step 20: Verify the HDFS Services/Daemons**

After starting the HDFS services, verify that all daemons are running.

#### **Steps to Verify:**

77. **Use the `jps` Command:**
    
    - Run the `jps` command to list all running Java processes:
        
        ```bash
         jps
                ```
    - Expected output:
        
        ```bash
        
        15015 NameNode
        15140 SecondaryNameNode
        15214 DataNode
        15335 Jps
                ```
    - If any of the processes (NameNode, SecondaryNameNode, DataNode) are missing, check the corresponding log files for errors.
        
78. **Check Log Files:**
    
    - Inspect the log files for any issues if a service fails to start:
        
        ```bash
         view /usr/local/hadoop/hadoop-3.6/logs/hadoop-<username>-<service>-<hostname>.log
                ```

---

### **Step 21: Start the YARN ResourceManager Service**

The ResourceManager is the central authority that manages resources and schedules applications in YARN.

#### **Steps to Start the ResourceManager:**

79. **Start the ResourceManager Service:**
    
    - Use the following command to start the ResourceManager:
        
        ```bash
        yarn --daemon start resourcemanager
                ```
80. **Verify the Logs:**
    
    - Check the ResourceManager log file to ensure there are no errors:
        
        ```bash
        
        view /usr/local/hadoop/hadoop-3.6/logs/yarn-<username>-resourcemanager-<hostname>.log
                ```

---

### **Step 22: Start the YARN NodeManager Service**

The NodeManager is responsible for managing resources on individual nodes and executing tasks.

#### **Steps to Start the NodeManager:**

81. **Start the NodeManager Service:**
    
    - Use the following command to start the NodeManager:
        
        ```bash
         yarn --daemon start nodemanager
                ```
82. **Verify the Logs:**
    
    - Check the NodeManager log file to ensure there are no errors:
        
        ```bash
        $ view /usr/local/hadoop/hadoop-3.6/logs/yarn-<username>-nodemanager-<hostname>.log
               ``` 

---

### **Step 23: Start the MapReduce JobHistoryServer Service**

The JobHistoryServer stores historical information about MapReduce jobs.

#### **Steps to Start the JobHistoryServer:**

83. **Start the JobHistoryServer Service:**
    
    - Use the following command to start the JobHistoryServer:
        
        ```bash
        mapred --daemon start historyserver
                ```
84. **Verify the Logs:**
    
    - Check the JobHistoryServer log file to ensure there are no errors:
        
        ```bash
        $ view /usr/local/hadoop/hadoop-3.6/logs/mapred-<username>-historyserver-<hostname>.log
        
        ```
---

### **Key Points to Include in Documentation:**

85. **Purpose of Each Service:**
    
    - Explain the role of each service (NameNode, SecondaryNameNode, DataNode, ResourceManager, NodeManager, JobHistoryServer).
        
86. **Verification:**
    
    - Emphasize the importance of verifying that all services are running using the `jps` command.
        
87. **Log Files:**
    
    - Highlight the location of log files and how to use them for troubleshooting.
        
88. **Common Issues:**
    
    - Warn users about common issues such as port conflicts or insufficient memory.
        
89. **Next Steps:**
    
    - After starting all services, the next steps typically involve running a sample MapReduce job to verify the setup.



### **Step 24: Verify All the Services/Daemons Are Running**

After starting all the Hadoop services, you need to verify that they are running correctly.

#### **Steps to Verify:**

90. **Use the `jps` Command:**
    
    - Run the `jps` command to list all running Java processes:
        
        ```bash  
         jps
                ```
    - Expected output:
        ```bash
        5783 NameNode
        5874 DataNode
        6218 SecondaryNameNode
        25584 ResourceManager
        25749 NodeManager
        9417 JobHistoryServer
        27779 Jps
                ```
    - **Explanation:**
        
        - **NameNode**: Manages the HDFS metadata.
            
        - **DataNode**: Stores the actual data blocks.
            
        - **SecondaryNameNode**: Performs periodic checkpointing of the NameNode's metadata.
            
        - **ResourceManager**: Manages resources and schedules applications in YARN.
            
        - **NodeManager**: Manages resources on individual nodes and executes tasks.
            
        - **JobHistoryServer**: Stores historical information about MapReduce jobs.
            
91. **Check Log Files for Missing Services:**
    
    - If any service is missing, check the corresponding log file for errors:
        
        ```bash
        $ view /usr/local/hadoop/hadoop-3.6/logs/<service>-<username>-<hostname>.log
        
        ```
---

### **Step 25: Check the HDFS NameNode Web UI**

The NameNode provides a web interface for monitoring the HDFS filesystem.

#### **Steps to Access:**

92. **Open the NameNode Web UI:**
    
    - Use a web browser to access the NameNode web UI:
        
        ```bash
        
        google-chrome http://localhost:9870
                ```
    - Alternatively, you can use any browser and navigate to `http://localhost:9870`.
        
93. **What to Check:**
    
    - Verify the health of the HDFS filesystem.
        
    - Check the storage capacity and usage.
        
    - View the list of DataNodes and their status.
        

---

### **Step 26: Check the YARN ResourceManager Web UI**

The ResourceManager provides a web interface for monitoring YARN applications and resource utilization.

#### **Steps to Access:**

94. **Open the ResourceManager Web UI:**
    
    - Use a web browser to access the ResourceManager web UI:
        
        ```bash
         google-chrome http://localhost:8088
                ```
    - Alternatively, navigate to `http://localhost:8088` in any browser.
        
95. **What to Check:**
    
    - View the list of running and completed applications.
        
    - Check resource utilization (memory, CPU, etc.).
        
    - Monitor the status of NodeManagers.
        

---

### **Step 27: Check the MapReduce JobHistoryServer Web UI**

The JobHistoryServer provides a web interface for viewing historical information about MapReduce jobs.

#### **Steps to Access:**

96. **Open the JobHistoryServer Web UI:**
    
    - Use a web browser to access the JobHistoryServer web UI:
        
        ```bash
        google-chrome http://localhost:19888
                ```
    - Alternatively, navigate to `http://localhost:19888` in any browser.
        
97. **What to Check:**
    
    - View details of completed MapReduce jobs.
        
    - Check job statistics, such as execution time and resource usage.
        

---

### **Step 28: Verify HDFS by Performing HDFS Operations**

To ensure HDFS is functioning correctly, you can perform basic file operations.

#### **Steps to Verify:**

98. **Download a Sample File:**
    
    - Download the "Moby Dick" book as a text file:
        
        ```bash
	    cd ~/Downloads
        wget http://www.gutenberg.org/files/2701/2701-0.txt
                ```
99. **Create a Directory in HDFS:**
    
    - Create a directory called `/in` in HDFS:
        
        ```bash
         hdfs dfs -mkdir /in
               ``` 
100. **Copy the File to HDFS:**
    
- Copy the downloaded file to the `/in` directory in HDFS:
        
    ```bash
        hdfs dfs -copyFromLocal ~/Downloads/2701-0.txt /in/mobydick.txt
                ```
101. **List the Contents of the Directory:**
    
- Verify that the file has been copied successfully:
        
    ```bash
        
        hdfs dfs -ls /in
        
        ```
---

### **Step 29: Verify YARN by Submitting and Executing a MapReduce Job**

To ensure YARN and MapReduce are functioning correctly, you can submit a sample MapReduce job.

#### **Steps to Verify:**

102. **Run a Word Count Job:**
    
- Use the `hadoop-mapreduce-examples` jar to run a word count job on the "Moby Dick" file:
        
    ```bash
         yarn jar /usr/local/hadoop/hadoop-3.3.6/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar wordcount /in /out
                ```
                
                
    - **Explanation:**
        
        - The `wordcount` job reads the input file (`/in/mobydick.txt`) and writes the output to the `/out` directory in HDFS.
            
103. **View the Output:**
    
- Once the job completes, view the output:
    ```bash
        hdfs dfs -cat /out/part-r-00000
                ```
    - The output will show the frequency of each word in the "Moby Dick" book.
        

---

### **Key Points to Include in Documentation:**

104. **Verification of Services:**
    
    - Emphasize the importance of verifying all services using the `jps` command.
        
105. **Web UIs:**
    
    - Explain the purpose of each web UI and how to use them for monitoring.
        
106. **HDFS Operations:**
    
    - Highlight the steps to perform basic HDFS operations for verification.
        
107. **MapReduce Job:**
    
    - Explain how to submit and monitor a sample MapReduce job to verify YARN functionality.
        
108. **Troubleshooting:**
    
    - Provide guidance on checking log files if any service fails to start or a job fails to execute.
