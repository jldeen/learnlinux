Learn Linux – and setup a Media Server
Setup Infrastructure

Deploy out one (1) resource group with the following:
♣	1 Windows Server 2012 R2 VM
♣	1 Ubuntu 16.04 VM 
♣	1 NSG
♣	2 NICs
♣	2 Public IPs
♣	1 Virtual Network
♣	2 Storage Accounts
Example:

Open the following ports in your NSG:
♣	80 – HTTP
♣	22 – SSH
♣	3389 – RDP
♣	8181 – PyPlex Web App
♣	32400 – Plex Media Server
Objective – Setup a Plex Media Server and Python Web App with a CIFS mount to pull log data
Bonus – Configure vhosts conf file and use a reverse proxy
Setup Windows Server 2012 R2

Install the following software:
♣	Plex Media Server (free) - https://www.plex.tv/downloads/ - Will require restart (Hint: From run: ‘shutdown /r /t 00’ will remotely restart
Configure Plex Media Server from Windows VM

♣	Web browser should automatically open to http://127.0.0.1:32400/web/index.html#!/setup/   
♣	Create a new account (it’s free)
♣	You can close the window for Plex Pass – not necessary to use Plex
♣	Add Libraries (predefined ones are music and photos) I used Home Videos for the demo and pointed it to my local Videos folder - C:\Users\demo\Videos
♣	Navigate to %LOCALAPPDATA%\Plex Media Server\Logs\ and share the folder
Setup Ubuntu 16.04 LAMP Stack

Install the following packages
♣	sudo apt-get update
♣	sudo apt-get upgrade
♣	sudo apt-get install apache2 mysql-server mysql-client php7.0-mysql php7.0-curl php7.0-json php7.0-cgi php7.0 libapache2-mod-php7.0 php-cli
Verify successful installation on Linux
♣	Verify Apache by going to IP Address / DNS from web browser
♣	Verify MySql install by typing: sudo systemctl status mysql
♣	Verify PHP 7 version install by typing: php -v
Setup PyPlex Web App from Linux VM

♣	cd /opt
♣	sudo git clone https://github.com/JonnyWong16/plexpy.git
♣	cd plexpy
♣	sudo python PlexPy.py
♣	PlexPy will be loaded in your browser or listening @ http://localhost:8181
♣	To start PlexPy on startup, refer to Install as a daemon  
(OPTIONAL) - Daemon instructions below:
♣	sudo adduser --system --no-create-home plexpy
♣	sudo chown plexpy:nogroup -R /opt/plexpy/
♣	sudo wget https://raw.githubusercontent.com/JonnyWong16/plexpy/master/init-scripts/init.ubuntu.systemd && mv init.ubuntu.systemd plexpy.service
♣	sudo mv plexpy.service /lib/systemd/system/plexpy.service
♣	systemctl daemon-reload
♣	systemctl enable plexpy.service
♣	sudo service plexpy start
Setup PyPlex on Linux VM to connect to Windows Plex

♣	From a web browser, http://ip-address-of-linux-vm:8181 type your Plex user account and password created when Plex was setup on Windows
♣	Under Plex Media Server (below Plex Authentication) type the IP address or hostname of your Windows Plex Server – hit verify to confirm it found it
♣	Hit next after the above steps are completed and leave all other settings as is. (Next will need to be clicked 3-4 times)
♣	To allow logging – From PyPlex web browser go to Settings –> Plex Media Server and in the field for Logs enter the mount path for the cifs mount in the following section. 
* In the demo I used /opt/plexpy/MediaServer

Windows shares mounted on Linux – Samba
Linux Ubuntu 16.04 Instructions

♣	sudo apt-get install cifs-utils (may already be installed)
♣	sudo update-rc.d -f umountnfs.sh remove
♣	sudo update-rc.d umountnfs.sh stop 15 0 6 .
* Update the unmount order to prevent CIFS from hanging during shutdown.

♣	Create a .smbcredentials file under /root/ 
* (tip: touch /root/.smbcredentials then use a text editor to edit)
* sudo su (elevate cli session to root)
♣	sudo chown root .smbcredentials
♣	sudo chmod 600 .smbcredentials

Example .smbcredentials file: 
username=user-account-name
password=super-secret-password

♣	mkdir /opt/plexpy/MediaServer
♣	sudo vim /etc/fstab add new line: 
//local-ip-address/Logs /opt/plexpy/MediaServer cifs credentials=/root/.smbcredentials 
♣	sudo mount -a -v (mounts all entries in /etc/fstab and enables verbose mode)
* cifs is type of mount (another is smbfs)
* yellow highlight is location on local linux system where mount point will be made
vhosts.conf and ProxyPass

♣	cd /etc/apache2/sites-available
♣	touch sites.conf
♣	sudo vim sites.conf
♣	Paste in text from sites.conf.example (in GitHub repo) and modify accordingly
♣	sudo a2enmod proxy
♣	sudo a2enmod proxy_http
♣	sudo a2enmod rewrite
♣	sudo a2ensite sites.conf
♣	sudo service apache2 reload
♣	sudo systemctl restart apache2
Setup Second Site for vhosts demo

♣	sudo vim /var/www/html/info.php
Add this text and save the file:

<?php
phpinfo();
?>

♣	sudo chmod -R 755 /var/www/html
♣	sudo systemctl restart apache2
♣	Go to DNS name of second site
Expectation: 
- Should see apache landing page (index.html in /var/www/html)
- Go to http://dns-name-of-2nd-site/info.php and should see PHP landing page
