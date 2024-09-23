To create an SSH user on an Ubuntu server and add the user to the sudo group, follow these steps:

### 1. **Log into your Ubuntu Server**
   If you're not already logged into the Ubuntu server, use SSH to connect to it:

   ```bash
   ssh your_username@your_server_ip
   ```

### 2. **Create a New User**
   Run the following command to create a new user. Replace `new_user` with the desired username.

   ```bash
   sudo adduser new_user
   ```

   You will be prompted to set a password and provide some additional user information (like full name). You can press `Enter` to skip the additional information if you don't want to provide it.

### 3. **Add the User to the Sudo Group**
   To grant the new user sudo privileges, use the following command:

   ```bash
   sudo usermod -aG sudo new_user
   ```

   This adds `new_user` to the `sudo` group, which allows them to run commands with superuser privileges.

### 4. **Set Up SSH Access for the New User**
   Now, you need to configure SSH access for the new user.

   - Switch to the new user:

     ```bash
     su - new_user
     ```

   - Create the `.ssh` directory in the user's home directory:

     ```bash
     mkdir ~/.ssh
     ```

   - Set the correct permissions for the `.ssh` directory:

     ```bash
     chmod 700 ~/.ssh
     ```

   - Create the `authorized_keys` file where you'll store the public key:

     ```bash
     nano ~/.ssh/authorized_keys
     ```

   - Paste your public SSH key (from your local machine) into this file and save it.

   - Set the correct permissions for the `authorized_keys` file:

     ```bash
     chmod 600 ~/.ssh/authorized_keys
     ```

### 5. **Test SSH Access for the New User**
   Open a new terminal on your local machine and connect using the new user's account:

   ```bash
   ssh new_user@your_server_ip
   ```

   If everything is set up correctly, you should be able to log in without using a password (if you added the SSH key correctly).
