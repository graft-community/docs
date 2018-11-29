# Guide Built for Ubuntu - Debian Graft Docker container

Thanks to Jason @jagerman42 (Telegram handle) for the alpha 3 code fork and optimizations included, including the wizard to install his packages and the watch-only-wallets download for testnet. https://github.com/graft-community/GraftNetwork  

Thanks to MustDie95 for the initial Docker build on Alpha3 code which I used as a backbone to get this running and tweaked his supervisor config to make the graftnoded and graft_server start automatically once the docker container starts. https://github.com/MustDie95/graft  

Thanks to Tiago S @el_duderino_007 (Telegram handle) for the blockchain download and initail guide for installing Alpha3 code on server.

Thanks to @mv1879 for the assistance in building this guide in markdown and in testing the guide, implementation and optimizing this guide.

Thanks to @CryptoRobo747 for assitance in testing the guide and implementation.

Tested on Debian 9.6 & Ubuntu 18.04 & Ubuntu 16.04 using stable Docker repo.
 
## Resources:
* Telegram Handle: @Fezz27
* Github: https://github.com/Fez29/fez-graft-docker.git
* Docker Hub: https://hub.docker.com/r/fez29/graftnoded-jagerman/

## Add User
Logged in as root and "username being your username you have chosen":
````bash
adduser username
````
* Set and confirm the new user's password at the prompt. A strong password is highly recommended!
````bash
Set password prompts:
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Note: if you are running as root please exclude any sudo references below:
````
* Follow the prompts to set the new user's information. It is fine to accept the defaults to leave all of this information blank.
````bash
User information prompts:
Changing the user information for username
Enter the new value, or press ENTER for the default
    Full Name []:
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
Is the information correct? [Y/n]
````
````bash
usermod -aG sudo username
````
By default, on Ubuntu, members of the sudo group have sudo privileges.
* Use the su command to switch to the new user account.
````bash
su - username
````
* As the new user, verify that you can use sudo by prepending "sudo" to the command that you want to run with superuser privileges.
````bash
username# sudo command_to_run
````
* For example, you can list the contents of the /root directory, which is normally only accessible to the root user.
````bash
username#  sudo ls -la /root
````
* The first time you use sudo in a session, you will be prompted for the password of the user account. Enter the password to proceed.
````bash
Output:
[sudo] password for username:
````
If your user is in the proper group and you entered the password correctly, the command that you issued with sudo should run with root privileges.

## Install Docker as non-root user
Note: if you are running as root please exclude any sudo references below:

````bash
sudo apt-get remove docker docker-engine docker.io
sudo apt-get update
sudo apt-get upgrade -y    # (for all promts just select keep as is) Optional
````

Install Docker dependencies (Copy entire command - all 6 lines and paste)

````bash
sudo apt-get install -y \
apt-transport-https \
ca-certificates \
curl \
gnupg2 \
software-properties-common
````
 
 Add Docker’s official GPG key:

````bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
````

Test success with:

````bash
sudo apt-key fingerprint 0EBFCD88
````

Should return:
````bash
	pub   4096R/0EBFCD88 2017-02-22
	      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
	uid                  Docker Release (CE deb) <docker@docker.com>
	sub   4096R/F273FCD8 2017-02-22
````

## Add docker repo

Stable repo:
````bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
````
Edge repo:
````bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic edge"
````

Update apt again and install docker Community Edition:
````bash
sudo apt-get update
sudo apt-get install -y docker-ce
````

Prepare Docker volume mount:
````bash
mkdir $HOME/.graft
````

## Download image and run container after with mounted volume 
<B>(Check [Optional](https://github.com/mv1879/docs/blob/master/Dockers%20by%20Fez.md#optional) at botom of this guide before continuing)</B>
:
````bash
sudo docker run --name graft -d -v $HOME/.graft:/root/.graft -p 28690:28690 -p 28680:28680 fez29/graftnoded-jagerman:Jagerman-Experiment_fez29
````

Use docker exec to login to the container as root:
````bash
sudo docker exec -ti graft /bin/bash
````

Note that graftnoded and graft_server are automatically started and restarted if they die by by supervisor - See optional if you want to disable/adjust behaviour. 
(Check section in optional regarding downloading watch-only-wallets now if want to action)
Check sync status:
````bash
graftnoded --testnet status
````

Check wallet address for Supernode:
````bash
graft-wallet-cli --wallet-file ~/.graft/supernode/data/stake-wallet/stake-wallet --password "" --testnet --trusted-daemon
````
Record seed and store safely especially on Mainnet (KEEP OFFLINE or written down and never reveal to anyone!)
when in wallet type: seed and follow prompt (no password just press enter)

Once graftnoded fully synced and stake in wallet, kill graft_server process to speed up process.  
Kill graft_server process:
````bash
kill -9 `pidof graft_server`
````
graft_server will start automatically again.

Viewing logs: (-n {150} - 150 equals lines to view – adjust accordingly)

Graft_server logs:
````bash
cd /home/graft-sn/supernode
tail -f -n 150 supernode.log
````
Graftnoded logs from anywhere:
````bash
tail -f -n 150 /$HOME/.graft/testnet/graft.log
````
Checking locally inside container if supernode list is being generated:
Install curl:
````bash
apt update
apt install curl -y
curl 127.0.0.1:28690/debug/supernode_list/0
````

Do not forget to open ports for 28690 and 28680 on VPS or port forward if hosting locally.

Test list from external:
````bash
<external-IP>:28690/debug/supernode_list/0
```` 
## Docker commands

Stop and Start container (use name from docker run command)
````bash
sudo docker stop graft
sudo docker start graft
````

To setup docker so that container starts automatically if machine restarted:

On running container after exiting container - just type exit when logged in to exit
````bash
sudo docker container update --restart unless-stopped graft
````
graft in above is the --name used in the docker run command


## Optional:
(Beware of using below on mainnet is much safer to download Blockchain youself | also please consult @jagerman42 before using 
any watch-only-wallets downloads on mainnet):

Before running the image download and docker run step, download provided blockchain directory.
````bash
mkdir $HOME/.graft		# (if not done already)
mkdir $HOME/.graft/testnet && mkdir $HOME/.graft/testnet/lmdb && cd ~/.graft/testnet/lmdb && wget https://rta.graft.observer/lmdb/data.mdb
````
Go back to: [Download image and run container after with mounted volume ](https://github.com/mv1879/docs/blob/master/Dockers%20by%20Fez.md#add-docker-repo)

## ADVANCED

Recommended Docker container updates
Restart container automatically on reboot of server:
````bash
sudo docker container update --restart unless-stopped graft
````
Limit cpu usage to 2 cores: (adjustable)
````bash
sudo docker container update --cpus=2 graft
````

Download Watch only wallets:
Login to already running container:
````bash
sudo docker exec -ti graft /bin/bash
mkdir -p ~/.graft/supernode/data/{watch-only-wallets,stake-wallet} && cd ~/.graft/supernode/data && curl -s https://rta.graft.observer/lmdb/watch-only-wallets.tar | tar xvf -
````
Kill graft_server process:
````bash
kill -9 `pidof graft_server`
````
graft_server will start automatically again.
cd back to location for supernode logs 
````bash
cd /home/graft-sn/supernode
tail -f -n 150 supernode.log	# to view logs
```` 

To adjust supervisor behaviour:
Install nano:
````bash
apt update
apt install nano -y
cd /etc/supervisor/conf.d/
nano graftnoded.conf
nano graft_server.conf
````
restart container by exiting container:
Type: exit  until you see you have exited the container.
````bash
sudo docker stop graft
sudo docker start graft
````

## To build image from scratch:
Install Git:
````bash
apt update
apt install git -y
````

Clone repo:
````bash
git clone https://github.com/Fez29/fez-graft-docker.git
cd fez-graft-docker
````

Build image yourself:
````bash
docker build - < Dockerfile -t <chosen_name>:<chosen_tag>
````
eg mine was built with:
````bash
docker build - < Dockerfile -t fez29/graftnoded-jagerman:Jagerman-Experiment_fez29
````
Then got to Prepare Docker volume mount: section of guide. Or optional section for Blockchain download

## Adding a second container

````bash
mkdir ~/.graft2
````
Use same docker run command but edit the exposed port like below:
````bash
sudo docker run --name graft2 -d -v $HOME/.graft2:/root/.graft2 -p 38690:28690 -p 38680:28680 fez29/graftnoded-jagerman:Jagerman-Experiment_fez29
````
Then go supervisor config and add p2p-external-flag switch to tell graftnoded exposed port is not asme as its bound to like (38680 being the port used in the above run command):

How to get there: 
````bash
cd /etc/supervisor/conf.d
````
````bash	
nano graftnoded.conf
````
````bash
command=graftnoded --testnet --p2p-external-port 38680 --detach
````