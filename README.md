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
       - Take note of the Virtual Network (Vnet) that gets created at this time.
       
<p align="center">
<img src="https://i.imgur.com/tvu44Yp.png" height="70%" width="70%" alt="Azure Free Account"/>



 - We need to set DC-1's virtual Network Interface Card (NIC) Private IP address to be static
     - Go to DC-1's network settings > select Networking > select the link next to Network Interface 
     - IP Configurations > select "ipconfig1..."
     - Change Assignment from dynamic to static (this guarantees DC-1's IP address will not change) > select Save
	   
<p align="center">
<img src="https://i.imgur.com/l9Zakrh.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/mUw9sHt.png" height="70%" width="70%" alt="Azure Free Services"/>  <img src="https://i.imgur.com/vJkPao0.png" height="70%" width="70%" alt="Azure Free Services"/>
</p>


- 2nd Virtual Machine will be the Client
     - Name: Client-1
     - Image: Windows 10 Pro
     - We will use the same resource group and Vnet as DC-1

<p align="center">
<img src="https://i.imgur.com/ZxU1LpI.png" height="70%" width="70%" alt="Azure Free Account"/>


<h3>Step 2: Ensure Connectivity between the client and Domain Controller</h3>

- Log in to Client-1 using Microsoft Remote Desktop
  - Search for Command Prompt and open it
  - ping DC-1's private IP Address (in my case, 10.0.0.4)
    - ping -t 10.0.0.4
      - Due to DC-1's firewall blocking ICMP traffic, the request is timing out. To fix this, we need to enable ICMPv4 on DC-1's local Windows firewall. 

<p align="center">
<img src="https://i.imgur.com/kbfxN73.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
- Log in to DC-1 using Microsoft Remote Desktop
    - Start > Windows Administrative Tools > Windows Defender Firewall with Advanced Security > Inbound Rules.
    	- Sort the list by protocols 
      	- Find Core Networking Diagnostics ICMPv4 and enable these two inbound rules one at a time.

<p align="center">
<img src="https://i.imgur.com/jfK4p7w.png" height="70%" width="70%" alt="Azure Free Account"/>

- Log back into Client-1 and in Command Prompt you should see the switch from the request timing out to receiving a reply. Thus, the ping to DC-1 is now successful.
    
<p align="center">
<img src="https://i.imgur.com/kaSTaTZ.png" height="70%" width="70%" alt="Azure Free Account"/> 


<h3>Step 3:  Install Active Directory</h3>

- Log back ito DC-1 
    - Open Server Manager
    - Select Add roles and features > Next > until you get to Server Roles.
    - At Server Roles, select Active Directory Domain Services > select Add Features > select Next for any of the remaining dependencies and finish installing.

<p align="center">
<img src="https://i.imgur.com/k7FXshb.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/gIVOzWp.png" height="50%" width="50%" alt="Azure Free Services"/>
</p>

   - At the top right of the Server Manager Dashboard, click on the flag containing a yellow triangle with an exclamation point.
        - Select Promote this server to a domain controller.

<p align="center">
<img src="https://i.imgur.com/HwP1r74.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
 - Select Add a new forest
          - Root domain name: mydomain.com (for the purpose of this demonstration we will use something simple: mydomain.com)
 - Select Next
          - Create password
 - Select Next for any of the remaining dependencies and finish installing. 


<p align="center">
<img src="https://i.imgur.com/ssGHbwP.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
- DC-1 will automatically restart.
	- Log back into DC-1 as user: "mydomain.com\labuser"              

<p align="center">
<img src="https://i.imgur.com/Cag7Btr.png" height="50%" width="50%" alt="Azure Free Account"/> 
	


<h3>Step 4: Create an Admin and Normal User Account in Active Directory</h3>
     
- Open Active Directory on DC-1 by opening up Server Manager
	- Select tools in the top right corner
		- Select Active Directory Users and Computers.

<p align="center">
<img src="https://i.imgur.com/YAnWOPr.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
- Right-click mydomain.com > New > select Oranizational Unit. We will create 2 folders.
	- Name one "_EMPLOYEES" and the other "_ADMINS"
	
<p align="center">
<img src="https://i.imgur.com/1TyQDMh.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
	
- For easier navigation, right-click mydomain.com and click referesh to sort the new organizational units to the top.
	- Create an admin account by going to _ADMINS organzational unit > right-click > New > User
		- First name: jane
		- Last name: doe
		- User logon name: jane_admin
			- Click next and create a password 
				- Uncheck "User must change password at next logon" for the purpose of this demonstration; select next and then select finish
<p align="center">
<img src="https://i.imgur.com/wWmcwe3.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/bnqzBW0.png" height="70%" width="70%" alt="Azure Free Account"/><img src="https://i.imgur.com/zyGGXTN.png" height="70%" width="70%" alt="Azure Free Services"/>
</p>
	
- To make our user an actual domain admin we need to assign it to the Domain Admin group: Go to _ADMINS organzational unit > right-click Jane Doe > select Properties > select Member Of tab > select Add > enter in domain admins > Check Names > OK > make sure Domain Admins is selected > Apply > OK
- Log out of DC-1 as "labuser" and log back in as “mydomain.com\jane_admin” using the password you created with the user jane doe.



<p align="center">
<img src="https://i.imgur.com/QNIfnxE.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/JmZpTx6.png" height="70%" width="70%" alt="Azure Free Services"/>
</p>
 
     

<h3>Step 5: Join Client-1 to your domain (mydomain.com)
</h3>

- From the Azure portal, we will set Client-1's DNS settings to the Domain Controller's private IP address. 
	- Go to Client-1 Virtual Machine in Azure
		- On the left hand side select Networking > select the link next to the NIC > DNS server > Custom > enter in DC-1's private IP address > Save
		- When finished updating > select Restart > Yes

<p align="center">
<img src="https://i.imgur.com/Squ6m0i.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/me4kDtO.png" height="70%" width="70%" alt="Azure Free Services"/> <img src="https://i.imgur.com/8n0FufP.png" height="70%" width="70%" alt="Azure Free Services"/>
</p>

- Log back into Client-1 using Microsoft Remote Desktop as our original local admin "labuser"
	- Right-click the start menu and select System
		- On the right hand side, select Rename this PC (advanced) > Change > Under Member of, select domain > type mydomain.com > OK
			- Username: mydomain.com\jane_admin
			- Type in password and select OK
- You must restart your computer to apply these changes > select Restart Now			


<p align="center">
<img src="https://i.imgur.com/mpnZVAd.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/JaNkU57.png" height="50%" width="50%" alt="Azure Free Services"/>
</p>

<h3>Step 6:  Setup Remote Desktop for non-adminitrative users on Client-1
</h3>

- Log back into Client-1 using "mydomain.com\jane_admin"
		- Right-click the start menu and select System
			- On the right hand side, select Remote Desktop > under User Accounts, click on Select users that can remotely access this PC > select Add > type: domain users > Check Names > OK > Ok

 
 <p align="center">
<img src="https://i.imgur.com/Yv1np61.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/ejHCUNB.png" height="60%" width="60%" alt="Azure Free Services"/> 
</p>

<h3>Step 7:   Create a bunch of additional users and attempt to log into client-1 with one of the users
</h3>

- Log back into DC-1 as jane_admin
	- Search for PowerShell_ISE > right-click on it and run as administrator > Yes
		- In the top left corner select new script > from the link, copy the raw contents > paste the contents of the script into PowerShell_ISE. You can find the script [here](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1).

<p align="center">
<img src="https://i.imgur.com/uT9AsCV.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/MUqvsyN.png" height="70%" width="70%" alt="Azure Free Services"/> <img src="https://i.imgur.com/xTHI7nB.png" height="70%" width="70%" alt="Azure Free Services"/>
</p>

- Select the green arrow button near the top middle of PowerShell_ISE to run the script
	- Now that the users are being created, we can go back to Active Directory to see all of the accounts. Go to Active Directory Users and Computers > mydomain.com > _EMPLOYEES 

- You can now log into Client-1 with one of the accounts that were created. 			

<p align="center">
<img src="https://i.imgur.com/9u8SG51.png" height="70%" width="70%" alt="Azure Free Account"/> <img src="https://i.imgur.com/NriysN7.png" height="70%" width="70%" alt="Azure Free Services"/>
</p>

- We will now log into Client-1 with one of the users that were created (in my case, I will use "bat.jafa", password will be Password1).
	- Go back to Client-1 and log out of jane_admin and log back into Client-1 using the user you've selected.

<p align="center">
<img src="https://i.imgur.com/MhHsg6v.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Once logged in, search and open Command Prompt to confirm we have logged in to the correct user.
	- In Command Prompt, type "whoami" to see our user is correct (mydomain.com\bat.jafa)
	- Then type "hostname" to see we are on Client-1
	
<p align="center">
<img src="https://i.imgur.com/fz0SFoU.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

🎉Congratulations! You have implementated on-premises Active Directory and created users within Azure Virtual Machines!🎉
