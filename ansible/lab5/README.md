# Ansible Dynamic Inventories

## Objective
The goal of this lab is to:

- Set up Ansible Dynamic Inventories to automatically discover and manage infrastructure.
- Use an Ansible Galaxy role to install Apache on target hosts.

## Steps

### 1. Set Up Dynamic Inventory
- **Create a dynamic inventory script**:
  Create a script (e.g., `dynamic_inventory.py`) that outputs JSON for inventory data.

- **Make the script executable**:
  ```bash
  chmod +x dynamic_inventory.py
  ```

- **Test the script**:
  ```bash
  ./dynamic_inventory.py
  ```

- **Verify Ansible can parse the dynamic inventory**:
  ```bash
  ansible-inventory -i dynamic_inventory.py --list
  ```

### 2. Install Ansible Galaxy Role
- **Install the geerlingguy.apache role**:
  ```bash
  ansible-galaxy role install geerlingguy.apache
  ```

### 3. Create Playbook
- **Write the playbook**:
  Create a file (e.g., `playbook.yml`) with the following content:
  
  ```yaml
  ---
  - name: Install Apache using Ansible Galaxy role
    hosts: all
    become: yes
    roles:
      - role: geerlingguy.apache
  ```

### 4. Run the Playbook
- **Execute the playbook using the dynamic inventory**:
  ```bash
  ansible-playbook -i dynamic_inventory.py playbook.yml
  ```

## Testing

### 1. Test Dynamic Inventory
- **Verify the dynamic inventory script outputs valid JSON**:
  ```bash
  ./dynamic_inventory.py
  ```

- **Check that Ansible can parse the inventory**:
  ```bash
  ansible-inventory -i dynamic_inventory.py --list
  ```

- **Test connectivity to all hosts**:
  ```bash
  ansible all -i dynamic_inventory.py -m ping
  ```
