<h1>RaspberryPi-Wifi-Accesspoint</h1>


<p>how to install a simple Wifi Accesspoint with dhcp on eth0 on raspberry Pi 4
or pi zero w with Ethernet hat</p>


<h2>Image for Pi</h2>


<p>raspios-buster-armhf-lite.img</p>
<p>donwload here: <a href:"https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwisiPS328XzAhVjzoUKHd_ZCwkQFnoECBEQAQ&url=https%3A%2F%2Fdownloads.raspberrypi.org%2Fraspios_lite_armhf_latest&usg=AOvVaw2JjWDCFntUKUID79GdvIJW">raspios-buster-armhf-lite</a><p>



<h1>Setup Pi4 and Pi_zero_w</h1>


<h3>Setup Pi4</h3>
<ol>
<li>flash image to sd card</li>
<li>connect to your network and ssh to Pi</li>
<li>change Password</li>
<li>update your Pi</li>
<li>install optional Packages</li>
<li>install needed Packages</li>
<li>config of the accesspoint</li>
</ol>

<h3>Setup Pi_zero_w</h3>
<ol>
<li>flash image to sd card</li>
<li>connect to your network and ssh to Pi</li>
<li>change Password</li>
<li>update your Pi</li>
<li>install optional Packages</li>
<li>install needed Packages</li>
<li>config of the accesspoint</li>
<li>setup ethernet-hat</li>
</ol>


<h1>Setup Step by Step Pi4</h1>


<h2>flash image to sd card</h2>


```
sudo dd if=/pfad/von/image.img of=/pfad/zur/sdcard bs=4M
```


<h2>ssh to your Pi</h2>

<p>detect the Pi in the network via arp-scan or nmap</p>
<p>arp-scan</p>

```
sudo arp-scan -interface=wlp1s0 --localnet
```
<p>nmap fast scan</p>

```
sudo nmap -sP 192.168.0.0/24
```
<p>if detectecd, login via ssh with standard-credentials</p>

```
user:pi
password:raspberry
```

```
ssh pi@192.168.0.173
```


<h2>change Password</h2>


<p>type passwd</p>

```
passwd
```
<p>change passwd for the root user</p>

```
sudo -i
passwd
exit
```


<h2>update your pi</h2>

```
sudo apt-get update && sudo apt-get upgrade
```


<h2>install optional packages</h2>

```
sudo apt install -y vim net-tools wireless-tools
```


<h2>install needed packages</h2>

```
sudo apt install -y iptables dhcpcd5 dnsmasq hostapd
```


<h1>config Wifi-Access-Point</h1>


<p>prepare config first to ignore wlan0 setup, static IP follows later</p>

```
sudo vim /etc/dhcpcd.conf
```

<p>Put this above any interface lines that may be in the file:</p>

```
denyinterfaces wlan0
```

<p>Now lets setup our interfaces eth0 with dhcp and wlan0 as wifi interface</p>

```
sudo vim /etc/network/interfaces
```

<p>add this section</p>

```

allow-hotplug eth0
iface eth0 inet dhcp

allow-hotplug wlan0  
iface wlan0 inet static  
    address 172.24.1.1
    netmask 255.255.255.0
    network 172.24.1.0
    broadcast 172.24.1.255
        
```

<p>restart dhcpcd with</p>

```
sudo service dhcpcd restart
```

<p>and then reload the configuration for wlan0 with:</p>

```
sudo ifdown wlan0 
sudo ifup wlan0
```

<H1>configure hostapd.conf file</h1>

```
vim /etc/hostapd/hostapd.conf
```

<h2>example config</h2>

```
<p>This is the name of the WiFi interface we configured above
interface=wlan0

# Use the nl80211 driver with the brcmfmac driver
driver=nl80211

# This is the name of the network
ssid=Some-Accesspoint-Name

# Use the 2.4GHz band
hw_mode=g

# Use channel 6
channel=6

# Enable 802.11n
ieee80211n=1

# Enable WMM
wmm_enabled=1

# Enable 40MHz channels with 20ns guard interval
ht_capab=[HT40][SHORT-GI-20][DSSS_CCK-40]

# Accept all MAC addresses
macaddr_acl=0

# Use WPA authentication
auth_algs=1

# Require clients to know the network name
ignore_broadcast_ssid=0

# Use WPA2
wpa=2

# Use a pre-shared key
wpa_key_mgmt=WPA-PSK

# The network passphrase
wpa_passphrase=password_you_chose

# Use AES, instead of TKIP
rsn_pairwise=CCMP

```


<p>test config with:</p>

```
sudo /usr/sbin/hostapd /etc/hostapd/hostapd.conf
```

<p>stop hostapd test</p>

<p>CTRL+C</p>


<p>tell hostapd where to find its config file</p>

```
sudo vim /etc/default/hostapd
```

<p>Find the line #DAEMON_CONF=”” and replace it with</p> 

```
DAEMON_CONF=”/etc/hostapd/hostapd.conf”
```

<h1>Let’s get dnsmasq set up:</h1>

```
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.bak  
sudo vim /etc/dnsmasq.conf
```

<p>The file should look like this:</p>

<p>example</p>

```
interface=wlan0
listen-address=172.24.1.1
bind-interfaces
server=8.8.8.8
domain-needed
bogus-priv
dhcp-range=172.24.1.50,172.24.1.150,12h
```
<p>example with comments</p>

```
interface=wlan0                         # Use interface wlan0  
listen-address=172.24.1.1               # Set our listening address
bind-interfaces                         # Bind to the interface to make sure we aren't sending things elsewhere  
server=8.8.8.8                          # Forward DNS requests to Google DNS  
domain-needed                           # Don't forward short names  
bogus-priv                              # Never forward addresses in the non-routed address spaces.  
dhcp-range=172.24.1.50,172.24.1.150,12h # Assign IP addresses between 172.24.1.50 and 172.24.1.150 with a 12 hour lease time
```


<h1>forward traffic</h1>

<p>Now, we have two interfaces active, and we have a DHCP client for our Pi and a DHCP server for our wireless guests.</p> 
<p>Now we need to forward traffic between the wifi and ethernet interfaces.</p> 
<p>We can make this happen immediately with a simple command to update /proc:</p>

<p>get root permission with</p>

```
sudo -i
echo 1 > /proc/sys/net/ipv4/ip_forward
cat /proc/sys/net/ipv4/ip_forward
```

<p>However, this change won’t stick between reboots. We need to make it permanent through sysctl:</p>

```
vim /etc/sysctl.conf
```

<p>..uncomment the line containing:</P> 

```
net.ipv4.ip_forward=1
```

<p>change to pi user</p>

``` 
exit
```



<h1>iptables setup</h1>
<h3>The forward isn’t quite enough to give our wifi guests Internet access (through our eth0 interface).</h3> 
<p>We need iptables to help us do this.</p>

```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  
sudo iptables -A FORWARD -i eth0 -o wlan0 \                                             
> -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

<p>Check out our rules:</p>

```
sudo iptables -S
```

```
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -i wlan0 -o eth0 -j ACCEPT
```


<p>Output our rules to a file:</p>

```
sudo iptables-save > /etc/iptables.ipv4.nat
```

<h1>Apply these rules every time we boot the Pi</h1> 

<p>by editing the /etc/rc.local file:</p>

```
sudo vim /etc/rc.local
```
<p>and adding at the end</p>

```
iptables-restore < /etc/iptables.ipv4.nat
```

<h3>Example</h3>

```
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi

iptables-restore < /etc/iptables.ipv4.nat
exit 0
```

<p>Make the file executable:</p>

```
sudo chmod 711 /etc/rc.local
ls -l /etc/rc.local
```

<h1>and finaly</h1>
<p>unmasking hostapd service</P>

```
sudo systemctl unmask hostapd
```

<p>restart and enable hostapd dnsmasq</p>

```
sudo systemctl start hostapd dnsmasq
sudo systemctl enable hostapd dnsmasq
sudo systemctl status hostapd dnsmasq
```

<h1>for pi_zero_w with ethernet hat enable ethernet-hat</h1>

```
sudo vim /boot/config.txt
```

<p>add following line at the end of the config.txt</p>

```
dtoverlay=enc28j60
```

****










