# Raspberry PI 4 - Docker Infrastructure

== OS: Ubuntu Server 18.04 ==

# Install Ubuntu Server OS

Install Ubuntu Server 18.04 build for Raspberry PI 4 from [official website download page](https://ubuntu.com/download/raspberry-pi/thank-you?version=18.04.4&architecture=arm64+raspi3).

(see documentation about it on their website)

# Installing Docker

## Set up the Docker repository

1. Update the apt packages:

`sudo apt-get update`

2. Install packages to allow apt to use a repository over HTTPS:

`sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common`

3. Add Docker’s official GPG key:

`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`

4. Set up a stable repository:

`sudo add-apt-repository \
   "deb [arch=amd4] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"`

5. Change the source:

`sudo nano etc/apt/sources.list`

6. Comment the docker source and add the following source:

(at the bottom of the file)

`deb [arch=arm64] https://download.docker.com/linux/ubuntu xenial stable`

## Install Docker Engine-Community

1. Update the apt package index.

`sudo apt-get update`

2. Install the latest version of Docker Engine

`sudo apt-get install docker-ce docker-ce-cli containerd.io`

3. Verify Docker Engine

`sudo docker run hello-world`

# Install Docker-Compose

1. Install dependencies

`sudo apt-get install libffi-dev libssl-dev`

`sudo apt-get install -y python python-pip`

`sudo apt-get remove python-configparser`

2. Install Docker-Compose

`sudo pip install docker-compose`

3. Test installation

`docker-compose`

# Install & Configure Apache (Reverse Proxy Portainer)

1. Install through apt

`sudo apt-get install apache2`

2. Enable extensions for reverse proxy

`sudo a2enmod proxy`

`sudo a2enmod proxy_http`

`sudo a2enmod proxy_balancer`

`sudo a2enmod rewrite`

3. Restart Apache
`sudo systemctl restart apache2`

4. Modify the default config file

`sudo nano /etc/apache2/sites-enabled/{default}.conf`

```
<VirtualHost *:80>

	# ServerName is optional
    # {ip} represents the machine IP address
    ServerName {ip}

    ProxyPreserveHost On
    RewriteEngine On
    ProxyRequests Off

    # Proxy for web application
    # {path} - URL segment that will be used to identify the web application
    # {port} - local port on wich the web application will run
    ProxyPass /{path}/ http://localhost:{port}/
    ProxyPassReverse /{path}/ http://localhost:{port}/

</VirtualHost>

```

## Docker Containers

### Portainer
1. Pull the Docker image

`docker pull portainer/portainer`

2. Run container from image

`docker run -d -p 127.0.0.1:9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer`

3. Add reverse proxy configuration to sites-enabled in Apache

```
ProxyPass /portainer/ http://localhost:9000/
ProxyPassReverse /portainer/ http://localhost:9000/
```

### MySql


1. Pull the image from [DockerHub](https://hub.docker.com/r/mysql/mysql-server).

`docker pull mysql/mysql-server`

2. Run container from image

`docker run -d -p 3308:3306 --name=mysql_container -e MYSQL_ROOT_PASSWORD=rootpw mysql/mysql-server`

3. Access container and grant root permissions to allow all hosts | single host

`docker exec -it mysql_container mysql -uroot -prootpw`

`CREATE USER 'root'@'%' IDENTIFIED BY 'rootpw';`

`GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;`

4. Test connection from remote computer


# Create multiple virtual IP addresses

1. Edit current netplan

`sudo nano /etc/netplan/50-cloud-init.yaml`

Add addresses section under the current network interface

```
network:
    ethernets:
        eth0:
        	# {IP1} - {IPN} are virtual IPs that will be bounded
            addresses: [{IP1}/24,{IPN}/24]
            dhcp4: true
            optional: true
    version: 2
```

2. Apply changes to the netplan

`sudo netplan apply`

### Utils

####Get system thermal (can be set as an alias = temp)

`cat /sys/class/thermal/thermal_zone0/temp`

#### Get RAM usage

`free -h` (h for human-readable output)

#### Get CPU info

`cat /proc/cpuinfo`

#### Check read / write speeds for MicroSD card

In order to check for read / write speeds you must be logged in as root.

Read speed

`sync; echo 3 | tee /proc/sys/vm/drop_caches` (clear cache)
`dd if=~/test.tmp of=/dev/null bs=500K count=1024`

Write speed

`sync; echo 3 | tee /proc/sys/vm/drop_caches` (clear cache)
`dd if=/dev/zero of=~/test.tmp bs=500K count=1024`

#### Disk Analyzer
A disk analyzer helps identify issues with phisical memory.

##### NCurses Disk Usage - ncdu
1. Install ncdu

`sudo apt-get install ncdu`

2. Scan system

`sudo ncdu -x -q`

Flag explanation:
```
-q Quiet mode, doesn't update the screen 10 times a second
   while scanning, reduces network bandwidth used

-x Don't cross filesystem borders (don't descend into a
   directory which is a mounted disk)
```

#### Thermals
Get system thermal (can be set as an alias = temp)

`cat /sys/class/thermal/thermal_zone0/temp`
