anisble is Configuration Management
<pre>
0. 리눅스 centos ip 설정

-bash-4.2# cat /etc/sysconfig/network-scripts/ifcfg-ens33

UUID=55c42a8d-2429-4421-ac72-25d69e9f10e4
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.100.100
NETMASK=255.255.255.0
GATEWAY=192.168.100.2
DNS1=8.8.8.8

VM 복사해서 사용할때 mac 충돌시
1. Select the virtual machine and select VM > Settings.
2. On the Hardware tab, select the virtual network adapter and click Advanced.
3. Type a new MAC address in the MAC Address text box, or click Generate to have Workstation generate a new address.
4. Click OK to save your changes.


-bash-4.2# service network restart
Restarting network (via systemctl):  [  OK  ]
-bash-4.2# 


1. ansible 테스트 환경 구축

A 192.168.100.100 Controller
B 192.168.100.101 host1
C 192.168.100.102 host2


A,B,C 공통 작업
- 호스트 설정
$ hostnamectl set-hostname cent100
$ hostnamectl set-hostname cent101
$ hostnamectl set-hostname cent102

- ansible 계정 추가
$ useradd -m ansible

- ansible 계정에 패스워드 없이 sudo 권한 주기
$ chmod u+w /etc/sudoers
$ vi /etc/sudoers
ansible ALL=(ALL) NOPASSWD: ALL
$ chmod u-w /etc/sudoers

- ansible 계정에서  python 설치 (http://snowdeer.github.io/python/2018/02/20/install-python3-on-centos/)
$ su - ansible
$ sudo yum install -y https://centos7.iuscommunity.org/ius-release.rpm
$ sudo yum install -y python36u python36u-libs python36u-devel python36u-pip
$ python -V

- SSH 패스워드 인증방식으로 변경
$ sudo vi /etc/ssh/sshd_config
# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no
PasswordAuthentication yes

# Change to no to disable s/key passwords
#ChallengeResponseAuthentication yes
ChallengeResponseAuthentication yes

$ sudo systemctl restart sshd


A 컨트롤러에 ansible 설치
$ sudo yum install epel-release
$ sudo yum -y install ansible
$ ansible-doc -l |wc -l
2080


[ansible@centos100 ansible]$ pwd
/etc/ansible
[ansible@centos100 ansible]$ sudo cat hosts
[controller]
192.168.100.100

[web1]
192.168.100.101

[web2]
192.168.100.102
[ansible@centos100 ansible]$ ansible-inventory --graph
@all:
  |--@controller:
  |  |--192.168.100.100
  |--@ungrouped:
  |--@web1:
  |  |--192.168.100.101
  |--@web2:
  |  |--192.168.100.102
[ansible@centos100 ansible]$ 


[ansible@cent100 ansible]$ ansible web1 -m ping
The authenticity of host '192.168.100.101 (192.168.100.101)' can't be established.
ECDSA key fingerprint is SHA256:PDtiKpK4YYANN77vR2y9Mts64eGxO7bGeiGMyTsfy0A.
ECDSA key fingerprint is MD5:89:21:d6:bb:e7:eb:d8:44:f0:8c:55:29:b1:a1:06:f9.
Are you sure you want to continue connecting (yes/no)? yes
192.168.100.101 | UNREACHABLE! => {
    "changed": false, 
    "msg": "Failed to connect to the host via ssh: Warning: Permanently added '192.168.100.101' (ECDSA) to the list of known hosts.\r\nPermission denied (publickey,gssapi-keyex,gssapi-with-mic,password,keyboard-interactive).", 
    "unreachable": true
}
[ansible@cent100 ansible]$ ansible web1 -m ping -k
SSH password: 
192.168.100.101 | UNREACHABLE! => {
    "changed": false, 
    "msg": "Invalid/incorrect password: ", 
    "unreachable": true
}

-m 모듈
-k 소문자: 계정 비밀번호
-K 대문자: SUDO 패스워드 입력

[ansible@centos100 ansible]$ sudo passwd ansible
ansible 사용자의 비밀 번호 변경 중
새  암호:
잘못된 암호: 암호는 8 개의 문자 보다 짧습니다
새  암호 재입력:
passwd: 모든 인증 토큰이 성공적으로 업데이트



[ansible@cent100 ansible]$ 
[ansible@cent100 ansible]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ansible/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/ansible/.ssh/id_rsa.
Your public key has been saved in /home/ansible/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:ERQhIcSRv6wsYLBPNujdIXasUQnoKAYx8mcLtjfIqAw ansible@cent100
The key's randomart image is:
+---[RSA 2048]----+
|+..o+ooo=o       |
|o+ .o. . .       |
|+ + +.. .        |
|+* * +.  .       |
|Eo+ *. .S        |
|B.+= =o          |
|+*oo*..          |
| .ooo.           |
|   .             |
+----[SHA256]-----+
[ansible@cent100 ansible]$ ssh-copy-id ansible@192.168.100.100
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/ansible/.ssh/id_rsa.pub"
The authenticity of host '192.168.100.100 (192.168.100.100)' can't be established.
ECDSA key fingerprint is SHA256:PDtiKpK4YYANN77vR2y9Mts64eGxO7bGeiGMyTsfy0A.
ECDSA key fingerprint is MD5:89:21:d6:bb:e7:eb:d8:44:f0:8c:55:29:b1:a1:06:f9.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'ansible@192.168.100.100'"
and check to make sure that only the key(s) you wanted were added.

[ansible@cent100 ansible]$ ssh-copy-id ansible@192.168.100.101
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/ansible/.ssh/id_rsa.pub"
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'ansible@192.168.100.101'"
and check to make sure that only the key(s) you wanted were added.

[ansible@cent100 ansible]$ 
[ansible@cent100 ansible]$ ssh-copy-id ansible@192.168.100.102
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/ansible/.ssh/id_rsa.pub"
The authenticity of host '192.168.100.102 (192.168.100.102)' can't be established.
ECDSA key fingerprint is SHA256:PDtiKpK4YYANN77vR2y9Mts64eGxO7bGeiGMyTsfy0A.
ECDSA key fingerprint is MD5:89:21:d6:bb:e7:eb:d8:44:f0:8c:55:29:b1:a1:06:f9.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
ansible@192.168.100.102's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'ansible@192.168.100.102'"
and check to make sure that only the key(s) you wanted were added.

[ansible@cent100 ansible]$ ansible all -m command -a 'hostname'
192.168.100.101 | CHANGED | rc=0 >>
cent101
192.168.100.102 | CHANGED | rc=0 >>
cent102
192.168.100.100 | CHANGED | rc=0 >>
cent100
[ansible@cent100 ansible]$ 

[ansible@cent100 ansible]$ ansible web1 -m command -a 'cat /etc/shadow'
192.168.100.101 | FAILED | rc=1 >>
cat: /etc/shadow: 허가 거부non-zero return code
[ansible@cent100 ansible]$ ansible web1 -m command -a 'cat /etc/shadow' -b
192.168.100.101 | CHANGED | rc=0 >>
root:$6$5M4uj/JMm/yAujTh$et43Fkr9Zg2yZ2UBIAVJ/ymhb3UI5VaHaHWJx1pvEwptlZqsrlQyNwgkig8r4n56iWOwoPNz7m8oBsi3mIGqZ0::0:99999:7:::
bin:*:17632:0:99999:7:::
daemon:*:17632:0:99999:7:::
adm:*:17632:0:99999:7:::
lp:*:17632:0:99999:7:::
sync:*:17632:0:99999:7:::
shutdown:*:17632:0:99999:7:::
halt:*:17632:0:99999:7:::
mail:*:17632:0:99999:7:::
operator:*:17632:0:99999:7:::
games:*:17632:0:99999:7:::
ftp:*:17632:0:99999:7:::
nobody:*:17632:0:99999:7:::
systemd-network:!!:17680::::::
dbus:!!:17680::::::
polkitd:!!:17680::::::
sshd:!!:17680::::::
postfix:!!:17680::::::
chrony:!!:17680::::::
dockerroot:!!:17680::::::
ansible:$6$lmpt4Q0I$CiS5mwwoAkbohvRiWyuvGMOozJLoN68n6HaHswdm6ywehPZLn8opGZnWlNhFJvZgjv9Q4qbnnAV.8.5vLeEPh1:17974:0:99999:7:::
[ansible@cent100 ansible]$ 

-b 는 권한 상승 become 즉 sudo

</pre>