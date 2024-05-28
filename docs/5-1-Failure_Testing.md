# 5일차 Availability
## Lab1. Infrastructure Level의 회복탄력성 실습

<br>

---
- [5일차 Availability](#5일차-availability)
  - [Lab1. Infrastructure Level의 회복탄력성 실습](#lab1-infrastructure-level의-회복탄력성-실습)
    - [1-1. Availability in Current Infrastructure](#1-1-availability-in-current-infrastructure)
    - [1-2. RDS Multi-AZ 구성](#1-2-rds-multi-az-구성)
    - [1-3. 테스트 App(Keycloak) 배포](#1-3-테스트-appkeycloak-배포)
    - [1-4. Node Failure](#1-4-node-failure)
    - [1-5. RDS Failure](#1-5-rds-failure)
---

ⓘ 실습목표 : AWS환경에서 인프라에 대한 가용성을 실습해보고 이해한다.

---

### 1-1. Availability in Current Infrastructure

1-1-1. Current Architecture

![](../images/5-1/5-1-1.svg)

👉 HA(High Availability) : 시스템, 네트워크, 서비스가 지속적이고 끊김없이 운영될 수 있도록 설계된 능력

<br>
<br>

---

### 1-2. RDS Multi-AZ 구성

▶ 현재 RDS는 Single AZ 지원으로 이를 Multi-AZ를 지원하도록 변경하여 가용성을 확보하고자 한다. 본 실습은 수강생의 이해를 돕기 위해 AWS Console에서 진행할 예정이다.

<br>

1-2-1. AWS Console에 접속하여 RDS메뉴에 접속한다.

![](../images/5-1/5-1-7.svg)

<br>

1-2-2. `Databases` 메뉴를 선택하고 `rds-cta-dev-kr` 데이터베이스를 클릭한다.

![](../images/5-1/5-1-8.svg)

<br>

- RDS의 Endpoint를 확인하고 메모장에 기록한다.

![](../images/5-1/5-1-42.svg)

<br>

1-2-3. `Configuration` 탭을 선택하고, `Multi-AZ` 항목을 확인한다.

![](../images/5-1/5-1-9.svg)
<br>

> 👉 `Multi-AZ`항목이 `No`로 되어있음을 확인할 수 있다.

<br>

1-2-4. `Summary`영역에서 `Region & AZ`항목을 확인한 후, 기록한다.

![](../images/5-1/5-1-10.svg)

<br>

1-2-5. `Modify` 버튼을 클릭하여 `Multi-AZ` 옵션을 활성화해보자.

![](../images/5-1/5-1-11.svg)

<br>

1-2-6. `Availability & durability` 항목에서 `Create a standby instance`를 체크하고 하단의 `Continue` 버튼을 클릭한다.

![](../images/5-1/5-1-12.svg)
<br>

![](../images/5-1/5-1-13.svg)

<br>

1-2-7. `Apply immediately`를 체크하고 `Modify DB instance` 버튼을 클릭한다.

![](../images/5-1/5-1-14.svg)

<br>

1-2-8. 이 작업은 10분 이상 소요된다. `Status`가 `Available`로 변경되면 작업이 완료된 것이다.

![](../images/5-1/5-1-15.svg)

<br>
<br>

---

### 1-3. 테스트 App(Keycloak) 배포

▶ Keycloak은 오픈소스 기반의 IAM(Identity and Access Management) 소프트웨어로 주로 애플리케이션과 서비스에 대한 사용자 인증(Authentication) 및 권한(Authorization)을 관리하는 데 사용된다. Single Sign-On(SSO)을 지원하며 표준 프로토콜(OIDC, SAML, OAuth2.0 등) 기반으로 기능을 제공한다.

Official : https://www.keycloak.org/

<br>

1-3-1. 테스트 검증을 위한 `keycloak` 데이터베이스 생성

- BastionHost 접속을 위해 EC2 메뉴에 접속한다.

![](../images/5-1/5-1-37.svg)

<br>

- BastionHost를 선택 후, `Connect`버튼을 클릭한다.

![](../images/5-1/5-1-38.svg)

<br>

- `Session Manager`로 탭이 활성화되어있는지 확인 후, `Connect` 버튼을 클릭한다.

![](../images/5-1/5-1-39.svg)

<br>

- BastionHost에 접속 후, mysql client가 설치되어있는지 확인한다.

```bash
mysql -V
```

✔ **(수행코드/결과 예시)**

```bash
$ mysql -V
mysql  Ver 15.1 Distrib 10.6.16-MariaDB, for debian-linux-gnu (x86_64) using  EditLine wrapper
```

<br>

👉 설치가 안된 경우엔 아래의 명령어로 설치
```bash
sudo apt-get update
sudo apt-get install -y mysql-client
```

<br>

- RDS 접속

❗ `<<mariadb_endpoint>>>`의 값은 수강생 RDS Endpoint로 수정한다.

```bash
mysql -h <<mariadb_endpoint>> -u admin -p
```

❗ `<<db_user_password>>>`의 값은 수강생 RDS Endpoint로 수정한다. (Default: password00!)
```bash
Enter Password: <<db_user_password>> 입력
```

✔ **(수행코드/결과 예시)**

```bash
$ mysql -h db-t3-cta-kr.cuk8nlvrmbrg.ap-northeast-2.rds.amazonaws.com -u admin -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 93804
Server version: 10.6.14-MariaDB-log managed by https://aws.amazon.com/rds/

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

<br>

- Database 생성

```bash
create database keycloak;
```

<br>

✔ **(수행코드/결과 예시)**

```bash
mysql> create database keycloak;
Query OK, 1 row affected (0.004 sec)
```

<br>

- Database 생성결과 확인

```bash
show databases;
```

<br>

✔ **(수행코드/결과 예시)**

```bash
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| innodb             |
| keycloak           |
| mysql              |
| performance_schema |
| tmp                |
+--------------------+
6 rows in set (0.001 sec)

mysql>
```

<br>

1-3-2. GitOps Console에 접속하여 수강생 개인별 Workspace를 선택한다.

![](../images/5-1/5-1-43.svg)

<br>

1-3-3. 우측 상단의 `Create App` 버튼을 클릭한다.

![](../images/5-1/5-1-44.svg)

<br>

1-3-4. PaC Source Type에 `Catalog`를 선택하고, `Search Catalog` 버튼을 클릭한다.

![](../images/5-1/5-1-45.svg)

<br>

1-3-5. `keycloak`을 선택하고, `Select` 버튼을 클릭한다.

![](../images/5-1/5-1-46.svg)

<br>

1-3-6. 이름과 경로를 확인하고, `Create` 버튼을 클릭한다.

![](../images/5-1/5-1-47.svg)

<br>

1-3-7. 개인 Github Repository에 PaC Catalog 코드가 복사된다. `REPO_CREATED` 상태를 확인하고, `keycloak`을 리스트에서 클릭한다.

![](../images/5-1/5-1-48.svg)

<br>

1-3-8. Target Namespace에 `default`를 확인하고 Select File 콤보박스에서 `override-values-sds.yaml`을 선택한다.

![](../images/5-1/5-1-49.svg)

<br>

1-3-9. 입력값을 확인 후, 하단의 `Register` 버튼을 클릭한다.

![](../images/5-1/5-1-50.svg)

<br>

1-3-10. `Source code`탭을 클릭한다.

![](../images/5-1/5-1-2.svg)

<br>

1-3-11. 좌측 디렉토리 View에서 `override-values-sds.yaml` 파일을 선택하면 우측 editor 창에 override-values-sds.yaml의 값이 보여진다. 데이터베이스 정보를 수정해보자.

- 우측 Text Edit 창의 아무곳이나 클릭한 후, `CTRL + G`를 누르면 line 이동 팝업이 나온다. `19`를 입력 후 `Go to line 19.`를 클릭하여 해당 라인으로 이동한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Line 이동 팝업 |`19`|🧲복사 & 📋붙여넣기|

![](../images/5-1/5-1-3.svg)

<br>

- `Line 19`의 `extraEnv` 하위에 아래의 정보를 수정하고 `Save` 버튼을 클릭한다.

❗ `<<mariadb_endpoint>>`의 값은 수강생 RDS Endpoint로 수정한다.

![](../images/5-1/5-1-4.svg)

<br>

1-3-12. 변경사항을 Github Repository에 반영할 수 있는 Commit 팝업을 확인할 수 있다. 아래와 같이 입력하고 `Commit` 버튼을 클릭한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Add Comment|`update db info`|🧲복사 & 📋붙여넣기|

![](../images/5-1/5-1-5.svg)

<br>

1-3-13. 리소스 목록을 확인 후, 하단의 `Sync` 버튼을 클릭한다.

![](../images/5-1/5-1-6.svg)

<br>

1-3-14. 2~3분 후 Status가 `Synced`로 바뀌면 배포가 완료된 것이다.

![](../images/5-1/5-1-51.svg)

<br>

1-3-15. keycloak이 정상적으로 설치되었는지를 확인한다.

- Cloud9에서 kubectl 명령어를 통해 POD가 정상적으로 설치되었는지 확인한다.

🧲 (COPY)
```bash
kubectl get pods | grep keycloak
```

✔ **(수행코드/결과 예시)**
```bash
mspuser:~/environment $ kubectl get pods | grep keycloak
keycloak-0              1/1     Running   0          3m30s
```

<br>

- 브라우저에서 아래 URL로 접속해본다.

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```yaml
접속 URL : https://www.<<YOUR_DOMAIN>>/identity/connect/auth
```

<br>

- 아래와 같이 keycloak UI가 보이면 keycloak이 정상적으로 설치된 것이다.

![](../images/5-1/5-1-52.svg)

<br>

1-3-16. RDS에 접속하여 keycloak 관련해서 생성된 테이블 목록을 확인한다.

- BastionHost에서 데이터베이스 접속

❗ `<<mariadb_endpoint>>>`의 값은 수강생 RDS Endpoint로 수정한다.

🧲 (COPY & Modify)
```yaml
mysql -h <<mariadb_endpoint>>> -u admin -p
```

✔ **(수행코드/결과 예시)**
```bash
$ mysql -h <<mariadb_endpoint>>> -u admin -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 82132
Server version: 10.6.14-MariaDB-log managed by https://aws.amazon.com/rds/

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

<br>

- `keycloak` 데이터베이스 접속

🧲 (COPY & Modify)
```yaml
use keycloak;
```

✔ **(수행코드/결과 예시)**
```bash
mysql> use keycloak;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql>
```

<br>

- 테이블 목록 조회

🧲 (COPY & Modify)
```yaml
show tables;
```

✔ **(수행코드/결과 예시)**
```bash
mysql> show tables;
+-------------------------------+
| Tables_in_keycloak            |
+-------------------------------+
| ADMIN_EVENT_ENTITY            |
| ASSOCIATED_POLICY             |
| AUTHENTICATION_EXECUTION      |
...중략...
| USER_SESSION                  |
| USER_SESSION_NOTE             |
| WEB_ORIGINS                   |
+-------------------------------+
92 rows in set (0.002 sec)

mysql>
```

<br>

1-3-17. 데이터의 유실을 체크해보기 위해 Keycloak Database에 간단한 데이터를 입력해 볼 예정이다. 우선 Keycloak admin 페이지에 접속해보자.

- Keycloak admin 페이지에 접속하여 좌측의 `Administration Console` 을 클릭한다.

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```YAML
접속 URL : https://www.<<YOUR_DOMAIN>>/identity/connect/auth
```

![](../images/5-1/5-1-52.svg)

<br>

- 아래와 같은 로그인 화면이 나타난다. Username/Password를 입력 후 `Sign in` 버튼을 클릭한다.

> |항목|내용|
> |---|---|
> ➕ Username | `keycloak` |
> ➕ Password | `keycloak` |
<br>

![](../images/5-1/5-1-53.svg)

<br>

1-3-18. 신규 User를 추가해보자.

- 관리 화면의 좌측 메뉴에서 Manage > `Users`를 클릭하고, Lookup 탭의 `View all users` 버튼을 클릭한다.

![](../images/5-1/5-1-16.svg)

<br>

- `keycloak` 이란 Username을 갖는 User가 1개 등록되어 있는 것을 확인할 수 있다. 새로운 User를 등록해보자. 우측의 `Add user`를 클릭한다.

![](../images/5-1/5-1-17.svg)

<br>

- Add User 화면이 나타난다. 아래와 같이 입력 후 `Save` 버튼을 클릭하여 저장한다.

> |항목|내용|
> |---|---|
> ➕ Username | admin@samsung.com |
> ➕ First Name | samsung |
> ➕ Last Name | admin |
<br>

![](../images/5-1/5-1-18.svg)

<br>

- 저장이 완료되고 아래와 같이 저장된 시간이 `Created At` 항목에 보여진다. 이 시간을 잘 기억해두자. (아래 예시에선 9:08:47 AM 이다.)

![](../images/5-1/5-1-19.svg)

<br>

- 이제 User List에 아래와 같이 2개의 User가 조회된다.

![](../images/5-1/5-1-20.svg)

<br>

1-3-19. 방금 등록한 User Data는 쿼리를 통해서도 확인이 가능하다.

- BastionHost에 접속하여 mariadb RDS에 접속해보자.

```bash
mysql -h <<mariadb_endpoint>> -u admin -p
```

```bash
Enter Password: <<db_user_password>> 입력 (Default: password00!)
```

✔ **(수행코드/결과 예시)**

```bash
$ mysql -h db-t3-cta-kr.cuk8nlvrmbrg.ap-northeast-2.rds.amazonaws.com -u admin -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 93804
Server version: 10.6.14-MariaDB-log managed by https://aws.amazon.com/rds/

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

<br>

- keycloak의 data는 Keycloak Database에 저장되기 때문에 keycloak database를 사용하자.

```bash
use KeycloakDb;
```

```bash
mysql> use keycloak
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql>
```

<br>

- Select 쿼리를 통해 Realm의 User 데이터를 조회해보자. 결과적으로 2개의 row가 검색되고, 1-3-18 실습에서 입력했던 `admin@samsung.com`의 email을 갖는 User가 잘 등록되어 있음을 확인할 수 있다.

```bash
select * from USER_ENTITY;
```

```bash
mysql> select * from USER_ENTITY;
+--------------------------------------+-------+--------------------------------------+----------------+---------+-----------------+------------+-----------+----------+-------------------+-------------------+-----    ------------------------+------------+
| ID                                   | EMAIL | EMAIL_CONSTRAINT                     | EMAIL_VERIFIED | ENABLED | FEDERATION_LINK | FIRST_NAME | LAST_NAME | REALM_ID | USERNAME          | CREATED_TIMESTAMP | SERV    ICE_ACCOUNT_CLIENT_LINK | NOT_BEFORE |
+--------------------------------------+-------+--------------------------------------+----------------+---------+-----------------+------------+-----------+----------+-------------------+-------------------+-----    ------------------------+------------+
| a63860bd-d6f6-430a-97bc-6ed960750d04 | NULL  | 7a3891fc-2c18-43f7-a0c1-97b47ee23ad2 |                |        | NULL            | NULL       | NULL      | master   | keycloak          |     1707377345808 | NULL                            |          0 |
| f6213263-103a-4690-a3f7-dd73acc5e330 | NULL  | e4d3997a-62f6-4169-b589-c8af3bb2dd15 |                |        | NULL            | samsung    | admin     | master   | admin@samsung.com |     1707610127401 | NULL                            |          0 |
+--------------------------------------+-------+--------------------------------------+----------------+---------+-----------------+------------+-----------+----------+-------------------+-------------------+-----    ------------------------+------------+
2 rows in set (0.002 sec)
```

<br>

- mysql client 접속을 종료한다.

```bash
exit
```

✔ **(수행코드/결과 예시)**

```bash
mysql> exit
Bye
```

<br>
<br>

---

### 1-4. Node Failure

1-4-1. Keycloak이 설치된 노드 확인하고 기록한다.

```bash
kubectl get pods -o wide | grep keycloak
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ kubectl get pods -o wide | grep keycloak
keycloak-0   2/2     Running   0          30h   10.0.30.231   ip-10-0-30-93.ap-northeast-2.compute.internal   <none>           <none>
```

<br>

1-4-2. 이제 EKS의 가용성 테스트를 위해 노드를 강제로 종료해보자.

- AWS Console에 로그인하여 EC2 메뉴에 접속한다.

![](../images/5-1/5-1-22.svg)

<br>

- Instance 메뉴를 클릭하고, 1-4-1에서 기록한 노드명을 검색해본다.

![](../images/5-1/5-1-23.svg)

<br>

- Instance를 선택하고, 오른쪽 상단의 [Instance state] - [Terminate instance] 메뉴를 클릭하여 노드를 강제 종료한다.

![](../images/5-1/5-1-24.svg)

<br>

- Instance 상태가 `Shutting-down`으로 변경되고, 잠시 후에 `Terminated`로 변경되는 것을 확인할 수 있다.

![](../images/5-1/5-1-25.svg)
<br>

![](../images/5-1/5-1-26.svg)

<br>

1-4-3. Keycloak Pod를 조회하여 노드를 확인해본다.

```bash
kubectl get pods -o wide | grep keycloak
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ kubectl get pods -o wide | grep keycloak
keycloak-0   2/2     Running   0          30h   10.0.30.189   ip-10-0-30-56.ap-northeast-2.compute.internal   <none>           <none>
```

> 👉 Keycloak Pod가 다른 노드에 스케쥴링된 것을 확인할 수 있다.

<br>

1-4-4. 노드 목록을 조회해본다.

```bash
kubectl get nodes
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ kubectl get nodes
NAME                                             STATUS   ROLES    AGE   VERSION
ip-10-0-20-121.ap-northeast-2.compute.internal   Ready    <none>   31d   v1.28.3-eks-e71965b
ip-10-0-30-56.ap-northeast-2.compute.internal    Ready    <none>   31d   v1.28.3-eks-e71965b
ip-10-0-30-68.ap-northeast-2.compute.internal    Ready    <none>   65s   v1.28.3-eks-e71965b
```

> 👉 새로운 노드가 생성된 것을 확인할 수 있다.

<br>

1-4-5. AWS Console에서도 확인해보자.

![](../images/5-1/5-1-27.svg)

> 👉 노드를 강제로 종료하여도 EKS의 워커노드 수를 3개로 유지한 것을 확인할 수 있다.

<br>

1-4-6. Keycloak에 접속하여 User 목록을 확인해보자.

![](../images/5-1/5-1-20.svg)

> 👉 문제없이 2개의 User가 조회되는 것을 확인할 수 있다.

<br>
<br>

---

### 1-5. RDS Failure

▶ 이제 RDS의 가용성을 확인해보자.

<br>

1-5-1. AWS Console에 로그인하여 RDS 메뉴에 접속한다.

![](../images/5-1/5-1-29.svg)

<br>

1-5-2. 데이터베이스를 선택하고, `Action` 버튼을 클릭하여 강제로 `Reboot`을 진행한다.

![](../images/5-1/5-1-30.svg)

👉 RDS가 실행되어있는 AZ를 기록한다.

<br>

1-5-3. `Reboot with Failover`를 체크하고 `Confirm`버튼을 클릭한다.

![](../images/5-1/5-1-31.svg)

<br>

1-5-4. `Failover` 결과를 확인해본다.

- Reboot을 진행하면 상태값이 `Rebooting`으로 변경된다.

![](../images/5-1/5-1-32.svg)

<br>

- 약 1-2분 후, 상태값이 `Available`으로 변경된 것을 확인할 수 있다.

![](../images/5-1/5-1-33.svg)

<br>

- 데이터베이스를 클릭하여 이벤트를 확인해본다.

![](../images/5-1/5-1-34.svg)
<br>

![](../images/5-1/5-1-35.svg)
<br>

👉 Failover 이벤트가 기록된 것을 확인할 수 있다.

<br>

1-5-5. RDS의 AZ를 확인해보자.

![](../images/5-1/5-1-36.svg)

👉 실제 AZ에 Fail이 발생하면 기존과는 다른 AZ로 RDS가 생성된다.
<br>
👉 본 테스트는 AWS AZ의 상황에 따라 AZ의 변경여부가 결정된다.

<br>

1-5-6. Keycloak에 접속하여 User 목록을 확인해보자.

![](../images/5-1/5-1-20.svg)

> 👉 문제없이 2개의 User가 조회되는 것을 확인할 수 있다.

<br>
<br>

😃 **Lab 1 완료!!!**

<br>

⏩ 다음 실습으로 [이동](5-2-Backup_Recovery.md)합니다.