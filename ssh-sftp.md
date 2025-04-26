# SSH Key Setup for Armbian NAS with FileZilla

This document outlines the steps taken to securely connect to your Armbian NAS using SSH keys for SFTP access.

### 1. Generate SSH Key on Windows

1. Open Command Prompt or PowerShell.
2. Run the following command to generate a new SSH key pair:
   ```
   ssh-keygen
   ```
3. Accept the default file location (`C:\Users\YourName\.ssh\id_rsa`) by pressing Enter.
4. Optionally, add a passphrase for extra security or press Enter to skip it.

### 2. Copy Public Key to the NAS

Since Windows doesn't have `ssh-copy-id` natively, use the manual method:

1. Copy the public key to the NAS. You can view the public key with the following command:
   ```
   type $env:USERPROFILE\.ssh\id_rsa.pub
   ```
2. Log in to the NAS via SSH:
   ```
   ssh username@192.168.1.99
   ```
3. On the NAS, ensure the `.ssh` directory exists and has correct permissions:
   ```
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   ```
4. Open the `authorized_keys` file:
   ```
   nano ~/.ssh/authorized_keys
   ```
5. Paste the public key (from `id_rsa.pub`) into the file, then save and exit (`Ctrl+O`, Enter, `Ctrl+X`).
6. Set the correct permissions for the `authorized_keys` file:
   ```
   chmod 600 ~/.ssh/authorized_keys
   ```

### 3. Test SSH Key Login

1. Log out from the NAS:
   ```
   exit
   ```
2. Test that the SSH key works by logging in again:
   ```
   ssh username@192.168.1.99
   ```

You should now be able to log in **without entering a password** (unless you set a passphrase for the key).

### 4. Set Up FileZilla for SFTP

1. Open FileZilla and go to **File -> Site Manager**.
2. Click **New Site** and configure it as follows:

   - **Host**: Your NAS IP (e.g., `192.168.1.99`).
   - **Port**: `22` (default for SFTP/SSH).
   - **Protocol**: **SFTP - SSH File Transfer Protocol**.
   - **Logon Type**: **Key file**.
   - **User**: Your NAS username (`username`).
   - **Key file**: Browse and select your private key (`id_rsa` from `C:\Users\YourName\.ssh\id_rsa`).

3. Click **Connect**. You should now be able to transfer files securely using SFTP.

### 5. Add New Computer Access

1. On the new computer, follow the same steps to generate a new SSH key:
   ```
   ssh-keygen
   ```
2. Copy the new public key to the NAS's `~/.ssh/authorized_keys` file by pasting it into the file.
3. Test the connection from the new computer by logging in via SSH or using FileZilla.

### 6. Optional: Disable Password Authentication on the NAS for Extra Security

1. Open the SSH config file:
   ```
   sudo nano /etc/ssh/sshd_config
   ```
2. Set `PasswordAuthentication` to `no` and `PermitRootLogin` to `no`:
   ```
   PasswordAuthentication no
   PermitRootLogin no
   ```
3. Restart the SSH service:
   ```
   sudo systemctl restart ssh
   ```

This will ensure that only SSH keys can be used to authenticate, providing stronger security.

#### [Go back](readme.md#whats-next)
