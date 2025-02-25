#  Ansible Environment Setup

This document outlines the steps taken to set up and test Ansible 

## Step 1: Update and Install Ansible

### Update the system:
```bash
sudo dnf update -y
```

### Install Ansible:
```bash
sudo dnf install ansible -y
```

## Step 2: Configure the Inventory File

### Edit the Ansible inventory file:
```bash
sudo vim /etc/ansible/hosts
```

### Add the target servers:
```ini
[servers]
servera
serverb
serverc
serverd

[all:vars]
ansible_user=student
```

## Step 3: Test Connectivity with Ad-Hoc Commands

### Ping the servers:
```bash
ansible servers -m ping
```

#### Expected output (sample from `servera`):
```json
servera | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
*Note: This output is a sample from `servera`, similar results are expected for all servers.*

### Create a test file on all servers:
```bash
ansible servers -m file -a "path=/tmp/testfile state=touch"
```

#### Expected output (sample from `servera`):
```json
servera | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "dest": "/tmp/testfile",
    "gid": 1000,
    "group": "student",
    "mode": "00644",
    "owner": "student",
    "secontext": "unconfined_u:object_r:user_tmp_t:s0",
    "size": 0,
    "state": "file",
    "uid": 1000
}
```
*Note: This output is a sample from `servera`, similar results are expected for all servers.*


## Step 4: Verify File Creation

### List the test file on all servers:
```bash
ansible servers -m command -a "ls /tmp/testfile"
```

#### Expected output (sample from `servera`):
```bash
servera | CHANGED | rc=0 =>
/tmp/testfile
```
*Note: This output is a sample from `servera`, similar results are expected for all servers.*



