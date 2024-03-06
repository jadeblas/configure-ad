<div align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</div>

<h1 align="center">Setting up Active Directory in Microsoft Azure</h1>
This guide illustrates the process of setting up Active Directory in Microsoft Azure. It involves configuring two virtual machines (VMs) within the same virtual network. One VM will serve as the <b>Domain Controller</b>, while the other will function as a <b>client</b>. The setup includes configuring the Active Directory to allow the client to join the domain and creating user accounts using a Powershell script.

<br />

<h2>Environments and Tools Utilized</h2>
<ul>
  <li>Microsoft Azure (Virtual Machines/Compute)</li>
  <li>Remote Desktop</li>
  <li>Active Directory Domain Services</li>
  <li>Powershell</li>
</ul>

<br />

<h2>Operating Systems Employed</h2>
<ul>
  <li>Windows Server 2022</li>
  <li>Windows 10 Pro (21H2)</li>
</ul>

<br />

<h2>Setup Process</h2>

<h3>Establishing Resources in Azure</h3>

<p>
  <ul>
    <li>Install the Client VM using the standard Windows 10 image (OS).</li>
    <ul>
      <li>Refer to the <b><a href="https://github.com/jadeblas/vm-network">tutorial</a></b> for instructions on VM installation and Remote Desktop access.</li>
    </ul>      
    <li>Create the Domain Controller VM with Active Directory, using the <b>Windows Server 2022 Datacenter: Azure Edition - x64 Gen2</b> image.</li>
    <ul>
      <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/7e85bce6-826f-4947-be95-b6d37d15ce4e" height="80%" width="80%"/></li>
    </ul>
    <li>Once VMs are created, assign a <i>static</i> IP Address to the Domain Controller. A dynamic IP configuration might hinder communication with the client VM.</li>
    <li>In Azure Virtual Machines, navigate to <b>Networking</b>, then access the link next to <b>Network Interface</b>. Proceed to <b>IP Configurations</b> under <b>settings</b>, and toggle the IP configuration to <b>Static</b>.</li>
    <ul>
      <li>IP Configuration for the Domain VM</li>
      <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/206c0af8-a995-4eb9-8a9a-1f024a4dc846" height="80%" width="80%"/></li>
    </ul>
  </ul>
</p>

<br />

<h3>Verifying Connectivity</h3>

<p>
  <ul>
    <li>Access the Client VM, open Command Prompt, and execute the command <b>ping [Domain Controller Private IP Address] -t</b> to continuously send pings and confirm connectivity with the Domain Controller. The first ping may timeout due to Firewall Settings on the Domain Controller.</li>
    <ul>
      <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/786416ed-167c-4b05-9111-2eb61efa7ae2" height="80%" width="80%"/></li>
    </ul>
    <li>Access the Domain Controller VM, navigate to <b>Windows Defender Firewall with Advanced Security</b>. Proceed to <b>Inbound Rules</b> and enable rules under <b>ICMPv4</b>, specifically <i>Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)</i>.</li>
    <ul>
	<li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/ab65e1bf-318a-451d-9ef6-c86580f2f846" height="80%" width="80%"/></li>
    </ul>
    <li>Return to the Client VM; you should now receive replies.</li>
    <ul>
	<li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/765abaca-3dc0-4c7a-af60-c5077eb37339" height="80%" width="80%"/></li>
    </ul>
  </ul>
</p>

<br />

<h3>Active Directory Installation on the Domain Controller</h3>

<p>
  <ul>
	  <li>In the Domain Controller VM, access the Server Manager Dashboard and select <b>Add Roles and Features</b>. Follow the installation process; under <b>Server Roles</b>, ensure to check <b>Active Directory Domain Services</b>.</li>
	  <ul>
	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/643b6428-b542-4baf-91d3-8341fed1c421" height="80%" width="80%"/></li>
	  </ul>
	 <li>After installation, promote the server to a domain controller. If prompted, click on the warning notification at the top right (flag icon) and select <b>Promote this server to a domain controller</b>. Choose <b>Add a new forest</b> and specify a domain name; for this tutorial, use <b>mydomain.com</b>. Set the password and complete the installation. You will be signed out automatically; re-login through Remote Desktop to finish installation.</li>
	  <ul>
	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/0c762bc9-869a-47d8-a0ab-fb1ea7ded961" height="80%" width="80%"/></li>
	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/6847c831-1ce0-42c5-a347-c4620d4b6693" height="80%" width="80%"/></li>
    	 </ul>
  </ul>
</p>

<br />

<h3>Logging In Note</h3>

<p>
  <ul>
    <li>When logging into the domain controller VM via Remote Desktop Connection, ensure to log in with the <b>domain context.</b></li>
    <li>Enter the domain path followed by the user name. For instance: <b>mydomain.com\labuser.</b></li>  
  </ul>
</p>

<br />

<h2>Configuration Process</h2>

<h3>Establishing Organizational Units (OUs) and Users</h3>

<p>
  <ul>
    <li>OUs serve as containers for organizing information, privileges, and login access of users within the directory</li>
    <li>In the Server Manager Dashboard, navigate to the <b>Tools</b> tab to access the Active Directory Users and Computers console. Right-click on the domain (mydomain.com) and create two OUs, namely <b>_ADMIN</b> and <b>_EMPLOYEES</b>.</li>
	  <ul>
		  <li>These OU names are required for a subsequent step where multiple accounts are created</li>
	  </ul>
    <li>In the _ADMIN OU, create the user <b>Jane Doe</b> with the username <b>jane_admin</b> and set a password of your choice</li>
    <ul>
	<li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/2949cb50-af57-4362-9384-418882c43d13" height="80%" width="80%" /></li>
    </ul>
    <li>Grant Jane administrative privileges. Using the <b>Security Group</b>, right-click on the user, open their <b>Properties</b>, click <b>Member Of</b>, then <b>Add</b> to assign the appropriate security group.</li>
    <ul>
	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/964f9493-306c-4f6c-bae0-01a361309c24)" height="80%" width="80%" /></li>
    </ul>
    <li>Going forward, use the user Jane for login purposes, utilizing the login username jane_admin.</li>
  </ul>
</p>

<br />

<h3>Joining the Client to the Domain</h3>

<p>
  <ul>
    <li>To begin, configure the Domain Name System (DNS) server. Navigate to your Client VM in the Azure Portal, proceed to <b>Networking</b>, then access the link next to <b>Network Interface</b>. Navigate to <b>DNS Servers</b> under <b>settings</b>, and set the DNS Server to <b>Custom</b>. Enter the Domain Controller's private IP address and save the changes. Restart the client VM to ensure the DNS changes are saved.</li>
    <ul>
	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/bcb60caa-296c-4c24-86ea-a40ebc7e45d8" height="80%" width="80%" /></li>
    </ul>
    <li>In the System menu of the client VM, select Rename this PC (advanced) and click Change.</li>
    <li>Input the domain and necessary credentials to allow the client to join the domain (log in as jane_admin). Note that the login credentials must be entered within the context of the domain path (mydomain.com\jane_admin).</li>
    <ul>
	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/5765bcea-4280-422c-83bb-1559f5495650" height="80%" width="80%" /></li>
    </ul>
    <li>The client should now be integrated into the domain (A popup should appear welcoming you to the domain). On the domain controller, the client should now be visible in Computers within the Active Directory Users and Computers panel.</li>
    <ul>
	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/d89d6325-2ad9-40b4-9b97-81d8653c0721" height="80%" width="80%" /></li>
    </ul>  
  </ul>
</p>

<br />

<h3>Setting Up Remote Desktop for Non-Administrative Users on Client VM</h3>

<p>
  <ul>
    <li>Prior to allowing users in the domain to use the client computer, enable Remote Desktop for non-administrative users.</li>
    <li>While logged in as the administrator (jane_admin), open <b>System Properties</b>. Click on <b>Remote Desktop</b> and select users who can remotely access this PC.</li>  
    <li>Grant Domain Users access to Remote Desktop. Non-administrative users can now log in to the Client.</li>
    <ul>
	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/f0ae679d-952d-4ada-93a7-db8abaccba6c" height="80%" width="80%" /></li>
    </ul>
  </ul>
</p>

<br />

<h3>Creating Users and Attempting to Log into the Client VM with One of the Users</h3>

<p>
  <ul>
    <li>While logged into the Domain Controller VM as jane_admin, launch <b>Powershell ISE</b> as an administrator.</li>
    <li>Utilize <a href="https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1">this PowerShell script</a> to generate thousands of randomly created accounts, all with the password "Password1".</li>
    <li>In Powershell ISE, create a new file, paste the PowerShell script into the file, and execute the script.</li>
    <ul>
	    <li>All these users will be generated and placed into the _EMPLOYEES Organizational Unit within the Active Directory.</li>
	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/820c192a-ff9f-4e48-867e-482be6490ed1" height="80%" width="80%" /></li>
    </ul>
    <li>Proceed to the Active Directory Users and Computers console, select a random username, and retrieve their login information by accessing <b>Properties</b> and navigating to the <b>Account</b> tab.</li>
    <ul>
	    <li>The generated username should appear as <b>[first name].[last name]</b>. In the image, the user is selecting "falojo.kugori".</li>
	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/dd81e596-6f20-4fe2-82fd-0b667dcd5812" height="80%" width="80%" /></li>
    </ul>
    <li>Attempt to log in to the Client VM using the selected generated username (username being <b>mydomain.com\username</b>) and the password "Password1".</li>
    	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/3a06b210-ea7b-4458-962b-b67d4b733c44" height="80%" width="80%" /></li>
    <li>After successfully logging in, you can verify your login under the new user by using the command line tools <b>“whoami”</b> and <b>“hostname”</b>.</li>
    	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/2bc4e457-d415-4850-9d9a-2640b0970292" height="80%" width="80%" /></li>
    <ul>
    <li>If a user gets <b>locked out</b> of their account due to too many login attempts, simply navigate to the user's properties in the Active Directory Users and Computers Console and check the box labeled <b>“Unlock Account.”</b> This should resolve the issue, allowing the user to log in again.</li>
      	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/7f27ff20-7fb0-4793-a446-1f8af6019b9f" height="80%" width="80%" /></li>
    <li>In case a user <b>forgets their password</b>, you can reset it by right-clicking on their user in the Active Directory Users and Computers console. Additionally, this menu allows you to manage account access by enabling or disabling it as needed.</li>
      	    <li><img src="https://github.com/jadeblas/configure-ad/assets/161860082/5fc893cb-976c-4b69-a634-35af952b9722" height="80%" width="80%" /></li>
  </ul>
</p>

<br />
