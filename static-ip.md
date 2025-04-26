# Armbian Static IP Configuration Guide

This guide provides instructions for configuring a static IP address on your device running Armbian.

## Prerequisites

- Armbian running
- Administrative access (sudo privileges)
- Knowledge of your network settings (gateway, subnet)

## Step 1: Identify Your Network Interface

First, identify your network interface name:

```bash
ip addr show
```

Look for your active network connection, which will typically be named something like `end1`, `eth0`, or similar. You'll see your current IP address listed next to `inet`.

## Step 2: Remove Any DHCP Configuration

If a DHCP configuration file exists, remove it to prevent conflicts:

```bash
sudo rm /etc/netplan/10-dhcp-all-interfaces.yaml
```

## Step 3: Create Static IP Configuration File

Create a new file for your static IP configuration:

```bash
sudo nano /etc/netplan/20-static-ip.yaml
```

Add the following configuration (adjust based on your needs):

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    end1: # Replace with your actual interface name from Step 1
      addresses:
        - 192.168.1.99/24 # Your desired static IP address
      routes:
        - to: default
          via: 192.168.1.1 # Your router/gateway IP address
      nameservers:
        addresses:
          - 8.8.8.8 # Google DNS
          - 1.1.1.1 # Cloudflare DNS
```

## Step 4: Set Proper Permissions

Set the correct permissions for the configuration file:

```bash
sudo chmod 600 /etc/netplan/20-static-ip.yaml
```

## Step 5: Apply the Configuration

Apply the new network configuration:

```bash
sudo netplan apply
```

You may see a warning message about "openvswitch" - this is normal and can be ignored as long as your IP address is updated correctly.

## Step 6: Verify the Configuration

Check that your new static IP has been applied:

```bash
ip addr show
```

Your interface should now show your configured static IP address.

## Step 7: Test Network Connectivity

Test that you can reach the internet:

```bash
ping -c 4 google.com
```

## Step 8: Make the Configuration Persistent

The configuration will persist across reboots automatically. To verify this after a reboot:

```bash
sudo reboot
```

After rebooting, check your IP again:

```bash
ip addr show
```

## Troubleshooting

### If you lose network connectivity:

Connect a monitor and keyboard to your device and:

1. Check your configuration file for typos
2. Verify that the gateway IP is correct
3. Make sure the interface name matches your actual network interface

### To revert to DHCP temporarily:

```bash
sudo ip addr flush dev end1  # Use your interface name
sudo dhclient end1
```

### To view network logs:

```bash
sudo journalctl -u systemd-networkd
```

#### [Go back](readme.md#whats-next)
