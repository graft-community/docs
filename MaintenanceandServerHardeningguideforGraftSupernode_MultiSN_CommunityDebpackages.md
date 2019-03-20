# Maintenance and Server Hardening guide for Graft Supernode | Multi-SN - Community Deb packages

- A big thanks goes out to ***@Jagerman42*** (Telegram Handle) for the effort that goes into making the Graft Community Builds and deb packages possible and his input into providing guidance on the below guides material.

- A big thanks also goes out to ***@el_duderino_007*** (Telegram Handle) for the time and effort put into creating the dummies guide and supporting the community in his role as community admin on multiple groups.

- In order to follow this guide please install nano if it does not exist on your system already or use your preferred text editor, vi for example. 

- This guide also assumes the name of the user performing the actions is "graft" please change at your discretion to the user your are using on your system.

- For the purpose of the guide we also assume the folder directories where we run supernode from is sn1, sn2, sn3 etc, this can be anything you like as long as each directory is unique for each Supernode. 

- We also include a logs directory inside of each Supernode directory, once again this is completely up to the user. 

- We will assume that the user has a general understanding of how operate the graftnoded and supernode applications, Linux distros and staking their supernode, if not please refer to the below directory for guides.

### [Graft Community Documentation](https://github.com/graft-community/docs)


- This guide does also assumes the user has the Graft Community deb packages installed and not the official binaries or community binaries.  
## To install the graft community deb packages, please refer to the below link for quick and easy to follow instructions:
### [deb.graft.community](https://deb.graft.community/)


## Setting up our folder structure

	mkdir -p ~/sn1/logs
	mkdir -p ~/sn2/logs
	mkdir -p ~/sn3/logs

- Copy config.ini into created sn directories

````
	cp /usr/share/doc/graft-supernode/config.ini ~/sn1/config.ini

	cp /usr/share/doc/graft-supernode/config.ini ~/sn2/config.ini

	cp /usr/share/doc/graft-supernode/config.ini ~/sn3/config.ini
````

- Edit config.ini files

````
	nano ~/sn1/config.ini
````

- The Following values are the important values to change when running multi-sn and should be unique per Supernode:

````
	http-address=0.0.0.0:18690
````

````
	data-dir=
````

- Change the port on ***http-address=***

- Change the directory according to the folders you created in the first step for ***data-dir=*** for each respective config.ini in each directory.
- Something like :

````
	data-dir=/home/graft/sn1/
````

````
	data-dir=/home/graft/sn2/
````

- Obviously as per all other guides instructions, add your wallet into config.ini that you will be using to stake with:

	wallet-public-address=<Your_wallet_address>


- Start supernode and check that supernode.keys is inside your directory for each sn. Refer to below where we setup systemd to launch each Supernode.


## Systemd configuration

- Create systemd service for graftnoded:

```
	sudo nano /etc/systemd/system/graftnoded.service
```

- Input below text into the new file:

````
[Unit]
Description=Graft node Mainnet
After=network-online.target

[Service]
User=graft
Type=simple
WorkingDirectory=~
Restart=always
ExecStart=/usr/bin/graftnoded --non-interactive
#LimitNOFILE=8192
Environment=TERM=xterm

[Install]
WantedBy=multi-user.target
````
- Cntrl + x to save

#### Enable graftnoded.service to automatically start graftnoded after boot.

````
	sudo systemctl enable graftnoded.service
````

- The output will advise you of a symlink being created something like below:
````
	Created symlink /etc/systemd/system/multi-user.target.wants/graftnoded.service → /etc/systemd/system/graftnoded.service.
````

#### Start Graftnoded
````
	sudo systemctl start graftnoded.service
````

#### Creating the graft-supernode.service file:
````
	sudo nano /etc/systemd/system/graft-supernode@.service
````

- Input below text into the new file:

````
[Unit]
Description=Graft supernode
After=graftnoded.service

[Service]
User=graft
Type=simple
WorkingDirectory=/home/graft/%i
Restart=always
ExecStart=/usr/bin/graft-supernode --config-file config.ini --log-file /home/graft/%i/logs/%i.log
Environment=TERM=xterm
#LimitNOFILE=8192
````
Cntrl + x to save

In the above
	
   - The "%i" is a variable which is defined when we run the enable command below. the input between @ and . is input as a variable into our file on creation and the same goes for when we start the service. 

In the below example: 
````    
    sn1 would be our variable in the first instance. 
````
````
    sn2 would be the variable in the second instance.
````

#### Enable graft-supernode@.service to automatically start graft-supernode after boot and to only start if graftnoded is up and running.

- sn1

````
	sudo systemctl enable graft-supernode@sn1.service
````
- sn2

````
	sudo systemctl enable graft-supernode@sn2.service
````

#### Starting graft-supernode with systemd

- sn1

````
	sudo systemctl start graft-supernode@sn1.service
````

- sn2

````
	sudo systemctl start graft-supernode@sn2.service
````

Now we can also stop the graft-supernode service with the below:
- sn1

````
	sudo systemctl stop graft-supernode@sn1.service
````

# Configuring Logrotate to rotate very 5 days and compress daily

```
	sudo nano /etc/logrotate.d/sn1-logs
````

- Input below text into the new file:

````
/home/graft/sn1/logs/sn1.log {
    daily
    rotate 5
    missingok
    compress
    create
}
````
- Cntrl + x to save
```
	sudo nano /etc/logrotate.d/sn2-logs
```
- Input below text into the new file:
````
/home/graft/sn2/logs/sn2.log {
    daily
    rotate 5
    missingok
    compress
    create
}
````

- Cntrl + x to save

- To check that logrotate is actually working we can run the below command:
````
sudo /usr/sbin/logrotate -f /etc/logrotate.conf
````
- Then check the log file directory for a compressed version of the log-file.

- We then check the chron job to ensure that force is enabled so we are assured the log-files will be compressed and rotated daily.
	- Check the logrotate chron script exists
````
sudo cat /etc/cron.daily/logrotate
````
- If it exists it will return something like:
```
#!/bin/sh

# skip in favour of systemd timer
if [ -d /run/systemd/system ]; then
    exit 0
fi

# this cronjob persists removals (but not purges)
if [ ! -x /usr/sbin/logrotate ]; then
    exit 0
fi

/usr/sbin/logrotate /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit $EXITVALUE

```

- In theory we now should be good to go. You can add the -f switch to the line:
```
/usr/sbin/logrotate -f /etc/logrotate.conf
```

- To force the logrotate to happen, in case of some files no meeting the requirements.

Port forwarding is required to the graftnoded port as of the time of writing to ensure that you supernode can communicate with other supernodes and appear as active.

Port for mainnet graftnoded at the time of writing is : ***18980***


## Hardening our server:

- The simplest and most effective way to keep our server safe is keep unnecessary software on our server to a minimum and and to always stay up to date with new packages.
	- sudo apt update && sudo apt ugrade -y
	- sudo apt autoremove

### Enabling ufw:
````
sudo apt install ufw -y
````
**Opening the neccessary ports:**

- SSH:
```
sudo ufw allow 22
```
- Graftnoded port:
````
sudo ufw allow 18980
````
- Enabling UFW (***BEWARE OF LOCKING YOURSELF OUT HERE***) ensure your SSH port is opened:
````
sudo ufw enable
````

### Configuring SSH
This assumes you are connecting to you machine via SSH but have used a password to login, lets enable login via a non-root user with no password. Again we will assume the user is "graft" that is being used on the local machine.

- Press enter to accept defaults unless you already have a SSH key present on your machine and that makes this step unnecessary
```
ssh-key-gen 2048
```
- Press: ENTER
```
ssh-copy-id graft@<remote-machine-ip
```
- Enter password.
- Now you can use:
```
ssh  graft@<remote-machine-ip
```
to Login with no password.

Next step is disable root logins via ssh, obviously ensure you have created another user to ensure you dont lock yourself out. 
- This step is general SSH best practice as every Linux/Unix distro uses root as the admin user and therefore is an easy target for brute force attacks. 
- Another good idea is to change the port to a custom port for ssh and use the switch "-p <port-number>" with the ssh command.
- Once again ensure you open the port on ufw if enabled to ensure you dont lock yourself out.

```
sudo nano /etc/ssh/sshd_config
```
- Then uncomment the below line and change value to "no"

````
PermitRootLogin no
````
- To make this take effect immediately:

	- Restart the SSH daemon to apply changes by issuing one of the below commands, specific to your Linux distribution.

```
sudo systemctl restart sshd
sudo service sshd restart
sudo /etc/init.d/ssh restart
```
- Now the root user cannot login via SSH.
- You can change to the root user at anytime by using the below command:
```
su -
```
and inputting the password.

- Changing the port is inside the same file as PermitRootLogin:
```
nano /etc/ssh/sshd_config
```
- Uncomment the Port=22 line and input your custom port instead of 22.
	- Again restart SSH daemon to make it take effect, beware as this could kick you off and lock you out, ***ENSURE PORT IS OPEN ON UFW!***



# USEFUL LINKS & RESOURCES

- [Digital Ocean Guide for SSH Setup](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-debian-9)

- [Digital Ocean Guide for Systemd management of services](https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units)

- [Linode using logrotate to manage log files](https://www.linode.com/docs/uptime/logs/use-logrotate-to-manage-log-files/)

Written by ***@Fezz27*** (Telegram Handle)
