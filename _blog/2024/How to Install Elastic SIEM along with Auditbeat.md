---
layout: blog
title: How to Install Elastic SIEM along with Auditbeat
tags: SIEM Blue-Team
comments: true
date: 2024-10-14
---
If you’re here you probably want to set up a SIEM to gather logs from a machine. There are a couple of open-source solutions, but the ELK Stack is one of the most robust ones. But first what exactly is the ELK Stack? 

# What is ELK Stack?

The ELK Stack combines three powerful open-source tools: Elasticsearch, Logstash, and Kibana. Elasticsearch is used for fast and scalable search, Logstash collects and processes logs or data, and Kibana helps visualize the data in dashboards. ELK is used for managing large volumes of log data, helping to search, analyse, and visualize it in real time. It’s popular in monitoring, troubleshooting, and security use cases, making it easier to track performance, identify issues, and gain insights from system logs or events.

We will be using Elasticsearch and Kibana since Logstash is not needed for our purposes. We will use Auditbeat to send logs from Linux Devices. 

# Installing ElasticSearch:

First, you need any Linux Distribution to install Elasticsearch. We are using Kali Linux in the below examples but you may use Ubuntu or any other distribution.
Please note that in this example we will do a bare-metal install of the ELK stack with security disabled. I will make another tutorial which has the security features enabled later on.

We will first run the following command, which will download the Elasticsearch GPG Key, which is used to verify the authenticity of the Elasticsearch Packages. 

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```  

We will now run the following command that will install a pre-requisite package that would enable the system package manager to handle package downloads over HTTPS: 

```bash
sudo apt-get install apt-transport-https
```

Now we will run the following command which basically adds the official Elasticsearch repository to our system’s APT Sources. 

```bash
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list 
``` 

Now we will run,

```bash
sudo apt-get update && sudo apt-get install elasticsearch
```

which will update the package list and further on also install Elasticsearch.   

{% if jekyll.environment == "production" %}
![Image1](https://raw.githubusercontent.com/MasterChief220/Blog/master/assets/images/blogs/2024/1_eA42er5ix7bS3wcbGwVSUg.png)
{% else %}
![Image1]({{ site.baseurl }}/assets/images/blogs/2024/1_eA42er5ix7bS3wcbGwVSUg.png)
{% endif %}

We need to now make some changes to the config file. Before you do that however we need to first ensure some things. If you’re running the Linux system on a virtual machine like Oracle VirtualBox or VMWare Workstation you need to ensure that your host system and the VM can communicate with each other i.e. their IPs are visible to each other. To do this you need to run `ip addr` command in the terminal.

![Image2]({{ site.baseurl }}/assets/images/blogs/2024/1_t5DMA8yllFHbGeVslMRnFw.png) 

Now as you can see the IP address of my machine is 192.168.240.128. I need to make sure my Windows machine can communicate with it. For that, we will open up the terminal on Windows and run the following command. ping 192.168.240.128. If the ping succeeds then we can go on. If it does not then you need to go into your virtual machine settings and change the network settings from NAT to Bridged Adapter. For Virtualbox the settings are here: 

![Image3]({{ site.baseurl }}/assets/images/blogs/2024/1_SkWg-WwK7Hdxn5uHWMWx0w.png)


Now we will open the config file and make some changes. To do that run the following command:

```bash
sudo vi /etc/elasticsearch/elasticsearch.yml
```
I am using Vim but you can use Nano or any other editor you feel comfortable with. Scroll down to the config file until you see the Network Section where you should change the network.host Ip address to your IP address. My ip right now is 192.168.240.128 but you should change it to your machine ip. Also, comment in the line. 

{% if jekyll.environment == "production" %}
![Image4](https://raw.githubusercontent.com/MasterChief220/Blog/master/assets/images/blogs/2024/1_9keDiriw9qbC_yqMUzPcsQ.png)
{% else %}
![Image4]({{ site.baseurl }}/assets/images/blogs/2024/1_9keDiriw9qbC_yqMUzPcsQ.png)
{% endif %}

It should look like the above.
Now scroll further down until you see the discovery section and remove hosts 1 and 2 and instead enter your MACHINE IP : 

![Image4]({{ site.baseurl }}/assets/images/blogs/2024/1_9keDiriw9qbC_yqMUzPcsQ.png)


We will now scroll down further until we come across Security Auto Configuration and set it all to false. 

![Image5]({{ site.baseurl }}/assets/images/blogs/2024/1_uv9QM-lp-RKKSXY3vgC5wA.png)


Now we will save the file and start elasticsearch. 

```bash
sudo systemctl daemon-reload # Tells system to reload the config files so it can organize new services 

sudo systemctl enable elasticsearch.service 

sudo systemctl start elasticsearch.service

``` 

Now by default, Elasticsearch runs on the loopback address and port 9200. Hence after Elasticsearch starts we can ensure whether it’s listening on the port by using the following command. 

```bash
curl 127.0.0.1:9200
```

![Image6]({{ site.baseurl }}/assets/images/blogs/2024/1_yO6fv1i5grkXnS8g-folwA.png)

Now once we get an output that looks like the above we can rest easy knowing that Elasticsearch is working. To test whether we can access it from our Windows machine run curl 192.168.240.128:9200 on the Windows terminal. We will see a similar output to the one above if it is installed properly. 

# Install Kibana: 

Kibana is used for the dashboard and to install it we will run:

```bash
sudo apt install -y kibana 
```

The -y parameter is used to automatically confirm yes to the confirmation.

We will now make some changes in the kibana.yml file using :

```bash
sudo vi /etc/kibana/kibana.yml
```

We will scroll down until we see the heading _System:Kibana server_. There change the server.host ip to the machine ip. Also, comment in the line.

Scroll further down until you see the heading _System:Elasticsearch_. There change the local host parameter in _elasticsearch.hosts:[“http://localhost:9200"]_ to your ip. Do not change anything else. It should look like this _elasticsearch.hosts:[“http://YOURMACHINEIP:9200"]_ Also comment in the line.

Once we have made those changes we will save the changes and start Kibana using:

```bash
sudo service kibana start 
sudo service kibana status # To ensure it started correctly
```

> Please keep in mind that Kibana might start but the dashboard takes some time to load fully. 

To ensure that the dashboard has loaded properly go to a web browser and enter **MACHINE_IP:5601** since 5601 is the default kibana port. If kibana loads successfully then everything is working properly. 


#Install Auditbeat: 

Now we will install Auditbeat on the Kali machine (or any Linux machine you want to). Keep in mind in our case this machine is the one which has both Elastic and Kibana running but this could be any Linux machine whose logs we want to send to our SIEM.

To install Auditbeat we will run:

```bash
sudo apt install -y auditbeat 
```
After it is installed we will modify the config file using

```bash
sudo vi /etc/auditbeat/auditbeat.yml
```

Scroll to the heading which says Kibana. There change the host parameter from _localhost:5601 to MACHINEIP:5601_. Do comment in the Line and then move below to the Elasticsearch Output heading. There change the hosts: from _localhost:9200 to MACHINEIP:9200_.

Now we will run the auditbeat setup. This validates whether Auditbeat can connect to Elasticsearch and Kibana. The command is:

```bash
sudo auditbeat -e setup
```

Most of the output is of not any use to us but 2 lines in particular need to be paid attention to.

![Image7]({{ site.baseurl }}/assets/images/blogs/2024/1_OEUdc-r90XcpjKuUpOQ3ew.png)


> If for some reason you do not see these highlighted lines then there was some issue and you need to fix it.

If it’s successful go ahead and start Auditbeat using: 

```bash
sudo service auditbeat start
```

To ensure that everything is working perfectly we will go to the dashboard which we can access using the browser on both the Windows Machine and the Linux one.

Entering the address in the browser yields the following:

![Image8]({{ site.baseurl }}/assets/images/blogs/2024/1_ZMSDNLP634Aovct_L7wBGQ.png)


**Since SSL and security options are not configured there will be no HTTPS address but a http address instead.**

However, to check if Auditbeat is sending logs we will open the left menu by clicking on the horizontal lines and then go to the discover section within Analytics.
If everything is working and sending logs we should see something like the following:

![Image9]({{ site.baseurl }}/assets/images/blogs/2024/1_Z3WVxRPiG7zpFPIblO5tuQ.png)



**The SIEM has been set up. Further tutorials will explore the Dashboard in more detail and also how to send logs from the Windows Machine to the SIEM Tool using Winlogbeat and Sysmon. Feel free to ask any questions.** 