# piserver-vpnrouter
Piserver with vpnrouter builtin with nordvpn


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
