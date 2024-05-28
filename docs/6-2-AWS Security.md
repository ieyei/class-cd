# 6일차 Security
## Lab2. AWS Service를 활용한 인프라 보안점검

<br>

---
- [6일차 Security](#6일차-security)
  - [Lab2. AWS Service를 활용한 인프라 보안점검](#lab2-aws-service를-활용한-인프라-보안점검)
    - [2-1. AWS ALB 생성](#2-1-aws-alb-생성)
    - [2-2. WAF 적용해보기](#2-2-waf-적용해보기)
    - [2-3. VPC Flow Log](#2-3-vpc-flow-log)
    - [2-4. Network Access Analyzer](#2-4-network-access-analyzer)
    - [2-5. Cloud Trail](#2-5-cloud-trail)
---

ⓘ 실습목표 : AWS의 상품 서비스를 활용하여 보안 강화를 위한 전략을 실습해보고 이해한다.

Keyword : 탐지, 분석, 조치
<br>

---

### 2-1. AWS ALB 생성

<br>

![](../images/6-2/6-2-12.svg)

> 출처 : https://medium.com/@nuatmochoi/aws-waf-%EC%A0%81%EC%9A%A9-%EC%A0%84-%EC%98%81%ED%96%A5%EB%8F%84-%EC%8B%9D%EB%B3%84%ED%95%98%EA%B8%B0-b89aa6b2499b

<br>

2-1-1. AWS ALB 생성을 위한 Target Group 생성

- AWS Console에 로그인하여 Target groups메뉴에 접속한다.

![](../images/6-2/6-2-13.svg)

<br>

- `Create target group`버튼을 클릭한다.

![](../images/6-2/6-2-14.svg)

<br>

- 아래의 값을 입력하고 `Next`버튼을 클릭한다.

**입력 및 수정 값**

> |항목|내용|
> |---|---|
> ➕ Target group name | `cta-alb` |
> ➕ Protocol: Port | `32543` |
> ➕ VPC | `10.0.0.0/16 CIDR이 설정되어있는 수강생 VPC` 선택 |
> ➕ Health check path | `/healthz` |
> ➕ Health check port | `Override` 선택하고 `31436` 입력 |
> ➕ Success codes | `200-399` |
그 외의 설정은 모두 default 상태로 변경 없이 저장
<br>

![](../images/6-2/6-2-15.svg)

<br>

![](../images/6-2/6-2-16.svg)

<br>

![](../images/6-2/6-2-17.svg)

<br>

![](../images/6-2/6-2-18.svg)

<br>

![](../images/6-2/6-2-19.svg)

<br>

- Available instances를 모두 선택하여 등록한다.

![](../images/6-2/6-2-20.svg)

<br>

- Review targets에 등록된 인스턴스 목록을 확인하고 `Create target group`버튼을 클릭한다.

![](../images/6-2/6-2-21.svg)

<br>

- Target group이 생성되고, 상태는 `Unused`인 것을 확인할 수 있다.

![](../images/6-2/6-2-22.svg)

<br>

2-1-2. AWS ALB를 위한 Security Group을 생성해보자.

- AWS Console에 로그인하여 EC2메뉴에 접속한다.

![](../images/6-2/6-2-23.svg)

<br>

- `Network & Security` 하위의 `Security Groups` 메뉴를 클릭한다.

![](../images/6-2/6-2-24.svg)

<br>

- 오른쪽 상단의 `Create security group` 버튼을 클릭한다.

![](../images/6-2/6-2-25.svg)

<br>

- 아래의 내용을 입력한다.

**입력 및 수정 값**

> |항목|내용|
> |---|---|
> ➕ Security group name | `cta-alb` |
> ➕ Description | `cta-alb` |
> ➕ VPC | `10.0.0.0/16 CIDR이 설정되어있는 수강생 VPC` 선택 |

<br>

![](../images/6-2/6-2-26.svg)

<br>

- Inbound rules 항목에서 `Add rule` 버튼을 클릭하여 아래의 내용을 입력하고, `Create security group`버튼을 클릭한다.

**입력 및 수정 값**

> |항목|내용|
> |---|---|
> ➕ Type | `All traffic` 선택 |
> ➕ Source | `0.0.0.0/0` |
그 외의 설정은 모두 default 상태로 변경 없이 저장

![](../images/6-2/6-2-27.svg)

<br>

![](../images/6-2/6-2-28.svg)

<br>

- Security Group이 잘 생성되었는지 확인한다.

![](../images/6-2/6-2-29.svg)

<br>

2-1-3. AWS ALB 생성

- EC2 메뉴에서 `Load Balancers` 메뉴에 접속하여 `Create load Balancer` 버튼을 클릭한다.

![](../images/6-2/6-2-31.svg)

<br>

- `Application Load Balancer`를 확인하고 `Create` 버튼을 클릭한다.

![](../images/6-2/6-2-32.svg)

<br>

- 아래의 내용을 입력하고 `Create load balancer`버튼을 클릭하여 ALB를 생성한다.

**입력 및 수정 값**

> |항목|내용|
> |---|---|
> ➕ Load balancer name | `cta-alb` |
> ➕ VPC | `10.0.0.0/16 CIDR이 설정되어있는 수강생 VPC` 선택 |
> ➕ Mappings | 모든 서브넷의 체크박스를 선택하고, Public Subnet으로 선택 |
> ➕ Security groups | `cta-alb` 선택 |
> ➕ Listener Protocol | `HTTPS` 선택 |
> ➕ Default action | `cta-alb` 선택 |
> ➕ Security policy name | `ELBSecurityPolicy-2016-08` 선택 |
> ➕ Certificate(from ACM) | 수강생 인증서 선택 |
그 외의 설정은 모두 default 상태로 변경 없이 저장

<br>

![](../images/6-2/6-2-33.svg)

<br>

![](../images/6-2/6-2-34.svg)

<br>

![](../images/6-2/6-2-35.svg)

<br>

![](../images/6-2/6-2-36.svg)

<br>

![](../images/6-2/6-2-37.svg)

<br>

2.1.4. Route53 연결하기

- `Route53` 메뉴에 접속하여 `Hosted zones` 메뉴를 클릭한다.

![](../images/6-2/6-2-38.svg)

<br>

- 수강생의 도메인을 클릭한다.

![](../images/6-2/6-2-39.svg)

<br>

- `Create record` 버튼을 클릭하고, 아래의 항목을 입력하여 A레코드를 생성한다.

![](../images/6-2/6-2-40.svg)

<br>

**입력 및 수정 값**

> |항목|내용|
> |---|---|
> ➕ Record name | `cta` |
> ➕ Alias | 활성화 |
> ➕ Choose endpoint | `Alias to Application and Classic Load Balancer` 선택 |
> ➕ Choose Region | `Asia Pacific(Seoul)` 선택 |
> ➕ Choose load balancer | 조회되는 ALB 선택 |
그 외의 설정은 모두 default 상태로 변경 없이 저장
<br>

![](../images/6-2/6-2-41.svg)

<br>

- 잠시 후에 새로 생성한 도메인을 활용하여 Grafana 페이지에 접속해보자.

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```YAML
접속 URL : https://cta.<<YOUR_DOMAIN>>/grafana
```
<br>

![](../images/6-2/6-2-42.svg)

> 👉 Grafana 접속페이지가 보이면 성공!

<br>
<br>

---

### 2-2. WAF 적용해보기

2-2-1. AWS Console의 `WAF & Shield` 메뉴에 접속한다.

![](../images/6-2/6-2-43.svg)

<br>

2-2-2. IP sets 메뉴를 클릭하여 현재 수강생이 접속하는 환경의 IP를 등록한다.

▶ URL : https://www.icanhazip.com/v4

>👉 자신의 ip를 확인할 수 있다. ip를 메모한다.

<br>

![](../images/6-2/6-2-45.svg)

<br>

- `Create IP set` 버튼을 클릭한다.

![](../images/6-2/6-2-46.svg)

<br>

- 아래의 값을 입력하고 `Create IP set`버튼을 클릭한다.

**입력 및 수정 값**

> |항목|내용|
> |---|---|
> ➕ IP set name | `cta-alb-restricted-ip` |
> ➕ Region | `Asia Pacific (Seoul)` |
> ➕ IP addresses | `수강생 IP/32` 선택 |
그 외의 설정은 모두 default 상태로 변경 없이 저장

![](../images/6-2/6-2-47.svg)

<br>

![](../images/6-2/6-2-48.svg)

<br>

2-2-3. `Web ACLs`메뉴를 선택하여 `Create web ACL` 버튼을 클릭한다.

![](../images/6-2/6-2-49.svg)

<br>

2-2-4. 아래의 항목을 입력한다.

**입력 및 수정 값**

> |항목|내용|
> |---|---|
> ➕ Name | `cta-alb-restricted-ip` |
<br>

![](../images/6-2/6-2-50.svg)

<br>

- `Add AWS resources`버튼을 클릭하여 `ALB 선택`하고 `Add`버튼 클릭

![](../images/6-2/6-2-51.svg)

<br>

![](../images/6-2/6-2-52.svg)

<br>

2-2-5. 등록된 AWS Resource를 확인하고, `Next` 버튼을 클릭한다.

![](../images/6-2/6-2-53.svg)

<br>

2-2-6. 나머지 설정은 모두 default 상태로 두고 `Next` 버튼을 눌러 Skip 한다.

- 마지막 Review 페이지가 나오면 `Create web ACL` 버튼을 클릭한다.

![](../images/6-2/6-2-54.svg)

<br>

![](../images/6-2/6-2-55.svg)

<br>

2-2-7. 생성한 `Web ACL`을 선택하여 `Rule`을 등록해보자.

- `Web ACLs` 메뉴에 접속하여 생성한 ACL인 `cta-alb-restricted-ip`를 선택한다.

![](../images/6-2/6-2-56.svg)

<br>

- 상세화면에서 `Rules`를 선택하고, `Add rules` 버튼을 클릭하여 `Add my own rules and rule groups`를 선택한다.

![](../images/6-2/6-2-57.svg)

<br>

- 아래의 값을 입력하고 `Add rule` 버튼을 클릭한다.

**입력 및 수정 값**

> |항목|내용|
> |---|---|
> ➕ Rule type | `IP set` 선택 |
> ➕ Name | `cta-alb-restricted-ip` |
> ➕ IP set | `cta-alb-restricted-ip` 선택 |
그 외의 설정은 모두 default 상태로 변경 없이 저장
<br>

![](../images/6-2/6-2-58.svg)

<br>

![](../images/6-2/6-2-59.svg)

<br>

- `Set rule priority` 화면이 나오면 `Save` 버튼을 클릭한다.

![](../images/6-2/6-2-60.svg)

<br>

2-2-8. Grafana 페이지에 접속해본다.

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```YAML
접속 URL : https://cta.<<YOUR_DOMAIN>>/grafana
```
<br>

![](../images/6-2/6-2-61.svg)

> 👉 403 Forbidden 페이지가 리턴되어 접근이 제한된 것을 확인할 수 있다.

> ❗❗수강생의 모바일을 통해 Grafana 페이지에 접속하면 정상 조회되는 것을 확인할 수 있다.

> 👉 추가로 Rule builder를 활용하여 접속국가 제한, SQL Injection 등도 설정할 수 있다.

<br>
<br>

---

### 2-3. VPC Flow Log

2-3-1. AWS Console에 로그인하여 CloudWatch메뉴에 접속한다.

![](../images/6-2/6-2-1.svg)

<br>

2-3-2. 왼쪽 메뉴에서 `[Logs] - [Log groups]`를 선택하고, 검색창에 `/aws/flowlog`를 입력한다.

![](../images/6-2/6-2-2.svg)

<br>

2-3-3. 로그그룹을 선택하여 상세정보를 확인한다.

![](../images/6-2/6-2-3.svg)

<br>

2-3-4. 브라우저에 새 창을 띄워서 아래의 url에 접속한다.

▶ URL : https://www.icanhazip.com/v4

>👉 자신의 ip를 확인할 수 있다. ip를 메모한다.

<br>

2-3-5. 다시 AWS Console로 돌아와 상단의 `Start tailing`버튼을 클릭하여 vpc 로그를 실시간으로 모니터링해보자.

![](../images/6-2/6-2-4.svg)

<br>

2-3-6. 화면 오른쪽에 실시간으로 네트워크 로그가 기록되는 것을 확인할 수 있다.

![](../images/6-2/6-2-5.svg)

<br>

2-3-7. 브라우저에 새 창을 띄워서 아래의 url에 접속한다.

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```yaml
접속 URL : https://cta.<<YOUR_DOMAIN>>/grafana
```

<br>

2-3-8. 다시 AWS Console로 돌아와 2-1-4에서 기록한 ip를 검색해보자.

![](../images/6-2/6-2-6.svg)

👉 사용자의 IP를 기반으로 시스템에 접속한 이력을 추적해볼 수 있다.

<br>
<br>

---

### 2-4. Network Access Analyzer

2-4-1. AWS Console에서 Network Access Analyzer메뉴에 접속한다.

![](../images/6-2/6-2-7.svg)

<br>

2-4-2. Network Access Scopes 목록을 확인할 수 있다.

![](../images/6-2/6-2-8.svg)

>👉 AWS는 기본적으로 4개의 Network Access Scopes를 생성한다.

<br>

2-4-3. `All-IGW-Ingress(Amazon created)`를 선택하고, 오른쪽 상단의 `Analyze`버튼을 클릭한다.

![](../images/6-2/6-2-9.svg)

<br>

2-4-4. `Analysis is in progress`라고 표시가 되고, 약 2-3분 후 분석이 완료된다.

![](../images/6-2/6-2-10.svg)

<br>

![](../images/6-2/6-2-11.svg)

<br>

>👉 `Findings`를 선택하면 화면 오른쪽 하단에 Network Path가 표시된다. 이로써 현재 구현된 Infrastructure의 네트워크 토폴로지를 분석할 수 있다.

<br>
<br>

---

### 2-5. Cloud Trail

2-5-1. AWS Console에서 CloudTrail메뉴에 접속한다.

![](../images/6-2/6-2-62.svg)

<br>

2-5-2. `Event history` 메뉴에 접속해서 AWS 계정에 대한 작업이력을 확인해보자.

![](../images/6-2/6-2-63.svg)

<br>

- `Lookup attributes`에서 `Event name`을 선택한다.

![](../images/6-2/6-2-64.svg)

<br>

- `Enter an event name` 영역에 `TerminateInstances`를 입력하여 인스턴스 종료이력을 확인해본다.

![](../images/6-2/6-2-64.svg)

>👉 가용성 실습 때 인스턴스를 종료했던 이력을 확인해볼 수 있다. 이렇게 AWS 계정 내 모든 작업에 대한 이력을 조회해볼 수 있다. 해킹이나 부정사용에 대한 확인을 통해 보안점검을 수행할 수 있다.

<br>

❗ 단, AWS에서 기본으로 제공하는 로그는 90일 이내의 로그이다. 90일이 지난 로그를 기록하고 싶다면 `Trail`을 생성하여 S3등의 저장소에 로그를 기록해야 한다.

<br>
<br>

😃 **Lab 2 완료!!!**