# AlpineAP
Alpine Linux Access POINT on Raspberry Pi
INSTALL OS
1. Create a 300 MiB FAT32 Partition and copy the content of untarred alpine-rpi-3.20.1-aarch64.tar.gz.
Create a second partition - ext4, aprox 1.7 GiB

2. mkdir /tmp/fat32 and mount /dev/sdb1  /tmp/fat32
Then copy untarred .gz to /tmp/fat32 and umount /tmp/32

3. Insert microSD card in Pi, login with root (without pass) and follow the setup-alpine steps:
- insert keyboard etc
- initilaize eth0, dhcp; wlan0 - done and add no manual config
- enable chrony as NTP - mandatory for APK Mirror - (f) Find and use fastest mirror
- select openssh as ssh server
DISK & install
Try boot media /media/mmcblk0p1?  yes
-> Available Disks are:
 mmcblk0 (8.0 GB)
Which disks would you like to use? mmcblk0
Would you like to use it? sys  - Disk is erased and file system created-



#############################################################################
ACCESS POINT
1.Configure interfaces:

 cat /etc/network/interfaces 

auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
iface eth0 inet6 dhcp
echo 2 > /proc/sys/net/ipv6/conf/eth0/accept_ra 

auto wlan0
iface wlan0 inet static
	address 172.25.1.1/24
iface wlan0 inet6 static
        address fdda:8765:4321:fdda::1
        netmask 64

-----------------------------------------------------
cat /etc/hosts  - de verificat daca trebuie
127.0.0.1	alpine.fritz.box alpine localhost.localdomain localhost
::1		localhost localhost.localdomain
:1             localhost ipv6-localhost ipv6-loopback
fe00::0         ipv6-localnet
ff00::0         ipv6-mcastprefix
ff02::1         ipv6-allnodes
ff02::2         ipv6-allrouters
ff02::3         ipv6-allhosts
--------------------------------------------------------------------------

DNS - OPTIONAL:
search d-ns.org
nameserver 85.214.172.223
Uncomment  below line 
#RESOLV_CONF="no"  in file /etc/udhcpc/udhcpc.conf
Reboot

###############################################################################
DHCP SERVER:
https://cylab.be/blog/221/a-light-nat-router-and-dhcp-server-with-alpine-linux

apk add dhcp
edit /etc/dhcp/dhcpd.conf
apk add dhcpcd
rc-update add dhcpd
rc-service dhcpd start  - or started after reboot!!!
-------------------------

-----------------------------


/etc/dhcp/dhcpd.conf should contain below lines:
option domain-name "d-ns.org";
option domain-name-servers www.d-ns.org;

default-lease-time 600;
max-lease-time 7200;
authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;


# This is a very basic subnet declaration.

subnet 172.25.1.0 netmask 255.255.255.0 {
  range 172.25.1.10 172.25.1.200;
 option domain-name-servers www.d-ns.org; 
  option routers 172.25.1.1;
}



#######################################################################

Enable NAT
Here is how to enable Network Address Translation (NAT) on an Alpine Linux server:

## enable IPv4 routing
echo "net.ipv4.ip_forward=1" | tee -a /etc/sysctl.conf
sysctl -w   - save
sysctl -p    - display

IPv4:
 apk add iptables
 rc-update add iptables
# wlan0 is the internal interface
 iptables -A FORWARD -i wlan0 -j ACCEPT
# eth0 is the external interface (connected to the internet)
 iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
 /etc/init.d/iptables save
 
 
### enable IPv6 routing 
echo "net.ipv6.conf.all.forwarding=1" | tee -a /etc/sysctl.conf
sysctl -w   - save
sysctl -p    - display
 
IPv6: 
apk add ip6tables 
rc-update add ip6tables
ip6tables -A FORWARD -i wlan0 -j ACCEPT
ip6tables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
/etc/init.d/ip6tables save
 
 
 net.ipv6.conf.all.forwarding= 1
net.ipv6.conf.he-ipv6.accept_ra=2
 
 
 -----------------------------------------
 
HOSTAPD 
 apk add hostapd
 rc-service hostapd start
 rc-update add hostapd
##### grep -v '^#' hostapd.conf   - print file without lines starting with #
 
 /etc/hostapd/hostapd.conf should contain below lines:
-----------------------------------//------------------------------------------  
interface=wlan0
logger_syslog=-1
logger_syslog_level=2
logger_stdout=-1
logger_stdout_level=2
ctrl_interface=/var/run/hostapd
ctrl_interface_group=0
ssid=iBlocker-ALPINE
hw_mode=g
channel=1
beacon_int=100
dtim_period=2
max_num_sta=255
rts_threshold=-1
fragm_threshold=-1
macaddr_acl=0
auth_algs=3
ignore_broadcast_ssid=0
wmm_enabled=1
wmm_ac_bk_cwmin=4
wmm_ac_bk_cwmax=10
wmm_ac_bk_aifs=7
wmm_ac_bk_txop_limit=0
wmm_ac_bk_acm=0
wmm_ac_be_aifs=3
wmm_ac_be_cwmin=4
wmm_ac_be_cwmax=10
wmm_ac_be_txop_limit=0
wmm_ac_be_acm=0
wmm_ac_vi_aifs=2
wmm_ac_vi_cwmin=3
wmm_ac_vi_cwmax=4
wmm_ac_vi_txop_limit=94
wmm_ac_vi_acm=0
wmm_ac_vo_aifs=2
wmm_ac_vo_cwmin=2
wmm_ac_vo_cwmax=3
wmm_ac_vo_txop_limit=47
wmm_ac_vo_acm=0
eapol_key_index_workaround=0
eap_server=0
own_ip_addr=127.0.0.1
wpa=2
wpa_passphrase=Test#1234

-----------------------------------//------------------------------------------
####################################################################################
