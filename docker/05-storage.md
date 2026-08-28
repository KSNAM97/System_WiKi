# Docker 05 — Docker Container Storage

## Docker Container Storage 개요

- 도커 컨테이너는 이미지 + **쓰기 계층(write layer)**으로 구성되며, 이 쓰기 계층은 휘발성이므로 DB 데이터나 웹 로그처럼 영구히 보존해야 하는 데이터는 Volume이나 Bind Mount(`-v /dbdata:/var/lib/mysql` 등)로 분리해 컨테이너 밖에 저장해야 한다.
- 이미지는 읽기 전용이며, 컨테이너는 이 이미지 위에 얇은 쓰기 가능한 레이어(Write Layer)가 추가된다.
  - Docker Image = 읽기 전용(Read-only)
  - Container = Image + 쓰기 가능 레이어(Write Layer)
  - 이 구조 덕분에 컨테이너는 가볍고 빠르게 생성되고 여러 컨테이너가 하나의 이미지 파일을 공유할 수 있다.

도커의 저장 구조 기본 개념 (도커 저장 구조를 이해하려면 다음 3가지)
- 이미지 레이어(Image Layer)
- 컨테이너 쓰기 레이어(Container Write Layer)
- 스토리지 드라이버(Storage Driver)

### Docker Image Layer (읽기 전용 계층)

- 도커 이미지는 여러 개의 층(Layer)으로 구성된다.
  - Ubuntu 이미지, Python 이미지, Nginx 이미지 — 이런 것들은 모두 OS 파일, 라이브러리, 바이너리 등이 레이어 단위로 쌓여 만들어진다.

특징
- Layer는 읽기 전용이다.
- 중복되지 않는 레이어만 다운로드한다.
- 여러 이미지가 동일한 레이어를 공유할 수 있다.
- Dockerfile의 일부 명령어(FROM, RUN, COPY, ADD 등)는 새로운 레이어를 생성한다.

EX) Dockerfile
```dockerfile
FROM ubuntu    # Layer 1
RUN apt update # Layer 2
RUN apt install # Layer 3
COPY app.py /app # Layer 4
```

결과 이미지
```
Layer 4 (최상단)
Layer 3
Layer 2
Layer 1
```

### Container Write Layer (쓰기 가능 계층)

- 이미지는 읽기 전용이므로 컨테이너가 실행될 때 읽기 전용 이미지 위에 쓰기 가능한 얇은 레이어가 추가된다.
- 예를 들어 컨테이너 안에서 파일 생성, 파일 수정, 로그 생성, 패키지 설치 — 이 모든 작업은 쓰기 레이어에 저장된다.
- 이 레이어는 컨테이너 삭제 시 같이 사라지고(휘발성) 다른 컨테이너와 공유되지 않는다.
- 따라서 컨테이너는 가볍고 빠르게 생성, 삭제할 수 있다.

### Storage Driver란?

- 도커는 이미지 레이어와 컨테이너 쓰기 레이어를 관리하기 위해 여러 레이어를 하나의 파일 시스템처럼 관리하는 기술을 사용한다. 이 기술이 바로 Storage Driver이다.

대표적인 Storage Driver

| 드라이버 | 설명 |
|----------|------|
| **overlay2** | 현재 대부분의 Linux에서 기본으로 사용하는 스토리지 드라이버. 성능이 좋고 안정적이며 현재 가장 많이 사용된다. |
| aufs | 과거 Ubuntu에서 많이 사용했던 스토리지 드라이버. 현재는 거의 사용되지 않는다. |
| devicemapper | 블록 장치(Block Device)를 이용해 컨테이너를 관리. 예전 CentOS/RHEL에서 많이 사용. |
| btrfs | Btrfs 파일 시스템 기반. 스냅샷, 압축 등의 기능을 제공. |
| zfs | ZFS 파일 시스템 기반. 데이터 무결성과 스냅샷 기능이 뛰어나지만 메모리 사용량이 많고 관리가 복잡. |

현재 Linux 환경에서는 overlay2가 사실상의 표준으로 사용된다.

overlay2 동작 순서
1. 읽기 전용 이미지 레이어(lowerdir)를 준비한다. — Docker Image의 내용을 저장하는 공간이다.
2. 컨테이너가 실행되면 쓰기 가능한 레이어(upperdir)를 생성한다. — 컨테이너에서 생성하거나 수정하는 파일은 모두 여기에 저장된다.
3. lowerdir와 upperdir를 하나의 파일 시스템처럼 합쳐서(merged) 컨테이너에 보여준다.

즉, 이미지는 변경되지 않고, 변경된 내용만 upperdir에 저장하는 방식이다.

### Docker Volume과의 차이

| 저장 구조 | 데이터 유지 | 속도 | 용도 |
|-----------|-------------|------|------|
| Container Write layer | 컨테이너 삭제 시 사라짐 | 느린 편 | 로그, 임시 파일 |
| Docker Volume | 삭제하지 않는 한 유지 | 빠른 편 | DB, 웹 데이터, 영구 저장 |

정리하면 컨테이너 레이어는 일회성, 볼륨은 영구 저장이다.

**정리**: 이미지 레이어, 컨테이너 쓰기 레이어, 스토리지 드라이버(overlay2)의 구조와 Volume의 필요성을 살펴봤다. 이어서 실제 MySQL 컨테이너로 익명 볼륨과 바인드 마운트를 실습한다.

---

## Docker Container Storage 실습 (MySQL)

### 익명 볼륨 방식 (-v /var/lib/mysql)

```bash
[guest@Server-A ~]$ docker  pull  mysql
Using default tag: latest
latest: Pulling from library/mysql
...
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest


[guest@Server-A ~]$ docker  images
IMAGE                    ID             DISK USAGE    CONTENT SIZE
mysql:latest             66aec17cd21a   1.29GB        290MB
nginx:latest             8541484afbc9   238MB         66MB


[guest@Server-A ~]$ docker  rm  -f  mysql_db


[guest@Server-A ~]$ docker  run  -d  --name mysql_db   -v  /var/lib/mysql   -e  MYSQL_ROOT_PASSWORD=admin1234  mysql:latest
c94169a43dd1f7cf5e5d9ea412533f7411d133cc5882b73684daaad024012184
```

- `-e`는 environment, 즉 컨테이너에 환경 변수(Environment Variable)를 전달하는 옵션이다.

```bash
[guest@Server-A ~]$ docker inspect  mysql_db
        "Mounts": [
            {
                "Type": "volume",
                "Name": "fc2e7a728e9ee0bd2ad1307b3020bb476f079ace3019edcb1e48d79750c35585",
                "Source": "/var/lib/docker/volumes/fc2e7a728e9ee0bd2ad1307b3020bb476f079ace3019edcb1e48d79750c35585/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
```

```bash
[root@Server-A ~]# ls -l  /var/lib/docker/volumes/
합계 32
drwx-----x 3 root root    19  8월  5 17:28 05c327209a0b1b524155426377884ee6340f520e1fe4bc52651aba261f1e325f
drwx-----x 3 root root    19  8월  7 10:59 fc2e7a728e9ee0bd2ad1307b3020bb476f079ace3019edcb1e48d79750c35585
-rw------- 1 root root 65536  8월  7 10:59 metadata.db


[root@Server-A ~]# cd /var/lib/docker/volumes/fc2e7a728e9ee0bd2ad1307b3020bb476f079ace3019edcb1e48d79750c35585/_data


[root@Server-A _data]# ls
'#ib_16384_0.dblwr'   binlog.000001   client-cert.pem   mysql
'#ib_16384_1.dblwr'   binlog.000002   client-key.pem    mysql.ibd
'#innodb_redo'        binlog.index    ib_buffer_pool    mysql.sock
'#innodb_temp'        ca-key.pem      ibdata1           mysql_upgrade_history
 auto.cnf             ca.pem          ibtmp1            performance_schema
```

### MySQL 접속 및 데이터 생성

```bash
[guest@Server-A ~]$ docker  exec  -it  mysql_db  /bin/bash

bash-5.1# mysql -u root -p
Enter password: admin1234
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 26.7.0 MySQL Community Server - GPL

mysql>

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.007 sec)


mysql> CREATE  DATABASE  sol_docker;
Query OK, 1 row affected (0.005 sec)


mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sol_docker         |
| sys                |
+--------------------+
5 rows in set (0.003 sec)


mysql> USE sol_docker;
Database changed


mysql> CREATE TABLE member (
    id VARCHAR(20) PRIMARY KEY,
    passwd VARCHAR(100),
    name VARCHAR(50),
    email VARCHAR(100),
    reg DATETIME DEFAULT CURRENT_TIMESTAMP
);


mysql> SHOW TABLES;
+---------------------------+
| Tables_in_sol_docker      |
+---------------------------+
| member                    |
+---------------------------+
1 row in set (0.003 sec)


mysql> DESC member;
+--------+--------------+------+-----+-------------------+-------------------+
| Field  | Type         | Null | Key | Default           | Extra             |
+--------+--------------+------+-----+-------------------+-------------------+
| id     | varchar(20)  | NO   | PRI | NULL              |                   |
| passwd | varchar(100) | YES  |     | NULL              |                   |
| name   | varchar(50)  | YES  |     | NULL              |                   |
| email  | varchar(100) | YES  |     | NULL              |                   |
| reg    | datetime     | YES  |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
+--------+--------------+------+-----+-------------------+-------------------+
5 rows in set (0.007 sec)


# reg 값을 적지 않아도 테이블의 DEFAULT 설정에 의해 현재시간 자동 저장
INSERT INTO member (id, passwd, name, email) VALUES ('soldesk', '1111', 'John', 'hong@soldesk.com');

# reg 컬럼을 명시하고 직접 NOW() 를 설정
INSERT INTO member VALUES ('admin', '1234', 'Tom', 'admin@soldesk.com', NOW());


mysql> SELECT * FROM member;
+---------+--------+------+-------------------+---------------------+
| id      | passwd | name | email             | reg                 |
+---------+--------+------+-------------------+---------------------+
| admin   | 1234   | Tom  | admin@soldesk.com | 2026-08-07 02:15:38 |
| soldesk | 1111   | John | hong@soldesk.com  | 2026-08-07 02:15:32 |
+---------+--------+------+-------------------+---------------------+
2 rows in set (0.001 sec)


[root@Server-A _data]# ls
'#ib_16384_0.dblwr'   binlog.000001   ...   sol_docker    <----
```

---

### 호스트 디렉터리 직접 연결 (Bind Mount)

```bash
[guest@Server-A ~]$ docker  run -d  --name mysql_db  -v  /dbdata:/var/lib/mysql   -e  MYSQL_ROOT_PASSWORD=admin1234  mysql:latest
0bd50758ba749ec31826f82301800eb21a16981e1f1c64ecc0b47f1b0ce6ab50


[guest@Server-A ~]$ docker  exec  -it  mysql_db  /bin/bash
bash-5.1#
bash-5.1# mysql  -u root  -p
Enter password:
Welcome to the MySQL monitor. ...

mysql>


mysql> CREATE  DATABASE  soldesk;
Query OK, 1 row affected (0.005 sec)


mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| soldesk            |
| sys                |
+--------------------+
5 rows in set (0.008 sec)


mysql> USE  soldesk;
Database changed
```

product_info 테이블 구성

| 컬럼명 | 자료형 | 설명 |
|--------|--------|------|
| pid | INT PRIMARY KEY AUTO_INCREMENT | 제품 ID |
| pname | VARCHAR(50) | 제품 이름 |
| category | VARCHAR(50) | 카테고리 |
| price | INT | 가격 |
| stock | INT | 재고 개수 |

```sql
mysql> CREATE TABLE product_info (
   pid      INT PRIMARY KEY AUTO_INCREMENT,
   pname    VARCHAR(50) NOT NULL,
   category VARCHAR(50) NOT NULL,
   price    INT NOT NULL,
   stock    INT NOT NULL
   );


mysql> SHOW TABLES;
+--------------------+
| Tables_in_soldesk  |
+--------------------+
| product_info       |
+--------------------+
1 row in set (0.003 sec)


INSERT INTO product_info (pname, category, price, stock)VALUES ('Laptop', 'Device', 1200, 15);
INSERT INTO product_info (pname, category, price, stock)VALUES ('Chair', 'Home', 160, 40);
INSERT INTO product_info (pname, category, price, stock)VALUES ('Bottle', 'Daily', 20, 120);


mysql> SELECT * FROM product_info;
+-----+--------+----------+-------+-------+
| pid | pname  | category | price | stock |
+-----+--------+----------+-------+-------+
|   1 | Chair  | Home     |   160 |    40 |
|   2 | Bottle | Daily    |    20 |   120 |
+-----+--------+----------+-------+-------+
2 rows in set (0.001 sec)
```

```bash
[guest@Server-A ~]$ ls  /dbdata/
'#ib_16384_0.dblwr'   binlog.000002   ib_buffer_pool   mysql_upgrade_history   soldesk    <----
```

**정리**: 익명 볼륨(`-v /var/lib/mysql`)과 호스트 디렉터리 바인드 마운트(`-v /dbdata:/var/lib/mysql`)로 MySQL 데이터를 영구 보존하는 방법을 실습했다. 이어서 같은 원리를 Nginx 로그에 적용한다.

---

## Nginx 웹 로그를 이용한 데이터 영구 보존 실습

### Volume을 사용하지 않는 경우

```bash
[guest@Server-A ~]$ docker  run  -d  --name  web1  -p  8080:80  nginx:latest
65a5c82145dca961e5fddfdbe5510a7d29e7a5460d25a85eaf03fdaf7d9311e2


[guest@Server-A ~]$ docker  exec  -it  web1  /bin/bash
root@65a5c82145dc:/#

root@65a5c82145dc:/# ls  -l  /var/log/nginx/
total 0
lrwxrwxrwx 1 root root 11 Aug  5 00:22 access.log -> /dev/stdout
lrwxrwxrwx 1 root root 11 Aug  5 00:22 error.log -> /dev/stderr


[guest@Server-A ~]$ docker  logs -f web1
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/08/07 02:50:04 [notice] 1#1: nginx/1.31.3
192.168.10.200 - - [07/Aug/2026:02:50:59 +0000] "GET / HTTP/1.1" 200 896 "-" "curl/8.5.0" "-"
192.168.10.200 - - [07/Aug/2026:02:52:28 +0000] "GET / HTTP/1.1" 200 896 "-" "curl/8.5.0" "-"
...


[guest@Server-A ~]$ docker  rm -f  web1
web1

[guest@Server-A ~]$ docker  logs -f web1
Error response from daemon: No such container: web1
```

- 컨테이너 삭제 시 로그도 함께 사라진다.

### Volume을 사용하여 Host 디렉터리로 Bind Mount한 경우

```bash
[guest@Server-A ~]$ docker  run  -d  --name  web1  -v  /web_log:/var/log/nginx  -p  8080:80  nginx:latest
907d48da2a54c49b1395f6bf7fa0531299eed9c96e1025c2630cc1770b2c771f


[guest@Server-A ~]$ ls  -l  /web_log
합계 4
-rw-r--r-- 1 root root    0  8월  7 12:01 access.log
-rw-r--r-- 1 root root 620  8월  7 12:01 error.log


[guest@Server-A ~]$ cat  /web_log/access.log
192.168.10.200 - - [07/Aug/2026:03:01:59 +0000] "GET / HTTP/1.1" 200 896 "-" "curl/8.5.0" "-"
192.168.10.200 - - [07/Aug/2026:03:04:18 +0000] "GET / HTTP/1.1" 200 896 "-" "curl/8.5.0" "-"
...
```

- 호스트의 `/web_log` 디렉터리에 로그가 영구 보존된다.

**정리**: 볼륨 없이 실행한 경우 로그가 컨테이너와 함께 사라지지만, 바인드 마운트를 사용하면 로그가 호스트에 영구 보존됨을 확인했다. 다음 절에서는 여기서 한 단계 더 나아가 읽기/쓰기 권한을 컨테이너별로 분리하는 실습을 진행한다.

---

## 웹데이터 Read-Only 서비스

- 컨테이너는 실행 환경이고, 데이터는 외부에 둬야 한다.
- 하나의 데이터를 여러 컨테이너가 서로 다른 권한으로 사용할 수 있다.
- 웹 서버는 읽기만 가능하고, 데이터 수정은 별도의 주체만 가능하도록 분리한다.

목표
- 호스트(Rocky)의 `/webdata` 디렉터리 안에 HTML 파일을 생성
- nginx(Rocky) 컨테이너가 이 디렉터리를 읽어서 웹 페이지로 서비스
- nginx(Rocky) 컨테이너에서는 이 디렉터리를 read-only(읽기 전용)으로만 사용
- 개발자는 별도의 writer 컨테이너에서만 파일을 수정할 수 있다.

구조 개념
```
호스트(Rocky)
/webdata               : 실제 파일 위치
writer 컨테이너        : /webdata 를 읽기/쓰기
web 컨테이너(nginx)    : /usr/share/nginx/html를 /webdata에 read-only 로 마운트 (파일은 읽기만 가능)
```

### 1단계: 호스트(Rocky Linux)에서 웹데이터 디렉터리 준비

```bash
[guest@Server-A ~]$ sudo mkdir  /webdata

[guest@Server-A ~]$ sudo chmod  755  /webdata
```

- `/webdata`는 실제 HTML 파일이 저장될 곳이다.

### 2단계: writer 컨테이너에서 HTML 파일 작성

```bash
[guest@Server-A ~]$ docker  pull  ubuntu:latest
...
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest


[guest@Server-A ~]$ docker  run -d  --name  writer  -v  /webdata:/webdata  ubuntu:latest  sleep  infinity
1904fd996033eaa67de91a043fe28c9381ad340b4f4ededbbbb94563790a98cf
```

- `sleep infinity` : 아무 작업을 하지 않고 무한 대기. 컨테이너의 메인 프로세스가 종료되지 않도록 하여 컨테이너를 계속 Running 상태로 유지한다.

```bash
[guest@Server-A ~]$ docker  exec  -it  writer  /bin/bash
root@1904fd996033:/#

root@1904fd996033:/# apt-get update
root@1904fd996033:/# apt-get install -y vim

root@1904fd996033:/# vi /webdata/index.html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Soldesk</title>
</head>
<body>
    <h1>Soldesk Academy</h1>
    <h2>IT 전문 교육기관</h2>
    <p>솔데스크는 IT 전문 교육을 제공합니다.</p>
    <h3>교육 과정</h3>
    <ul>
        <li>Linux</li>
        <li>Network</li>
        <li>Docker</li>
        <li>Cloud</li>
    </ul>
    <p>Soldesk와 함께 IT 전문가로 성장하세요.</p>
</body>
</html>
```

### 3단계: nginx 웹 서버 컨테이너를 Read-Only로 실행

- NGINX의 파일 경로 = `/webdata:/usr/share/nginx/html/`

```bash
[guest@Server-A ~]$ docker run  -d  --name web  -v /webdata:/usr/share/nginx/html:ro  -p  8080:80  nginx:latest


[guest@Server-A ~]$ docker ps
CONTAINER ID   IMAGE           COMMAND                   CREATED         STATUS       PORTS                    NAMES
154f80b9836b   nginx:latest    "/docker-entrypoint.…"   3 seconds ago   Up 2 seconds 0.0.0.0:8080->80/tcp     web
1904fd996033   ubuntu:latest   "sleep infinity"          23 minutes ago  Up 23 min                             writer
```

### 4단계: 웹 페이지 접속 테스트

```bash
# 웹 브라우저 접속
http://192.168.10.100:8080/

# curl로 테스트
guest@Server-B:~$ curl  http://192.168.10.100:8080
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Soldesk</title>
</head>
<body>
    <h1>Soldesk Academy</h1>
    ...
</body>
</html>
```

### 5단계: Read-Only 동작 확인

```bash
root@154f80b9836b:/# echo "html code change" >> /usr/share/nginx/html/index.html
bash: /usr/share/nginx/html/index.html: Read-only file system
```

- `:ro` 옵션 때문에 nginx 컨테이너는 `/usr/share/nginx/html`에 읽기만 할 수 있고, 쓰기는 금지된다.
- 즉, 웹 서버는 오직 서비스만 하고, 데이터 수정은 writer 쪽에서만 할 수 있게 분리된 구조이다.

### 6단계: writer에서 수정하고 웹에서 확인

```bash
[guest@Server-A ~]$ docker  exec  -it  writer  /bin/bash

root@1904fd996033:/# export LANG=C.UTF-8
root@1904fd996033:/# export LC_ALL=C.UTF-8

root@1904fd996033:/# vi /webdata/index.html
# ... AWS, TERRAFORM 추가 후 교육자료 목록 보기 링크 추가 ...
<a href="/text/">교육자료 목록 보기</a>    # 추가 설정
```

**정리**: `writer` 컨테이너는 쓰기 가능, `web` 컨테이너는 `:ro`로 읽기 전용 마운트하여 데이터 수정 권한을 역할별로 분리하는 패턴을 실습했다. 이어서 이 구조에 텍스트 자료 디렉터리와 Nginx의 autoindex 기능을 추가한다.

---

## 텍스트 파일 작성 + Nginx autoindex 설정

### 1단계: Host에 디렉터리 생성

```bash
[guest@Server-A ~]$ mkdir -p /webdata/text
```

- `/webdata` : 웹 서비스에 필요한 데이터가 저장되는 Host 디렉터리
- `/webdata/text` : 웹 서비스외에 텍스트 자료를 저장하는 디렉터리

### 2단계: writer 컨테이너에서 텍스트 파일 생성

```bash
[guest@Server-A ~]$ docker  exec  -it  writer  /bin/bash

root@1904fd996033:/# echo "Linux Server Administration" > /webdata/text/linux.txt
root@1904fd996033:/# echo "Docker Container Technology" > /webdata/text/docker.txt
root@1904fd996033:/# echo "Network Infrastructure" > /webdata/text/network.txt

root@1904fd996033:/# ls  -l  /webdata/text/
total 12
-rw-r--r-- 1 root root 28 Aug  7 14:42 docker.txt
-rw-r--r-- 1 root root 28 Aug  7 14:42 linux.txt
-rw-r--r-- 1 root root 23 Aug  7 14:42 network.txt
```

### Nginx autoindex 설정 파일 생성

- NGINX의 autoindex는 디렉터리 안의 파일 목록을 웹 브라우저에 보여주는 기능이다.

```bash
[guest@Server-A ~]$ sudo mkdir  /nginx_config

[guest@Server-A ~]$ vi  /nginx_config/default.conf
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /text/ {
        autoindex on;
        autoindex_exact_size off;
        autoindex_localtime on;
    }
}
```

설정 설명
- `listen 80` : 80번 포트로 들어오는 HTTP 요청을 처리
- `server_name localhost` : localhost로 들어오는 요청을 처리
- `root /usr/share/nginx/html` : 웹 서버의 기본 문서 디렉터리 지정
- `index index.html` : 디렉터리 요청 시 기본으로 보여줄 파일 지정
- `location /` : `try_files $uri $uri/ =404` — 요청한 파일이 있는지 확인, 없으면 디렉터리 확인, 둘 다 없으면 404 반환
- `location /text/` : `/text/` 경로로 들어오는 요청 처리
  - `autoindex on` : index.html이 없을 경우 해당 디렉터리의 파일 목록을 웹 페이지에 출력
  - `autoindex_exact_size off` : 파일 크기를 KB, MB 등 읽기 쉬운 형태로 출력
  - `autoindex_localtime on` : 파일의 수정 시간을 서버의 로컬 시간으로 표시

### Nginx 컨테이너 Read-Only 실행 (설정 파일 포함)

```bash
[guest@Server-A ~]$ docker run -d \
    --name web \
    -v /webdata:/usr/share/nginx/html:ro \
    -v /nginx_config/default.conf:/etc/nginx/conf.d/default.conf:ro \
    -p 8080:80 \
    nginx:latest
```

- `-v /webdata:/usr/share/nginx/html:ro` : 웹 컨텐츠용 볼륨 (읽기 전용)
- `-v /nginx_config/default.conf:/etc/nginx/conf.d/default.conf:ro` : NGINX의 설정 파일을 공유하기 위한 볼륨 (읽기 전용)

접속 확인
```
Index of /text/
../
docker.txt      07-Aug-2026 05:42    28    <--- 클릭 시 텍스트 내용 출력
linux.txt       07-Aug-2026 05:42    28    <--- 클릭 시 텍스트 내용 출력
network.txt     07-Aug-2026 05:42    23    <--- 클릭 시 텍스트 내용 출력
```

**정리**: Nginx의 `autoindex` 설정과 설정 파일 자체를 read-only 볼륨으로 마운트하는 방법을 실습했다. 마지막으로 여러 웹 컨테이너가 하나의 DB 컨테이너를 공유하는 멀티 컨테이너 구조를 실습한다.

---

## 2대의 Web 서버가 1대의 DB를 공유하는 실습

### 1단계: 작업 디렉터리 및 네트워크 구성

```bash
[guest@Server-A ~]$ sudo mkdir  -p  /web-multi-db/web

[guest@Server-A ~]$ cd /web-multi-db/


# Docker 네트워크 생성
[guest@Server-A web-multi-db]$ docker  network create webnet
f9f4654680bf0ac02806765164a89d4aeea70ad693bbfc1ea381fd7c99ebd61e


[guest@Server-A web-multi-db]$ docker  network  ls
NETWORK ID     NAME     DRIVER   SCOPE
de07c1cd08a1   bridge   bridge   local
488e8e247e8e   host     host     local
f38215cbe6b3   none     null     local
f9f4654680bf   webnet   bridge   local


[guest@Server-A web-multi-db]$ docker  network   inspect webnet
[
    {
        "Name": "webnet",
        "Id": "f9f4654680bf0ac02806765164a89d4aeea70ad693bbfc1ea381fd7c99ebd61e",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        ...
    }
]
```

### 2단계: MySQL DB 컨테이너 생성 (mydb)

```bash
[guest@Server-A web-multi-db]$
docker  run  -d  --name mydb  --network  webnet \
-e  MYSQL_ROOT_PASSWORD=1234 \
-e  MYSQL_DATABASE=labdb \
-v  /mysqldb:/var/lib/mysql \
-p  3306:3306  mysql:latest


[guest@Server-A web-multi-db]$ sudo systemctl stop mariadb.service    # DB동작시 중지


[guest@Server-A web-multi-db]$ docker ps
CONTAINER ID   IMAGE          COMMAND                   CREATED          STATUS          PORTS                         NAMES
4406f9fae5a2   mysql:latest   "docker-entrypoint.s…"   45 seconds ago   Up 45 seconds   0.0.0.0:3306->3306/tcp        mydb


# MySQL 컨테이너 접속
[guest@Server-A web-multi-db]$ docker  exec  -it  mydb  /bin/bash

bash-5.1# mysql  -u root -p
Enter password:
Welcome to the MySQL monitor. ...

mysql>

mysql> USE labdb;
Database changed


# labdb 데이터베이스 안에 products 테이블 생성
mysql> CREATE TABLE products (
  id    INT PRIMARY KEY AUTO_INCREMENT,
  name  VARCHAR(50) NOT NULL,
  price INT NOT NULL
);
Query OK, 0 rows affected (0.010 sec)


# 테스트용 데이터 입력
INSERT INTO products (name, price) VALUES ('apple', 1000);
INSERT INTO products (name, price) VALUES ('banana', 1500);
INSERT INTO products (name, price) VALUES ('orange', 2000);


mysql> SELECT * FROM products;
+----+--------+-------+
| id | name   | price |
+----+--------+-------+
|  1 | apple  |  1000 |
|  2 | banana |  1500 |
|  3 | orange |  2000 |
+----+--------+-------+
3 rows in set (0.000 sec)
```

### 3단계: 웹 애플리케이션 파일 준비

```bash
[guest@Server-A web]$ pwd
/web-multi-db/web

[guest@Server-A web]$ sudo vi dockerfile
FROM  php:8.2-apache

RUN  docker-php-ext-install  mysqli

COPY  .  /var/www/html/
```

`/web-multi-db/web/index.php` 작성 (Rocky Linux)

```php
<?php
// DB 접속 설정
$DB_HOST = '192.168.10.100';
$DB_USER = 'root';
$DB_PASS = '1234';
$DB_NAME = 'labdb';

$message = '';
$rows = [];

$conn = new mysqli($DB_HOST, $DB_USER, $DB_PASS, $DB_NAME);

if ($conn->connect_error) {
    die('DB 접속 실패: ' . $conn->connect_error);
}

// 폼에서 값이 넘어온 경우 → DB에 INSERT
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name  = trim($_POST['name'] ?? '');
    $price = trim($_POST['price'] ?? '');

    if ($name === '' || $price === '') {
        $message = '이름과 가격을 모두 입력해야 합니다.';
    } elseif (!ctype_digit($price)) {
        $message = '가격은 숫자로만 입력해야 합니다.';
    } else {
        $stmt = $conn->prepare('INSERT INTO products (name, price) VALUES (?, ?)');
        $price_int = (int)$price;
        $stmt->bind_param('si', $name, $price_int);

        if ($stmt->execute()) {
            $message = '저장 완료: ' . htmlspecialchars($name) . ' (' . $price_int . '원)';
        } else {
            $message = '저장 실패: ' . $stmt->error;
        }
        $stmt->close();
    }
}

// 항상 DB의 전체 목록을 읽어서 출력
$result = $conn->query('SELECT id, name, price FROM products ORDER BY id ASC');

if ($result) {
    $rows = $result->fetch_all(MYSQLI_ASSOC);
    $result->free();
}

$conn->close();
?>
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>상품 입력 및 전체 목록 조회 (웹 2대 + DB 1대)</title>
</head>
<body>
    <h1>상품 입력 페이지</h1>
    <p>아래에 상품 이름과 가격을 입력하면 DB에 저장되고, 아래 테이블에 전체 목록이 출력됩니다.</p>

    <form method="post">
        <label for="name">상품 이름:</label>
        <input type="text" id="name" name="name">

        <label for="price">가격:</label>
        <input type="text" id="price" name="price">

        <button type="submit">저장</button>
    </form>

    <hr>

    <?php if ($message !== ''): ?>
        <p><?php echo htmlspecialchars($message); ?></p>
    <?php endif; ?>

    <h2>상품 전체 목록</h2>
    <table border="1" cellpadding="5" cellspacing="0">
        <tr>
            <th>ID</th>
            <th>이름</th>
            <th>가격(원)</th>
        </tr>
        <?php if (count($rows) === 0): ?>
            <tr>
                <td colspan="3">등록된 상품이 없습니다.</td>
            </tr>
        <?php else: ?>
            <?php foreach ($rows as $row): ?>
                <tr>
                    <td><?php echo htmlspecialchars($row['id']); ?></td>
                    <td><?php echo htmlspecialchars($row['name']); ?></td>
                    <td><?php echo htmlspecialchars($row['price']); ?></td>
                </tr>
            <?php endforeach; ?>
        <?php endif; ?>
    </table>

    <hr>
    <p>이 페이지는 같은 소스(index.php)를 사용하는 두 개의 웹 컨테이너(web1, web2)가
       하나의 DB 컨테이너(mydb)에 데이터를 저장하고, DB의 전체 데이터를 조회하는 구조입니다.</p>
</body>
</html>
```

### 4단계: 웹 이미지 빌드

```bash
# Rocky Linux에서 이미지 빌드
[guest@Server-A web]$ docker  build  -t  web_db_conn:1.0  .

[guest@Server-A web]$ docker  images | grep web_db*
web_db_conn:1.0    d63bfb94e03a    701MB    176MB


# Ubuntu Linux에서 이미지 빌드
guest@Server-B:/web-multi-db/web$ docker  build  -t  web_db_conn:1.0  .

guest@Server-B:/web-multi-db/web$ docker  images | grep web_db*
web_db_conn:1.0    fe9a72df8f78    708MB    176MB
```

### 5단계: 웹 서버 2대 컨테이너 실행 (web1, web2)

```bash
# Rocky Linux에서 web1 컨테이너 생성
[guest@Server-A web]$ docker  run  -d  --name web1  --network webnet  -p  80:80  web_db_conn:1.0
7538056201e3ea21f134f0ccc1dfa8459176e81208e5016529720abea39abc2e

[guest@Server-A web]$ docker  ps
CONTAINER ID   IMAGE             COMMAND                  CREATED        STATUS         PORTS                    NAMES
7538056201e3   web_db_conn:1.0   "docker-php-entrypoi…"  2 minutes ago  Up 2 minutes   0.0.0.0:80->80/tcp       web1
2246647ecb1e   mysql:latest      "docker-entrypoint.s…"  29 minutes ago Up 29 minutes  0.0.0.0:3306->3306/tcp   mydb


# Ubuntu Linux에서 web2 컨테이너 생성
guest@Server-B:/web-multi-db/web$ docker  run  -d  --name web2  -p 80:80  web_db_conn:1.0
237ff3553cc4d2d337732f3baea848d672c88117e5f95f725cc8e3d5ca822bab

guest@Server-B:/web-multi-db/web$ docker  ps
CONTAINER ID   IMAGE             COMMAND                   CREATED            STATUS             PORTS                  NAMES
237ff3553cc4   web_db_conn:1.0   "docker-php-entrypoi…"   About a minute ago Up About a minute  0.0.0.0:80->80/tcp     web2
```

접속 확인
```
http://192.168.10.100/    # Rocky의 web1 접속
http://192.168.10.200/    # Ubuntu의 web2 접속
```

```bash
[guest@Server-A web]$ ls  /mysqldb/
'#ib_16384_0.dblwr'  binlog.000001  ibdata1   mysql   performance_schema
...     labdb    ...
```

- 두 웹 서버 모두 동일한 DB에서 데이터를 읽고 쓸 수 있다.
