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
    
    기본적으로 동기화
    ansible-playbook test01.yml
    
    비동기화 (그냥 지나감)  - async, poll
    ansible-playbook test01.yml 

    다시 동기화 - async_status, util, retries
    ansible-playbook test03.yml     
<pre>    