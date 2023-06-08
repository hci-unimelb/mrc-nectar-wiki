<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
<!-- *generated with [DocToc](https://github.com/thlorenz/doctoc)* -->

# Melbourne Research Cloud Wiki
A guide on how to request and set up a Melbourne Research Cloud server, from an HCI perspective including data collection, internet facing services and even data analysis.

## Official Guides
**Your First Instance**: https://docs.cloud.unimelb.edu.au/training/first_instance/
**Setting up an SSH Client**: https://docs.cloud.unimelb.edu.au/guides/ssh_client/ 

## **Table of Contents**  

- [Concepts](#concepts)
  - [Project and Allocations](#project-and-allocations)
  - [Instance](#instance)
  - [Volume](#volume)
  - [Key Pair](#key-pair)
  - [Security Groups](#security-groups)
- [Allocation request](#allocation-request)
- [Setup and Launch an instance](#setup-and-launch-an-instance)
  - [Details](#details)
  - [Source](#source)
  - [Flavour](#flavour)
  - [Networks](#networks)
  - [Security Groups](#security-groups-1)
  - [Key Pair](#key-pair-1)
  - [Configuration](#configuration)
  - [Server Groups](#server-groups)
  - [Metadata](#metadata)
  - [Launch it!](#launch-it)
- [SSH and SFTP access](#ssh-and-sftp-access)
  - [SSH Keys Setup and Permissions](#ssh-keys-setup-and-permissions)
  - [Software](#software)
  - [Converting private PEM keys to new format](#converting-private-pem-keys-to-new-format)
  - [Connecting to your instance](#connecting-to-your-instance)
  - [Terminal and Powershell](#terminal-and-powershell)
  - [Windows (Solar Putty)](#windows-solar-putty)
  - [Connect](#connect)
- [Set up Security Groups](#set-up-security-groups)
- [Set up VSCode on the server](#set-up-vscode-on-the-server)
  - [Port Forwarding and Security Groups](#port-forwarding-and-security-groups)
    - [**Edit code-server config**](#edit-code-server-config)
  - [Allow access through webservice (OPTIONAL)](#allow-access-through-webservice-optional)
    - [**Edit NGINX Configuration**](#edit-nginx-configuration)
  - [Debugging](#debugging)
- [Setup the Database Instance](#setup-the-database-instance)
  - [Details](#details-1)
  - [Networking](#networking)
  - [Database Access](#database-access)
  - [Initial Databases](#initial-databases)
  - [Advanced](#advanced)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Concepts
### Project and Allocations
When you submit an allocation request you are actually requesting a project, not a server. A project can include compute instances, volumes, database instances etc...
### Instance
Servers as we usually intend them so a machine running an OS like Ubuntu. There can be multiple instances, but all together they cannot exceed the quota of virtual CPUs (vCPUs in short) allocated. Technically you are not getting your own dedicated machine but just some resources on a shared machine. This means that your CPUs are virtualized, the server might have 100 physical CPUs and only allocated 24 of those to you as virtual CPUs

### Volume
You can see volumes as an external drive that you can connect to your instance(s) at will. It can be extremely useful to avoid large amount of data taking space on your instance's internal drive's space. Besides, you can attach the same volume to multiple instances, effectively sharing data between them. You can obviously mount a volume on a specific directory and create a symbolic link to it.

For instance say you have a ML project with a huge data folder, that data folder could be a symlink to the volume, so it won’t be using your machine’s internal SSD space but the folder will still appear in the path where you need it. The data is safely stored in the volume so whatever happens to your instance, your data won't be deleted.

### Key Pair
For security reasons, only devices using the correct ssh key can connect to an instance.
You can use the same key pair on multiple instances.

It would be better if the SSH key is unique per device and not per person: e.g. Have two SSH keys, one from your Uni MacBook and another one for another device.

The Key Pair is well... a pair, consisting of: A public key which resides on the instance, and a private key that you should store safely on your machine.

To get access to an instance you should first generate a key paid and only send the public key to the server admin (or the server admin will provide you with one). Either way, you should not share your private key with anyone.
### Security Groups
Again for security reasons, instances only allow certain type of connections to access them. The most common type of connections are `HTTP`, `HTTPS` and `SSH` and they use port `80`, `443` and `22` respectively.

Say you want to host a Jupyter Notebook on your server and make it available to anyone through the instance's IP address and port.
If the notebook is running on port `9000` then you should create a new security group allowing TCP connections from/to the instance through port `9000`
## Allocation request
TODO
## Setup and Launch an instance
From your Dashboard's sidebar, go to Compute and then Instances.
Click on "Launch Instance". On left side of the popup that just opened you’ll see the different steps, you can follow each subsection here to configure that step if you need it.

### Details
* Give it whatever name you want (it does not really matter especially if you’ll only have one instance)
   1. Count = 1 if you only need one server (machine) and you most likely only asked for one instance anyway
   2. Availability zone you can keep the default one
* Choose the settings in the next subsection

### Source
* Boot source "Image"
* Create new Volume (So you don’t have to do it yourself afterwards)
   1. Select whatever size you have available (e.g 500-100gb)
   2. Make sure to select "NO" for Delete Volume on Instance" so that even if you delete the instance by mistake, your data on that volume is safe (e.g that ML project data)
* I’d probably go for `Ubuntu 18.04 LTS` (version 20 or 22 might not be compatible with certain packages) but now they have new images with pre-installed stuff like the image with JupyterLab, at the time of writing we used this one:
   1. `NeCTAR Jupyter Notebook (Ubuntu 18.04 LTS Bionic) [v28]`
   2. Click on the little up-arrow to add this to the allocated instances (only one in this case, but you can select multiple if you have more than 1 instance)


### Flavour
Now it’s time to choose the server "flavour" and it depends on what has been allocated to you
* You can check your available resources in Compute -> Overview
   1. Check how many cpus you have available and what’s the RAM limit 
   2. The ones above your resources should be already flagged
* Now choose the flavour that sounds the best for your needs e.g.:
   1. Large datasets to analyse (no deep learning training), it’d be nice to have A LOT of RAM and lots of CPUs
   2. Just a simple webpage then even 4gb of RAMs and 2 vCPUs are ok
   3. A common one could be the `24 vCPUs` and `1000 GB` of RAM (976Gb actually)
* Click on NEXT and go to "Networks" tab:        

### Networks
We usually do not need to configure this if we only have one instance. This would be needed if you have multiple instances where only some of them should be facing the internet (public subnets) while others would only be communicating internally through private subnets

### Security Groups
Here you might want to add all 3 groups to your instance or you might not even be able to access it, so make sure that under "Allocated" you’ll see
* Default
* HTTP and HTTPS (80 and 443)
* RDP (3389)
* SSH (22)


### Key Pair
This is where you give SSH/SFTP access to people through public and private keys.
* Click on create key pair
* Give it a name that reminds you which machine is using this key
   * E.g. Gabry-UniMacbook
* Key type: SSH Key
* Create it
* Copy the whole content (you can also just click the button to copy to clipboard)
* Paste EVERYTHING (do not delete anything) to a new file called something like MRC_Server.pem
* Make sure the newly created SSH appears under "Allocated"

### Configuration
This is a bash script that will run when setting up the instance you can leave this empty, you can always do it afterwards

### Server Groups
We only have one instance so we don’t really need to create a server group and add it to the group, skip this step

### Metadata
This can be useful to know what services the instance offers. For instance if you have multiple instances and one offers MySQL and the other one offers RStudio, you can add these tags. 
Keep in mind that this does not affect the instance or the configuration, it’s just a way to immediately know what services the instance offers and which port they are attached to.
Let’s say you’re going to install certain services, you can add the tags so you know what’s available through the instance’s IP address:
* MySQL (port 3606)
* Python, add which version you’re going to use so you know this machine uses that version of python
* Make sure you not leave empty metadata fields or it won’t let you launch the instance


### Launch it!
It might take a while, just wait for it to get built and ready
Make a note of the IP address so you can start setting up the SSH access.


## SSH and SFTP access
### SSH Keys Setup and Permissions
Store your private keys into a folder `~/.ssh`. Each file should have the correct permissions:
* SSH directory (`~/.ssh`): `700 (drwx------) `
  ```bash
  $ sudo chmod 700 ~/.ssh
  ```
* Public keys, the `.pub` files on the machine you are going to use to connect to the server: `644 (-rw-r--r--)`
  ```bash
  $ sudo chmod 644 ~/.ssh/*.pub
  ```
* Private keys id_rsa ant authorized_keys stored in ~/.ssh on the server you want to access: `600 (-rw-------)`
  ```bash
  $ sudo chmod 600 ~/.ssh/id_rsa
  ```
* Lastly your home directory should not be writeable by the group or others (at most `755 (drwxr-xr-x)`)
  ```bash
  $ sudo chmod 755 ~
  ```

If for some reasons you want to use Windows PowerShell to connect via ssh you can set the right persmissions in this way. If this is the case, please reconsider your life choices (I'm just kidding of course! But Unix or MacOS with their terminals definitely make life much easier)
```powershell
icacls .\private-key.ppk /inheritance:r
icacls .\private-key.ppk  /grant:r "%username%":"(R)"
icacls .\private-key.ppk  /grant:r ${env:UserName}:F
TakeOwn /F .\private-key.ppk
Icacls .\private-key.ppk /c /t /Grant:r ${env:UserName}:F
```
### Software
**SSH**
* On Unix and MacOS machines you can just use the **terminal**.
* On MacOS I use a program called [**Termius**](https://termius.com/) or a modern new terminal app called [**Warp**](https://www.warp.dev/)
* On Windows you can use good ol [**PuTTY**](https://www.putty.org/) or the newest updates [**Solar-PuTTY**](https://www.solarwinds.com/free-tools/solar-putty)

**FTP/SFTP**
* You can use [**Termius**](https://termius.com/)  again on MacOS
* You can also use [**FileZilla**](https://filezilla-project.org/) which is available for all platforms
* You can also use [**WinSCP**](https://winscp.net/eng/download.php) on windows
* Or [**Cyberduck**](https://cyberduck.io/) on Mac


### Converting private PEM keys to new format
The private key PEM format used by MRC seems to updated and Solar Putty or other software won’t let you use it.

**Windows**
* Download PuTTY and PuTTYGen
* In PuTTYGen, in the 'Conversions' menu choose 'Import' and load your .pem key
* From the top menu bar click Key -> Parameters for Saving key files…
   * Select PPK file version 2 (NOT 3, that’s TOO new…)
* 'Save private key' to a different file
* Make sure to select the new key if you were using the old one before

**Unix**
```bash
$ sudo apt install putty-tools
$ puttygen server-priv-key.pem -O private -o server-priv-key.ppk
```

**MacOs**
```bash
$ brew install putty
$ puttygen server-priv-key.pem -O private -o server-priv-key.ppk
```


### Connecting to your instance
You can always connect your instance from your MRC Dashboard:
* Go to Instances
* Click on the little arrow in the dropdown menu that says "Create Snapshot"
* Click on Console

### Terminal and Powershell
You can very quickly connect to your instance like this:
```bash
$ sudo ssh -i ~/.ssh/privatekey.pem ubuntu@<ip-address>
```
You can also create an alias for it
```bash
$ alias connect_instance="sudo ssh -i ~/.ssh/privatekey.pem ubuntu@<ip-address>"
```

On Windows PowerShell you can connect with this (after setting the right permissions as shown in one of the previous steps:
```powershell
ssh -i D:\SSH\gabryxx7.pem ubuntu@103.6.254.6
```
### Windows (Solar Putty)
Create a new session
* Name: Something like "MRC UniMelb [PROJECT]"
* IP: Your instance’s IP address (it does not matter if it’s not ready and running yet)
* Port: `22`
* Type of connection: `SSHv2`
* Credentials (create new credentials)
   * Username: Your machine’s username (the default one for MRC instances is `ubuntu`)
   * Password: leave blank, you should use the private key (converted `.pem` file you created earlier) instead
   * Private key: Select the private key (converted to the new format) file you created earlier
   * Passphrase: If you created the key through MRC it should be empty, but other key generation programs might allow you to use a passphrase when generating it.
   * Credentials name: Just a name to remember what credentials these are (you can just use the filename if you want)
* Click Save

Now try to connect to your instance:
* If it does not work you might want to enable logging and check the logs in `C:/ProgramData/SolarWinds/Logs`
* It usually is a problem with the old public key format (old PEM format), check the previous subsection to convert it 

### Connect
Wait for the instance to show "Running" (you actually have to refresh the page on MRC, it does not update automatically!)

Now try to connect to your instance...
* The first time it will ask you whether you trust the connection and want to add it to the trusted servers, say yes


## Set up Security Groups
This is probably the most important step when it comes to publishing services to the internet or just overall being able to access your instance from a browser.

Here you can choose which port or port ranges are open to the internet, here is a list of DEFAULT ports you might want/need for different services:
* HTTP: `80`
* SSL/HTTPS: `443`
* FTP: `21`
* SSH: `22`
* Jupyter Notebook: `8888`
* MYSQL: `3306`
* code-server: `8080`



## Set up VSCode on the server
The easiest way to edit code and access the terminal on your instance is to just have an online version of visual studio code running on your server, and which you can reach through the browser.

The server version of VSCode can be found here: Code-server https://github.com/coder/code-server

To install it, just run this line in the terminal after you connected through SSH:
```bash
$ curl -fsSL https://code-server.dev/install.sh | sh
```
Enable the daemon to make sure the server runs in the background, automatically restarts and starts with the server:
```bash
$ sudo systemctl enable –now code-server@$USER
```
You can check its status and what port or address is running on with this (give it a tab at the end to autocomplete):

```bash
$ sudo systemctl status code-server
```

The status will tell you which port it’s being served on like `http://127.0.0.1:8080`.

You can try to access `code-server` on your browser by going to `http://<server-ip-address>:8080` Make sure you're using `http` as we have not set up any certificate yet.

> It’s likely it won’t work on the first time because the security groups are not properly set up.


### Port Forwarding and Security Groups
From the sidebar go to Network -> Security Groups
* Create a new security group
* Call it with the name of the service whose ports you’re forwarding (opening) like `VSCode`
* Create it
* Add Rule:
   * Custom TCP Rule
   * Ingress
   * Port: 8080
   * CIRD 0.0.0.0
* Add Rule:
   * Custom TCP Rule
   * Edgress
   * Port: 8080
   * CIRD 0.0.0.0
* Add Rule:
   * Custom UDP Rule
   * Ingress
   * Port: 8080
   * CIRD 0.0.0.0
* Add Rule:
   * Custom UDP Rule
   * Egress
   * Port: 8080
   * CIRD 0.0.0.0        
Go back to Compute -> Instances
From the dropdown that says "Create snapshot" click "Edit Security Groups", now add the newly created security group


#### **Edit code-server config**
```bash
$ sudo nano ~/.config/code-server/config.yaml
```
Make sure the bind address is something like `0.0.0.0:8080`. If you leave it as 127.0.0.1 it will only allow people connecting locally to access it, `0.0.0.0` binds to any IP address, this is especially true with `SSL`.
The yaml file should be something like this:
```yaml
bind-addr: 0.0.0.0:8080
cert: false
auth: password
password: xxxxxxxx
```

Restart code-server:
```bash
$ sudo systemctl restart code-server@$USER
```

### Allow access through webservice (OPTIONAL)
In case this is still not working and in case you want to serve code-server on a subdomain or on a different URL path, you can try these steps to configure it.

First we need to figure out what webservice is running (there should be one already if you use a JupyterLab OS image). You can check if you’re using nginx or apache by running these two:
```bash
$ sudo systemctl list-units --type=service | grep nginx
$ sudo systemctl list-units --type=service | grep apache
```

#### **Edit NGINX Configuration**
You can find the `code-server` guide here: https://coder.com/docs/code-server/latest/guide#using-lets-encrypt-with-nginx

* Move to the nginx config folder:
  ```bash
  $ cd /etc/nginx
  ```
* Check what’s available:
  ```bash
  $ ls -lh
  ```
* Check the available sites:
  ```bash
  $ ls -lh ./sites-available
  $ cat ./sites-available/default
  ```
* Check the enabled sites:
  ```bash
  $ ls -lh ./sites-enabled
  $ cat ./sites-enabled/vhosts.conf
  ```

For some reason the Jupyter OS Image has the configuration file `vhosts.conf` under `sites-enabled` which is usually frowned upon. The config file should be in the folder `sites-available` and the folder `sites-enabled` should only have symlinks to these files, as the official `nginx` guide explains.

Anyway, let’s copy that vhosts config since we know it’s working for JupyterLab and it's a good starting point.
If you have the file you can copy it to the sites-available folder:
```bash
$ sudo cp ./sites-enabled/vhosts.conf ./sites-available/code-server.conf
```
Otherwise just create a new file with:
```bash
$ sudo touch ./sites-available/code-server.conf
```

We can edit it to use the port we need so it. You can run this:
```bash
$ sudo nano ./sites-available/code-server.conf
```
Edit the file with `nano`, you can comment out lines by adding a `#` in front.
Generally speaking configurations work like this:
```nginx
# resolve domain with no port or port 80
server {
  listen 80;
  server_name example.com www.example.com;
  ...
}

# resolve domain for port 8080
server {
  listen 8080;
  server_name example.com www.example.com;
  ...
}

# resolve with IP on port 8080
server {
  listen 8080 default_server;
  server_name example.com www.example.com;
  ...
}
```

If you don't have a domain for your instance you can configure it to use your IP address instead. 
Edit the file with:
```bash
$ sudo nano /etc/nginx/sites-available/code-server.conf
```
This is the official code-server nginx config (https://github.com/coder/code-server/blob/main/docs/guide.md)
```nginx
server {
    listen 8080;
    listen [::]:8080;
    server_name doesnotmatter.com;
    location / {
      proxy_pass http://127.0.0.1:8080/;
      proxy_set_header Host $host;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection upgrade;
      proxy_set_header Accept-Encoding gzip;
    }
}
```

Save the file by pressing `CTRL + X` then `Y` and then `Enter` to save the file.
Now let’s enable the new config by creating a symlink (Make sure to use absolute paths and not relative ones!):
```bash
$ sudo ln -s /etc/nginx/sites-available/code-server.conf /etc/nginx/sites-enabled/code-server.conf
```
Reload NGINX:
```bash
sudo systemctl reload nginx
```

### Debugging
Sometimes stuff just goes wrong. First you should check the status of `nginx` and `code-server` services:
```bash
$ sudo systemctl status nginx
$ sudo systemctl status code-server
```
See if you can figure out what's wrong.
You can check the logs like this:
```bash
$ cat /var/log/nginx/access.log
$ cat /var/log/nginx/error.log
$ cat ~/.local/share/code-server/logs
```

If one of the two says anything about the address like `0.0.0.0` or `127.0.0.1` not being available or being in use already, you can find out what is listening to a certain port
```bash
$ sudo ss -lptn 'sport = :8080'
```
Or
```bash
$ sudo netstat -nlp | grep :8080
```




## Setup the Database Instance

### Details
Volume Size: 500gb (or whatever is available)
Datastore; MySQ:L 8.0-41
Flavour: There is not description for each flavour but we went for db3.large which ended up having 16GB of RAM
Locality: Affinity

### Networking
Maybe add the two zones where you server is located, for sure add:
* `qh2-uom`
* `qh2-uom-internal`

### Database Access
ALL CIDRs `0.0.0.0/0` are already allowed so you should not need to add that


### Initial Databases
Initial databases: Just a comma separated list of the new databases you want to create
Initial admin user: I mean just avoid stuff like admin or root for security reasons, anything else should be ok
Allowed Host: Leave empty so you can access it from any machine

### Advanced
You can restore from a backup if you have one


Connect to the instance  via terminal
Ubuntu:
```bash
$ sudo apt-get install mysql
```
Macos: 
```bash
$ brew install mysql
```
You’ll find the host on the Databases -> instances page
You can then connect with:
```bash
$ mysql -host xxxx.db.cloud.edu.au -u ubuntu -p <password>
```