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

<h2>High-Level Deployment and Configuration Steps</h2>

- Set up resources and virtual machines in Azure
- Test connectivity Between virtual machines
- Install Active Directory
- Create an administrator account in Active Directory
- Join a virtual machine the domain
- Set up remote desktop for all domain users
- Create additional user accounts using a Powershell script

<h2>Deployment and Configuration Steps</h2>

  <h3><strong>Set up the domain controller in Azure</strong></h3>
  <ul>
    <li>Create a new resource group named “RG-1.”</li>
  </ul>
  <ul>
    <li>In the Azure portal, create a new virtual machine (VM) running a Windows 2022 Datacenter Server. Name this machine <strong>“DC-1.”</strong> This will be our domain controller.
      <ul>
        <li>Note: Ensure that the VM-1 machine is inside of your RG-1 resource group!</li>
      </ul>
      <img src="https://github.com/amaraphi/configure-ad/assets/144752187/832f348e-c8fa-41d1-b024-f5a3017abbb6"/>
    </li>
  </ul>
  <h3>Change DC-1’s <strong>IP address from “dynamic” to “static.”</strong></h3>
  <ul>
    <li>On the Azure portal home page, click on Virtual Machines —&gt; DC-1.</li>
  </ul>
  <ul>
    <li>Under settings, click on Networking.</li>
  </ul>
  <ul>
    <li>Click on the virtual Network Interface card.</li>
  </ul>
  <ul>
    <li>Under Settings, click on IP configurations</li>
  </ul>
  <ul>
<img src="https://github.com/amaraphi/configure-ad/assets/144752187/7c50b570-8170-4b7d-8a6d-ac7591a772cc"/>
    <li>We can observe that DC-1’s private IP address is dynamic. Click on “ipconfig1.”
      <ul>
        <li>Note: write down or take note of this private IP address, which we’ll need for a later step in this tutorial.</li>
      </ul>
    </li>
  </ul>
  <ul>
    <li>Change the Allocation settings from Dynamic to Static. Click Save.</li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/f749a88b-2ada-4ae8-9aca-52584ac8c51e"/>
  <h3><strong>Create another VM, this time called “Client-1.” This is the client that we’ll join to Active Directory, which will allow any user within the domain to log in remotely.</strong></h3>
  <ul>
    <li>Under Subscription —&gt; Resource Group, select the RG-1 resource group that contains VM-1.</li>
  </ul>
  <ul>
    <li>Under Instance Details —&gt; image, select “Windows 10 Pro.”</li>
  </ul>
  <ul>
    <li>For size, your Client-1 VM must have at least 2-4 vCPUs.</li>
  </ul>
  <ul>
    <li>Before creating the VM, confirm that Client-1 will be within our domain controller’s virtual network by clicking on the Networking tab.</li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/44fd2055-0c1a-44a3-b22f-2389d450e952"/>

  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/a7eeb49c-ecbf-4b19-929b-fd1ab600c2f7"/>

  <ul>
    <li>After confirming, click “Review + Create” then “Create.”</li>
  </ul>
  <ul>
    <li>To confirm that both VM-1 and Client-1 are within the same resource group, search for “topology” in the search bar, and click on “Network Watcher” under Services.</li>
  </ul>
  <h3><strong>Testing Connectivity between the domain controller and Client-1</strong></h3>
  <ul>
    <li>Copy the public IP address for Client 1 and log in via Windows Remote Desktop.</li>
  </ul>
  <ul>
    <li>In Client-1’s remote environment, open the Windows command prompt.</li>
  </ul>
  <ul>
    <li>Initiate a perpetual ping to DC-1 using its private IP address.
      <ul>
        <li>To find DC-1’s private IP address, return to the Azure portal</li>
      </ul>
    </li>
  </ul>
  <ul>
    <li>Note that our ping requests have timed out. This is because the domain controller has blocked inbound ICMP traffic.</li>
  </ul>
 <img src="https://github.com/amaraphi/configure-ad/assets/144752187/7466d010-793d-402b-a1f3-2a3875610035"/>

  <ul>
    <li>Open a new session of Microsoft Remote Desktop.</li>
  </ul>
  <ul>
    <li>Login to the domain controller, DC-1, using its public IP address.
      <ul>
        <li>To find DC-1’s public IP addresss…</li>
      </ul>
    </li>
  </ul>
  <ul>
    <li>Open Windows firewall by using the taskbar to search for wf.msc.</li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/f7b6a888-4f1f-4ff2-889d-b567fa448878"/>

  <ul>
    <li>Under Inbound Rules, filter rules by protocol &gt; ICMPv4</li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/71d224a5-ddf2-49b6-889d-62456e1df1aa"/>

  <ul>
    <li>Right-click and select Enable Rule for:
      <ul>
        <li>Core Networking Diagnostics - ICMP Echo Request - Profile: Domain</li>
      </ul>
      <ul>
        <li>Core Networking Diagnostics - ICMP Echo Request - Profile: Private, Public</li>
      </ul>
      <img src="https://github.com/amaraphi/configure-ad/assets/144752187/8b33bcbd-8649-4920-b2f1-7e90194936a3"/>
    </li>
  </ul>
  <ul>
    <li>Return to Client 1’s command line in Windows Remote Desktop. Note that Client-1’s ping (echo) requests are now receiving replies from DC-1.</li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/1a0a0d3c-9866-4d6f-adec-61f6c0383590"/>
  <h3>Install Active Directory on DC-1</h3>
  <ul>
    <li>Return to DC-1 session in Windows Remote Desktop.</li>
  </ul>
  <ul>
    <li>Open the Server Manager. (If not already open, use the taskbar to search.)</li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/715b2768-c817-4d39-b448-0e0dfc31b581"/>
  <ul>
    <li>Under Configure this local server, select Add roles and features. This should open the installation wizard.
      <ul>
        <li>Select the following options to install Active Directory:
          <ul>
            <li>Installation type —&gt; Role based or feature based</li>
          </ul>
          <ul>
            <li>In Server Selection, confirm that Active Directory will be installed on DC-1 (confirm correct private IP address and that it will be installed on a Windows Datacenter Server)</li>
          </ul>
          <img src="https://github.com/amaraphi/configure-ad/assets/144752187/a41686f7-9316-4d87-9432-eab59282d751"/>
          <ul>
            <li>Server Roles —&gt; Active Directory Domain Services</li>
          </ul>
              <img src="https://github.com/amaraphi/configure-ad/assets/144752187/ed491e4d-7c34-45d1-9590-61acfd7daf66"/>
              <ul>
                <li>A pop up suggesting the installation of the additional features. Click add features.</li>
              </ul>
            </li>
          </ul>
          <ul>
            <li>On the Results page, click “Install.</li>
          </ul>
        </li>
      </ul>
    </li>
  </ul>
  <h3>Promote DC-1 as a domain controller</h3>
  <ul>
    <li>Back on the Server Manager homepage, select the Notifications in the top right. (Yellow triangular sign with exclamation mark.)</li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/93d2d6c9-d46b-48aa-8b6b-3778d8419eb8"/>
  <ul>
    <li>Click Promote this Server to a Domain Controller. You should now see the Active Directory Domain Services Configuration Wizard.
      <ul>
        <li>Select the following options:
          <ul>
            <li>Deployment Configuration —&gt; add a new forest</li>
          </ul>
          <ul>
            <li>For the purposes of this tutorial, let’s give our domain a root domain name of mydomain.com</li>
          </ul>
          <img src="https://github.com/amaraphi/configure-ad/assets/144752187/6329b441-b4ae-486b-abe7-ceb7b3159bc6"/>
          <ul>
            <li>Under Domain Controller Options, create a Directory Services Restore Mode password.</li>
          </ul>
          <img src="https://github.com/amaraphi/configure-ad/assets/144752187/744ebbb4-ef34-4253-a84c-afa9f2c364b1"/>
          <ul>
            <li>Click Next for all other options.</li>
          </ul>
          <ul>
            <li>Prerequisites check: Once prerequisites have been passed, click Install.
              <ul>
                <li><strong>Note: You will be automatically signed out of DC-1’s remote desktop session.</strong></li>
              </ul>
              <img src="https://github.com/amaraphi/configure-ad/assets/144752187/2e91fae6-9ecb-4c1c-a186-37b7143f0b4f"/>
            </li>
          </ul>
        </li>
      </ul>
    </li>
  </ul>
  <ul>
    <li>Refresh DC-1 in Azure
      <ul>
        <li>Once Active Directory has been installed, return to the Azure portal.</li>
      </ul>
      <ul>
        <li>Click on Virtual Machines &gt; DC-1.</li>
      </ul>
      <ul>
        <li>Restart DC-1 by clicking “Stop,” then “Start.”</li>
      </ul>
    </li>
  </ul>
  <h3><strong>Now that DC-1 has been promoted to a domain controller, we need to use its fully qualified domain name, or FQDN, to log back in with Remote Desktop.</strong></h3>
  <ul>
    <li>Open a new Remote Desktop Session.</li>
  </ul>
  <ul>
    <li>Log to DC-1:
      <ul>
        <li>Computer: DC-1’s public IP address</li>
      </ul>
      <ul>
        <li><code>Username: mydomain.com\username</code></li></ul>
      </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/0bf4987f-f527-4931-8705-73431047d67b"/>
    </li>
  </ul>
  <h3>Create Organizational Units (OUs)</h3>
  <ul>
    <li>After logging back in to our newly-created domain controller, DC-1, return to the Server Manager.</li>
  </ul>
  <ul>
    <li>Click Tools &gt; Active Directory Users and Computers</li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/2651d979-7074-4db2-bed8-bd8c81cf6c2d"/>
  <ul>
    <li>Right-click on <a href="http://mydomain.com">mydomain.com</a> and select New &gt; Organizational Unit
    </li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/8c1f4ab5-9014-49bd-a95e-1aed959366ab"/>
  <ul>
    <li>Create a new OU named <code>_EMPLOYEES.</code></li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/7365f1ff-fa91-4f32-8978-3b37682afd91"/>
  <ul>
    <li>Repeating the same steps, create another OU named _ADMINS</li>
  </ul>
  <ul>
    <li>Right-click on <code>mydomain.com">mydomain.com</code> and select Refresh.
    </li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/57ec43ca-2968-4dec-9074-cca785d9d3ef"/>
  <h3>Create a new user</h3>
  <p><strong>So far we’ve been using a generic account, lab_user, to configure our domain controller. Next we need to create an account that is tied to a specific person. We’ll create new user, “Jane Doe,” in the <code>_ADMINS</code> organizational unit.</strong></p>
  <ul>
    <li>Right-click on the _ADMINS folder and select New &gt; User:
      <ul>
        <li>First name: Jane</li>
      </ul>
      <ul>
        <li>Last name: Doe</li>
      </ul>
      <ul>
        <li>User logon name: jane_admin</li>
      </ul>
      <img src="https://github.com/amaraphi/configure-ad/assets/144752187/2336d4b4-15fc-4b49-8a59-5dab633a4886"/>
      <ul>
        <li>Create a password.</li>
      </ul>
      <ul>
        <li>Uncheck “User must change password at next logon.”</li>
      </ul>
      <img src="https://github.com/amaraphi/configure-ad/assets/144752187/28809477-fc3f-4a37-a12f-18f8734456db"/>
    </li>
  </ul>
  <h3>Assign Permissions with Security Groups</h3>
  <p><strong>Next, we need to assign permissions to our new user account. Since the Jane Doe is in the <code>_ADMINS</code> organizational unit, we’ll add this account to the Domain Admins security group.</strong></p>
  <ul>
    <li>Right click on the Jane Doe user account and select “Add to a group.”</li>
  </ul>
  <ul>
    <li>Type “domain” and select “Check Names.” You should see a list of all domain security groups.</li>
  </ul>
  <ul>
    <li>Select “Domain Admins” and click OK.</li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/a524fcc2-9ee1-49b2-909f-c8ec5483da93"/>
  <h3>Log in to the domain controller as an administrator</h3>
  <ul>
    <li>Open the command prompt and type logoff.</li>
  </ul>
  <ul>
    <li>Use Remote Desktop to log into DC-1, this time as jane_admin. Remember to use the fully qualified domain name, or FQDN.</li>
  </ul>
  <img src="https://github.com/amaraphi/configure-ad/assets/144752187/aee6d712-bc06-4603-b6f8-2047867ed2fe"/>
  <h3><strong>Configure Client-1 DNS settings</strong></h3>
  <ul>
    <li>Before Client-1 can be joined to the domain controller, we need to set its DNS settings to our domain controller’s private IP address.
      <ul>
        <li>Note: To complete this step, you’ll need to know DC-1’s private IP address.</li>
      </ul>
    </li>
  </ul>
  <ul>
    <li>In the Azure portal, use the toolbar to search for and open Network Interfaces.</li>
  </ul>
  <ul>
    <li>Click on Client-1’s network interface, then select “DNS Servers.”</li>
  </ul>
  <ul>
    <li>Change DNS settings to “Custom.”</li>
  </ul>
  <ul>
    <li>Paste or type DC-1’s private IP address then click Save.</li>
  </ul>
  <ul>
    <li>After the DNS settings have been updated, restart Client-1 from within the portal.</li>
  </ul>
  <h3>Join Client-1 to the domain controller</h3>
  <ul>
    <li>Use Remote Desktop to log in to Client-1</li>
  </ul>
  <ul>
    <li>Use the Windows Task Bar to search “about,” then click on “About your PC”:
      <ul>
        <li>Click on Rename this PC &gt; Change</li>
      </ul>
      <ul>
        <li>Member of &gt; Domain: <a href="http://mydomain.com">mydomain.com</a> &gt; OK.
        </li>
      </ul>
    </li>
  </ul>
  <ul>
    <li>You will be prompted to enter the username and password of an account with permission to join the domain.</li>
  </ul>
  <ul>
    <li>Enter the jane_admin username and password.</li>
  </ul>
  <ul>
    <li>You will be prompted to restart Client-1. Click OK then Restart Now.</li>
  </ul>
  <h3>Confirm that Client-1 is joined to the domain controller</h3>
  <ul>
    <li>In Active Directory Users and Computers, click on the root of the domain (mydomain.com) &gt; Computers to confirm that Client-1 is now inside of Active Directory.</li>
  </ul>
  <p>Allow all domain users to log in to Client-1</p>
  <ul>
    <li>Log in to client one with the jane_admin account.</li>
  </ul>
  <ul>
    <li>Use the taskbar to search for “About.”</li>
  </ul>
  <ul>
    <li>Click on Remote Desktop &gt; Select users that can remotely access this PC.</li>
  </ul>
  <ul>
    <li>Under “object names” type “domain” then click on “Check Names.”</li>
  </ul>
  <ul>
    <li>Select Domain Users. Click OK.</li>
  </ul>
  <h3>Log in to Client-1 as a domain user</h3>
  <p><strong>Now that any user in the domain can access Client-1, we’ll create a randomly generated a list of domain users using a Powershell script. These users will be part of the _EMPLOYEES organizational unit.</strong></p>
  <ul>
    <li>Log into DC-1 with the jane_admin account.</li>
  </ul>
  <ul>
    <li>Use the taskbar to search for “Powershell ISE.” Right click on “run as Administrator.”</li>
  </ul>
  <ul>
    <li>Open a new file and paste the Powershell script.</li>
  </ul>
  <ul>
    <li>Click File —&gt; Run to run the script.</li>
  </ul>
  <ul>
    <li>You can observe the list of users being generated:
      <ul>
        <li>This will randomly generate a list of 1,000 users with names that consist of alternating vowels and consonants.</li>
      </ul>
      <ul>
        <li>These users will be automatically added to the _EMPLOYEES organizational unit.</li>
      </ul>
      <ul>
        <li>The password for all of the user accounts will be Password1.</li>
      </ul>
    </li>
  </ul>
  <ul>
    <li>When the script has finished executing, return to Active Directory Users and Computers and click on the _EMPLOYEES folder. You’ll see a list of our newly-created users.</li>
  </ul>
  <ul>
    <li>To log in to Client-1 using one of these accounts, pick a random user account from the list.</li>
  </ul>
  <ul>
    <li>Log out of Client-1.</li>
  </ul>
  <ul>
    <li>Log in to Client-1 using the random username you chose. Remember that the Powershell script has set the default for all user passwords as Password1.</li>
  </ul>
  <ul>
    <li>You should be able to log in to Client-1 as a domain user.</li>
  </ul>
