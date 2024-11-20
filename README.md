# Attack-Technique-Emulation-Wazuh
A project showcasing attack technique emulation using MITRE ATT&amp;CK and detection with Wazuh, Sysmon, and Atomic Red Team.
**Introduction**:

This repository showcases the emulation of adversary tactics from the MITRE ATT&CK framework using Atomic Red Team and demonstrates how Wazuh can be configured to detect these threats effectively, with detailed monitoring provided by Sysmon.

**Emulating ATT&CK Techniques**:

Using Red Canary’s [Atomic Red Team](https://github.com/redcanaryco/invoke-atomicredteam), we emulate **T1053.005 – Scheduled Task/Job**, a common adversarial technique for automating malicious activities. This simulation demonstrates how Wazuh can monitor, detect, and alert on the creation and execution of potentially harmful scheduled tasks, providing a realistic assessment of our detection capabilities.

**Setup and Installation Instructions**: 

We are using wazuh docker deployment, so we will need to install Docker and Docker-Composer

Install Docker:

```jsx
sudo apt install [docker.io](http://docker.io/)
sudo apt install docker-compose

//use this command to check the version
sudo docker --version 

//use this command to check if your docker is running
sudo systemctl status docker
```

**Changing tha vm.max_map_count value:**

The default value of vm.max_map_count on many systems is 65536. Recommended setting it to at least 262144 to prevent out-of-memory exceptions.

```jsx
//To check the current value of vm.max_map_count
sysctl vm.max_map_count

//To open the file in text editor use this command
sudo nano /etc/sysctl.conf

//now put this command in that file and save the file
vm.max_map_count=262144
```

# **Sysmon Configuration**

Sysmon, a system monitoring tool from Microsoft Sysinternals, can be downloaded from the [official Sysinternals page](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon). It is installed using a configuration file, **sysmonconfig.xml**, which maps Sysmon event monitoring to MITRE ATT&CK techniques.

To install Sysmon with the configuration file via PowerShell, use the following command:

```powershell
sysmon.exe -accepteula -i sysmonconfig.xml
```

This command installs Sysmon and loads the specified configuration file to start monitoring system activities. Be sure to install Sysmon on the endpoint you wish to monitor for detailed event logging and analysis.

# **Wazuh Docker Deployment**

In this section, we will guide you through setting up Wazuh using Docker. For detailed instructions and configuration files, refer to my GitHub repository:

🔗 [Wazuh Docker Deployment Repository](https://github.com/Ghost-7A/wazuh-docker-deployment.git)

This repository contains all the necessary steps to deploy Wazuh in a Docker environment efficiently.

# **Emulating ATT&CK Techniques**

We leverage Red Canary’s [Atomic Red Team](https://github.com/redcanaryco/invoke-atomicredteam) to emulate **T1053.005 – Scheduled Task/Job**, a technique frequently used by adversaries to automate malicious operations. This simulation provides an opportunity to test Wazuh’s ability to monitor, detect, and alert on the creation and execution of suspicious scheduled tasks. By doing so, we gain a realistic assessment of our detection capabilities and identify areas for potential improvement.

We will implement this simulation on our designated victim endpoint.

### Basic Commands:

Get details of a particular technique

- The command below is used to show details of technique **T1053.005**:

```jsx
Invoke-AtomicTest **T1053.005** -ShowDetailsBrief
```

- Check/Get prerequisites of a technique

To check the prerequisites needed to test  **T1053.005**, the command below is used:

```jsx
Invoke-AtomicTest **T1053.005** -CheckPrereqs
```

- There may be some prerequisites that are not met. We will satisfy them by running the following command:

```jsx
Invoke-AtomicTest **T1053.005** -GetPrereqs
```

- Run the test for a particular technique

To run the test that emulates the **T1053.005** technique, the following command is used:

```jsx
Invoke-AtomicTest **T1053.005**
```

- Clean-up on completion of the test

After a test has been carried out, the changes made can be reverted with the following command. This command will clean-up test for **T1053.005**:

```jsx
Invoke-AtomicTest **T1053.005** -Cleanup
```

# **Monitoring and Detection**:

## **Configuring Wazuh agent**

Installation and enrollment of the Wazuh agent are done on the Windows sandbox. The agent is configured to capture Sysmon events by adding the following settings to the agent configuration file in  C:\Program Files (x86)\ossec-agent\ossec.conf

```jsx
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

To apply changes, we restart the agent by running the following PowerShell command as an administrator:

```jsx
Restart-Service -Name wazuh
```

# **Creating detection rules on Wazuh manager:**

To generate alerts for the previously selected MITRE ATT&CK techniques, the following rules are added to the local_rules.xml file in the rules section on the Wazuh manager.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/04cace58-755d-4c8b-9aa4-7044a0c11346/86e41797-7841-4b24-a9fb-ac2d52aa4e98/image.png)

```jsx
<group name="windows,sysmon,">

<rule id="100001" level="10">
  <if_group>windows</if_group>
  <field name="win.eventdata.ruleName" type="pcre2" >technique_id=T1053,technique_name=Scheduled Task</field>
  <description>A Newly Scheduled Task has been Detected on $(win.system.computer)</description>
  <mitre>
    <id>T1053</id>
  </mitre>
</rule>

</group>
```

After we save the rules file, We restart the Wazuh manager so it starts using the new rules.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/04cace58-755d-4c8b-9aa4-7044a0c11346/c2a22bfb-747d-42b6-8846-1755cb4d11e5/image.png)

# **Monitoring with Wazuh Dashboards:**

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/04cace58-755d-4c8b-9aa4-7044a0c11346/6865004a-d4ee-49c5-bf33-05bef51bd1d0/image.png)

The above image showcases our Wazuh dashboard for the specific endpoint. Since this is a fresh installation, it currently does not display extensive data.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/04cace58-755d-4c8b-9aa4-7044a0c11346/8e4ed84d-6e14-4229-845f-3daff2e81ff4/image.png)

Now we will go to Discover section and look for the alerts.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/04cace58-755d-4c8b-9aa4-7044a0c11346/b1e84eb0-8aaa-4fda-8015-8b197b2df0ca/image.png)

The alert have been generated 

# **Conclusion:**

In this project, we explored the emulation of MITRE ATT&CK techniques and the detection of these techniques using Wazuh, Sysmon, and Atomic Red Team. This journey provided valuable insights into the capabilities and challenges of modern threat detection mechanisms.
