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

## Installation and Configuration

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
Past this [script](https://github.com/MISP/MISP/blob/2.4/app/files/feed-metadata/defaults.json) here : 
<br>
<br>
![Alt text](Images/image5.png)
<br>
DON'T FORGET TO ACTIVATE AND COLLECT THE FEEDS
<br>
<br>
![Alt text](Images/image6.png)
<br>
#### Generate an API key
- Click on administration >> list auth keys >> generate a key

![Alt text](Images/image18.png)
- We generate an authentication key to allow the API to recognize and authorize the user. Fields such as user, comment, and authorized IPs must be configured as needed before submitting.

![Alt text](Images/image19.png)
- Please make sure to write down the authentication key

![Alt text](Images/image20.png)
#### Set up a Cronjob to update feeds daily
```
0 1 * * * /usr/bin/curl -XPOST --insecure --header "Authorization: **YOUR_API_KEY**" --header "Accept: application/json" - header "Content-Type: application/json" https://**YOUR_MISP_ADDRESS**/feeds/fetchFromAllFeeds
```
### Installing Wazuh
- Wazuh offers an installation method called `Quick Start`
- Download and run the Wazuh installation assistant :
```
curl -sO https://packages.wazuh.com/4.11/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
- Once the installation is complete, the wizard will give us a username and password to connect to the indexer

![Alt text](Images/image7.png)
### Initial Wazuh Configuration
- we identify ourselves using the credentials given previously

![Alt text](Images/image8.png)
- Home page :

![Alt text](Images/image9.png)

#### Adding agents
- Click on Deploy new agent
- Select your agent's system

![Alt text](Images/image10.png)
- Enter the server IP address. Then, name your agent, and add it to an existing group

![Alt text](Images/image11.png)
- Open PowerShell as an administrator and run the displayed installation command to download the agent. Then, start the agent using the `NET START WazuhSvc` command

![Alt text](Images/image12.png)
![Alt text](Images/image13.png)
- This will create a directory under `C:\Program Files (x86)\ossec-agent`, which we can use later to manage the events sent to the wazuh manager

![Alt text](Images/image14.png)
- and there you have it! the agent is deployed.

![Alt text](Images/image15.png)

#### Wazuh-Sysmon Integration
##### Step 1 : Installing and Configuring Sysmon
- Download Sysmon from the Microsoft Sysinternals [page](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon).
- Download the Sysmon configuration file from this [link](https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml).
- Extract the Sysmon zip file and place the downloaded configuration file in the extracted folder.
- Install Sysmon with the configuration file using PowerShell (as administrator):
```
.\sysmon64.exe -accepteula -i .\sysmonconfig-export.xml
```
##### Step 2 : Configure the Wazuh agent
- Edit the Wazuh agent's `ossec.conf` file:
`C:\Program Files (x86)\ossec-agent\ossec.conf`
- Add the following configuration to collect Sysmon logs:
```
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```
- Restart the Wazuh agent with the command:
```
Restart-Service -Name wazuh
```
##### Step 3 : Configure the Wazuh server
- Add the following rules to the file `/var/ossec/etc/rules/local_rules.xml` :
```
<group name="win-sysmon">
  <rule id="100502" level="2">
    <if_sid>921101</if_sid>
    <field name="win.system.eventID" type="pcre2">^3$</field>
    <field name="win.eventdata.image" type="pcre2">^C:\\\\Windows\\\\System32\\\\WindowsPowerShell\\\\v1.0\\\\powershell.exe$</field>
    <description>Network connection initiated by PowerShell</description>
    <mitre>
      <id>T1059.001</id>
    </mitre>
  </rule>

  <rule id="100503" level="13" frequency="5" timeframe="60">
    <if_matched_sid>100502</if_matched_sid>
    <description>Multiple network connections initiated by PowerShell to "$(win.eventdata.destinationIp)" on port "$(win.eventdata.destinationPort)"</description>
    <mitre>
      <id>T1059.001</id>
    </mitre>
  </rule>
</group>
```
- Restart the Wazuh manager:
```
systemctl restart wazuh-manager
```
## MISP-Wazuh Integration
### Step 1 : Add the Python script
- Place [this Python script](https://github.com/karelumair/MISP-Wazuh-Integration/blob/main/custom-misp.py) at `/var/ossec/integrations/custom-misp`

**Note :**
ensure that you didn't add extention `.py`
- Change the URL and API key in the script.

![Alt text](Images/image16.png)
- Make sure to set the permissions:
```
sudo chown root:wazuh custom-misp && sudo chmod 750 custom-misp
```
- Make sure wazuh is already alerting for the desired sysmon events. You will likely need to create a custom rule if it isn't already alerting.
- For example, in our test we will need DNS queries from sysmon event 22
- We will change the above rule level from `0` to `4` in the file `/var/ossec/ruleset/rules/0595-win-sysmon_rules.xml`
```
<rule id="61650" level="4">
   <if_sid>61600</if_sid>
   <field name="win.system.eventID">^22$</field>
   <description>Sysmon - Event 22: DNS Query event</description>
   <options>no_full_log</options>
   <group>sysmon_event_22,</group>
 </rule>
```
**Note :** We found the rule for event `22` in `0595-win-sysmon_rules.xml` because it falls between `05`-`95`. Follow the same approach to find the desired event.
<br>
**Note :** There are 16 levels of rules `0`-`15`. Check [this page](https://documentation.wazuh.com/current/user-manual/ruleset/rules/rules-classification.html)to recognize each one.
### Step 2 : Configure the integration in Wazuh
- Edit the Wazuh manager's `/var/ossec/etc/ossec.conf` file to add the integration block:
```
<integration>
  <name>custom-misp</name>
  <group>sysmon_event1,sysmon_event3,sysmon_event6,sysmon_event7,sysmon_event_15,sysmon_event_22,syscheck</group>
  <alert_format>json</alert_format>
</integration>
```
**Note :** the manager will only run the script when one of the Sysmon groups is triggered
- Restart the Wazuh manager.
```
systemctl restart wazuh-manager
```
### Step 3 : Also add the following rule to the wazuh manager
- Go to `Server Management` > `Rules` > `Add New Rule`. Name it `misp.xml`, add the following and save.
```
<group name="misp,">
  <rule id="100620" level="10">
    <field name="integration">misp</field>
    <match>misp</match>
    <description>MISP Events</description>
    <options>no_full_log</options>
  </rule>
  <rule id="100621" level="5">
    <if_sid>100620</if_sid>
    <field name="misp.error">\.+</field>
    <description>MISP - Error connecting to API</description>
    <options>no_full_log</options>
    <group>misp_error,</group>
  </rule>
  <rule id="100622" level="12">
    <field name="misp.category">\.+</field>
    <description>MISP - IoC found in Threat Intel - Category: $(misp.category), Attribute: $(misp.value)</description>
    <options>no_full_log</options>
    <group>misp_alert,</group>
  </rule>
</group>
```
![Alt text](Images/image17.png)
### Step 4 : Restart Wazuh services
```
systemctl restart wazuh-manager
systemctl restart wazuh-indexer
systemctl restart wazuh-dashboard
```
## Integration testing
In the integration test, you can use any attribute from the feed. However, we’ll create our own event and add a domain attribute to it, allowing us to test with that domain later.
### create our own event
- Access the MISP interface via its URL (e.g.: http://<MISP_IP_address>).
- Create a new event with a title, distribution, and threat level, then submit.
- Add a domain attribute with a fictitious name, like `lolo.koko.co`, and save it.
- Publish the event by clicking on `Publish Event`

![Alt text](Images/image21.png)

![Alt text](Images/image22.png)

- On a Windows machine with the Wazuh agent installed, use PowerShell to interact with the added domain:

![Alt text](Images/image23.png)

- Check if the malicious domain is detected and marked as a critical alert in the Sysmon logs transmitted to Wazuh.

![Alt text](Images/image24.png)

![Alt text](Images/image25.png)












