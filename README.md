<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)



<h2>Deployment and Configuration Steps</h2>

<h3>Step 1: Setup Resources in Azure</h3>

- Create two Virtual Machines
    - 1st Virtual Machine will be the Domain Controller
       - Name: DC-1
       - Image: Windows Server 2022
       - Take note of the Virtual Network (Vnet) that gets created at this time
       
<p align="center">
<img src="https://i.imgur.com/AsS2sLq.png" height="70%" width="70%" alt="Azure Free Account"/>



 - We need to set DC-1's virtual Network Interface Card (NIC) Private IP address to be static
     - Go to DC-1's network settings --> select Networking --> select the link next to Network Interface 
     - IP Configurations --> select "ipconfig1..."
     - Change Assignment from dynamic to static (this guarantees DC-1's IP address will not change) --> select Save
	   
<p align="center">
<img src="https://i.imgur.com/5B66tdZ.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/2LusfFQ.png" height="70%" width="70%" alt="Azure Free Services"/>  <img src="https://i.imgur.com/il0ARpO.png" height="70%" width="70%" alt="Azure Free Services"/>
</p>


- 2nd Virtual Machine will be the Client
     - Name: Client-1
     - Image: Windows 10 Pro
     - we will use the same resource group and Vnet as DC-1

<p align="center">
<img src="https://i.imgur.com/FW3r6qA.png" height="70%" width="70%" alt="Azure Free Account"/>


<h3>Step 2: Ensure Connectivity between the client and Domain Controller</h3>

- Log in to Client-1 using Microsoft Remote Desktop
  - Search for Command Prompt and open it
  - ping DC-1's private IP Address (in my case, 10.0.0.4)
    - ping -t 10.0.0.4
      - Due to DC-1's firewall blocking ICMP traffic, the request is timing out. To fix this, we need to enable ICMPv4 on DC-1's local Windows firewall. 

<p align="center">
<img src="https://i.imgur.com/kbfxN73.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
- Log in to DC-1 using Microsoft Remote Desktop
    - Start --> Windows Administrative Tools --> Windows Defender Firewall with Advanced Security --> Inbound Rules.
    	- Sort the list by protocols 
      	- Find Core Networking Diagnostics ICMPv4 and enable these two inbound rules one at a time

<p align="center">
<img src="https://i.imgur.com/jfK4p7w.png" height="70%" width="70%" alt="Azure Free Account"/>

- Log back into Client-1 and in Command Prompt you should see the switch from the request timing out to receiving a reply. Thus, the ping to DC-1 is now successful.
    
<p align="center">
<img src="https://i.imgur.com/kaSTaTZ.png" height="70%" width="70%" alt="Azure Free Account"/> 


<h3>Step 3:  Install Active Directory</h3>

- Log back ito DC-1 
    - Open Server Manager
    - Select "Add roles and features." Follow the prompts.
    - At Server Roles, check "Active Directory Domain Services." (Ignore how the picture below already says "Installed") Then select next Add Features. Select next and finish installing.

<p align="center">
<img src="https://i.imgur.com/DQRVNnm.png" height="80%" width="80%" alt="Azure Free Account"/> <img src="https://i.imgur.com/RpzngRi.png" height="50%" width="50%" alt="Azure Free Services"/>
</p>

   - At the top right of the Server Manager Dashboard, click on the flag.
        - Select Promote this server to a domain controller.

<p align="center">
<img src="https://i.imgur.com/GOYiTFe.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
 - Select Add a new forest
          - Root domain name: mydomain.com
 - Select next
          - Create password
 - Select next, follow the prompts and finish up by selecting install. 


<p align="center">
<img src="https://i.imgur.com/IjfUZ0a.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
- DC-1 will automatically restart.
	- Log back into DC-1 as user: mydomain.com\labuser               

<p align="center">
<img src="https://i.imgur.com/oNp39DK.png" height="70%" width="70%" alt="Azure Free Account"/> 
	


<h3>Step 4: Create an Admin and Normal User Account in Active Directory v1.15.8</h3>
     
- On DC-1, open up Server Manager
	- Click tools at the top right hand side
		- select Active Directory Users and Computers.

<p align="center">
<img src="https://i.imgur.com/udGHbGs.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
- Right click mydomain.com -> New -> Select Oranizational Unit (We will be creating 2 folders.)
	- Name one _EMPLOYEES and the other _ADMINS
	
<p align="center">
<img src="https://i.imgur.com/5wSZuA4.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
	
- Right click mydomain.com and click referesh to sort the new organizational units to the top.
	- Go to _ADMINS organzational unit -> right click -> New -> User
		- First/Last name: jane doe
		- user login name: jane_admin
			- Select next and create a password 
				- uncheck all boxes, select next and then select finish
<p align="center">
<img src="https://i.imgur.com/nv6jc9p.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/uLopQTZ.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
- Go to _ADMINS organzational unit -> right click Jane doe -> select properties
	- Click "member of" tab -> Select Add -> type in domain admins -> Check Names -> OK -> Apply
- Log out of DC-1 as "labuser" and log back in as “mydomain.com\jane_admin”



<p align="center">
<img src="https://i.imgur.com/EapMhBs.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/vGb8Kx8.png" height="70%" width="70%" alt="Azure Free Services"/>
</p>
 
     

<h3>Step 5: Join Client-1 to your domain (mydomain.com)
</h3>

- Go back to the Azure portal. 
	- Go to Client-1 Virtual Machine
		- On the left hand side select Networking -> select the link next to the NIC -> DNS server -> Custom -> type in DC-1's private IP address -> Save
		- After it is done updating, select restart and select yes

<p align="center">
<img src="https://i.imgur.com/z6UesO7.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/bt0yK17.png" height="70%" width="70%" alt="Azure Free Services"/>  <img src="https://i.imgur.com/sB5edH5.png" height="70%" width="70%" alt="Azure Free Services"/>
</p>

- Log back into Client-1 using Microsoft Remote Desktop as original local admin (labuser)
	- Right click the start menu and select System
		- On right hand side, select Rename this PC (advanced) -> Change -> Under Member of, select domain -> type mydomain.com and select OK
			- Username: mydomain.com\jane_admin
			- Type in password and press OK
- Restart the computer 			


<p align="center">
<img src="https://i.imgur.com/3HxJLpe.png" height="80%" width="80%" alt="Azure Free Account"/> <img src="https://i.imgur.com/J8M4zBU.png" height="50%" width="50%" alt="Azure Free Services"/>
</p>

<h3>Step 6:  Setup Remote Desktop for non-adminitrative users on Client-1
</h3>

- Log back into Client-1 
	- use mydomain.com\jane_admin
		- Right click the start menu and select System
			- On the right hand side, select Remote Desktop -> under User Accounts, click on select users that can remotely access this PC -> Select add
			- Type: domain users -> Check Names -> OK. Select Ok again

 
 <p align="center">
<img src="https://i.imgur.com/HgAXVMX.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/0QDUk5l.png" height="60%" width="60%" alt="Azure Free Services"/>
</p>

<h3>Step 7:   Create a bunch of additional users and attempt to log into client-1 with one of the users
</h3>

- Log back into DC-1 as jane_admin
	- Search for Powershell_ise, right click on it and open as an administrator
		- At the top left, select new script and paste the contents of the script into it. You can find the script [here](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1).

<p align="center">
<img src="https://i.imgur.com/MpvLIbB.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/V4vIvre.png" height="70%" width="70%" alt="Azure Free Services"/>
</p>

- Click the green arrow button near the top middle to run the script
	- Once the users have been created, go back to Active Directory Users and Computers -> mydomain.com -> _EMPLOYEES
		- You will see all the accounts that were created, in here. 
- You can now log into Client-1 with one of the accounts that were created. 			

<p align="center">
<img src="https://i.imgur.com/3HN1Nf4.png" height="80%" width="80%" alt="Azure Free Account"/> <img src="https://i.imgur.com/CeE8LGh.png" height="50%" width="50%" alt="Azure Free Services"/>  <img src="https://i.imgur.com/7ZVBp8a.png" height="70%" width="70%" alt="Azure Free Services"/>
</p>

Lets log into Client-1 with one of the users that were created (in our instance "base.milu", password will be Password1)

<p align="center">
<img src="https://i.imgur.com/EzgHWRs.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/hYFodxu.png" height="70%" width="70%" alt="Azure Free Services"/>
</p>



🎉Congratulations! You have implementated on-premises Active Directory and created users within Azure Virtual Machines!🎉
