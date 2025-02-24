# README: Configuring Sudo Privileges and Installing Nginx on Red Hat

## Steps 

### 1. Create a New Group
Run the following command to create a new group named `devgroup`:
```bash
sudo groupadd devgroup
```

### 2. Add a New User
Create a new user (e.g., `aml`):
```bash
sudo useradd aml
```

### 3. Add the User to the Group
Add the newly created user (`aml`) to the `devgroup` group:
```bash
usermod -aG devgroup aml
```

Verify the user has been added to the group:
```bash
grep devgroup /etc/group
```

### 4. Edit the `sudoers` File
Use `vim` to edit the `sudoers` file:
```bash
vim /etc/sudoers
```

Add the following line to grant `aml` the ability to run `dnf install nginx` without a password:
```plaintext
aml ALL=(ALL) NOPASSWD: /usr/bin/dnf install nginx
```

Make sure no other commands can bypass the password prompt by ensuring the line does not include `ALL` or unrestricted privileges.

Save and exit the editor.

### 5. Switch to the New User
Switch to the new user (`aml`) to test the configuration:
```bash
su - aml
```

### 6. Test the Configuration
- Run the following command to install Nginx without being prompted for the sudo password:
  ```bash
  sudo dnf install nginx
  ```
- Try running any other sudo command (e.g., `sudo dnf update`) to confirm it prompts for the password.

If you encounter any errors (e.g., missing repositories), ensure the system is updated:
```bash
sudo dnf update
```

