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
  - ansible 버그 발견 hosts 에서 그룹으로 사용하게 되면 비정상 동작하는 함
  hosts: web-server => hosts: web1, web2 이런식으로 사용하는 것을 권장 
  
- [롤 기반2](https://github.com/anisble-simple-example/study/tree/master/study/temp04) 
  - 8081 포트 버전
  - webdeploy role: web1, wev2 에 web server 설치
  - haproxy role: controller 에 load balance 설치
  - 80 이외의 포트로 할 경우 서비스가 기동이 안 됨 ansible 문제 아님 
  운영체제에서 확인 할 예정  


3. serial 정리 
4. 3-3일차 마저 정리
5. 기존의 정리했던것 보기 좋게 리뉴얼 



