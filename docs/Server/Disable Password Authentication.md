
To disable password authentication and add your SSH keys for secure login, follow these steps:

### 1. **Disable Password Authentication**

1. **Open the SSH Configuration File**:
   
   You will need to edit the SSH configuration file to disable password authentication:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. **Update the Configuration**:
   
   In the SSH configuration file, find and modify the following lines:

   - Disable password authentication:
   
     ```bash
     PasswordAuthentication no
     ```

   - Disable root login (optional but recommended for security):
   
     ```bash
     PermitRootLogin no
     ```

3. **Save and Exit**:
   
   Press `Ctrl + X`, then `Y`, and `Enter` to save and exit the file.

4. **Restart the SSH Service**:

   After editing the file, restart the SSH service to apply changes:

   ```bash
   sudo systemctl restart ssh
   ```

Now, only users with SSH keys will be able to log in to the server.

---

### 2. **Adding Your SSH Keys**

To add your SSH key to the new user’s account, follow these steps:

#### **Generate an SSH Key Pair (if you don’t have one)**

1. **On your local machine**, generate an SSH key if you don't already have one:

   ```bash
   ssh-keygen -t rsa -b 4096
   ```

   You can press `Enter` to accept the default file location (`~/.ssh/id_rsa`). If you want to set a passphrase for additional security, you can do so during this process.

2. **Check for an existing SSH key**:
   
   To check if you already have an SSH key, run the following:

   ```bash
   ls ~/.ssh/id_*
   ```

   If you see files like `id_rsa` and `id_rsa.pub`, that means you already have a key pair. You can skip key generation.

#### **Add the Public Key to the Server**

1. **Copy the Public Key**:

   Use the following command to copy your public key to the Ubuntu server. Replace `new_user` with your username and `your_server_ip` with your server’s IP address.

   ```bash
   ssh-copy-id new_user@your_server_ip
   ```

   This command automatically appends your public key to the server’s `~/.ssh/authorized_keys` file.

   Alternatively, you can manually add the key using:

   ```bash
   cat ~/.ssh/id_rsa.pub | ssh new_user@your_server_ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
   ```

2. **Set Permissions (If Needed)**:

   Once the key is added, ensure the correct permissions are set on the server:

   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

#### **Test the SSH Connection**

Now that the key has been added, try logging in with the new user account to test the SSH key authentication:

```bash
ssh new_user@your_server_ip
```

If everything is configured correctly, you should be able to log in without being prompted for a password.

---

### Recap of Commands:
1. **Disable Password Authentication**:
   ```bash
   sudo nano /etc/ssh/sshd_config
   # Set PasswordAuthentication no and PermitRootLogin no
   sudo systemctl restart ssh
   ```

2. **Add SSH Key**:
   ```bash
   ssh-copy-id new_user@your_server_ip
   # Or manually copy the public key:
   cat ~/.ssh/id_rsa.pub | ssh new_user@your_server_ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
   ```

Let me know if you need any further assistance with this!


# At Last
### **Disable Password Authentication**
   For added security, you can disable password-based login for SSH, allowing only key-based authentication.

   - Edit the SSH configuration file:

     ```bash
     sudo nano /etc/ssh/sshd_config
     ```

   - Look for the following lines and ensure they are set as shown:

     ```bash
     PasswordAuthentication no
     PermitRootLogin no
     ```

   - Save the file and restart the SSH service:

     ```bash
     sudo systemctl restart ssh
     ```

Now, you have successfully created a new SSH user and added them to the `sudo` group on your Ubuntu server!
