# Fabric Minecraft Server Setup Guide

This guide provides detailed instructions for setting up an optimized Fabric Minecraft server (version 1.21.5) on an Armbian.

## Prerequisites

- Armbian running
- At least 4GB of RAM (8GB+ recommended)

## Step 1: Install Java 21

```bash
# Update package lists
sudo apt update

# Add the Adoptium repository
wget -qO - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo tee /etc/apt/trusted.gpg.d/adoptium.asc
echo "deb [signed-by=/etc/apt/trusted.gpg.d/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list

# Update and install Java 21
sudo apt update
sudo apt install temurin-21-jre

# Verify installation
java -version
```

## Step 2: Create Server Directory Structure

```bash
# Create a directory for your Minecraft server
mkdir -p ~/minecraft_server/1.21.5_server
cd ~/minecraft_server/1.21.5_server
```

## Step 3: Download Fabric Server

```bash
# Download the Fabric installer for Minecraft 1.21.5
curl -OJ https://meta.fabricmc.net/v2/versions/loader/1.21.5/0.16.14/1.0.3/server/jar

# Rename the downloaded file to something simpler
mv fabric-server-mc.*.jar server.jar
```

## Step 4: First Run Setup

```bash
# Run the server once to generate configuration files
java -Xmx4G -jar server.jar nogui

# Accept the EULA
echo "eula=true" > eula.txt
```

## Step 5: Install Performance Optimization Mods

```bash
# Create mods directory
mkdir -p mods

# Download optimization mods
cd mods

# Lithium - general performance improvements
wget https://cdn.modrinth.com/data/gvQqBUqZ/versions/VWYoZjBF/lithium-fabric-0.16.2%2Bmc1.21.5.jar

# ScalableLux - lighting engine optimization (Starlight replacement)
wget https://cdn.modrinth.com/data/Ps1zyz6x/versions/UueJNiJn/ScalableLux-0.1.3%2Bbeta.1%2Bfabric.4039a8d-all.jar

# FerriteCore - memory usage optimization
wget https://cdn.modrinth.com/data/uXXizFIs/versions/CtMpt7Jr/ferritecore-8.0.0-fabric.jar

# Krypton - networking optimization
wget https://cdn.modrinth.com/data/fQEb0iXm/versions/neW85eWt/krypton-0.2.9.jar

# Concurrent Chunk Management Engine - chunk loading improvements
wget https://cdn.modrinth.com/data/VSNURh3q/versions/eL3rprSq/c2me-fabric-mc1.21.5-0.3.2.0.0.jar

# Very Many Players - high playercount optimisation
wget https://cdn.modrinth.com/data/wnEe9KBa/versions/EKg6v67t/vmp-fabric-mc1.21.5-0.2.0%2Bbeta.7.198-all.jar

cd ..
```

## Step 6: Configure Server Properties

```bash
# Create/edit server properties with optimized settings
nano server.properties
```

Add/modify these key settings:

```properties
view-distance=8
simulation-distance=6
spawn-protection=0
```

## Step 7: Create Optimized Startup Script

```bash
nano start.sh
```

Add the following content (adapt -Xms and -Xmx values (allocated ram)):

```bash
#!/bin/bash
while [ true ]; do
    java -Xms8G -Xmx8G --add-modules=jdk.incubator.vector -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikars.new.flags=true -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -jar server.jar --nogui
    echo "Server crashed or stopped. Restarting in 10 seconds..."
    echo "Press CTRL + C to stop."
    sleep 10
done
```

Make the script executable:

```bash
chmod +x start.sh
```

## Step 8: Running the Server with Screen

```bash
# Install screen if not already available
sudo apt install screen

# Create a new screen session for the Minecraft server
screen -S minecraft

# Start the server
./start.sh
```

To detach from the screen session (leaving the server running), press `Ctrl+A` followed by `D`.

To reattach to the session later:

```bash
screen -r minecraft
```

## Step 9: Port Forwarding and Dynamic DNS

1. Set up port forwarding on your router:

   - Forward TCP/UDP port 25565 to your device local IP (192.168.1.60)

2. Set up port forwarding

   ```bash
   sudo apt install ufw  # if not already installed
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   sudo ufw allow 22/tcp  # SSH
   sudo ufw allow 25565/tcp  # Minecraft
   sudo ufw enable
   ```

3. Set up a dynamic DNS service like No-IP (if you need one, otherwise you can skip this):
   - Create an account at noip.com
   - Create a hostname (e.g., yourserver.ddns.net)
   - Install the DUC (Dynamic Update Client):
   ```bash
   cd ~
   wget https://www.noip.com/client/linux/noip-duc-linux.tar.gz
   tar xzf noip-duc-linux.tar.gz
   rm noip-duc-linux.tar.gz
   cd noip-*
   sudo make
   sudo make install
   ```
   - Configure No-IP with your credentials:
   ```bash
   sudo noip2 -C
   ```
   - Start the client:
   ```bash
   sudo noip2
   ```
   - Make No-IP start on boot:
   ```bash
   echo "@reboot sudo noip2" | sudo tee -a /etc/crontab
   ```

## Performance Tuning Tips

### Server Properties Optimization

- Reduce `view-distance` and `simulation-distance` for better performance
- Use `entity-broadcast-range-percentage=50` to reduce entity updates
- Set `network-compression-threshold=256` for better network performance

### Memory Management

- For 16GB RAM systems, allocate 4-10GB to Minecraft
- For 8GB RAM systems, allocate 4-6GB to Minecraft
- Never allocate more than 12GB as it can worsen GC behavior
- Always leave at least 1–2 GB free for your operating system

## Troubleshooting

### Server Won't Start - Lock Error

If you see "already locked" errors:

```bash
rm -f ~/minecraft_server/1.21.5_server/world/session.lock
```

### Lag Spikes During Play

- Check server logs for garbage collection pauses
- Reduce view-distance further
- Remove mods one by one to identify problematic ones

### Connection Issues

- Verify your port is properly forwarded
- Check if your No-IP hostname is resolving correctly
- Ensure your server is not being blocked by a firewall

## Useful Commands

### Server Control

- Stop server gracefully: Type `stop` in the server console
- Force stop: Press `Ctrl+C` in the screen session

### Screen Management

- List screen sessions: `screen -ls`
- Reattach to session: `screen -r minecraft`
- Create new screen session: `screen -S [name]`
- Detach from session: `Ctrl+A` then `D`
- Kill a session: `screen -X -S 3952 quit` (3952 is the id of the session (ex: 3952.minecraft))

### Server Backup

```bash
# Simple backup script
mkdir -p ~/minecraft_backups
tar -czf ~/minecraft_backups/minecraft_$(date +%Y%m%d).tar.gz -C ~/minecraft_server 1.21.5_server
```

#### [Go back](readme.md)
