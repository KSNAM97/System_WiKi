# Shell Script 실행 방법 (Script Execution Methods)

같은 스크립트 파일이라도 어떤 방식으로 실행하느냐에 따라 **새 프로세스(서브쉘)에서 도는지, 현재 쉘에서 그대로 도는지**, **Shebang을 참고하는지 무시하는지**가 달라진다. (Shebang 자체 문법은 `08-syntax-master.md`의 "Shebang과 실행 방식" 참고)

## 실행 권한으로 직접 실행 (./script.sh)

- `./script.sh`처럼 파일 경로를 직접 실행하면, 커널이 파일 맨 첫 줄의 **Shebang**을 읽어 그 인터프리터로 스크립트를 실행한다.
- 이 방식은 파일에 **실행 권한(x)**이 반드시 있어야 하며, 실행 권한이 없으면 Shebang이 있어도 거부된다.

```bash
[root@Server-A ~]# vi ./script/run_test.sh
#!/bin/bash
echo "실행 방식 : $0"

:wq


[root@Server-A ~]# ./script/run_test.sh
-bash: ./script/run_test.sh: 허가 거부됨		# 실행 권한이 없어서 거부


[root@Server-A ~]# chmod +x ./script/run_test.sh

[root@Server-A ~]# ./script/run_test.sh
실행 방식 : ./script/run_test.sh
```

**정리**: `./script.sh` 실행은 Shebang과 실행 권한(`chmod +x`)이 모두 갖춰져야 하며, Shebang에 지정된 인터프리터가 새 프로세스를 만들어 스크립트를 실행한다.

## bash · sh 명령으로 실행

- `bash script.sh` 또는 `sh script.sh`처럼 인터프리터를 직접 지정해서 실행하면, **실행 권한이 없어도** 스크립트를 실행할 수 있다.
- 이때는 Shebang을 아예 읽지 않는다. 파일 첫 줄에 `#!/bin/bash`가 적혀 있어도 무시하고, 명령어로 지정한 `bash` 또는 `sh`가 그대로 사용된다.
- `bash`/`sh` 둘 다 **새로운 서브쉘(자식 프로세스)**을 만들어 그 안에서 스크립트를 실행한다.

```bash
[root@Server-A ~]# ls -l ./script/run_test.sh
-rw-r--r-- 1 root root 32  8월 20 10:00 ./script/run_test.sh	# 실행 권한 없음


[root@Server-A ~]# bash ./script/run_test.sh
실행 방식 : ./script/run_test.sh


[root@Server-A ~]# sh ./script/run_test.sh	# Shebang이 #!/bin/bash여도 sh로 강제 실행됨
실행 방식 : ./script/run_test.sh
```

- 서브쉘에서 실행되므로, 스크립트 안에서 `cd`로 디렉터리를 옮기거나 변수를 만들어도 **부모 쉘(현재 터미널)에는 영향을 주지 않는다.**

```bash
[root@Server-A ~]# vi ./script/cd_test.sh
#!/bin/bash
cd /tmp
myvar="hello"

:wq


[root@Server-A ~]# pwd
/root

[root@Server-A ~]# bash ./script/cd_test.sh

[root@Server-A ~]# pwd			# 서브쉘 안에서만 이동했으므로 현재 쉘 위치는 그대로
/root

[root@Server-A ~]# echo $myvar		# 서브쉘의 지역 변수도 부모 쉘로 전달되지 않는다.
(출력 없음)
```

**정리**: `bash script.sh`/`sh script.sh`는 실행 권한 없이도 동작하지만 Shebang을 무시하고 지정한 인터프리터로 강제 실행하며, 항상 새로운 서브쉘에서 돌기 때문에 `cd`나 변수 변경이 현재 쉘에 반영되지 않는다.

## source · . 으로 실행 (현재 쉘에서 실행)

- `source script.sh` 또는 `. script.sh`(점 하나 + 공백 + 경로)는 **새로운 프로세스를 만들지 않고, 현재 실행 중인 쉘 안에서** 스크립트의 명령을 한 줄씩 그대로 실행한다.
- 그래서 스크립트 안에서 `cd`로 이동하거나 변수를 만들면, **그 변화가 현재 쉘에 그대로 남는다.**
- `.bashrc`, `.bash_profile`처럼 환경설정 파일을 현재 쉘에 즉시 반영할 때(`source ~/.bashrc`) 이 방식을 사용한다.

```bash
[root@Server-A ~]# pwd
/root

[root@Server-A ~]# source ./script/cd_test.sh

[root@Server-A ~]# pwd			# 현재 쉘이 실제로 /tmp로 이동함
/tmp

[root@Server-A ~]# echo $myvar		# 현재 쉘에 변수가 그대로 남음
hello


[root@Server-A ~]# cd ~
[root@Server-A ~]# . ./script/cd_test.sh	# '.'은 source와 완전히 동일하게 동작

[root@Server-A /tmp]# pwd
/tmp
```

- `source`/`.`는 실행 권한이 없어도 동작하며, Shebang도 참고하지 않는다(항상 현재 쉘 자체의 문법으로 해석).

**정리**: `source script.sh`(`. script.sh`)는 서브쉘을 만들지 않고 현재 쉘 안에서 스크립트를 그대로 실행하므로, 변수와 `cd` 결과가 호출한 쉘에 그대로 유지된다. 환경설정을 즉시 반영하거나, 여러 스크립트 사이에서 변수를 공유해야 할 때 사용한다.

## 세 가지 실행 방식 비교

| 방식 | 실행 권한 필요 | Shebang 참고 | 실행 위치 | 변수/cd 영향 |
|---|---|---|---|---|
| `./script.sh` | 필요 | O (참고함) | 새 프로세스(서브쉘) | 현재 쉘에 영향 없음 |
| `bash script.sh` / `sh script.sh` | 불필요 | X (무시, 지정한 인터프리터 강제) | 새 프로세스(서브쉘) | 현재 쉘에 영향 없음 |
| `source script.sh` / `. script.sh` | 불필요 | X (현재 쉘 문법으로 해석) | 현재 쉘 그대로 | 현재 쉘에 그대로 반영 |

**정리**: 스크립트를 "독립된 프로그램"으로 실행하려면 `./script.sh`나 `bash script.sh`를, 스크립트의 결과(변수, 현재 디렉터리)를 **현재 쉘에 반영**해야 한다면 `source`/`.`를 사용한다. 세 방식 모두 결과가 같아 보여도 프로세스 구조와 영향 범위가 다르다는 점이 핵심이다.
