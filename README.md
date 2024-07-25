# configure-ad
<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines. We will create a Domain Controller and a Client VM, ensure connectivity, and set up Active Directory with user accounts. <br />



<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>


## Table of Contents
- [Setup Resources in Azure](#setup-resources-in-azure)
- [Ensure Connectivity between the Client and Domain Controller](#ensure-connectivity-between-the-client-and-domain-controller)
- [Install Active Directory](#install-active-directory)
- [Create Admin and Normal User Accounts in AD](#create-admin-and-normal-user-accounts-in-ad)
- [Join Client-1 to Your Domain](#join-client-1-to-your-domain)
- [Setup Remote Desktop for Non-Administrative Users on Client-1](#setup-remote-desktop-for-non-administrative-users-on-client-1)
- [Create Additional Users](#create-additional-users)

<h2>Deployment and Configuration Steps</h2>

<p> 

![image](https://github.com/user-attachments/assets/de681697-992f-4909-a1a6-31d2ce6b3d7c)
  
 **Create the Domain Controller VM (Windows Server 2022) named `DC-1`**
Take note of the Resource Group and Virtual Network (Vnet) that get created at this time.
Set the Domain Controller’s NIC Private IP address to be static.

**Create the Client VM (Windows 10) named `Client-1`**

Use the same Resource Group and Vnet created in Step 1.
Ensure both VMs are in the same Vnet

</p>

## Ensure Connectivity between the Client and Domain Controller

Login to Client-1 with Remote Desktop

Login to the Domain Controller

Enable ICMPv4 on the local Windows Firewall.
Check back at Client-1 to see the ping succeed
<br />

![image](https://github.com/user-attachments/assets/b7b6493a-299c-4281-a925-dd65d97f68ae)

![image](https://github.com/user-attachments/assets/e87dee1e-6802-4e8d-9494-ccaf05a5471c)

Locate DC-1's Private IP Address in the Azure portal and copy it. Proceed to Client-1 and open the Command Prompt, type "ping -t (DC-1 Private IP Address)"


Notice how the request continues to time out, this is due in part because ICMPv4 traffic is blocked by default on DC-1's firewall. Therefore, we will have to enable inbound ICMP traffic to allow for Client-1's continuous ping to be successful.

![image](https://github.com/user-attachments/assets/40b60a0e-0eda-49ef-8bae-21a9affb71ed)

check back at `Client-1` to see the ping succeed. Once the traffic is enabled, the continuos ping is now successful

![image](https://github.com/user-attachments/assets/f2e872a7-51b8-4b3d-bb94-ddc095d12e01)

## Install Active Directory

**Login to `DC-1` and install Active Directory Domain Services**
 **Promote as a DC**
   - Set up a new forest as `mydomain.com` (it can be anything; just remember what it is).
   - Restart and log back into `DC-1` as user: `mydomain.com\labuser`.
   - 
   ![image](https://github.com/user-attachments/assets/b4f3c448-a130-46ed-8cd9-b0d37f83ae53)

<br />

## Create Admin and Normal User Accounts in AD

 **In Active Directory Users and Computers (ADUC)**
   - Create an Organizational Unit (OU) called `_EMPLOYEES`.
   - Create a new OU named `_ADMINS`.
   - Create a new employee named `Jane Doe` (same password) with the username `jane_admin`.
   - Add `jane_admin` to the `Domain Admins` Security Group.
   - Log out/close the Remote Desktop connection to `DC-1` and log back in as `mydomain.com\jane_admin`.
   - Use `jane_admin` as your admin account from now on.

   - ![image](https://github.com/user-attachments/assets/2d7bdd7e-8c39-48f8-af7d-bf988c2055fb)


![image](https://github.com/user-attachments/assets/15c5fcf5-5074-4f06-8c08-75494e08a3b4)


![image](https://github.com/user-attachments/assets/ec32278f-0855-4e22-b587-9274d04ca470)


## Join Client-1 to Your Domain

 **From the Azure Portal**
   - Set `Client-1`’s DNS settings to the DC’s Private IP address.
   - Restart `Client-1`.

 **Login to `Client-1` (Remote Desktop) as the original local admin (labuser)**
   - Join it to the domain (computer will restart).

 **Login to the Domain Controller (Remote Desktop)**
   - Verify that `Client-1` shows up in Active Directory Users and Computers (ADUC) inside the `Computers` container on the root of the domain.
   - Create a new OU named `_CLIENTS` and drag `Client-1` into it (this step is not necessary; it is just for organizational purposes).

## Setup Remote Desktop for Non-Administrative Users on Client-1

 **Log into `Client-1` as `mydomain.com\jane_admin`**
   - Open system properties.
   - Click “Remote Desktop”.
   - Allow “domain users” access to remote desktop.

 **You can now log into `Client-1` as a normal, non-administrative user**
   - (Normally, you’d want to do this with Group Policy, which allows you to change MANY systems at once.)

## Create Additional Users

 **Login to `DC-1` as `jane_admin`**
 **Open PowerShell_ise as an administrator**
   - Create a new File and paste the contents of the script into it ([Generate-Names-Create-Users.ps1](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1)).
   - Run the script and observe the accounts being created.

   - ![image](https://github.com/user-attachments/assets/a08b614d-5bbb-48bf-8995-195712e929e9)


 **When finished, open ADUC and observe the accounts in the appropriate OU**
   - Attempt to log into `Client-1` with one of the accounts (take note of the password in the script).
