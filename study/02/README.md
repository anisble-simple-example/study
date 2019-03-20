<pre>

★ [ copy ] 매뉴얼 보고 필수 옵션을 알기  OPTIONS (= is mandatory)
[ansible@centos100 ansible]$ ansible-doc copy
> COPY    (/usr/lib/python2.7/site-packages/ansible/modules/files/copy.py)

        The `copy' module copies a file from the local or remote machine to a location on the
        remote machine. Use the [fetch] module to copy files from remote locations to the local
        box. If you need variable interpolation in copied files, use the [template] module. For
        Windows targets, use the [win_copy] module instead.
...
= dest
        Remote absolute path where the file should be copied to. If `src' is a directory, this
        must be a directory too. If `dest' is a nonexistent path and if either `dest' ends with
        "/" or `src' is a directory, `dest' is created. If `src' and `dest' are files, the parent
        directory of `dest' isn't created: the task fails if it doesn't already exist.

★ 파일 생성 후 복사 
[ansible@centos100 study02]$ ansible web1 -m copy -a 'src=test.txt dest=/tmp/text.txt'
192.168.100.101 | CHANGED => {
    "changed": true, 
    "checksum": "4c49b08e9b258e0e5867d76ce583c159597b6857", 
    "dest": "/tmp/text.txt", 
    "gid": 1000, 
    "group": "ansible", 
    "md5sum": "2debfdcf79f03e4a65a667d21ef9de14", 
    "mode": "0664", 
    "owner": "ansible", 
    "secontext": "unconfined_u:object_r:user_home_t:s0", 
    "size": 5, 
    "src": "/home/ansible/.ansible/tmp/ansible-tmp-1553078211.23-114887080614169/source", 
    "state": "file", 
    "uid": 1000
}
[ansible@centos100 study02]$ ansible web1 -m copy -a 'src=test.txt dest=/tmp/text.txt'
192.168.100.101 | SUCCESS => {
    "changed": false, 
    "checksum": "4c49b08e9b258e0e5867d76ce583c159597b6857", 
    "dest": "/tmp/text.txt", 
    "gid": 1000, 
    "group": "ansible", 
    "mode": "0664", 
    "owner": "ansible", 
    "path": "/tmp/text.txt", 
    "secontext": "unconfined_u:object_r:user_home_t:s0", 
    "size": 5, 
    "state": "file", 
    "uid": 1000
}
★ [ copy ] 첫번째 명령어 실행했을 경우와 두번째 실행했을 때 리턴 CHANGED -> SUCCESS 즉 idempontent 멱등법칙이 적용!!!! 

★ [ command ]  생성 확인 (상태를 보면 항상 CHABGED 즉 not idemptent)
[ansible@centos100 study02]$ ansible web1 -m command -a 'cat /tmp/text.txt'
192.168.100.101 | CHANGED | rc=0 >>
TEST

★ [ copy ] 필수 옵션 =dest 대신 -content 사용 
[ansible@cent100 ~]$ ansible web1 -m copy -a 'content=test2 dest=/tmp/test2.txt'
192.168.100.101 | CHANGED => {
    "changed": true, 
    "checksum": "109f4b3c50d7b0df729d299bc6f8e9ef9066971f", 
    "dest": "/tmp/test2.txt", 
    "gid": 1000, 
    "group": "ansible", 
    "md5sum": "ad0234829205b9033196ba818f7a872b", 
    "mode": "0664", 
    "owner": "ansible", 
    "secontext": "unconfined_u:object_r:user_home_t:s0", 
    "size": 5, 
    "src": "/home/ansible/.ansible/tmp/ansible-tmp-1553090510.18-278017001186220/source", 
    "state": "file", 
    "uid": 1000
}

★ [ command ]  생성 확인 (상태를 보면 항상 CHABGED 즉 not idemptent)
[ansible@cent100 ~]$ ansible web1 -m command -a 'cat /tmp/test2.txt'
192.168.100.101 | CHANGED | rc=0 >>
test2
[ansible@cent100 ~]$ 

★ [ file ]  파일 생성 삭제
[ansible@cent100 ~]$ ansible web1 -m file -a 'path=/tmp/test3.txt'
192.168.100.101 | FAILED! => {
    "changed": false, 
    "msg": "file (/tmp/test3.txt) is absent, cannot continue", 
    "path": "/tmp/test3.txt", 
    "state": "absent"
}
[ansible@cent100 ~]$ ansible web1 -m file -a 'path=/tmp/test3.txt state=touch'
192.168.100.101 | CHANGED => {
    "changed": true, 
    "dest": "/tmp/test3.txt", 
    "gid": 1000, 
    "group": "ansible", 
    "mode": "0664", 
    "owner": "ansible", 
    "secontext": "unconfined_u:object_r:user_tmp_t:s0", 
    "size": 0, 
    "state": "file", 
    "uid": 1000
}
[ansible@cent100 ~]$ ansible web1 -m file -a 'path=/tmp/test3.txt'
192.168.100.101 | SUCCESS => {
    "changed": false, 
    "gid": 1000, 
    "group": "ansible", 
    "mode": "0664", 
    "owner": "ansible", 
    "path": "/tmp/test3.txt", 
    "secontext": "unconfined_u:object_r:user_tmp_t:s0", 
    "size": 0, 
    "state": "file", 
    "uid": 1000
}
[ansible@cent100 ~]$ ansible web1 -m file -a 'path=/tmp/test3.txt state=absent'
192.168.100.101 | CHANGED => {
    "changed": true, 
    "path": "/tmp/test3.txt", 
    "state": "absent"
}
[ansible@cent100 ~]$ ansible web1 -m file -a 'path=/tmp/test3.txt state=absent'
192.168.100.101 | SUCCESS => {
    "changed": false, 
    "path": "/tmp/test3.txt", 
    "state": "absent"
}
[ansible@cent100 ~]$ 

★ [ yum ]  package 설치, 삭제 (권한 상승이 필요)
[ansible@cent100 ~]$ ansible web1 -m yum -a 'name=git'
192.168.100.101 | FAILED! => {
    "ansible_facts": {
        "pkg_mgr": "yum"
    }, 
    "changed": false, 
    "msg": "You need to be root to perform this command.\n", 
    "rc": 1, 
    "results": [
        "Loaded plugins: fastestmirror\n"
    ]
}
[ansible@cent100 ~]$ ansible web1 -m yum -a 'name=git' -b
192.168.100.101 | CHANGED => {
    "ansible_facts": {
        "pkg_mgr": "yum"
    }, 
    "changed": true, 
    "msg": "", 
    "rc": 0, 
    "results": [
[ansible@cent100 ~]$ ansible web1 -m yum -a 'name=git state=absent' 
192.168.100.101 | FAILED! => {
    "ansible_facts": {
        "pkg_mgr": "yum"
    }, 
    "changed": false, 
    "msg": "You need to be root to perform this command.\n", 
    "rc": 1, 
    "results": [
        "Loaded plugins: fastestmirror\n"
    ]
}
[ansible@cent100 ~]$ ansible web1 -m yum -a 'name=git state=absent' -b
192.168.100.101 | CHANGED => {
    "ansible_facts": {
        "pkg_mgr": "yum"
    }, 
    "changed": true, 
    "msg": "", 
    "rc": 0, 
    "results": [    
    
★ [ setup ]  변수 설정 확인 가능    
[vagrant@controller ~]$ ansible web1 -m setup
192.168.56.101 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.0.2.15",
            "192.168.56.101"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::5054:ff:fe26:1060",
            "fe80::a00:27ff:feb2:996f"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "12/01/2006",
        "ansible_bios_version": "VirtualBox",
        "ansible_cmdline": {
            "BOOT_IMAGE": "/boot/vmlinuz-3.10.0-957.5.1.el7.x86_64",
            "LANG": "en_US.UTF-8",
            "biosdevname": "0",
            "console": "ttyS0,115200n8",
            "crashkernel": "auto",
            "elevator": "noop",
            "net.ifnames": "0",
            "no_timer_check": true,
            "ro": true,
    
</pre>

<pre>
ansible convert 절차를 생각해 볼 필요... 일단 쉘로 간 후 해야겠지?

-b --become          권한 상승
-a --args            모듈 아규먼트
-k --ask-pass        ssh 패스워드
-K --ask-become-pass sudo 패스워드
</pre>