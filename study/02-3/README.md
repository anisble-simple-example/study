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



</pre>