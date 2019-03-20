<pre>
★ [ yml, yaml 언어 ](https://ko.wikipedia.org/wiki/YAML , https://onlineyamltools.com/)
 - {name: John Smith, age: 33}
 - name: Mary Smith
   age: 2

 men: [John Smith, Bill Jones]
 women:
   - Mary Smith
   - Susan Williams

★ [ playbook basic ]
[ansible@cent100 test1]$ cat play1.ymal
- hosts: web1
  tasks:
    - ping:
        data: power

[ansible@cent100 test1]$ ansible-playbook play1.ymal --syntax-check

playbook: play1.ymal
[ansible@cent100 test1]$ ansible-playbook play1.ymal 

PLAY [web1] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]

TASK [ping] ************************************************************************************************************************
ok: [192.168.100.101]

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=2    changed=0    unreachable=0    failed=0   

[ansible@cent100 test1]$ 

★ [ playbook ] 실패 경우 play1.retry  생성
[ansible@cent100 test1]$ cat play1.ymal 
- hosts: web1
  tasks:
    - ping:
        data: power
    - yum:
        name: vsftpd
[ansible@cent100 test1]$ ansible-playbook play1.ymal --syntax-check

playbook: play1.ymal
[ansible@cent100 test1]$ ansible-playbook play1.ymal 

PLAY [web1] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]

TASK [ping] ************************************************************************************************************************
ok: [192.168.100.101]

TASK [yum] *************************************************************************************************************************
fatal: [192.168.100.101]: FAILED! => {"changed": false, "msg": "You need to be root to perform this command.\n", "rc": 1, "results": ["Loaded plugins: fastestmirror\n"]}
        to retry, use: --limit @/home/ansible/test1/play1.retry

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=2    changed=0    unreachable=0    failed=1 

[ansible@cent100 test1]$ ls -al
합계 8
drwxrwxr-x. 2 ansible ansible  62  3월 21 01:16 .
drwx------. 6 ansible ansible 136  3월 21 00:36 ..
-rw-rw-r--. 1 ansible ansible   0  3월 21 00:36 ansible.cfg
-rw-rw-r--. 1 ansible ansible  16  3월 21 01:16 play1.retry
-rw-rw-r--. 1 ansible ansible  86  3월 21 01:16 play1.ymal
[ansible@cent100 test1]$ cat play1.retry 
192.168.100.101
[ansible@cent100 test1]$ 

★ [ playbook ] -b 혹은 become: true 
[ansible@cent100 test1]$ ansible-playbook play1.ymal -b

PLAY [web1] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]

TASK [ping] ************************************************************************************************************************
ok: [192.168.100.101]

TASK [yum] *************************************************************************************************************************
changed: [192.168.100.101]

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=3    changed=1    unreachable=0    failed=0 


[ansible@cent100 test1]$cat play1.ymal 

- hosts: web1
  tasks:
    - ping:
        data: power
    - yum:
        name: vsftpd
        state: absent
      become: yes

[ansible@cent100 test1]$ ansible-playbook play1.ymal

PLAY [web1] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]

TASK [ping] ************************************************************************************************************************
ok: [192.168.100.101]

TASK [yum] *************************************************************************************************************************
ok: [192.168.100.101]

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=3    changed=0    unreachable=0    failed=0 

★ [ playbook ] hosts 에 두개 정의 
[ansible@cent100 test1]$ cat play1.ymal 
- hosts: web1, web2
  tasks:
    - ping:
        data: power
    - yum:
        name: vsftpd
      become: yes
[ansible@cent100 test1]$ ansible-playbook play1.ymal

PLAY [web1, web2] ******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]
ok: [192.168.100.102]

TASK [ping] ************************************************************************************************************************
ok: [192.168.100.102]
ok: [192.168.100.101]

TASK [yum] *************************************************************************************************************************
changed: [192.168.100.101]
changed: [192.168.100.102]

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=3    changed=1    unreachable=0    failed=0   
192.168.100.102            : ok=3    changed=1    unreachable=0    failed=0  

★ [ playbook ] web 설치, index 파일 수정, httpd 수행
[ansible@cent100 test1]$ cat play2.yml
- name: SETUP WEB
  hosts: web1, web2
  tasks:
    - name: WEB INSTALL
      become: yes
      yum:
        name: httpd
    - name: MAKE index.html 
      become: yes
      copy: 
        content: hello
        dest: /var/www/html/index.html
    - name: RUN WEB
      become: yes
      service:
        name: httpd
        state: started
  
[ansible@cent100 test1]$ ansible-playbook play2.yml --syntax-check

playbook: play2.yml
[ansible@cent100 test1]$ ansible-playbook play2.yml 

PLAY [SETUP WEB] *******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]
ok: [192.168.100.102]

TASK [WEB INSTALL] *****************************************************************************************************************
ok: [192.168.100.102]
ok: [192.168.100.101]

TASK [MAKE index.html] *************************************************************************************************************
ok: [192.168.100.102]
ok: [192.168.100.101]

TASK [RUN WEB] *********************************************************************************************************************
changed: [192.168.100.102]
changed: [192.168.100.101]

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=4    changed=1    unreachable=0    failed=0   
192.168.100.102            : ok=4    changed=1    unreachable=0    failed=0   


★ [ shell, command, raw ] 멱등성 없으며 인베디스 리눅스를 사용하는 네트워크 장비는 raw를 이용
===============================
 module   shell   python
---------------------------------------
command   X        O
shell     O        O
raw       O        X    (순수 ssh를 사용하는 것)
===============================
[vagrant@controller playbook]$ ansible web1 -m command -a 'ps -ef|grep httpd'
192.168.56.101 | FAILED | rc=1 >>
error: unsupported SysV option

Usage:
 ps [options]

 Try 'ps --help <simple|list|output|threads|misc|all>'
  or 'ps --help <s|l|o|t|m|a>'
 for additional help text.

For more details see ps(1).non-zero return code

[vagrant@controller playbook]$ ansible web1 -m shell -a 'ps -ef|grep httpd'
192.168.56.101 | CHANGED | rc=0 >>
root     17943     1  0 06:29 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   17944 17943  0 06:29 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   17945 17943  0 06:29 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   17946 17943  0 06:29 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   17947 17943  0 06:29 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   17948 17943  0 06:29 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
vagrant  22871 22870  0 06:53 pts/1    00:00:00 /bin/sh -c ps -ef|grep httpd
vagrant  22873 22871  0 06:53 pts/1    00:00:00 grep httpd

[vagrant@controller playbook]$ ansible web1 -m raw -a 'ps -ef|grep httpd'
192.168.56.101 | CHANGED | rc=0 >>
root     17943     1  0 06:29 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   17944 17943  0 06:29 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   17945 17943  0 06:29 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   17946 17943  0 06:29 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   17947 17943  0 06:29 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache   17948 17943  0 06:29 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
vagrant  22944 22820  0 06:53 pts/1    00:00:00 bash -c ps -ef|grep httpd
vagrant  22958 22944  0 06:53 pts/1    00:00:00 grep httpd
Shared connection to 192.168.56.101 closed.

</pre>