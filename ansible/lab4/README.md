# Ansible Roles for Jenkins, Podman, and OpenShift CLI

## Project Setp

### Step 1: Create a Project Directory
```bash
mkdir ~/ansible-roles-lab
cd ~/ansible-roles-lab
```

### Step 2: Create Ansible Role Structure
Create new roles for Jenkins, Podman, and OpenShift CLI:
```bash
ansible-galaxy role init roles/jenkins
ansible-galaxy role init roles/podman
ansible-galaxy role init roles/openshift-cli
```
This will create a structure for each role inside the `roles` directory.

Structure of each role:
```
roles/
├── jenkins/
│   ├── tasks/
│   │   └── main.yml
│   ├── defaults/
│   │   └── main.yml
│   ├── handlers/
│   │   └── main.yml
├── podman/
│   └── ...
└── openshift-cli/
    └── ...
```

### Step 3: Configure Roles

#### 1. Jenkins Role
Edit the `roles/jenkins/tasks/main.yml` file:
```yaml
- name: Add Jenkins repository
  yum_repository:
    name: jenkins
    description: Jenkins Repository
    baseurl: https://pkg.jenkins.io/redhat-stable
    gpgkey: https://pkg.jenkins.io/redhat-stable/jenkins.io.key
    gpgcheck: yes

- name: Install Jenkins
  yum:
    name: jenkins
    state: present

- name: Start and enable Jenkins service
  service:
    name: jenkins
    state: started
    enabled: yes
```

#### 2. Podman Role
Edit the `roles/podman/tasks/main.yml` file:
```yaml
- name: Install Podman
  yum:
    name: podman
    state: present
```

#### 3. OpenShift CLI Role
Edit the `roles/openshift-cli/tasks/main.yml` file:
```yaml
- name: Download OpenShift CLI
  get_url:
    url: https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable/openshift-client-linux.tar.gz
    dest: /tmp/openshift-client-linux.tar.gz

- name: Extract OpenShift CLI
  unarchive:
    src: /tmp/openshift-client-linux.tar.gz
    dest: /usr/local/bin
    remote_src: yes
    mode: '0755'

- name: Clean up downloaded file
  file:
    path: /tmp/openshift-client-linux.tar.gz
    state: absent
```

### Step 4: Create and Run the Playbook

Create a main playbook file:
```bash
touch deploy-apps.yml
```
Edit the `deploy-apps.yml` file:
```yaml
---
- name: Deploy Jenkins, Podman, and OpenShift CLI
  hosts: servera
  become: yes
  roles:
    - jenkins
    - podman
    - openshift-cli
```
Run the playbook:
```bash
ansible-playbook -i inventory deploy-apps.yml --ask-become-pass
```

## Testing on `servera`

### 1. Jenkins
- **Check if Jenkins is running:**
  ```bash
  sudo systemctl status jenkins
  ```
  Ensure the output shows Jenkins as **active (running)**.

- **Access Jenkins in the browser:**
  Open the following URL in your browser:
  ```
  http://servera:8080
  ```

### 2. Podman
- **Verify the Podman installation:**
  ```bash
  podman --version
  ```
  This will display the installed Podman version.

- **Run a test container:**
  ```bash
  podman run hello-world
  ```
  The output should confirm that the container ran successfully.

### 3. OpenShift CLI (OC)
- **Check the OC version:**
  ```bash
  oc version
  ```
  This will display the OpenShift CLI version if installed correctly.


