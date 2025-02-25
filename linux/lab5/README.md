# Lab 5: SSH Configurations

## Objective
Set up SSH access to another VM using public and private keys, and simplify the SSH command to just `ssh ivolve`.

## Steps

### 1. Generate SSH Keys
Run the following command to create public and private keys:
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
```
- Leave the passphrase empty for simplicity.

### 2. Copy Public Key to Remote Machine
Use the `ssh-copy-id` command to copy your public key to the remote machine:
```bash
ssh-copy-id username@remote-ip
```
Replace `username` with your remote username and `remote-ip` with the IP address of the remote machine.

### 3. Configure SSH for Simplicity
Edit the SSH configuration file:
```bash
vim ~/.ssh/config
```
Add the following configuration:
```
Host ivolve
    HostName remote-ip
    User username
    IdentityFile ~/.ssh/id_rsa
```
Replace `remote-ip` and `username` with your remote machine's IP and username.

### 4. Test the Configuration
Run the following command to test:
```bash
ssh ivolve
```
You should connect to the remote machine without specifying the username, IP, or key manually.

