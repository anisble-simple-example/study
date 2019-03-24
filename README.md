Ansible Education 20190319 ~ 20190322

WEB 서버 설치 및 소스 배포 

- [YML 파일 분리 하기](https://github.com/anisble-simple-example/study/tree/master/study/temp00)

- [ansible-galaxy 템플릿 다운받기](https://github.com/anisble-simple-example/study/tree/master/study/temp01)

- [롤 기반](https://github.com/anisble-simple-example/study/tree/master/study/temp02) 
  - webdeploy role: web1, wev2 web server 설치

- [롤 기반1](https://github.com/anisble-simple-example/study/tree/master/study/temp03) 
  - 80 포트 버전
  - webdeploy role: web1, wev2 에 web server 설치
  - haproxy role: controller 에 load balance 설치

- [롤 기반2](https://github.com/anisble-simple-example/study/tree/master/study/temp04) 
  - 8081 포트 버전
  - webdeploy role: web1, wev2 에 web server 설치
  - haproxy role: controller 에 load balance 설치
  - 80 이외의 포트로 할 경우 서비스가 기동이 안 됨 ansible 문제 아님 운영체제에서 확인 할 예정  




 - 기타 
    - Git 부분 체크 아웃
    - 테스트 자동화 스크립트
<pre>
    cd ~
    rm -rf temp03
    git init temp03
    cd temp03
    git config core.sparseCheckout true
    git remote add -f origin https://github.com/anisble-simple-example/study.git
    echo "study/temp03" >> .git/info/sparse-checkout
    git pull origin master
    cd ./study/temp03
    ansible-playbook site.yml

    cd ~
    rm -rf temp04
    git init temp04
    cd temp04
    git config core.sparseCheckout true
    git remote add -f origin https://github.com/anisble-simple-example/study.git
    echo "study/temp04" >> .git/info/sparse-checkout
    git pull origin master
    cd ./study/temp04
    ansible-playbook site.yml
    
    cd ~
    rm -rf tests
    git init tests
    cd tests
    git config core.sparseCheckout true
    git remote add -f origin https://github.com/anisble-simple-example/study.git
    echo "study/tests" >> .git/info/sparse-checkout
    git pull origin master
    cd ./study/tests
    ansible-playbook test01.yml        
</pre>


