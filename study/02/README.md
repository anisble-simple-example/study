<pre>
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
★ [ copy ] 첫번째 명령어 실행했을 경우와 두번째 실행했을 때 리턴 CHANGED -> SUCCESS
★ [ copy ] idempontent 멱등법칙이 적용!!!! 

★ [ command ]  생성 확인 (상태를 보면 항상 CHABGED 즉 not idemptent
[ansible@centos100 study02]$ ansible web1 -m command -a 'cat /tmp/text.txt'
192.168.100.101 | CHANGED | rc=0 >>
TEST
[ansible@centos100 study02]$ ansible web1 -m command -a 'cat /tmp/text.txt'
192.168.100.101 | CHANGED | rc=0 >>
TEST
[ansible@centos100 study02]$ 
</pre>