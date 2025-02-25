# Ping Servers Script

This script checks the availability of servers in a given subnet by pinging each IP address and reporting whether it's reachable or not.

## Usage

1. Create the script file:
   ```bash
   vim ping_servers.sh
   ```
2. Copy the script code into the file. The script iterates over all IP addresses in the specified subnet, constructs each IP address dynamically, and uses the `ping` command to check connectivity. For each address, it outputs whether the server is reachable or not based on the response. 

   Key points:
   - The subnet is defined by modifying the `SUBNET` variable.
   - The loop covers the range of hosts (0-255).
   - The `ping` command sends a single ICMP request with a 1-second timeout.
   - Results are determined by checking the exit status of the `ping` command.

   Script Code:
   ```bash
   #!/bin/bash

   # Define the subnet to scan
   SUBNET="172.16.17"

   # Loop through all possible host addresses (0-255) in the subnet
   for i in {0..255}; do
       # Construct the full IP address
       IP="$SUBNET.$i"

       # Ping the IP with a single packet and a 1-second timeout
       ping -c 1 -W 1 "$IP" > /dev/null 2>&1

       # Check the exit status of the ping command
       if [ $? -eq 0 ]; then
           echo "Server $IP is up and running"
       else
           echo "Server $IP is unreachable"
       fi
   done
   ```

3. Make it executable:
   ```bash
   chmod +x ping_servers.sh
   ```
5. Save results to a file (optional):
   ```bash
   ./ping_servers.sh > results.txt
   ```

## Example Output
- Reachable: `Server 172.16.17.1 is up and running`
- Unreachable: `Server 172.16.17.2 is unreachable`



