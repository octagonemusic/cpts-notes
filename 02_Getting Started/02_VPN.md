# Connect to HTB VPN via OpenVPN
### Step 1:  Connect to the VPN
```bash
sudo openvpn user.ovpn
```

### Step 2: Check if you're connected to the VPN
```bash
ifconfig
```
You should be able to see a `tun` adapter if you have successfully connected to the VPN

### Step 3: Check networks accessible via the VPN
```bash
netstat -rn
```