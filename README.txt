# piserver-vpnrouter
Piserver with vpnrouter pihole builtin with nordvpn and password manager and entire network protection Now with retropie also! wohooo

MAKE SURE YOU VIEW THIS IN RAW FORMAT!


step 1:

Hardware= Raspberry pi 4 4gb or 8gb ( might work with pi400 but unsure! dont own one so dont know untested )
1 network cable duuh :p
1 memory card again duuh :P i reccomend 32gb size minimum
optional drives ( harddrives ssds etc etc )
enclosure and cooler ( i recomend retropie 4 nintendo box comes with fan and hotswap ssd cassette good stuff )
Pc or linux to make images and ssh

Step 2

grab Raspberry Pi OS Lite flash it to the memory card, add a empty ssh file so you can ssh into it. ( pi raspberry )

step 3

download the pdf and follow this guide https://forum.openmediavault.org/index.php?thread/28789-installing-omv5-on-raspberry-pi-s-armbian-sbc-s-i386-32-bit-platforms/

step 4

once ur setup is complete i reccomend you go through all settings in open media vault and fix everything to your likeing and network ( THIS NEEEDS TO BE DONE before you go next step failure to make all settings and ip adresses and such will result in you haveing to reinstall everything! )

step 5

make 2 admin accounts 1 for you 1 for emergency.

once you tested ur new admin accounts both ssh and homepage. remove the pi user since its a security risk.
reverify all ur settings and make sure your ip adress is static and doublecheck everything.

step 6

Once ur firewalls and settings and static ip is set and all other stettings are completed and your ready to go to next step.

step 7 

Setup a storage device make a storage device to host ur files for the server other than the memory card ( hardrive or ssd )
this is where we will store our serverfiles configurations and backups.

step 8

once you made a storage device and made few folders ( configs backup and so on )

( if you dont see docker or omv extra go to plugin and click omv extra and install it and reboot )

time to install docker

go to omv extra menu make sure backports are on then click docker and install it DO not install portainer yet or yacht.
once docker is installed restart ur raspberry pi

time to change a few things.
in general setting change the port to another number besides 80 and change the timeout to 60+

step 9

restart ur pi and login on new port example 85
install portainer
once ur portainer is installed open it and setup ur admin account.
you going to go back here later and fix networks and such.

step 10

now that we got nas and docker installed time to get vpn setup
now in order to install some vpn we need the librarys for it easiest way of doing that is to aktually use the openmediavault  menu.
go to plugins add fail 2 ban and openvpn and iscsi targets and whatever else floats ur boat.
reboot ur pi

DO NOT OPEN THE OPENVPN MENU! we not going to use it at all! we only installed it for the dependancy and library.

step 11 

SSh into the pi

time to navigate to /etc/openvpn

Download OVPN Config files (NordVPN)
cd /etc/openvpn
sudo wget https://downloads.nordcdn.com/configs/archives/servers/ovpn.zip
sudo unzip ovpn.zip

replace CC with country code and #### with server number you wish to use check nord vpn homepage for the numbers and codes

sudo nano /etc/openvpn/connect.sh
------------------------------------
sudo openvpn --config "/etc/openvpn/ovpn_udp/CC####.nordvpn.com.udp.ovpn" --auth-user-pass /etc/openvpn/auth.txt
------------------------------------

sudo nano /etc/openvpn/auth.txt
------------------------------------
Username@email.com
Password
------------------------------------

replace #.#.#.# with pi ip
sudo nano /etc/openvpn/iptables.sh
--------------------------------------------------------------
#!/bin/bash
# Flush
iptables -t nat -F
iptables -t mangle -F
iptables -F
iptables -X

# Block All
iptables -P OUTPUT DROP
iptables -P INPUT DROP
iptables -P FORWARD DROP

# allow Localhost
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Make sure you can communicate with any DHCP server
iptables -A OUTPUT -d 255.255.255.255 -j ACCEPT
iptables -A INPUT -s 255.255.255.255 -j ACCEPT

# Make sure that you can communicate within your own network
iptables -A INPUT -s #.#.#.#/24 -d #.#.#.#/24 -j ACCEPT
iptables -A OUTPUT -s #.#.#.#/24 -d #.#.#.#/24 -j ACCEPT

# Allow established sessions to receive traffic:
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow TUN
iptables -A INPUT -i tun+ -j ACCEPT
iptables -A FORWARD -i tun+ -j ACCEPT
iptables -A FORWARD -o tun+ -j ACCEPT
iptables -t nat -A POSTROUTING -o tun+ -j MASQUERADE
iptables -A OUTPUT -o tun+ -j ACCEPT

# allow VPN connection
iptables -I OUTPUT 1 -p udp --destination-port 1194 -m comment --comment "Allow VPN connection" -j ACCEPT

# Block All
iptables -A OUTPUT -j DROP
iptables -A INPUT -j DROP
iptables -A FORWARD -j DROP

# Log all dropped packages, debug only.

iptables -N logging
iptables -A INPUT -j logging
iptables -A OUTPUT -j logging
iptables -A logging -m limit --limit 2/min -j LOG --log-prefix "IPTables general: " --log-level 7
iptables -A logging -j DROP

echo "saving"
iptables-save > /etc/iptables.rules
echo "done"
#echo 'openVPN - Rules successfully applied, we start "watch" to verify IPtables in realtime (you can cancel it as usual CTRL + c)'
#sleep 3
#watch -n 0 "sudo iptables -nvL"
-------------------------------------------------------------------------

Enable IP Forwarding
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"

Create rc.local service
sudo nano /etc/systemd/system/rc-local.service
----------------------------------------------------
[Unit]
Description=/etc/rc.local Compatibility
ConditionPathExists=/etc/rc.local


[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99


[Install]
WantedBy=multi-user.target
------------------------------------------------------------
Create rc.local script
sudo nano /etc/rc.local
-------------------------------------------------------
#!/bin/sh -e
sudo bash /etc/openvpn/iptables.sh &
sleep 10
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
sudo bash /etc/openvpn/connect.sh &

exit 0
----------------------------------------------------------------------------------------

sudo chmod +x /etc/rc.local
Enable/Start rc.local service
sudo systemctl enable rc-local
sudo systemctl start rc-local.service

TO TEST
sudo bash /etc/openvpn/iptables.sh
sudo bash /etc/openvpn/connect.sh

Point another device’s default gateway at your VPN server. Ping google.com

CTRL-C to kill OpenVPN

Ping should stop on remote box.

If all is working, enable the service and reboot. IPTables are assigned each boot, and then the VPN connects. This allows troubleshooting or maintenance by commenting out rc.local calls and boot the server without IPTables restrictions.

step 12 OPTIONAL getting all containers their own ip ( only if you want each container a seperate ip for easy management otherwise IGNORE THIS )

instead of netplan yaml use the openmedia one 

https://www.youtube.com/watch?v=5XD-fHDmNtg


step 13

add pihole

use this guide https://www.youtube.com/watch?v=AFm6nd6GSJQ&list=PLhMI0SExGwfAU-UMeKxd1Lu5_a60AlA9N&index=9

and install what ever you wish from here https://www.youtube.com/playlist?list=PLhMI0SExGwfAU-UMeKxd1Lu5_a60AlA9N

( IF YOU WISH TO HAVE A FREE BITWARDEN PASSWORD MANAGER THEN DONT USE THE ONE IN DBTECH! IT REQUIRES DOMAIN PURCHASE USE THIS INSTEAD! )

https://www.youtube.com/watch?v=eCJA1F72izc

Now that you got a working pi server with vpn protection for your entire network and pihole and dns server and password manager and NAS server all in one! enjoy 

Step 14 optinal Retropie emulator games!

SSH into ur new nas all in one and time to add some gameing capability couse why not.

inside ur nas we wanna add some monitor capability couse that usually gets removed by openmedia installation.
cd into /boot
sudo nano config.txt

check for HDMI safe 1
uncomment that line

now you can close the ssh connection for now.

open the webbrowser and point it to ur all in one nas and log in with ur admin

go to users

create a new user for retropie ( DO NOT name it retropie but name it games or what ever ) make a group with SAME NAME aka games:games
give the new user a strong password and give it most of the linux rights except for openmedia sections, ( dont forget input since duuuh we need controller )

once you completed the task. head to ssh and ssh into the new retropie user games as in example.

once inside games in ur home directory add a new folder mkdir retropie
cd into it

we going to install retropie manually its easier than it sounds.

first do sudo apt update && sudo apt upgrade
press yes

attention! this next step will only work if you installed with us language local.
otherwise you need to verify ur local settings to the following. ( if you installed us language then u dont need this )
LANG=en_US.UTF-8
LANGUAGE=en_US:en
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=en_US.UTF-8

update-locale

sudo update-locale LC_ALL="en_US.UTF-8"

you can also use sudo raspi-config

wich brings uss to the next task gpu memory we need memory for the emulators.
sudo raspi-config and go to performance and set gpu memory 512 or 256

next we need some packages for retro pie

sudo apt install git lsb-release

sudo git clone --depth=1 https://github.com/RetroPie/RetroPie-Setup.git

cd RetroPie-Setup
sudo chmod +x retropie_setup.sh
sudo ./retropie_setup.sh

once ur here just basic install
when propmted about boot select BOOT TO CONSOLE ( autologin or no login dont matter just boot to console )

once installation finnishes takes about 20 min reboot
when thats done attach a controller to the usb or keyboard and hdmi
in ur ssh type emulationstation
when it boots up follow the instructions and then click menu and settings.
in settings disable the samba share.
thats it ur done ;I enjoy

input ur roms here /RetroPie/roms (or /home/pi/RetroPie/roms
and in correct folder and pew pew

if you wanna send the monitor over the network just export the display to the ip number of ur screen server and you can game remotely without hdmi
or just use hdmi and bluetooth controller ;i

enjoy

