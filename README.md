ufw-cheat-sheet

# Securing Your Admin Dashboard with UFW and VPN

This guide will walk you through the process of securing an Nginx admin dashboard so it's only accessible via VPN, using UFW (Uncomplicated Firewall) to implement OS-level security.

## Overview

We'll use a methodical approach to:
1. Install and configure UFW
2. Identify the VPN client subnet CIDR
3. Gradually lock down the server while testing at each step
4. Secure both SSH and the admin dashboard to only be accessible through VPN

## Prerequisites

- Ubuntu/Debian-based server running your Nginx admin dashboard on port 5000
- A separate VPN server already configured
- A client computer that can connect to your VPN

## Step 1: Check if UFW is Installed and Enabled

First, verify if UFW is installed on your system:

```bash
which ufw
```

Or check using the package manager:

```bash
dpkg -l | grep ufw
```

If UFW is not installed, install it:

```bash
sudo apt update
sudo apt install ufw
```

Check the current status of UFW:

```bash
sudo ufw status
```

If enabled, you'll see "Status: active" along with any configured rules.

## Step 2: Open Access Initially for Testing

Start with an open configuration to establish baseline connectivity:

```bash
# Reset UFW to default settings (if previously configured)
sudo ufw reset

# Set default policies to allow everything initially
sudo ufw default allow incoming
sudo ufw default allow outgoing

# Enable UFW with the open configuration
sudo ufw enable
```

## Step 3: Identify Your VPN Client Subnet

1. Connect to your VPN from your client computer
2. SSH into your server
3. Check the SSH connection details to identify the client IP:

```bash
# While connected via SSH, run:
who
netstat -tn | grep ":22"
# or
ss -tn | grep ":22"
```

4. Make note of your IP address while connected to VPN

5. Test accessing the admin dashboard (http://your-server-ip:5000) via web browser while on VPN

6. Determine the CIDR subnet of your VPN client connections (e.g., 10.8.0.0/24)

## Step 4: Secure SSH Access First

Apply restrictions to SSH while keeping the dashboard open:

```bash
# Clear existing rules
sudo ufw reset

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH only from VPN subnet (replace with your actual VPN subnet)
sudo ufw allow from 10.8.0.0/24 to any port 22 proto tcp

# Allow all access to port 5000 temporarily
sudo ufw allow 5000/tcp

# Enable the firewall
sudo ufw enable
```

Now test SSH access:
1. While connected to VPN - should work
2. While disconnected from VPN - should fail

## Step 5: Secure Admin Dashboard Access

After confirming SSH is properly restricted:

```bash
# Remove the open access to port 5000
sudo ufw delete allow 5000/tcp

# Allow port 5000 only from VPN subnet
sudo ufw allow from 10.8.0.0/24 to any port 5000 proto tcp
```

## Step 6: Verify Your Configuration

Check that your rules are properly configured:

```bash
sudo ufw status verbose
```

You should see your configured rules, including the VPN subnet restriction for port 5000.

## Testing Your Setup

To ensure everything is working correctly:

1. Connect to your VPN
2. Try accessing the admin dashboard (http://your-server-ip:5000)
3. Disconnect from VPN and verify you can no longer access the dashboard
4. Verify you can still SSH into the server regardless of VPN connection

## Troubleshooting

If you can't access your dashboard even when connected to VPN:

1. Verify you're receiving the correct IP from the VPN server using `ip addr show`
2. Check that the subnet in your UFW rule matches the VPN client subnet
3. Check UFW logs for blocked connections:
   ```bash
   sudo grep UFW /var/log/syslog
   ```

If you accidentally lock yourself out of SSH:
1. You'll need to access the server directly (console access through your cloud provider)
2. Disable UFW: `sudo ufw disable`
3. Reconfigure your rules correctly

## Additional Security Recommendations

1. Consider changing SSH to a non-standard port
2. Set up SSH key authentication and disable password authentication
3. Configure rate limiting for SSH attempts:
   ```bash
   sudo ufw limit ssh
   ```
4. Regularly update your system packages:
   ```bash
   sudo apt update && sudo apt upgrade
   ```

By following this guide, your admin dashboard will only be accessible to users connected to your VPN, with the restriction enforced at the OS level before requests even reach Nginx.
