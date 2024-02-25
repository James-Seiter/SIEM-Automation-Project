<h1>SOC Project</h1>

<h2>Description</h2>
The primary objectives for this project are to deploy a SIEM/XDR solution in the cloud, connect an agent to the SIEM, write rules to generate an alert based on indicators of compromise, and finally send those alerts to a SOAR.  The SOAR will automate case creation and email an analyst for incident response.  Wazuh is an open source security information and event management (SIEM) tool, and extended detection and response (XDR) solution.  The Wazuh manager will be deployed on an Ubuntu server running in the cloud.  TheHive is an open source incident response platform that will be used for case management.  This will also be deployed on an Ubuntu server in the cloud.  Shuffle is an open source security orchestration automation and response (SOAR) platform that utilizes API calls to automate a workflow.  Windows 10 Pro will be used for the on prem workstation running a Wazuh agent.  <br />
<br />
In addition to the many tools listed above, TheHive requires some prerequisite applications be installed.  Apache Cassandra is an open source NoSQL database that is well known for its ability to handle large volumes of data without sacrificing performance. ElasticSearch is a search engine that will be used to manage data indices.
<br />
<br />
This was an extremely long, difficult, and rewarding project.  Almost every step along the way required either extensive research and/or laborious troubleshooting, which was a phenomenal learning experience. This was a great test of several skills, including but not limited to: Linux/Windows administration, cloud/VM, networking, and implementation of cybersecurity concepts.

<h2>Environments and Tools Used </h2>

- <b>Digital Ocean</b>
- <b>Oracle VirtualBox</b>
- <b>Ubuntu Server 22.04 LTS - Gen2</b>
- <b>Windows 10 Pro</b>
- <b>Wazuh</b>
- <b>TheHive</b>
- <b>Shuffle</b>

<h2>Project walk-through:</h2>

<p align="center">
Logical Topology: <br/>
<img src="https://i.imgur.com/71oiW22.png" height="80%" width="80%"/>
<br />
<br />
Begin by provisioning the Wazuh and TheHive Ubuntu servers in the cloud. Once created, they are placed behind a firewall with a rule that only allows traffic from the on prem public IP address:  <br/>
<img src="https://i.imgur.com/i9fTG5S.png" height="80%" width="80%"/>
<br />  
<br />
SSH into the server and run 'apt-get update && apt-get upgrade' to update the package lists and then update them. Do this for TheHive server as well: <br/>
<img src="https://i.imgur.com/MVhQe1H.png" height="80%" width="80%" />
<br />
<br />
Use the Curl command found here: https://documentation.wazuh.com/current/installation-guide/wazuh-server/installation-assistant.html, to install the Wazuh manager:  <br/>
<img src="https://i.imgur.com/ZBIKp8X.png" height="80%" width="80%" />
<br />
<br />
After a few minutes the installation will be complete and login credentials will be generated.  Be sure to copy these:  <br/>
<img src="https://i.imgur.com/496lM9u.png" height="80%" width="80%" />
<br />
<br />
Open a browser and navigate to the public IP address of the Wazuh server.  In this case it would be https://68.183.147.154. Enter the credentials from the previous step to log in:  <br/>
<img src="https://i.imgur.com/BkWWGmt.png" height="80%" width="80%" />
<br />
<br />
Now that the Wazuh manager is successfully installed and deployed in the cloud, it's time to do the same for TheHive. To install TheHive, some dependencies and prerequisites must be installed first.  For the sake of brevity, these installations are not shown.  The prerequisites include Java, Cassandra, and ElasticSearch.  Finally TheHive can be successfully installed:  <br/>
<img src="https://i.imgur.com/mLyQ3A7.png" height="80%" width="80%" />
<br />
<br />
Now that Wazuh and TheHive are installed, it's time to edit some configuration files to make sure everything works properly, starting with Cassandra:  <br/>
<img src="https://i.imgur.com/wYq2Z1v.png" height="80%" width="80%" />
<br />
<br />
In the /etc/cassandra/cassandra.yaml file, configure the cluster_name(optional), listen_address, rpc_address, and seed values.  The latter three should be configured with the public IP address of TheHive server:  <br />
<img src="https://i.imgur.com/sncnpBF.png" height="80%" width="80%" />
<img src="https://i.imgur.com/pbujnjL.png" height="80%" width="80%" />
<img src="https://i.imgur.com/Qso5og9.png" height="80%" width="80%" />
<img src="https://i.imgur.com/AfZujRB.png" height="80%" width="80%" />
<br />
<br />
Once the above configuration changes are complete, save the file. Next stop the cassandra service, remove old files, restart the service, and check the status to make sure it is 'active (running)':  <br />
<img src="https://i.imgur.com/ZjiJcjL.png" height="80%" width="80%" />
<br />
<br />
ElasticSearch needs to be configured next:  <br />
<img src="https://i.imgur.com/yTtwnB6.png" height="80%" width="80%" />
<br />
<br />
In the /etc/elasticsearch/elasticsearch.yml file, uncomment/configure cluster.name, node.name, network.host, http.port, and cluster.initial_master_nodes. network.host should be the public IP address of TheHive server. http.port can be left with its default setting of 9200:  <br/>
<img src="https://i.imgur.com/zm7Tafn.png" height="80%" width="80%" />
<img src="https://i.imgur.com/ZiRXHIY.png" height="80%" width="80%" />
<br />
<br />
Once the above configurations are complete, save the file.  Now the elasticsearch.service can be started/enabled. Check the status of the service to confirm it is 'active (running)':
<img src="https://i.imgur.com/PLJ3Zjz.png" height="80%" width="80%" />
<br />
<br />
With the configuration of Cassandra and ElasticSearch complete, it's time to configure TheHive. To begin, TheHive user/group will need access to the /opt/thp filepath:  <br/>
<img src="https://i.imgur.com/2UanbKd.png" height="80%" width="80%"/>
<br />  
<br />
After that bit of maintenance, edit TheHive configuration file: <br/>
<img src="https://i.imgur.com/7G7cAgt.png" height="80%" width="80%" />
<br />
<br />
In the /etc/thehive/application.conf file, under Database and index configuration, edit both hostnames to be the public IP address of TheHive server. The cluster-name should be the same as the one used in /etc/cassandra/cassandra.yaml. Note: (Not pictured here) further down in the file, application.baseUrl should also be changed to reflect TheHive server public IP:  <br/>
<img src="https://i.imgur.com/casBj5B.png" height="80%" width="80%" />
<br />
<br />
Save the file and start/enable thehive.service. As always, check the status to make sure it is 'active (running)':  <br/>
<img src="https://i.imgur.com/DjeFhCN.png" height="80%" width="80%" />
<br />
<br />
Open a browser and navigate to the public IP address of TheHive server over port 9000.  In this case it would be http://174.138.74.192:9000. Log in using the default credentials: admin@thehive.local with password secret:  <br/>
<img src="https://i.imgur.com/IBWkpp8.png" height="80%" width="80%" />
<br />
<br />
With Wazuh and TheHive up an running in the cloud, it's time to move on to the second phase of the project.  Over on the Wazuh dashboard there are currently 0 agents.  That needs to change, so click the link to add a new agent. Select the radio button under the Windows header, enter the public IP address of the Wazuh manager, and optionally name the agent to generate the Powershell commands:  <br/>
<img src="https://i.imgur.com/MjVixAs.png" height="80%" width="80%"/>
<img src="https://i.imgur.com/P00Ghlc.png" height="80%" width="80%"/>
<br />  
<br />
Copy the aforementioned commands. In the Windows 10 Pro VM running in VirtualBox, open an admin session of Powershell.  Paste and run the commands: <br/>
<img src="https://i.imgur.com/9dzbZ7I.png" height="80%" width="80%" />
<br />
<br />
Once complete, start the Wazuh service. Check the status to confirm that it is running:  <br/>
<img src="https://i.imgur.com/83Wdnjw.png" height="80%" width="80%" />
<br />
<br />
Back on the Wazuh dashboard, the newly joined agent should be visible and showing as active:  <br/>
<img src="https://i.imgur.com/huYfObQ.png" height="80%" width="80%" />
<br />
<br />
Now that the Wazuh agent is installed, it requires some configuration. Open the ossec.conf file in an admin session of notepad:  <br/>
<img src="https://i.imgur.com/Ul7xEG9.png" height="80%" width="80%" />
<br />  
<br />
Sysmon is an incredibly useful service that monitors and logs activities such as process creation and network connections to the Windows event log. It was previously installed on this Windows 10 Pro VM. In order to ingest these logs on the Wazuh agent, edit the ossec.conf file under the Log analysis comment to include the location of these logs: <br/>
<img src="https://i.imgur.com/1TegU4C.png" height="80%" width="80%" />
<br />
<br />
Save the file and restart the Wazuh service:  <br/>
<img src="https://i.imgur.com/EWy3OL0.png" height="80%" width="80%" />
<br />
<br />
Now that the appropriate logs are being ingested on the agent, it's time to generate some telemetry. The goal here will be to download the open source malware program, mimikatz, and have Wazuh alert on its process creation. Mimikatz is a tool often used by threat actors to extract credentials from Windows machines. This is obviously a major threat, and one that a security professional would need to be alerted to as quickly as possible. <br />
<br />
By default, Wazuh will only display logs that are triggered by a rule or alert. In order to make Wazuh log everything and index those logs, some configuration will need to be done on the Wazuh server. This will make it possible to search for certain events. Start by editing the /var/ossec/etc/ossec.conf file on the Wazuh server and change logall to yes:  <br/>
<img src="https://i.imgur.com/aT85KYI.png" height="80%" width="80%" />
<img src="https://i.imgur.com/QSz2FxI.png" height="80%" width="80%" />
<br />
<br />
Save the file and restart the wazuh-manager.service. Now Wazuh will log everything and save it to the /var/ossec/logs/archives directory:  <br/>
<img src="https://i.imgur.com/49ktYUa.png" height="80%" width="80%" />
<br />
<br />
To make Wazuh ingest these logs, edit the /etc/filebeat/filebeat.yml file and change archives: enabled: from false to true. Don't forget to restart the filebeat service:  <br/>
<img src="https://i.imgur.com/Vcsw0nx.png" height="80%" width="80%" />
<br />
<br />
With all of that complete, create a new Index Pattern in the Wazuh manager dashboard. This will make it possible to search all of the logs, even if they do not trigger an alert:  <br/>
<img src="https://i.imgur.com/HFn8sSA.png" height="80%" width="80%" />
<br />
<br />
Now that all of the logs are searchable, simulate an indicator of compromise, or IOC, by executing mimikatz:  <br/>
<img src="https://i.imgur.com/iokyxuR.png" height="80%" width="80%" />
<br />  
<br />
This will generate a Sysmon log with event ID 1 which indicates process creation. Because the Windows 10 Pro VM ossec.conf file was previously edited to send Sysmon logs over to Wazuh, the mimikatz process creation should be a searchable event in the newly created index: <br/>
<img src="https://i.imgur.com/Z46uolP.png" height="80%" width="80%" />
<br />
<br />
By clicking the arrow to expand the log with event ID 1, the data.win.eventdata.originalFileName field can be located. This field can be used to create a custom rule in Wazuh that will generate an alert when a certain criteria is met:  <br/>
<img src="https://i.imgur.com/HzCZaBl.png" height="80%" width="80%" />
<br />
<br />
From the Wazuh dashboard home screen, navigate to Management > Rules:  <br/>
<img src="https://i.imgur.com/4MtdJGH.png" height="80%" width="80%" />
<br />
<br />
From there, create the following custom rule. This rule will generate an alert that reads, 'Mimikatz Usage Detected' when Wazuh receives a log that contains mimikatz.exe in the original file name field. As a side note, this alert will have a MITRE ID of T1003 which can be researched to reveal this IOC as a common technique used by adversaries for OS credential dumping:  <br/>
<img src="https://i.imgur.com/xtzwWmm.png" height="80%" width="80%" />
<br />
<br />
Save the new rule and follow the prompt to restart. Because the originalFileName field was used and not the file ID, Wazuh should alert on this process creation even if the application were to be renamed.  To demonstrate this, exit mimikatz, rename the .exe file to something else and rerun the program:  <br/>
<img src="https://i.imgur.com/njVe7OM.png" height="80%" width="80%" />
<img src="https://i.imgur.com/Iv6Uz1q.png" height="80%" width="80%" />
<img src="https://i.imgur.com/Eae3RAw.png" height="80%" width="80%" />
<br />
<br />
If everything is configured correctly, Wazuh should generate an alert. Expand the alert and take note that even though the executable was renamed to peekaboo.exe, the rule picked up on the original file name of mimikatz.exe:  <br/>
<img src="https://i.imgur.com/lrG6o1c.png" height="80%" width="80%" />
<img src="https://i.imgur.com/uI1qMpC.png" height="80%" width="80%" />
<br />  
<br />
The next major step in this project is to have these newly generated alerts be automatically forwarded to Shuffle. First, create a new workflow over on shuffler.io and copy the webhook URI: <br/>
<img src="https://i.imgur.com/u7Ws6ad.png" height="80%" width="80%" />
<br />
<br />
Next, on the Wazuh server, edit the /var/ossec/etc/ossec.conf file and add the integration tag with copied URI from Shuffle. Also add the custom rule ID that was created earlier in Wazuh. Save the file and restart the wazuh-manager.service:  <br/>
<img src="https://i.imgur.com/TtNbaUZ.png" height="80%" width="80%" />
<br />
<br />
With that configuration complete, run the workflow on shuffle and expand the execution arguments to confirm that Shuffle received the alert:  <br/>
<img src="https://i.imgur.com/Ae2gtYy.png" height="80%" width="80%" />
<br />
<br />
The next step will be to extract the SHA256 hash value from the execution argument JSON data using regular expression. Start by clicking the Change Me icon. In the Find Actions drop-down menu, select 'Regex capture group'. Set the Input data as the JSON path for the file hashes and input the Regex that is shown:  <br/>
<img src="https://i.imgur.com/eZ3LqbG.png" height="20%" width="20%" />
<br />
<br />
This can be tested by rerunning the workflow and expanding the results for Change Me:  <br/>
<img src="https://i.imgur.com/tz1TV6F.png" height="80%" width="80%" />
<br />
<br />
The third step in this workflow will be to send this SHA256 hash value over to Virustotal to generate a hash report. First, add the Virustotal app and connect it to the workflow in Shuffle:  <br/>
<img src="https://i.imgur.com/ZfnolDd.png" height="80%" width="80%" />
<br />  
<br />
To make this transfer of data possible, create a Virustotal account, generate an APIkey, and copy it: <br/>
<img src="https://i.imgur.com/6UGsAtu.png" height="30%" width="30%" />
<br />
<br />
Back in Shuffle, click on the Virustotal app. In the Find Actions drop-down menu select 'Get a hash report'. Paste in the APIkey and set the Hash value to the Regex output JSON path generated by the previous step in the workflow:  <br/>
<img src="https://i.imgur.com/jTvWfw3.png" height="20%" width="20%" />
<br />
<br />
Rerun the workflow and expand the results for Virustotal and check out all that useful JSON data. To better understand what is being displayed, paste the SHA256 file hash into the Virustotal website and see the results:  <br/>
<img src="https://i.imgur.com/Wva4Vnh.png" height="80%" width="80%" />
<img src="https://i.imgur.com/kAmXYGZ.png" height="80%" width="80%" />
<img src="https://i.imgur.com/8uO6s0J.png" height="80%" width="80%" />
<br />
<br />
The fourth step in this workflow will be to send all this data to TheHive to generate an alert for case management and incident response. Over on the browser from earlier with TheHive, log back in and create two users. One user will be an analyst account while the other user will be a service account. Once created, select preview on the analyst account and set a password. Click preview on the service account and generate an APIkey. With the APIkey in hand, move back over to Shuffle and add TheHive application to the workflow. Click the newly added application and click the plus button next to authentication.  From here, paste in the APIkey and change the url field to the public IP address of TheHive server with port number 9000. Note that this port will need to be opened in the firewall running in the cloud for this to work:  <br/>
<img src="https://i.imgur.com/ZU3a29v.png" height="80%" width="80%" />
<br />  
<br />
After configuring the desired fields in TheHive Shuffle application, rerun the workflow to send the data to TheHive server and generate a new case alert: <br/>
<img src="https://i.imgur.com/rghpPLB.png" height="80%" width="80%" />
<img src="https://i.imgur.com/OkK00ch.png" height="80%" width="80%" />
<img src="https://i.imgur.com/lMWEwQB.png" height="80%" width="80%" />
<br />
<br />
The final step in this workflow will be to automatically generate an email and send it to the analyst to begin the incident response process. Simply add the email application to the workflow in Shuffle and input the desired email address:  <br/>
<img src="https://i.imgur.com/tvPIAlH.png" height="80%" width="80%" />
<br />
<br />
Just to demonstrate that Wazuh works on multiple platforms, a linux agent was added to this deployment and joined to the server:  <br/>
<img src="https://i.imgur.com/iPIS8De.png" height="80%" width="80%" />
<br />
<br />
Fin.  

























</p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
