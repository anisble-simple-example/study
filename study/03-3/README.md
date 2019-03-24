<pre>
    cd ~
    rm -rf tests
    git init tests
    cd tests
    git config core.sparseCheckout true
    git remote add -f origin https://github.com/anisble-simple-example/study.git
    echo "study/tests" >> .git/info/sparse-checkout
    git pull origin master
    cd ./study/tests
</pre>
    
기본적으로 동기화
- ansible-playbook [test01.yml](https://github.com/anisble-simple-example/study/blob/master/study/tests/test01.yml)
    
비동기화 (그냥 지나감)  - async, poll
- ansible-playbook [test02.yml](https://github.com/anisble-simple-example/study/blob/master/study/tests/test02.yml)
                                  
다시 동기화 - async_status, util, retries
- ansible-playbook [test03.yml](https://github.com/anisble-simple-example/study/blob/master/study/tests/test03.yml)     
    
병렬 실행 - serial
- ansible-playbook [test04.yml](https://github.com/anisble-simple-example/study/blob/master/study/tests/test04.yml)   
   