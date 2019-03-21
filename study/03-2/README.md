<pre>
★ [ playbook when ] setup 변수에서 값을 참조하여 when 조건 구현

[ansible@cent100 03-1]$ cat test03.yml 
---
- hosts: webs
  tasks:
    - command: hostname
      when: ansible_default_ipv4.address == "192.168.100.101"
[ansible@cent100 03-1]$ ansible-playbook test03.yml

PLAY [webs] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]
ok: [192.168.100.102]

TASK [command] *********************************************************************************************************************
skipping: [192.168.100.102]
changed: [192.168.100.101]

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=2    changed=1    unreachable=0    failed=0   
192.168.100.102            : ok=1    changed=0    unreachable=0    failed=0   

★ [ playbook when ] setup 변수에서 값을 참조하여 when 리스트 조건 구현 ( AND 임 )
[ansible@cent100 03-1]$ cat test03.yml 
---
- hosts: webs
  tasks:
    - command: hostname
      when: 
        - ansible_default_ipv4.address == "192.168.100.101"
        - ansible_distribution =="CentOS"
[ansible@cent100 03-1]$ ansible-playbook test03.yml

PLAY [webs] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]
ok: [192.168.100.102]

TASK [command] *********************************************************************************************************************
skipping: [192.168.100.102]
changed: [192.168.100.101]

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=2    changed=1    unreachable=0    failed=0   
192.168.100.102            : ok=1    changed=0    unreachable=0    failed=0   

★ [ playbook ignore_errors ] 에러가 발생하더라고 그냥 고고
★ [ playbook register, when ] register 이용하여 result 에 결과를 등록하고 when 조건에 failed, succeeded, skiped

[ansible@cent100 03-1]$ cat test04.yml
---
- hosts: web1
  tasks:
    - copy:
        src: no.txt
        dest: /tmp/no.txt
      register: result
      ignore_errors: true 
    - debug:
        msg: "copy fail"
      when: result is failed

    - ping:
      register: result1
    - debug:
        msg: "ping success"
      when: result1 is succeeded
[ansible@cent100 03-1]$ ansible-playbook test04.yml

PLAY [web1] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]

TASK [copy] ************************************************************************************************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: If you are using a module and expect the file to exist on the remote, see the remote_src option
fatal: [192.168.100.101]: FAILED! => {"changed": false, "msg": "Could not find or access 'no.txt'\nSearched in:\n\t/home/ansible/03-1/files/no.txt\n\t/home/ansible/03-1/no.txt\n\t/home/ansible/03-1/files/no.txt\n\t/home/ansible/03-1/no.txt on the Ansible Controller.\nIf you are using a module and expect the file to exist on the remote, see the remote_src option"}
...ignoring

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.101] => {
    "msg": "copy fail"
}

TASK [ping] ************************************************************************************************************************
ok: [192.168.100.101]

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.101] => {
    "msg": "ping success"
}

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=5    changed=0    unreachable=0    failed=0   





</pre>