<pre>
★ [ handler ] 사용법
[ansible@cent100 test1]$ cat play4.yml 
- name: Web Source Deployment 
  hosts: web1, web2
  become: yes
  tasks:
    - name: web install
      yum:
        name: httpd
    - name: copy
      copy:
        content: hello1
        dest: /var/www/html/test.html
      notify:
        - "job copy after"
    - name: web run
      service:
        name: httpd
        state: started
  handlers:
    - name: "job copy after"
      service:
        name: httpd
        state: restarted

       
 
[ansible@cent100 test1]$ ansible-playbook play4.yml

PLAY [Web Source Deployment] *******************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.102]
ok: [192.168.100.101]

TASK [web install] *****************************************************************************************************************
ok: [192.168.100.102]
ok: [192.168.100.101]

TASK [copy] ************************************************************************************************************************
changed: [192.168.100.102]
changed: [192.168.100.101]

TASK [web run] *********************************************************************************************************************
ok: [192.168.100.102]
ok: [192.168.100.101]

RUNNING HANDLER [job copy after] ***************************************************************************************************
changed: [192.168.100.102]
changed: [192.168.100.101]

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=5    changed=2    unreachable=0    failed=0   
192.168.100.102            : ok=5    changed=2    unreachable=0    failed=0   

[ansible@cent100 test1]$ 


★ [ 독립적 공간 ] 사용법
$ cat ansible.cfg 
[defaults]
inventory=inventory

$ cat inventory 
[psc1]
192.168.100.100

$ ansible all --list-hosts
  hosts (1):
    192.168.100.100

$ ansible-inventory --graph
@all:
  |--@psc1:
  |  |--192.168.100.101
  |--@ungrouped:
[ansible@cent100 03-1]$ 
</pre>