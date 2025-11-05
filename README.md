# IM NOT TAKING CREDIT FOR THE ORIGINAL AUTHOR. Please visit his website and his github page he created this not me this is just my working version of his compose file because the authors compose file did not work
https://wg-easy.github.io/wg-easy/latest/examples/tutorials/basic-installation/
https://github.com/wg-easy/wg-easy

Easy WireGuard VPN server setup using wg-easy with web UI.

## Configuration

- **FQDN**: FIll in YOURS
- **WireGuard Port**: 4750 (UDP)
- **Web UI Port**: 51821 (TCP)
- **Client IP Range**: 10.8.0.x
- **LAN Access**: 192.168.1.0/24 (configured)
- **Allowed IPs**: 0.0.0.0/0,192.168.1.0/24

## Prerequisites

1. Docker and Docker Compose installed
2. Make sure ports 4750/UDP and 51821/TCP are open in your firewall
3. Point your DNS A record for your FQDN to your server's public IP address
4. Enable IP forwarding on the host (already configured in container)

## Quick Start

1. **Set a secure password** (edit `PASSWORD` in docker-compose.yml):
   ```yaml
   - PASSWORD=your_secure_password_here
   ```

2. **Start the WireGuard server:**
   ```bash
   docker-compose up -d
   ```

3. **Access the Web UI:**
   - Open your browser and go to: `http://your-server-ip:51821`
   - Or if accessing locally: `http://localhost:51821`
   - Login with the password you set

4. **View logs:**
   ```bash
   docker-compose logs -f wg-easy
   ```

## Using the Web UI

The wg-easy web interface allows you to:
- Create and manage client configurations
- View QR codes for easy mobile setup
- Download client configuration files
- Monitor connected clients
- Remove clients

## Accessing Your LAN

Your LAN (192.168.1.0/24) is already configured in `WG_ALLOWED_IPS`. To access LAN resources:

1. **Enable IP forwarding** on the host (if not already enabled):
   ```bash
   # Linux
   echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   
   # Or temporarily:
   sudo sysctl -w net.ipv4.ip_forward=1
   ```

2. **Add iptables rules** to allow forwarding (Linux):
   ```bash
   # Replace 'eth0' with your actual LAN interface name
   sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
   sudo iptables -A FORWARD -i wg0 -j ACCEPT
   sudo iptables -A FORWARD -o wg0 -j ACCEPT
   ```

3. **If you need to change the LAN subnet**, edit `WG_ALLOWED_IPS` in docker-compose.yml:
   ```yaml
   - WG_ALLOWED_IPS=0.0.0.0/0,192.168.1.0/24
   ```

## Network Configuration

- **Container Network**: 10.42.42.0/24 (IPv4) and fdcc:ad94:bacf:61a3::/64 (IPv6)
- **Container IP**: 10.42.42.42 (IPv4) and fdcc:ad94:bacf:61a3::2a (IPv6)
- **WireGuard Client Subnet**: 10.8.0.0/24

## Important Notes

- Configuration is stored in the `etc_wireguard` Docker volume
- **Change the default password** before exposing the web UI to the internet!
- The web UI is accessible on port 51821 - consider restricting access with a firewall
- Restart the container after changing environment variables:
  ```bash
  docker-compose down
  docker-compose up -d
  ```

## Troubleshooting

- Check logs: `docker-compose logs wg-easy`
- Verify ports are open: `netstat -uln | grep 4750` and `netstat -tln | grep 51821`
- Test WireGuard connection: `docker-compose exec wg-easy wg show`
- If web UI doesn't load, check firewall rules for port 51821

