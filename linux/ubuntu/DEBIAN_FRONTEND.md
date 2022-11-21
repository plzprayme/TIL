ref: https://www.cyberciti.biz/faq/explain-debian_frontend-apt-get-variable-for-ubuntu-debian/

# DEBIAN FRONTEND
ubuntu 를 비롯한 debian cloned linux는 Debian package configuration system은 debconf를 호출한다.  
대표적으로 pakcage install 할 때 debconf를 참조하게 된다.  

apt command 에 대한 variable 이 여러개 있는데 그 중 `noninteractive` 를 사용하면 package intall, upgrade을 zero interaction 모드로 수행을 가능하다.  
모든 질문에 대한 default answer이 입력되고 totally slient하기 때문에 완벽하게 automatic install이 가능하다.  
Dockerfile, shell script, cloud-init script 등에 사용하기가 좋다.
