ref: https://wiki.alpinelinux.org/wiki/Setting_up_a_new_user
version: alpine

# adduser

## Overview
root 말고 다른 유저를 생성해보자. 새로운 유저는 고유의 $HOME 디렉토리를 가질 수 있고, os configuration file에 대한 access 권한 제어를 할 수 있다.  

## Creating a new user
다음 커맨드로 `adduser [-g "<Full Name>"] <username>` 유저를 생성할 수 있다.  
  
이 커맨드를 입력하면 default로 다음과 같은 일이 일어난다.  
* set password prompt
* `/home/<username> 디렉토리 생성
* root가 사용하던 shell로 설정
* user ID, group ID을 할당 (1000부터 시작)
* `Linux User,,," 에 GECOS(fullname) 필드 설정 
  
> Tip: Optional 인 `-g "<Full Name>"` 설정에 의해서 GECOS 필드가 설정된다
이로 인해서 유저를 특정하기에 아주 용이해진다. 그룹을 최소한 유저네임과 동일하게라도 설정하는 것이 로그인 화면에서 유저가 리스트 되었을 때 같은 상황에서 사용자를 구별할 수 있게 해줄 것이다.  
  
