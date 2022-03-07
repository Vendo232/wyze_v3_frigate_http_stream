# wyze_v3_frigate_http_stream
How to setup Wyze V3 camera to work better with Frigate
How to use and setup Wyze V3 for Frigate Person Detection NVR
  
Version 1 ( March 2022 )
This guide is for those who want to use Wyze V3 camera on Frigate. If you experience issues and instability using Wyze`s original RTSP Firmware or your Bridge docker does not work correctly. This guide is for you. Setting up HTTP stream hack on Wyze V3 will help you use this camera with Frigate without any issues, securing smooth and continuouse stream between the cam and Frigate using your Wifi. 

Prerquisites
1.	SD Card ( 32 or 16GB )
2.	2 linux machines on your network ( could be virtual , running Ubuntu or like OS ) 
3.	DHCP Router ( your home network )

Downgrade WYZE CAM V3
1.	Downgrade your Wyze cam to earlier version 
2.	I`m using this version 4.36.0.228 (December 28, 2020)
Later revision could be used as well but since we will be using this cam for HTTP stream and all the post processing will be done outside Wyze installing newer version might not be needed and could cause issues installing the Telnet Hack.

Installing DNSMASQ to spoof Wyze server
1.	For this you will need 2 PC with Linux.
2.	On Machine 1 ( will call it DNS server ) run following commands ( assuming it is Ubuntu type OS )
3..	Find IP of this PC by typing ifconfig -a ( e.g. 192.168.1.100 )
4.	Your Machine DNS server for spoofing is 192.168.1.100 ( example )
5.	- sudo apt update
6.	sudo apt install dnsmasq
7.	sudo nano /etc/dnsmasq.conf
8.	to the bottom of the file add following lines. Make sure all other lines are deactivated using # in the begging of the line 
9.	no-dhcp-interface=
10.	server=8.8.8.8
11.	no-hosts
12.	addn-hosts=/etc/dnsmasq.hosts
13.	Sudo nano /etc/dnsmasq.hosts
14.	This should be blank file, add one line
15.	192.168.1.200 s3-us-west-2.amazonaws.com
16.	192.168.1.200 is IP  of the Machine 2 where you will run installation script from.
17.	Go to your Wifi router and under DNS add 192.168.1.100 as your DNS server. 
18.	Start DNSMASQ
19.	killall -9 dnsmasq
20.	dnsmasq --no-daemon --log-queries

Install Telnet
1.	This is installing modified WyzeHack which will enable you to Telnet to the cam.
2.	On your Machine 2 with IP 192.168.1.200 ( example ) running Ubuntu.
3.	Download wyze_hacks_0_7_00.zip  from https://github.com/Vendo232/wyze_v3_frigate_http_stream 
4.	Unzip your release archive a directory, and change your current working directory to there.
5.	Rename "config.inc.TEMPLATE" to "config.inc", and then update the content properly.
6.	Make sure your wyze_updater.py file contains this URL part. 
8.	Run "./remote_install.sh"
9.	Run “sudo python3 ./wyze_updater.py --token ~/.wyze_token --debug update -m WYZEC1-JZ -m WYZECP1_JEF -m WYZE_CAKP2JFUS -m WYZEDB3 -f ./firmware.bin -p 80” 
11.	First time, it will ask your Wyze account and password, and it may also ask for 2FA authentication.
12.	The login credentials will be stored in a local file named ".tokens" for future use so you don't need to enter username and password and 2FA everytime. Make sure you don't share this file with anyone you don't trust.
13.	The token seems to have an expiration period. So next time if you run into error with something like "Access token error" please delete ".tokens" file and restart.
14.	Once correctly authenticated, it will go through all the Wyze Cameras under your account, asking you for each of them if you want to push the wyzehack onto that camera. Enter 'Y' if yes, otherwise 'N'. Press "Ctrl + C" twice to interrupt the process.
And this is the tricky part where the installation script will be creating DNS loop between your Machine 1 running DNSMASQ and Machine 2 assuming it talks to legit Amazon server.
15.	When you run command you should see WyzeCam V3 LED goe purple whichindicates it is flashing FW. On Machine 1 ( assuming you use SSH ) you should run “dnsmasq --no-daemon --log-querie” and see that WyzeCam from IP 192.168.1.6 ( example ) is requesting domain from 192.168.1.200.
16.	If all goes well the purple LED will turn to Red and then Blue and you should be able to telnet to your Wyze Cam using Putty for example. 
17.	If it gets stuck on Broken Pipe msg or other error the Wyze cam will stay on Purple LED and you have to repeat the process and disconect and reconnect again the camera and run the command from point 9 again.
18.	Some cams go on first try some will take time unfortunatelly.

Install HTTP stream hack
1.	Use Putty to telnet to your Wyze Cam
2.	Login is “root” password is “WYom2020”
3.	cd /configs
4.	wget crb.users.sonic.net/current/install.sh
5.	chmod +x install.sh
6.	./install.sh
7.	You should see msg “ Installed Successfully”

Frigate
1.	Almost there!
2.	In your Frigate config.yaml file your camera setup will be something like this
3.	inputs:
4.	        - path: http://192.168.1.174:12345 
5.	          roles:
6.	            - detect
7.	            – record
8.	The path to your camera is http://192.168.X.XXX:12345 replacing XXX with actual IP of the Wyze cam.
9.	Also it is a good practice to lock IP on your DHCP but that you already know.

