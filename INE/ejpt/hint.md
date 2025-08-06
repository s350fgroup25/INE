## Website Recon & Footprinting
	Ip addresses
	Director 
	Name
	Email 
	Phone
	Physical address
	Web tech (server os)

## Tools 
	host xxx.org 
	Robots.txt
	Sitemap
	
	whatweb  xxx.org 
	Whatis  xxx.org 
	
	Website–online : Netcraft//dnsrecon//wafw00f//sublist3r
	Good Dorks
	theHarvester: find email
	Have been pwned: leaker password
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	https://github.com/08hakr/learn-/tree/main/EJPT/001%20INE-Assessment-Methodologies-Information-Gathering-Course-Files
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
### Host Discovery : ip a s
	=> nmap -sn 192.168.2.0/24
	netdiscovery -i eth0 -r 192.168.2.0/24

### dirb https://target.com
	https://target.com/robots.txt
	robots.txt/sitemap.xml/author-sitemap.xml/page-sitemap.xml/category-sitemap.xml/post-sitemap.xm/

### Extension :BuiltWith/Wappalyzer

### whois google.com 
	Domain Name: GOOGLE.COM
	 Registry Domain ID: 2138514_DOMAIN_COM-VRSN
	Registrar WHOIS Server: whois.markmonitor.com
	 Registrar URL: http://www.markmonitor.com

### netcraft.com  : 
   SSL, TLS, WhatWeb, Mail Servers, Nameserver, and as well

### DNS Recon
	A        IPv4
	AAAA     IPv6
	NS       Name Server
	MX       Mail Server
	TXT      Text Record

### Tools : 
	dnsdumpster.com
	wafw00f: is a tool that use to see is the target is behind the Firewall OR not.
	Subdomains Findings
	sublist3r -d medium.com   
	mail.elonmusk--medium.com
	webdisk.elonmusk--medium.com
	webmail.elonmusk--medium.com

## Extra tips:
	go with the version vulnerabilities.
	go for the config files of the server is there any way to access them?
	check the source code of the web server for more links.
	and save the Nmap scan of each target in notes.
	there is a tool called wp-scan check it if you find WordPress.
	nano /etc/hosts: I got a site where I was unable to access this with a URL at this moment you need to add that machine pair IP with the Target URL in the /etc/hosts file after that you can easily access that site it takes time longer than usual.
	even MySQL ID and Passwords play a good role try to read the databases and find the other user’s password hashes.

### Nmap Scripting Engine (NSE)
	WebDAV Vulnerabilities
	ls -al /etc/cron*

### 檢查 WordPress : 
	<meta name="generator" content="WordPress x.x.x" />

### 檢查 SAMBA: 
	nmap -p 139,445 <target-ip>
	smbclient -L //<target-ip>

### 檢查Drupal: 
	<meta name="generator" content="Drupal x.x" />

### 查找MySQL版本: 
	mysql -u username -p -e "SELECT VERSION();"

### DMZ（非軍事區）網路上
	=> 特定子網路（例如，192.168.1.0/24）

	在 Drupal 網站上運行的 Linux 發行版
	=> cat /etc/*release

### Syntex 提供的服務

### Drupal 網站上的管理員電子郵件: 
	SELECT name, mail FROM users WHERE uid = 1;
	目錄settings.php中sites/default是否有任何電子郵件配置

### WordPress 上活動主題的名稱
	登入 WordPress 管理面板，前往外觀 > 主題。將標記活動主題

### 匿名存取的FTP伺服器

### Drupal站點漏洞
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	運行Drupal的Linux主機上的權限提升漏洞
	
	WordPress 上的反向 Shell 漏洞
	
	
	無法透過 DMZ 存取內網主機
	網路映射：nmap使用識別主機等工具執行內部網路掃描。
	防火牆規則：檢視防火牆規則以確定哪些主機與 DMZ 隔離。
	
	
	主機轉向內部網絡
	受感染的 DMZ 主機：尋找 DMZ 中任何有權存取內部網路的受感染電腦。
	VPN 或堡壘主機：識別充當內部連接網關的主機。
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	
	「lawrence」帳戶的密碼
	hydra -l lawrence -P <path-to-password-list> -t 4 rdp://<target-ip>
	
	
	WINSERVER-037 上使用者帳號「mary」的密碼
	hydra -l mary -P <path-to-password-list> -t 4 rdp://WINSERVER-037

### 指令注入
	Drupalgeddon2 的 CVSS V3.x 評級
	
	帶有“todo.txt”的Web伺服器
	目錄暴力破解：使用 Dirb 或 Gobuster 等工具暴力破解 Web 伺服器上的目錄。
	檢查已知位置：某些應用程式可能在常見位置有「todo.txt」；嘗試訪問http://<target-ip>/todo.txt.

### WordPress 上「admin」使用者帳號的密碼
	hydra -l admin -P <path-to-password-list> -t 4 http://<wordpress-site>

	WordPress 網站上安裝的外掛程式數量
	管理儀表板：登入 WordPress 管理面板並前往外掛程式 > 安裝的外掛程式
	SELECT COUNT(*) FROM wp_plugins;
	
	WordPress檔案儲存資料庫配置
	wp-config.php檔案中

### DMZ 網路上的主機在連接埠 3307 上執行資料庫伺服器
	nmap -p 3307 <DMZ-subnet-range>
	
	託管 Drupal 網站的系統上執行的 Linux 核心版本
	uname -r
	cat /etc/*release
### 運行Drupal的伺服器上MySQL資料庫的root密碼
	hydra -l root -P <path-to-password-list> mysql://<target-ip>

### DMZ 網路中的主機運作啟用了 WebDAV 的 Web 伺服器
	nmap -p 80 --script http-webdav-scan <DMZ-subnet-range>
	
### 使用者帳號「lawrence」的網路上託管
	net user lawrence /domain
### WINSERVER-02 上存在的使用者帳戶
	PS: Get-LocalUser
	nmap -p 445 --script smb-enum-users WINSERVER-02

### WINSERVER-017 上安裝的修補程式數量
	PS : Get-HotFix | Measure-Object
	type \Users\mike\Documents\flag.txt

### nmap 
	nmap -sV -sC 
	nmap -sn 192.168.100.0/24
	nmap -p 80 192.168.1.0/24

### ftp anonymous 

	Drupla site vulnerability 
	=> searchsploit version
### be root 
	reshell 

cmdasp.aspx

settings.php
