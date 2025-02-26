# Ansible Playbook for MySQL Setup



## **Steps**

1. **Ansible Installed:**
   - Ensure Ansible is installed on your workstation. You can check the version using:
     ```bash
     ansible --version
     ```

2. **Connectivity to `servera`:**
   - Verify that `servera` is reachable from your workstation:
     ```bash
     ping servera
     ```


## **Steps to Run the Playbook**

### 1. **Create the Inventory File:**
   - Edit the Ansible inventory file to include `servera`:
     ```bash
     sudo vim /etc/ansible/hosts
     ```
   - Add the following content:
     ```ini
     [servers]
     servera
     ```



### 2. **Create the Playbook File:**
   - Create a file named `mysql_setup.yml`:
     ```bash
     nano mysql_setup.yml
     ```
   - Add the following content to the file:
     ```yaml
     ---
     - name: Install MySQL and setup ivolve database
       hosts: servera
       become: yes
       vars:
         ansible_python_interpreter: /usr/bin/python3
       vars_files:
         - secrets.yml
       tasks:
         - name: Install MySQL
           yum:
             name: mysql-server
             state: present

         - name: Start MySQL service
           service:
             name: mysqld
             state: started
             enabled: yes

         - name: Create ivolve database
           community.mysql.mysql_db:
             name: ivolve
             state: present

         - name: Create user with all privileges on ivolve DB
           community.mysql.mysql_user:
             name: "{{ db_user }}"
             password: "{{ db_password }}"
             priv: "ivolve.*:ALL"
             state: present
     ```



### 3. **Create the Vault File:**
   - Create an encrypted `secrets.yml` file using Ansible Vault:
     ```bash
     ansible-vault create secrets.yml
     ```
   - Add the following content to the file:
     ```yaml
     db_user: ivolve_user
     db_password: 
     ```
   



### 4. **Install Required Python Modules:**
   - SSH into `servera` and install the required Python module for MySQL:
     ```bash
     ssh student@servera
     sudo yum install python3-pymysql -y
     ```



### 5. **Run the Playbook:**
   - Execute the playbook with the following command:
     ```bash
     ansible-playbook --ask-vault-pass --ask-become-pass -i /etc/ansible/hosts mysql_setup.yml
     ```
   - When prompted:
     - Enter the Vault password (used to encrypt `secrets.yml`).
     - Enter the sudo password (for the user on `servera`).



### 6. **Verify the Setup:**
   - SSH into `servera`:
     ```bash
     ssh student@servera
     ```
   - Check if MySQL is running:
     ```bash
     sudo systemctl status mysqld
     ```
   - Log in to MySQL:
     ```bash
     mysql -u ivolve_user -p
     ```
   - Verify the `ivolve` database:
     ```sql
     SHOW DATABASES;
     ```
   - Verify the user's privileges:
     ```sql
     SHOW GRANTS FOR 'ivolve_user'@'localhost';
     ```


           
