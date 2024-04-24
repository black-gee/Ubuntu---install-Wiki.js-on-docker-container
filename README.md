# Ubuntu---install-Wiki.js-on-docker-container
This guide is a fully detailed guide to install everything necessary to run Wiki.js on a brand new Ubuntu >= 18.04 LTS machine.

#### How to install Wiki.js on Ubuntu 23.10 (Server-Minimum)
#### 
#### At the end of the guide, you'll have a fully working Wiki.js instance with the following components:
#### 
####     Docker
####     PostgreSQL 11 (dockerized)
####     Wiki.js 2.x (dockerized, accessible via port 80)
####     Wiki.js Update Companion (dockerized)
####     OpenSSH with UFW Firewall preconfigured for SSH, HTTP and HTTPS

#### Update & Install Prerequisite 

sudo apt -qqy update
sudo apt install nano && sudo apt install  ufw  
sudo DEBIAN_FRONTEND=noninteractive apt-get -qqy -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold' dist-upgrade

#### Install Docker

sudo apt -qqy -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold' install ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt -qqy update && sudo apt -qqy -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold' install docker-ce docker-ce-cli containerd.io docker-compose-plugin

#### Setup Containers
#### Create internal docker network
docker network create wikinet

#### Create data volume for PostgreSQL
docker volume create pgdata
sudo mkdir -p /etc/wiki
sudo openssl rand -base64 32 > /etc/wiki/.db-secret

#### Note if you get a permissions error on the above break it into two commands copy the output from the first and paste it into the file like:

sudo openssl rand -base64 32
sudo vi /etc/wiki/.db-secret
sudo docker network create wikinet
sudo docker volume create pgdata

#### Create the Containers

sudo docker create --name=db -e POSTGRES_DB=wiki -e POSTGRES_USER=wiki -e POSTGRES_PASSWORD_FILE=/etc/wiki/.db-secret -v /etc/wiki/.db-secret:/etc/wiki/.db-secret:ro -v pgdata:/var/lib/postgresql/data --restart=unless-stopped -h db --network=wikinet postgres:11

sudo docker create --name=wiki -e DB_TYPE=postgres -e DB_HOST=db -e DB_PORT=5432 -e DB_PASS_FILE=/etc/wiki/.db-secret -v /etc/wiki/.db-secret:/etc/wiki/.db-secret:ro -e DB_USER=wiki -e DB_NAME=wiki -e UPGRADE_COMPANION=1 --restart=unless-stopped -h wiki --network=wikinet -p 80:3000 -p 443:3443 ghcr.io/requarks/wiki:2

sudo docker create --name=wiki-update-companion -v /var/run/docker.sock:/var/run/docker.sock:ro --restart=unless-stopped -h wiki-update-companion --network=wikinet ghcr.io/requarks/wiki-update-companion:latest

#### Setup Firewall

sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo systemctl start ufw
sudo ufw --force enable

#### Start the containers

sudo docker start db
sudo docker start wiki
sudo docker start wiki-update-companion
![image](https://github.com/black-gee/Ubuntu---install-Wiki.js-on-docker-container/assets/147333383/6b349fdf-d820-4f2c-9a0e-e5f59fd45c08)

