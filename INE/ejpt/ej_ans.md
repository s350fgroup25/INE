# eJpt (updated:2025/1/31 )
## 1. 運行 SAMBA 的主機的 IP 位址是多少？
	What is the IP address of the host running SAMBA?
	nmap -sV 192.168.100.52
	139/tcp  open  netbios-ssn   Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
	ans (192.168.100.52)
## 2.DMZ 網路上有多少台主機正在執行 Windows？
	How many hosts on the DMZ network are running Windows?
	ans (4)/5
## 3.DMZ 網路上有多少主機在連接埠 80 上執行 Web 伺服器？
	How many hosts on the DMZ network are running a web server on port 80?
	4(open)/6 (filter)
## 4.DMZ 網路上有多少台主機正在執行資料庫伺服器？
	How many hosts on the DMZ network are running a database server?
	3306/tcp open  mysql         MySQL 5.5.5-10.3.34-MariaDB-0ubuntu0.20.04.1
	ans (1)
## 5. What Linux distribution is running on the host running the Drupal site?
	運行 Drupal 網站的主機上正在運行什麼 Linux 發行版？
	=> Apache/2.4.41 (Ubuntu) Server at 192.168.100.52 Port 80
	ans (Ubuntu)
## 6.What is the name of the user account that published a blog post on the Drupal site?
	在 Drupal 網站上發布部落格文章的使用者帳戶的名稱是什麼？
	
	Syntex Dynamics - What we do
	Submitted by auditor on Sun, 04/17/2022 - 18:30
	
	ans(auditor)
## 7.Drupal 網站上管理員使用者的電子郵件地址是什麼？
	ans (admin@syntex.com)
## 8.Drupal 網站上執行的 Drupal 版本是什麼？
	ans => 7.57
## 9.易受 SSH 暴力攻擊的主機的 IP 位址是多少？
	ans => 192.168.100.52
## 10.包含名為 update.txt 的檔案的 FTP 伺服器的 IP 位址是什麼？
	ans 192.168.100.52 
## 11.什麼類型的漏洞可被利用來提升您在執行 Drupal 的 Linux 主機上的權限？
	ans (Misconfigured SUDO Permissions)
## 12.可以利用什麼類型的漏洞來存取 WINSERVER-03？
	ans (SMB Brute Force)
## 13.下列哪一個 MSF 模組可用來在 WINSERVER-02 上取得提升的反向 shell？
	set RHOSTS 192.168.100.51 
	exploit/windows/http/kaseya_uploader
	(  0   Kaseya VSA v7 to v9.1)
	exploit/windows/ftp/ms09_053_ftpd_nlst
	(   0   Windows 2000 SP4 English/Italian (IIS 5.0))
	[*] Started reverse TCP handler on 192.168.100.5:1234 
	[-] Exploit failed [bad-config]: Rex::BindFailed The address is already in use or unavailable: (0.0.0.0:8080).

	exploit/windows/smb/smb_login
	 use exploit/windows/smb/smb_login
	[-] No results from search
	[-] Failed to load module: exploit/windows/smb/smb_login
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ans (exploit/windows/misc/hta_server)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
## 14.內部網路的子網路是什麼？
	192.168.1.0/24
	192.168.200.0/24
	192.168.0.0/24
	192.168.2.0/24
	
	ans (192.168.0.0/24)
## 15.Which one of the following meterpreter commands can be used to add a network route?
	ans(autoroute)
## 16.內部網路中的一台 Linux 伺服器正在運行易受攻擊的服務。易受攻擊的服務在哪個連接埠上運作？
	One of the Linux servers in the internal network is running a vulnerable service. 
	What port is the vulnerable service running on?
	ans (80) wrong , should be 3389

## 17.What is the password of the "Administrator" user on WINSERVER-03?
	WINSERVER-03 上「管理員」使用者的密碼是什麼？

	ans(swordfish)
## 18 A target system has a user account called "lawrence". What is the password for this account?
	目標系統有一個名為「lawrence」的使用者帳號。該帳號的密碼是什麼？
	ans (computadora)
## 19.WINSERVER-01 上使用者「mike」的密碼是什麼？
	ans(diamond)
## 20.Drupalgeddon2 漏洞的 CVSS V3.x 評級是多少？
	ans (9.8)
## 21.在內網 Linux 伺服器上運行的存在漏洞的 Web 應用程式的名稱是什麼？
	Webmin
	Apache Tomcat
	Jenkins
	phpMyAdmin
	wrong (idk)
## 22.哪個網頁伺服器包含名為「todo.txt」的檔案？
	ans (WINSERVER-03)
## 23.WordPress 上「admin」使用者帳號的密碼是什麼？
	What is the password for the "admin" user account on WordPress?
	ans(estrella)
## 24.什麼 WordPress 文件儲存資料庫配置？
	What WordPress file stores the database configuration?
	ans (wp-config.php)
## 25. DMZ 網路上的哪台主機在連接埠 3307 上執行資料庫伺服器？
	ans (192.168.100.50)
## 26 WINSERVER-01 上執行的是哪個版本的 WordPress？
	ans (5.9.3)
## 27 託管 Drupal 網站的系統上運行的 Linux 核心版本是什麼？
	ans (5.13.0)
## 28 除來賓帳戶外，WINSERVER-01 上有多少個使用者帳戶？
	Excluding the guest account, how many user accounts are present on WINSERVER-01?
	4
## 29.WINSERVER-02 上執行的開放 TCP 連接埠總數是多少？
	What is the total number of open TCP ports running on WINSERVER-02?
	
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
	ans (14)
## 30. 網路上的哪一台主機包含名為「lawrence」的使用者帳號？
	What host on the network contains a user account called "lawrence"?
	ans(03)
## 31. WINSERVER-02 上存在下列哪一個使用者帳號？
	Which one of the following user accounts is present on WINSERVER-02?
	ans(steven)
## 32. c系統包含檔案 C:\Users\mike\Documents\flag.txt；旗幟的價值是多少？
	=> ans (b8424c81d39f4364b8036a0ab2e245aa)
## 33. WINSERVER-01 上安裝了幾個 HotFix？
	ans(220)
## 34. 執行 Drupal 的主機上標誌 /root/flag.txt 的值是多少？
	ans (e38e8ed144da4214a9b862f29d5d6cfe) 
## 35. 什麼 Windows 實用程式可用於從遠端 Web 伺服器下載檔案？
	ans(certutil)

