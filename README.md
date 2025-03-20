# MISP-Wazuh Integration

## Introduction
This project provides a detailed guide and necessary scripts to integrate MISP (Malware Information Sharing Platform) with Wazuh, a security monitoring solution. By combining these tools, security teams can automatically check Sysmon events against MISP's threat intelligence database, enabling real-time detection of known threats and indicators of compromise (IoCs).

## Prerequisites
Before starting the integration, ensure you have the following:
- A machine with Ubuntu Server installed (for MISP and Wazuh installation)
- VMware or another virtualization platform (if using a VM)
- Docker installed (if using the Docker installation method)
- Basic knowledge of Linux command line, Docker, and network configuration
- Python 3 and `pip3` installed (for the integration script)

## Installation

### Installing MISP
- MISP can be installed using three methods: automatic script, manual installation, or Docker. Choose the method that best suits your needs.
- In this guide, we will configure and run MISP using **Docker** For a faster and isolated deployment on an **Ubuntu Server** (virtual machine on VMware).
#### Installing Docker
##### First, uninstall any old versions of Docker
```
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
##### Add Docker's official GPG key
```
sudo apt-get update

sudo apt-get install ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
##### Add the Docker repository
```
echo \  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/
linux/ubuntu \  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \  sudo tee /etc/apt/sources.list.d/
docker.list > /dev/null

sudo apt-get update
```
##### Install the Docker packages
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
##### Finally, verify Docker was installed successfully by executing
```
sudo docker run hello-world
```
#### Installing the MISP Docker Image
##### Clone the MISP Docker repository
```
git clone https://github.com/MISP/misp-docker
```
##### Configure files
```
cd misp-docker
cp template.env .env
vim .env
```
Modify the `MISP_BASEURL` variable in `.env` to reflect the machine's IP address.
<br>
<br>
![Alt text](Images/image1.png)
<br>
##### Next, build the Docker containers
```
sudo docker compose build
```
#### Running MISP Using Docker
##### edit the docker-compose.yml file
This file holds the configuration settings for the Docker environment running MISP. In particular, you need to update the `MISP_BASEURL` variable to match the IP address of the machine hosting MISP.
<br>
<br>
![Alt text](Images/image2.png)
<br>
##### Launch MISP containers
```
sudo docker compose up -d
```
##### To stop the Docker environment
```
sudo docker compose down
```
### Initial MISP Configuration
#### Logging into MISP
You can access your MISP instance through ports 80 and 443 on the machine hosting MISP. Accept the security certificate, then log in as the default Administrator using the credentials `admin@admin.test`:`admin`.
<br>
<br>
![Alt text](Images/image3.png)
<br>
#### Adding feeds
A MISP feed is a structured data source that automatically provides up-to-date information on cyber threats
<br>
<br>
![Alt text](Images/image4.png)
<br>


























