ref: baeldung.com/linux/envsubst-command

# envsubst

## Overview
template, configuration, initialization file 들에 정의되어 있는 bash variable placeholders를 replace 해준다.  

## Basic Features
envsubst 커맨드는 `$VARIABLE` or `${ARIABLE}` 패턴을 찾아서 그에 맞는 bash variable 값으로 교체한다.  
단, envbust는 exported variable 만 인식한다.  

`envsubst [OPTION] [SHELL-FORMAT]`

## Usage
```bash
echo "Hello user $USER in $DESKTOP_SESSION. It;s time to say $HELLO!" > welcom.txt
$ export HELLO="good morning"
$ envsubst < welcome.txt

output:
Heelo user joe in Lubuntu. It's time to say good morning!
```
---
```bash
$ unset HELLO
$ envsubst < welcom.txt
Hello user joe in Lubuntu. It's time to say !
```

## Substituting All Env Var
```bash
$ envsubst "$(printf '${%s} ' $(env | cut -d'=' -f1))" < welcome.txt
Hello user joe in Lubuntu. It's time to say $HELLO!
```

