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

★ [ playbook 변수를 이용한 debug ]
[ansible@cent100 03-1]$ cat test05.yml 
---
- hosts: web1
  vars:
    print1: false
  tasks:
    - command: "echo 'print1'"
      when: print1
    - command: "echo 'print2'"
[ansible@cent100 03-1]$ ansible-playbook test05.yml 

PLAY [web1] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]

TASK [command] *********************************************************************************************************************
skipping: [192.168.100.101]

TASK [command] *********************************************************************************************************************
changed: [192.168.100.101]

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=2    changed=1    unreachable=0    failed=0  


★ [ playbook loop, when ]
[ansible@cent100 03-1]$ cat test06.yml
---
- hosts : web1, web2
  tasks:
    - debug:
        msg: "{{ item }}"
      with_items: 
        - AAA
        - BBB
        - CCC
      when: (item == "AAA") or (item == "BBB") 

[ansible@cent100 03-1]$ ansible-playbook test06.yml 

PLAY [web1, web2] ******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]
ok: [192.168.100.102]

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.101] => (item=AAA) => {
    "msg": "AAA"
}
ok: [192.168.100.101] => (item=BBB) => {
    "msg": "BBB"
}
skipping: [192.168.100.101] => (item=CCC) 
ok: [192.168.100.102] => (item=AAA) => {
    "msg": "AAA"
}
ok: [192.168.100.102] => (item=BBB) => {
    "msg": "BBB"
}
skipping: [192.168.100.102] => (item=CCC) 

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=2    changed=0    unreachable=0    failed=0   
192.168.100.102            : ok=2    changed=0    unreachable=0    failed=0   

★ [ playbook with_nested ] 이중 for문

[ansible@cent100 03-1]$ cat test07.yml
---
- hosts: web1
  tasks: 
    - debug:
        msg: "{{ item[0] }}{{ item[1] }}"
      with_nested:
        - [ 1, 2 ]
        - [ 10, 20, 30 ]
[ansible@cent100 03-1]$ ansible-playbook test07.yml

PLAY [web1] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.101] => (item=[1, 10]) => {
    "msg": "110"
}
ok: [192.168.100.101] => (item=[1, 20]) => {
    "msg": "120"
}
ok: [192.168.100.101] => (item=[1, 30]) => {
    "msg": "130"
}
ok: [192.168.100.101] => (item=[2, 10]) => {
    "msg": "210"
}
ok: [192.168.100.101] => (item=[2, 20]) => {
    "msg": "220"
}
ok: [192.168.100.101] => (item=[2, 30]) => {
    "msg": "230"
}

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=2    changed=0    unreachable=0    failed=0   

★ [ playbook with_file ] 반복문을 이용한 파일 내용 보기

[ansible@cent100 03-1]$ cat test1.txt 
test11
test12
[ansible@cent100 03-1]$ cat test2.txt 
test21
test22
[ansible@cent100 03-1]$ ansible-playbook test08.yml 

PLAY [web1] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.101] => (item=test11
test12) => {
    "msg": "test11\ntest12"
}
ok: [192.168.100.101] => (item=test21
test22) => {
    "msg": "test21\ntest22"
}

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=2    changed=0    unreachable=0    failed=0   

★ [ block ] block, rescue, always 으로 구분하고 관리함 

[ansible@cent100 03-1]$ cat test09.yml
- hosts: web1
  tasks:
    - name: block teste
      block:
        - debug: msg='step1'
        - command: /bin/false
        - debug: msg='step2'
      rescue:
        - debug: msg='step3'
        - command: /bin/false
        - debug: msg='step4'
      always:
        - debug: msg='step5'
[ansible@cent100 03-1]$ ansible-playbook test09.yml 

PLAY [web1] ************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [192.168.100.101]

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.101] => {
    "msg": "step1"
}

TASK [command] *********************************************************************************************************************
fatal: [192.168.100.101]: FAILED! => {"changed": true, "cmd": ["/bin/false"], "delta": "0:00:00.001829", "end": "2019-03-22 00:07:24.073375", "msg": "non-zero return code", "rc": 1, "start": "2019-03-22 00:07:24.071546", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.101] => {
    "msg": "step3"
}

TASK [command] *********************************************************************************************************************
fatal: [192.168.100.101]: FAILED! => {"changed": true, "cmd": ["/bin/false"], "delta": "0:00:00.001827", "end": "2019-03-22 00:07:24.405744", "msg": "non-zero return code", "rc": 1, "start": "2019-03-22 00:07:24.403917", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}

TASK [debug] ***********************************************************************************************************************
ok: [192.168.100.101] => {
    "msg": "step5"
}
        to retry, use: --limit @/home/ansible/03-1/test09.retry

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=4    changed=0    unreachable=0    failed=2   

★ [ playbook ]  파일을 생성한 이후에 파일 내용을 shell 리턴 값으로 받아 디버그로 내용을 보여줌
[ansible@centos100 study01]$ cat time.yml 
---
- name: Time Stamp
  hosts: web1
  tasks:
    - command: hostname
      register: result
    - debug:
        msg: "{{ result }}"
    - debug:
        msg: "{{ result.rc }}"
    - copy:
        content: "ABCDEF"
        dest: "/tmp/test1.txt"
    - shell: cat /tmp/test1.txt
      register: result
    - debug:
        msg: "file content is {{ result.stdout }}"
[ansible@centos100 study01]$ ansible-playbook time.yml

PLAY [Time Stamp] ********************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************
ok: [192.168.100.101]

TASK [command] ***********************************************************************************************************
changed: [192.168.100.101]

TASK [debug] *************************************************************************************************************
ok: [192.168.100.101] => {
    "msg": {
        "changed": true, 
        "cmd": [
            "hostname"
        ], 
        "delta": "0:00:00.004589", 
        "end": "2019-03-22 13:17:33.129069", 
        "failed": false, 
        "rc": 0, 
        "start": "2019-03-22 13:17:33.124480", 
        "stderr": "", 
        "stderr_lines": [], 
        "stdout": "cent101", 
        "stdout_lines": [
            "cent101"
        ]
    }
}

TASK [debug] *************************************************************************************************************
ok: [192.168.100.101] => {
    "msg": "0"
}

TASK [copy] **************************************************************************************************************
ok: [192.168.100.101]

TASK [shell] *************************************************************************************************************
changed: [192.168.100.101]

TASK [debug] *************************************************************************************************************
ok: [192.168.100.101] => {
    "msg": "file content is ABCDEF"
}

PLAY RECAP ***************************************************************************************************************
192.168.100.101            : ok=7    changed=2    unreachable=0    failed=0   


★ [ playbook ]  콜백 타입 리스트 보기
[ansible@centos100 ~]$ ansible-doc -t callback -l
actionable          shows only items that need attention                                                                       
cgroup_memory_recap Profiles maximum memory usage of tasks and full execution using cgroups                                    
context_demo        demo callback that adds play/task context                                                                  
counter_enabled     adds counters to the output items (tasks and hosts/task)                                                   
debug               formatted stdout/stderr display                                                                            
default             default Ansible screen output                                                                              
dense               minimal stdout output                                                                                      
foreman             Sends events to Foreman                                                                                    
full_skip           suppresses tasks if all hosts skipped                                                                      
grafana_annotations send ansible events as annotations on charts to grafana over http api.                                     
hipchat             post task events to hipchat                                                                                
jabber              post task events to a jabber server                                                                        
json                Ansible screen output as JSON                                                                              
junit               write playbook output to a JUnit file.                                                                     
log_plays           write playbook output to log file                                                                          
logdna              Sends playbook logs to LogDNA                                                                              
logentries          Sends events to Logentries                                                                                 
logstash            Sends events to Logstash                                                                                   
mail                Sends failure events via email                                                                             
minimal             minimal Ansible screen output                                                                              
null                Don't display stuff to screen                                                                              
oneline             oneline Ansible screen output                                                                              
osx_say             oneline Ansible screen output                                                                              
profile_roles       adds timing information to roles                                                                           
profile_tasks       adds time information to tasks                                                                             
selective           only print certain tasks                                                                                   
skippy              Ansible screen output that ignores skipped status                                                          
slack               Sends play events to a Slack channel                                                                       
splunk              Sends task result events to Splunk HTTP Event Collector                                                    
stderr              Splits output, sending failed tasks to stderr                                                              
sumologic           Sends task result events to Sumologic                                                                      
syslog_json         sends JSON events to syslog                                                                                
timer               Adds time to play stats                                                                                    
tree                Save host events to files                                                                                  
unixy               condensed Ansible output                                                                                   
yaml                yaml-ized Ansible screen output     




★ [ playbook ]  profile_tasks, time 활성화하여 걸린 시간 보기
[ansible@centos100 study01]$ cat ansible.cfg 
[defaults]
inventory          = inventory
callback_whitelist = profile_tasks, timer
[ansible@centos100 study01]$ ansible-playbook time.yml 

PLAY [Time Stamp] ******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
Friday 22 March 2019  13:44:28 +0900 (0:00:00.127)       0:00:00.127 ********** 
ok: [192.168.100.101]

TASK [command] *********************************************************************************************************************
Friday 22 March 2019  13:44:59 +0900 (0:00:31.767)       0:00:31.894 ********** 
changed: [192.168.100.101]

TASK [debug] ***********************************************************************************************************************
Friday 22 March 2019  13:45:01 +0900 (0:00:02.018)       0:00:33.913 ********** 
ok: [192.168.100.101] => {
    "msg": {
        "changed": true, 
        "cmd": [
            "hostname"
        ], 
        "delta": "0:00:00.014505", 
        "end": "2019-03-22 13:45:01.551623", 
        "failed": false, 
        "rc": 0, 
        "start": "2019-03-22 13:45:01.537118", 
        "stderr": "", 
        "stderr_lines": [], 
        "stdout": "cent101", 
        "stdout_lines": [
            "cent101"
        ]
    }
}

TASK [debug] ***********************************************************************************************************************
Friday 22 March 2019  13:45:02 +0900 (0:00:00.279)       0:00:34.192 ********** 
ok: [192.168.100.101] => {
    "msg": "0"
}

TASK [copy] ************************************************************************************************************************
Friday 22 March 2019  13:45:02 +0900 (0:00:00.259)       0:00:34.451 ********** 
ok: [192.168.100.101]

TASK [shell] ***********************************************************************************************************************
Friday 22 March 2019  13:45:05 +0900 (0:00:03.471)       0:00:37.923 ********** 
changed: [192.168.100.101]

TASK [debug] ***********************************************************************************************************************
Friday 22 March 2019  13:45:07 +0900 (0:00:01.373)       0:00:39.296 ********** 
ok: [192.168.100.101] => {
    "msg": "file content is ABCDEF"
}

PLAY RECAP *************************************************************************************************************************
192.168.100.101            : ok=7    changed=2    unreachable=0    failed=0   

Friday 22 March 2019  13:45:07 +0900 (0:00:00.186)       0:00:39.483 ********** 
=============================================================================== 
Gathering Facts ------------------------------------------------------------------------------------------------------------ 31.77s
copy ------------------------------------------------------------------------------------------------------------------------ 3.47s
command --------------------------------------------------------------------------------------------------------------------- 2.02s
shell ----------------------------------------------------------------------------------------------------------------------- 1.37s
debug ----------------------------------------------------------------------------------------------------------------------- 0.28s
debug ----------------------------------------------------------------------------------------------------------------------- 0.26s
debug ----------------------------------------------------------------------------------------------------------------------- 0.19s
Playbook run took 0 days, 0 hours, 0 minutes, 39 seconds
[ansible@centos100 study01]$ 

★ [ playbook ]  ssh_connection pipelining 설정으로 통신 빠르게!!!
[ansible@centos100 study01]$ cat ansible.cfg 
[defaults]
inventory          = inventory
callback_whitelist = profile_tasks, timer

[ssh_connection]
pipelining = true
[ansible@centos100 study01]$ ansible web1 -m ping -vvvv
ansible 2.7.8
  config file = /home/ansible/study01/ansible.cfg
  configured module search path = [u'/home/ansible/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /bin/ansible
  python version = 2.7.5 (default, Apr 11 2018, 07:36:10) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]
Using /home/ansible/study01/ansible.cfg as config file
setting up inventory plugins
/home/ansible/study01/inventory did not meet host_list requirements, check plugin documentation if this is unexpected
/home/ansible/study01/inventory did not meet script requirements, check plugin documentation if this is unexpected
Parsed /home/ansible/study01/inventory inventory source with ini plugin
Loading callback plugin minimal of type stdout, v2.0 from /usr/lib/python2.7/site-packages/ansible/plugins/callback/minimal.pyc
META: ran handlers
Using module file /usr/lib/python2.7/site-packages/ansible/modules/system/ping.py
<192.168.100.101> ESTABLISH SSH CONNECTION FOR USER: None
<192.168.100.101> SSH: EXEC ssh -vvv -C -o ControlMaster=auto -o ControlPersist=60s -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o ConnectTimeout=10 -o ControlPath=/home/ansible/.ansible/cp/be7bc5e234 192.168.100.101 '/bin/sh -c '"'"'/usr/bin/python && sleep 0'"'"''
<192.168.100.101> (0, '\n{"invocation": {"module_args": {"data": "pong"}}, "ping": "pong"}\n', 'OpenSSH_7.4p1, OpenSSL 1.0.2k-fips  26 Jan 2017\r\ndebug1: Reading configuration data /etc/ssh/ssh_config\r\ndebug1: /etc/ssh/ssh_config line 58: Applying options for *\r\ndebug1: auto-mux: Trying existing master\r\ndebug2: fd 3 setting O_NONBLOCK\r\ndebug2: mux_client_hello_exchange: master version 4\r\ndebug3: mux_client_forwards: request forwardings: 0 local, 0 remote\r\ndebug3: mux_client_request_session: entering\r\ndebug3: mux_client_request_alive: entering\r\ndebug3: mux_client_request_alive: done pid = 10029\r\ndebug3: mux_client_request_session: session request sent\r\ndebug1: mux_client_request_session: master session id: 2\r\ndebug3: mux_client_read_packet: read header failed: Broken pipe\r\ndebug2: Received exit status from master 0\r\n')
192.168.100.101 | SUCCESS => {
    "changed": false, 
    "invocation": {
        "module_args": {
            "data": "pong"
        }
    }, 
    "ping": "pong"
}
META: ran handlers
META: ran handlers
[ansible@centos100 study01]$ 

</pre>