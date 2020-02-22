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

`nano etc/apt/sources.list` 

6. Modify the docker source to look like the following:

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

# Install Portainer
1. Pull the Docker image

`docker pull portainer/portainer`

2. Run container from image

`docker run -d -p 127.0.0.1:9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer`

# Install & Configure Apache (Reverse Proxy Portainer)

1. Install through apt

`apt-get install apache2`

2. Enable extensions for reverse proxy

`sudo a2enmod proxy`

`sudo a2enmod proxy_http`

`sudo a2enmod proxy_balancer`

3. Restart Apache
`systemctl restart apache2`

4. Modify the default config file

`nano /etc/apache2/sites-enabled/{default}.conf`

```
<VirtualHost *:80>

    ServerName yourip

    ProxyPreserveHost On
    RewriteEngine On
    ProxyRequests Off

    # Proxy for Portainer
    ProxyPass /portainer/ http://localhost:9000/
    ProxyPassReverse /portainer/ http://localhost:9000/
    
</VirtualHost>

```

# Install & Configure MySql Docker Container


1. Pull the image from [DockerHub](https://hub.docker.com/r/mysql/mysql-server).

`docker pull mysql/mysql-server`

2. Run container from image

`docker run -d -p 3308:3306 --name=mysql_container -e MYSQL_ROOT_PASSWORD=rootpw mysql/mysql-server`

3. Access container and grant root permissions to allow all hosts | single host

`docker exec -it mysql_container mysql -uroot -prootpw`

`GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;`

`GRANT ALL PRIVILEGES ON *.* TO 'root'@'youripaddres' WITH GRANT OPTION;`


### Utils
Get system thermal (can be set as an alias = temp)

`cat /sys/class/thermal/thermal_zone0/temp`
