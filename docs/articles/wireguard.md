# Wireguard

WireGuard is a new, experimental VPN protocol that aims to offer a simpler, faster, and more secure solution for VPN tunneling than the existing VPN protocols. WireGuard has some major differences when compared to OpenVPN and IPSec, such as the code size (under 4,000 lines!), speed, and encryption standards. 

You can learn more about it here https://www.wireguard.com/ and https://restoreprivacy.com/wireguard/

## Setting up Wireguard

First you'll need to install wireguard as it hasn't been submitted to the mainline kernel yet. Then as you user (NOT ROOT) you'll want to generate your keys similar to generating Private an Public SSH keys. THis will need done both Server and Client(s) side.
<pre class="clear">
sudo equo install wireguard
sudo modprobe wireguard
mkdir .wireguard 
cd .wireguard
wg genkey | tee privatekey | wg pubkey > publickey
</pre>

Next, as root, you'll want to add your config for a wireguard interface

<pre class="clear">
# mkdir /etc/wireguard
# touch /etc/wireguard/wg0.conf
# nano /etc/wireguard/wg0.conf
</pre>


### ***SERVER SIDE CONFIG***
You'll want to have you're PRIVATE key handy from `~/.wireguard/privatekey` for the server and you'll want to have the connecting client's PUBLIC key as well. Also, due to wireguard not having a dynamic IP function, you'll need to enter static IPs. Enter the data as necessary like below:

***/etc/wireguard/wg0.conf***
<pre class="clear"> 
[Interface]
Address = 172.16.0.1
ListenPort = 51820
PrivateKey = {Your Server PRIVATE Key HERE}

[Peer]
PublicKey = {Your Client PUBLIC Key HERE}
AllowedIPs = 172.16.0.2
</pre>

Afterwards you'll want to connect to your wireless router and setup port-forwarding to send inbound communications on a port of your choice (51820 is fine) to the Reserved or Static IP of your server on port 51820. That is the default port, but can be changed to whatever you wish. If you have a Dynamic DNS, it will make your life a lot easier to reach your VPN.

### ***CLIENT SIDE CONFIG***
You'll want to have you're PRIVATE key handy from `~/.wireguard/privatekey` for the Client and you'll want to have the connecting Servers's PUBLIC key as well. Also, due to wireguard not having a dynamic IP function, you'll need to enter static IPs. You can also add alternative DNS Addresses. Enter the data as necessary like below:

***/etc/wireguard/wg0.conf***
<pre class="clear"> 
[Interface]
Address = 172.16.0.2
DNS = 1.1.1.1
PrivateKey = {Your client PRIVATE Key HERE}

[Peer]
AllowedIPs = 0.0.0.0/0
Endpoint = {Your SERVER's External IP or Dynamic DNS Name}:51820
PersistentKeepalive = 25
PublicKey = {Your SERVER's PUBLIC Key HERE}
</pre>

### Enabling your Wireguard Interface
Systemd is used to enable/disable/start/stop all services. This includes your VPNs.

<pre class="clear"> 
sudo systemctl start wg-quick@wg0
</pre>

View the systemd guide for more information on enabling upon boot, starting/stopping/restarting/disabling/etc.

And there you have it! You have your own personal VPN using wireguard. As a side note, wireguard also has a free Android app so you can connect your new VPN to you phone. This makes you safe on those sketchy free wifi connections in stores and restuarants.
