# HackTheBox_CheatSheet

## 段階別対応
### 偵察(search) 
#### 初めの一歩
ターゲットをhostsに入れ、ポート確認とディレクトリ検索を実施
```
TARGET=<IP>
echo "$TARGET <マシン名>.htb" >> /etc/hosts
rustscan -b 1000 ${TARGET} > rustscan.txt
gobuster dir -e -u ${TARGET} --wildcard  -w /usr/share/wordlists/dirb/common.txt > gobuster.txt
```

### (足がかり)Initial Footprint
#### 実績ありポートとサービス(特別脆弱性を突いた例)
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



#### 着目すべきポート
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
 
### その他ポート
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
 
#### Webの確認ポイント

### (一般権限奪取)user level privilege
#### フォルダ/ファイルあさり
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

#### Webシェル/リバースシェル
#### Meterpreterによるexploit確認

### (特権昇格)privilege escalation
#### 利用できるツール
#### Kernel Exploit

## ツール色々
### ポート調査
#### nmap
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

#### rustscan
nmapを高速化したもの、ポート特定までやってnmapに引き渡し
```
rustscan -b 1000 10.10.10.197
```

### Web探索
#### nikto
webサービスについて、ディレクトリ探査や脆弱性を確認できる
```
nikto -h XX.XX.XX.XX
```

#### dirb
webサービスについて、ディレクトリ探査が可能、ネストも調べてくれるが遅い
```
dirb http://XX.XX.XX.XX
```

#### gobuster
dirbと同様に、ディレクトリ総当たりだが1階層のみで早い
```
gobuster dir -e -u http://10.10.10.197:8080/ --wildcard  -w /usr/share/wordlists/dirb/common.txt
```

### サービス探索
#### wpscan
wordpressの内容把握に利用可能
wordpressはそのもののバージョンから脆弱性を探したり、wp-content/plugin配下の脆弱なプラグインを探したりする
```
wpscan --url <url> --enumerate p,u
```

## 小技色々
#### SMB匿名接続
```
kali@kali:~$ rpcclient -U "" lame.htb
Enter WORKGROUP\'s password: 
Cannot connect to server.  Error was NT_STATUS_IO_TIMEOUT
```



## 知らないことリスト

## WriteUp
### HackTheBox
[Legacy](https://syachineko.hatenablog.com/entry/2020/06/21/163536)

[Lame](https://syachineko.hatenablog.com/entry/2020/06/16/231952)

[Devel](https://syachineko.hatenablog.com/entry/2020/06/27/230424)

[Cap](https://syachineko.hatenablog.com/entry/2021/10/16/202928)

