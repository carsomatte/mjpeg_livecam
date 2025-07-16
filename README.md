# Secure Remote Camera Streaming over WireGuard

## Overview
This project configures a Linux-based webcam streaming server using mjpg_streamer, with access restricted to clients connected through a secure WireGuard VPN tunnel. 
Firewall rules managed by UFW ensure that only traffic over the VPN interface is allowed to reach the stream.


## Stack
- WireGuard VPN (peer-to-peer tunnel)
- Oracle Cloud (central wireguard VPN server)
- mjpg_streamer (lightweight video stream via HTTP)
- UFW firewall (interface-specfifc access control)


## Network Design
### IP Assignment (WireGuard Subnet: `10.0.0.0/24`)

| Device         | Role                        | WireGuard IP |
|----------------|-----------------------------|--------------|
| Main Computer  | VPN Client (viewer/admin)   | `10.0.0.3`   |
| Oracle Server  | VPN Hub / Relay Node        | `10.0.0.1`   |
| Camera Host    | VPN Client (stream server)  | `10.0.0.2`   |

### Ports Used

| Service         | Port  | Interface | Purpose                         |
|-----------------|-------|-----------|-------------------------------  |
| mjpg_streamer    | 8080  | `wg0`     | Video stream (HTTP)            |
| SSH              | 22    | `wg0`     | Secure remote admin (optional) |
| WireGuard VPN    | 51820 | `ens3`    | Encrypted VPN handshake port   |



## Setup Instructions
1. **Set up Oracle Cloud Instance and WireGuard**
   - Use the provided config files (`oracle-server.conf`, `camera.conf`, `main_client.conf`) for all three hosts.
   - Ensure your Oracle instance has a public IP and port `51820` open.
   - On the Oracle instance, enable IP forwarding:

     ```bash
     # Edit sysctl configuration to persist IP forwarding
     sudo nano /etc/sysctl.conf
     # Add or uncomment:
     net.ipv4.ip_forward=1

     # Apply the setting immediately
     sudo sysctl -w net.ipv4.ip_forward=1
     ```
2. **Configure UFW rules**
   - On Oracle Instance, allow
     ```bash
     sudo ufw allow 51820/udp    # WireGuard VPN
     sudo ufw allow 22/tcp       # SSH access to Oracle
     ```
   - On Camera Host, allow 
      ```bash
     sudo ufw allow in on wg0 to any port 22 proto tcp    # SSH access over VPN
     sudo ufw allow in on wg0 to any port 8080 proto tcp  # Camera stream over VPN
     ```


4. **Install `mjpg_streamer` on the Camera Host**
   - Install dependencies and compile `mjpg_streamer` or use a package manager.
   - Start the stream, ensuring it binds to all interfaces or just the VPN IP:
5. **Test from Main Client**
   - Make sure you're connected to the WireGuard VPN on your main computer.
   - Access the video stream via:
     ```
     http://10.0.0.2:8080/
     ```
   - Optional: SSH into the camera host over the VPN:
     ```bash
     ssh user@10.0.0.2
     ```
