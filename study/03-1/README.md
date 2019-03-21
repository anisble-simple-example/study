<pre>

★ [ ansible ] 구조 파악 필수 
[ansible@cent100 03-1]$ ansible --version
ansible 2.7.8
  config file = /home/ansible/03-1/ansible.cfg
  configured module search path = [u'/home/ansible/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /bin/ansible
  python version = 2.7.5 (default, Apr 11 2018, 07:36:10) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]
[ansible@cent100 03-1]$ cat ansible.cfg 
[defaults]
inventory=inventory
[ansible@cent100 03-1]$ cat inventory 
[controller]
192.168.100.100

[web1]
192.168.100.101

[web2]
192.168.100.102

[webs]
192.168.100.101
192.168.100.102
[ansible@cent100 03-1]$ ansible-inventory --graph
@all:
  |--@controller:
  |  |--192.168.100.100
  |--@ungrouped:
  |--@web1:
  |  |--192.168.100.101
  |--@web2:
  |  |--192.168.100.102
  |--@webs:
  |  |--192.168.100.101
  |  |--192.168.100.102
[ansible@cent100 03-1]$ 

★ [ playbook - 변수 사용법 ]  -e >  vars_files > vars 위치에 상관 없이 우선순위 기억!!! 개 중요
[ansible@cent100 03-1]$ cat test01.yml 
---
- hosts: controller
  vars: 
    a: A1
    b:
      b1: B1
  tasks:
    - debug:
         msg: "{{a}}{{b.b1}}"
[ansible@cent100 03-1]$ 

[ansible@cent100 03-1]$ ansible-playbook test01.yml 

PLAY [controller] ******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.100]

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.100] => {
    "msg": "A1B1"
}

PLAY RECAP *************************************************************************************************************************
192.168.100.100            : ok=2    changed=0    unreachable=0    failed=0   

[ansible@cent100 03-1]$ ansible-playbook test01.yml -e "a=B1"

PLAY [controller] ******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.100]

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.100] => {
    "msg": "B1B1"
}

PLAY RECAP *************************************************************************************************************************
192.168.100.100            : ok=2    changed=0    unreachable=0    failed=0   

[ansible@cent100 03-1]$ cat test01.yml 
---
- hosts: controller
  vars: 
    a: A1
    b:
      b1: B1
  vars_files:
    - web
  tasks:
    - debug:
         msg: "{{a}}{{b.b1}}"
[ansible@cent100 03-1]$ echo "a: AAA1" > web
[ansible@cent100 03-1]$ ansible-playbook test01.yml 

PLAY [controller] ******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.100]

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.100] => {
    "msg": "AAA1B1"
}

PLAY RECAP *************************************************************************************************************************
192.168.100.100            : ok=2    changed=0    unreachable=0    failed=0   

[ansible@cent100 03-1]$ cat test01.yml 
---
- hosts: controller
  vars_files:
    - web
  vars: 
    a: A1
    b:
      b1: B1
  tasks:
    - debug:
         msg: "{{a}}{{b.b1}}"
[ansible@cent100 03-1]$ ansible-playbook test01.yml 

PLAY [controller] ******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.100]

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.100] => {
    "msg": "AAA1B1"
}

PLAY RECAP *************************************************************************************************************************
192.168.100.100            : ok=2    changed=0    unreachable=0    failed=0   

[ansible@cent100 03-1]$ ansible-playbook test01.yml -e "a=1818"

PLAY [controller] ******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.100]

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.100] => {
    "msg": "1818B1"
}

PLAY RECAP *************************************************************************************************************************
192.168.100.100            : ok=2    changed=0    unreachable=0    failed=0   


★ [ playbook template ] 사용법
[ansible@cent100 03-1]$ cat source.txt 
{{ message }}
[ansible@cent100 03-1]$ cat test02.yml 
---
- hosts: web1
  vars:
    message: hello world
  tasks:
    - template:
        src: source.txt
        dest: /tmp/source.txt
[ansible@cent100 03-1]$ ansible-playbook test02.yml 

PLAY [web1] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]

TASK [template] ********************************************************************************************************************
changed: [192.168.100.101]

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=2    changed=1    unreachable=0    failed=0   

[ansible@cent100 03-1]$ ansible web1 -m command -a "cat /tmp/source.txt"
192.168.100.101 | CHANGED | rc=0 >>
hello world

[ansible@cent100 03-1]$ cat web
a: AAA1
message: hello var files
[ansible@cent100 03-1]$ cat test02.yml
---
- hosts: web1
  vars_files:
    - web
  vars:
    message: hello world
  tasks:
    - template:
        src: source.txt
        dest: /tmp/source.txt
[ansible@cent100 03-1]$ ansible-playbook test02.yml 

PLAY [web1] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]

TASK [template] ********************************************************************************************************************
ok: [192.168.100.101]

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=2    changed=0    unreachable=0    failed=0   

[ansible@cent100 03-1]$ ansible web1 -m command -a "cat /tmp/source.txt"
192.168.100.101 | CHANGED | rc=0 >>
hello var files
[ansible@cent100 03-1]$ 

</pre>