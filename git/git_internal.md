# 10.1 Git의 내부 - Plumbing 명령과 Porcelain 명령
git은 content-addressable file system이다.

## Plumbing 명령과 Porcelain 명령
저수준의 명령어를 Plumbing이라고 부른다. Porcelain은 사용자에게 친숙한 고수준의 명령어를 말한다.  
Plumbing 명령어를 사용하면 새로운 도구를 만들거나 각자가 특별하게 필요한 스크립트를 만들기 위해서 사용한다.  
`git init` 을 입력하면 `.git` 디렉토리 아래에 아래와 같은 파일들이 생성된다.   
```
config
description
HEAD
hooks/
info/
objects/
refs/
```

`description` 파일은 GitWeb에서만 사용된다.  
`config` 에서는 설정 옵션이 들어있다.  
`info` 디렉토리에는 `.gitignore` 파일처럼 무시할 파일의 패턴을 적어두는 곳이다. `.gitignore` 와는 다르게 Git으로 관리되지 않는다.  
`hooks` 디렉토리에는 서버 훅 혹은 클라이언트 훅이 위치한다.  
`objects` 디렉토리는 모든 컨텐트를 저장하는 데이터베이스이다.  
`refs` 디렉토리에는 커밋 개체의 포인터(브랜치, 태그 리모트 등)을 저장한다.  
`HEAD` 파일은 현재 checkout한 브랜치를 가리한다.  
`index` 파일은 Staging Area의 정보를 저장한다.  

# 10.2 Git의 내부 - Git 개체
Git은 Content-addressable file System이다. 다른말로 하면 Key-Value 형태의 데이터 저장소이다.  
어떤 형식의 데이터라도 저장할 수 있고 Key로 조회할 수 있다.  
`git hash-object` 명령어에 저장하고 싶은 컨텐츠를 전달하면 `.git/objects` 아래에 저장된다.  

## Blobk
`git hash-object` 명령어가 출력한 해시값을 `git cat-file -p {hash}` 명령어와 함께 사용하면 컨텐츠를 출력할 수 있다.  
git이 저장하는 데이터의 타입은 blob, tree가 있다. 데이터 타입을 확인하려면 `git cat-file -t {hash}` 명령어를 사용하면 된다.  

## Tree
`tree` 타입을 저장해보자. Git은 Staging Area(index) 상태대로 Tree 객체를 만들고 기록한다. 그래서 Tree 개체를 만드려면 우선 Staing Area에 Index를 추가해야한다.  
Plumbing 명령어인 `update-index` 명령어로 `test.txt` 파일이 들어있는 index를 만들자. `git update-index --add --cacheinfo 10064 {hash} text.txt` 
인덱스를 만들었으면 저장하자. `git write-tree` 그 후 잘 저장되었는지 확인해보자 `git cat-file -p {hash}`. 

## Commit
`git commit-tree` 명령어로 커밋도 만들 수 있다.  

#10.3 Git의 내부 - Git Refs

## Refs
Git의 SHA-1 값을 기억하기 어려우니까 Git Refs를 사용해야한다. 이 값은 `.git/refs` 에 저장된다.   
`git update-ref refs/heads/master 1a410efbd13591db07496601ebc7a059dd55cfe9` 와 같은 명령어를 사용하면 branch를 사용하는 것과 같다.  
`git branch <branch>` 명령어를 사용하면 내부적으로는 `git update-ref` 명령어를 사용한다.  
그리고 `git branch` 명령어는 현재 브랜치를 알아내기 위해서 `HEAD` 파일을 참조한다. `HEAD` 파일은 심볼릭 링크이다.  
`git symbolic-ref` 명령어를 사용해서 수정할 수 있다. `git symbolic-ref HEAD refs/heads/test` 

## Tag
blob, tree, commit 개체 외에도 tag 개체가 있다. commit 은 Tree를 가리키고 tag는 commit 을 가리킨다.  
tag 개체는 참조를 옮길 수 없다. lightweight 태그와 annotated 태그가 있는데, lightweight은 commit 개체를 가리키고 annotated는 모든 Git 개체를 가리킬 수 있다.  

## Remote
리모트 Refs도 있는데 `refs/remote` 디렉토리에 저장된다.

# 10.4 Git의 내부 - Packfile
git은 파일을 저장할 때 zlib 으로 파일을 압축해서 저장한다. Git이 처음 개체를 저장하는 형식은 `Loose` 포멧이라고 부른다. 이 개체를 하나의 파일로 압축 할 수 있다. 
Remote 서버로 Push 할 때 `git gc` 명령어로 압축된다. `git verify-pack` 명령어를 사용하면 얼마나 많은 공간이 절약되었는지 알 수 있다.  
