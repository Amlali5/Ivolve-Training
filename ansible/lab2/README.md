#  Ansible Playbooks for Web Server Configuration


## Steps

### 1. Inventory File Configuration
Define the target hosts in the Ansible inventory file (`/etc/ansible/hosts`). The `[webservers]` group includes the following servers:

```
[webservers]
servera
serverb
serverc
serverd
```



### 2. Ansible Playbook: `webserver.yml`
The playbook performs the following tasks:

- **Install Apache (`httpd`)**: Ensures the Apache package is present on the target servers.
- **Enable and Start Apache**: Ensures the Apache service is enabled and running.
- **Configure Firewall**: Allows HTTP traffic by adding the `http` service to the firewall.
- **Create a Sample Webpage**: Creates an `index.html` file in the `/var/www/html` directory with the following content:
  ```html
  <html>
  <body>
  <h1>Welcome to the Web Server!</h1>
  </body>
  </html>
  ```
- **Restart Apache**: Restarts the Apache service to apply the changes.



### 3. Execution
Run the playbook using the following command:

```bash
ansible-playbook webserver.yml --ask-become-pass
```

Example output:
- Tasks execute successfully for all servers (`servera`, `serverb`, `serverc`, `serverd`).
- Each task is marked as either `ok` or `changed`.



### 4. Verification
After the playbook execution, verify the setup:

1. **Check Apache Service Status**:
   ```bash
   sudo systemctl status httpd
   ```
   Expected output:
   - Apache service is `active (running)`.

2. **Access the Web Server**:
   Use `curl` or a web browser to check the webpage:
   ```bash
   curl http://<server-name>
   ```
   Expected output:
   ```html
   <html>
   <body>
   <h1>Welcome to the Web Server!</h1>
   </body>
   </html>
   ```

3. **Verify Firewall Configuration**:
   ```bash
   sudo firewall-cmd --list-services
   ```
   Expected output includes `http` in the list of allowed services.



