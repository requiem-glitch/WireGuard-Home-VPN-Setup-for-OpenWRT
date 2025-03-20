# WireGuard-Home-VPN-Setup-for-OpenWRT
This repository contains configuration and instructions for configuring WireGuard VPN on a router with OpenWRT.
## Server (Router)
1. Install wireguard:
* ```opkg update && opkg install wireguard-tools ```
2. Create a pairs of keys (private and public) for router and client:
* ```wg genkey | tee privatekey | wg pubkey > publickey```
* ```wg genkey | tee client_privatekey | wg pubkey > client_publickey```  
save these files to yourself, the values from there will also be needed to configure the client.  
3. Configuring the WireGuard interface on the router:
* Open the network configuration file: ```nano /etc/config/network```
* Add a new WireGuard interface (configuration example):
```
config interface 'wg0'
    option proto 'wireguard'
    option private_key 'SERVER_PRIVATE_KEY'
    option listen_port '51820'
    option route '10.0.0.1/24'

config wireguard_wg0
    option public_key 'CLIENT_PUBLIC_KEY'
    option allowed_ips '10.0.0.2/32'

config route
    option interface 'wg0'
    option target 'YOUR_LOCAL_NETWORK'
    option gateway '10.0.0.1'
```
4. Firewall configuration:
* Open the firewall configuration file: ```nano /etc/config/firewall```
* Add a rule to allow traffic through WireGuard:
```
config zone
    option name 'vpn'
    option input 'ACCEPT'
    option forward 'ACCEPT'
    option output 'ACCEPT'
    option masq '1'
    option network 'wg0'

config forwarding
    option src 'vpn'
    option dest 'lan'

config forwarding
    option src 'vpn'
    option dest 'wan'

config rule
    option name 'Allow-WireGuard'
    option src 'wan'
    option dest_port '51820'
    option proto 'udp'
    option target 'ACCEPT'
```
5. Restart network services and firewall:
```
/etc/init.d/network restart
/etc/init.d/firewall restart
```
### Warning
* some settings might not be set after reboot: ip & route for wg0, check: ```ip a show wg0 && ip route```
* If there are no parameters, you must set them manually:
```
ip address add 10.0.0.1/24 dev wg0
ip route add 10.0.0.0/24 dev wg0
```
## Client
1. Configuration
* Create config file (file.conf):
```
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 10.0.0.2/24
DNS = YOUR_ROUTER_LOCAL_IP

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = YOUR_PUBLIC_IP:51820
AllowedIPs = 0.0.0.0/0
```
2. Import file in wireguard.

at this point, the vpn should work fine, to check its performance, you can ping the server on the client (I use termux for the phone) and ```curl ifconfig.me``` to see which ip we are working from
