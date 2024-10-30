<p align="center">
  
![image](https://github.com/user-attachments/assets/680b2427-2b78-43ac-b846-602246e928b9)
</p>

<h1>Azure Sentinel | SIEM Map with Live Cyber Attacks</h1>
This tutorial outlines the steps to setup Azure Sentinel (SIEM) and connect it to a live virtual machine acting as a honey pot to observe live attacks (RDP Brute Force) from all around the world.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Internet Information Services (IIS)

<h2>Operating Systems Used </h2>

- Windows 10</b> (21H2)

<h2>Technical Steps</h2>

-  Create Virtual Machine
-  Allow all in Firewall
-  Create Log Analytics Workspace
-  Enable gathering VM logs in Microsoft Defender
-  Create a Data collection rule.
-  Connect Log Analytics to VM
-  Setup Azure Sentinel
-  Observe Event Viewer Logs in VM
-  Turn off Windows Firewall on VM
-  Get Geolocation.io API Key
-  Run Script To get Geo Data from attackers
-  Create custom log in LAW to bring in our custom log
-  Create custom fields/extract fields from raw custom log data
-  Setup map in sentinel with Latitude and Longitude (or country)

<p>
  
![image](https://github.com/user-attachments/assets/4aa31c42-415c-456a-9987-9b75dd5c6938) ![image](https://github.com/user-attachments/assets/ece9c0d8-5742-4393-9d7c-59778bfaaba4)
</p>
<p>
1.) Create a virtual machine within Azure that will act as our "honeypot" and customize it's NIC network group firewall rule to allow any and everything to connect to the VM. The goal here is to make the "honeypot" VM as vulnerable as possible. You can create a new resource group while creating the VM. All resources we create will go into this resource group.
</p>
<br />

<p>
  
![image](https://github.com/user-attachments/assets/098fe3fc-3bcc-4fae-8245-52fe1b955851)
</p>
<p>
2.) Next, create a log analytics workspace resource and assign it to the same resource group as the VM.
</p>
<br />

<p>
  
![image](https://github.com/user-attachments/assets/b44a431c-250f-4bb4-8640-58aa1d43c09c) ![image](https://github.com/user-attachments/assets/5fa0e71a-07e5-4cf7-81f3-fa9edf934444) ![image](https://github.com/user-attachments/assets/1eef6856-70c6-4634-9d4c-9f3ccff0be49)
</p>
<p>
3.)  Now to enable the abaility for the LAW to gather logs from the VM, navigate to Microsoft Defender for Cloud -> Enviorment settings -> select the subscription your LAW is connected to -> turn on "Defender CSPM" and "Servers" -> Click save
</p>
<br />

<p>

![image](https://github.com/user-attachments/assets/1363defe-21f3-45b4-9592-fe73f1ac7538)
</p>
<p>
4.) Create a Data collection rule.
</p>
<br />

<p>
  
![image](https://github.com/user-attachments/assets/8258d097-ac7f-46ec-b309-94103297ac39) ![image](https://github.com/user-attachments/assets/c81c7831-fe5c-41f2-8521-2365a1d9d83f)
</p>
<p>
5.) Connect the honeypot LAW to the honeypot VM and add Microsoft Sentinel to your Log Analytics workspace. This is the SIEM that will be used to visualize the attack data.
</p>
<br />

<p>
  
![image](https://github.com/user-attachments/assets/8b787ad8-49c2-4dc0-8357-e0b5e2dca021)
</p>
<p>
6.) Remote desktop into the honeypot VM.
</p>
<br />

<p>
  
![image](https://github.com/user-attachments/assets/4f635fbe-3944-4b45-a32d-27320bb1106e) ![image](https://github.com/user-attachments/assets/10ecd0aa-aec1-4644-b47c-e85cc3e76ef1) ![image](https://github.com/user-attachments/assets/7a0864cc-0ac2-4be0-8522-852e683a4b31)
</p>
<p>
7.) Observe Event Viewer Logs in VM.
</p>
<br />

<p>
  
![image](https://github.com/user-attachments/assets/5c547771-9a54-4300-b30b-70098472cc88) ![image](https://github.com/user-attachments/assets/31f1dae9-9725-4eaf-b2bf-001f165d0316) ![image](https://github.com/user-attachments/assets/3e27957e-8569-4cb4-a919-e4eade086e4f) ![image](https://github.com/user-attachments/assets/bf41e6ca-d7f7-475b-8733-05f0f86edc4e) ![image](https://github.com/user-attachments/assets/0fb082c9-7076-4b3a-9692-075ed10b6b5b)
</p>
<p>
8.) Open command prompt from your actual device and perpetually ping the honeypot VM "ping ..*.* -t" and observe the request time outs. This means the firewall is enable on the VM and it needs to be turned off to allow all internet traffic through.  Press the start menu on the VM -> search "wf.msc" and open it -> click "Windows Defender Firewall Properties -> turn off the firewalls -> observe the successful pings in the command prompt, letting you know that the firewalls were succesfully disabled.
</p>
<br />

<p>

![image](https://github.com/user-attachments/assets/5dc66f0a-c08f-4900-bb6a-3424192eed36) ![image](https://github.com/user-attachments/assets/e0d37289-ee9c-4712-87cb-5dd0f98fbf3a) ![image](https://github.com/user-attachments/assets/c8226abe-38c1-4b35-afcc-357903544f50) ![image](https://github.com/user-attachments/assets/1fc265be-21bf-4ad2-87d0-dc1b12f83d8a) ![image](https://github.com/user-attachments/assets/cfbee080-dad6-4173-8cca-bda69873e102) ![image](https://github.com/user-attachments/assets/54b5453d-4c26-4e01-aa3b-4e97a5328189) ![image](https://github.com/user-attachments/assets/3af96167-9a0d-4e2a-b35d-1786a3b50275)

</p>
<p>
9.) Open Windows Powershell ISE and run a script that runs perpetuality. Essentially looking through the event log that we were looking at earlier and logs all the failed events to login into the "honeypot" VM, recording IP addresses, longitude, latitude, etc. Once this data starts to be collected, it creates a new log file. (C:\Windows\Programdata\failed_RDP.log). Whenever there’s a failed log on in the event viewer we will see a purple output (look at image of Windows Powershell ISE). Basically what’s happening is it looks through these failed logs, send them to created log file. We're going to use this to train our Log Analytics Workspace.
P.S. ipgeolocation.io which offers FREE IP lookup API and accurate IP location finder is what I used in the perpetual script to look up the IP addresses that are connected to our failed log events.
</p>
<br />

<p>
  
![image](https://github.com/user-attachments/assets/588c0ef8-a0a8-4179-a325-55a626839d6f) ![image](https://github.com/user-attachments/assets/40da8023-fc58-43f6-a87c-7740d6c8af74) ![image](https://github.com/user-attachments/assets/5c862de3-c630-47ca-9b95-886168456c9a)
</p>
<p>
10.) Create custom log in LAW to bring in our custom log.
</p>
<br />

<p>
  
![image](https://github.com/user-attachments/assets/81a6db96-a931-4763-9482-092d9dd3ddf8) ![image](https://github.com/user-attachments/assets/0118f0f9-c9a1-41f7-aa12-302e99a6b5c8)
</p>
<p>
11.) 
</p>
<br />

<p>
  
![image](https://github.com/user-attachments/assets/9e7dc04b-80a1-413c-91d1-5187dcc5acb4)
</p>
<p>
12.) 
</p>
<br />

<p>

![image](https://github.com/user-attachments/assets/aad90454-9d64-40d1-b5e8-9836a9f693eb) ![image](https://github.com/user-attachments/assets/e17840f8-870b-40fa-bf3f-322642a7db7a) ![image](https://github.com/user-attachments/assets/d184fd1a-c7bc-4809-ac1c-2b8d83b4a2fa)


</p>
<p>
13.) 
</p>
<br />

<p>
  
![image](https://github.com/user-attachments/assets/6d0f29ca-360e-456e-aa3b-0277da90893c)
</p>
<p>
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
</p>
<br />
