# Filebeat

Filebeat configuration to ship the logs to logstash. I have commited a whole respository to setup the ELK. If you want to go for whole ELK Setup. where We are setting up Elasticsearch, kibana and logstash for log storage, visulazation and aggregation respectively. Here we are fetching the logs through filebeat and sending that logs to logstash. 
You can also go through the respository where you can see, how to setup the spaces, role and user at kibana and create the specific live log dashboard for user. 
you can also check how we can raise the alert using elastalert and postfix. apart from that curator also to delete the older logs at elasticsearch to save overburden of logs. 


So Here we are going for basic Filebeat configuration that will work fine for you. you can edit based on your requirements. 

###### you can install filebeat on master or client server based on from where you want to fetch your logs. Here we have configured it for the client machine server fetching the logs and shipping to another master machine where my ELK is configured.  

**Note: setup your filebeat before the logstash, so that logstash grok patterns can be configured based on your logs need.** 

# Install Filebeat

Run the following command 
```
sudo apt-get install filebeat
```

# Configure Filebeat 

to configure the filebeat, we will run following command. 
```
sudo nano /etc/filebeat/filebeat.yml
```

**Important Note: In the below file configuration there are log_type, that is basically the name that should be same in logstash and filebeat file. and with the same name you will find the index at kibana. Define your path based on from where you are going to fetch the logs. Apart from these given configuration, i have everything else as comment. you can uncomment and edit based on your need. but this configuration worked well for me. 

Copy and paste the given configuration. change paths, log_type and public ip of master server based on your setup. 

```
filebeat.inputs: 

- type: log

# Change to true to enable this input configuration. 
enabled: true 
fields:
  log_type: access

# Paths that should be crawled and fetched. Glob based paths. 
paths:
    - /var/log/nginx/*.access.log
  

- type: log
enabled: true 
fields:
  log_type: errors 
 paths:
    - /var/log/nginx/live-1.error.log
    
    

filebeat.config.modules:
# Glob pattern for configuration loading 
  path: ${path.config}/modules.d/*.yml

# Set to true to enable config reloading 
  reload.enabled: false

setup.kibana: 
#
#
#

output.logstash:
# The Logstash hosts 
  hosts: [“public-ip:5044”] 

processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  #- add_cloud_metadata: ~
  #-add_docker_metadata: ~
  #- add_kubernetes_metadata: ~

```

*Here replace public-ip with your master public ip address. In the updated version of filebeat above 7. you won't be able to use -type : log, there you will have to use -type: filestream. you can check out more input type based on your file type.**

To start your filebeat and check the status, you can run the following commands. 
```
sudo service filebeat start
```
```
sudo service filebeat status
```
```
sudo service filebeat stop
```

**if your filebeat is running in status or failing again and again, then you can run the following command and check where exactly issue is with the filebeat. yml files are too identation sensitive. so there can be issue with that in filebeat.yml file. so check your issue properly by running following command.**
```
sudo filebeat -e
```

Now we can start configuring our logstash. you will have to prepare the grok patterns based on your paths and log file patterns. keep in mind the index name that are gonna be same in both file for logstash rules.conf file and filebeat.yml file. 

