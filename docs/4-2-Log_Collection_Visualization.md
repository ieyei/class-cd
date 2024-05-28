# 4일차 Observability
## Lab2. Log 수집 및 Visualization

<br>

---
- [4일차 Observability](#4일차-observability)
  - [Lab2. Log 수집하기](#lab2-log-수집하기)
    - [2-1. 로그 수집 대상 확인하기](#2-1-로그-수집-대상-확인하기)
    - [2-2. 로그 수집 대상 추가하기](#2-2-로그-수집-대상-추가하기)
    - [2-3. 수집 결과 확인하기](#2-3-수집-결과-확인하기)
    - [2-4. JSON 포맷의 로그 수집하기](#2-4-json-포맷의-로그-수집하기)
    - [2-5. Discover 활용하여 TableView 생성하기](#2-5-discover-활용하여-tableview-생성하기)
    - [2-6. Dashboards 활용하기](#2-6-dashboards-활용하기)
---

ⓘ 실습목표 : Pod의 로그를 수집하여 Opensearch-Dashboard로 조회해보고 다양한 시각화 기능을 실습해본다.

---
### 2-1. 로그 수집 대상 확인하기

▶ 수집하고자 하는 POD의 로그를 확인해보고, 로그가 기록되는 경로를 확인해본다.

<br>

2-1-1. Cloud9에서 POD배포를 위한 Yaml 파일을 생성한다.

- Cloud9 왼쪽의 EXPLORER 에서 `New File`을 선택한다.

![](../images/4-2/4-2-1.svg)

- 파일명은 `test-pod.yaml`로 입력하고 엔터를 누른다.

![](../images/4-2/4-2-2.svg)

- 아래 내용을 복사하여 파일 내용에 붙여넣고, `Ctrl + S`를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: app
    image: ubuntu
    command: ["/bin/bash", "-c"]
    args:
      - 'i=0; while [ $i -le 3600 ]; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'
    volumeMounts:
    - name: varlog
      mountPath: /var/log/containers/
  volumes:
    - name: varlog
      hostPath:
        path: /var/log/containers/
```

![](../images/4-2/4-2-3.svg)

<br>

2-1-2. 하단의 Terminal 창에서 생성된 `test-pod.yaml` 파일을 실행시켜준다.

```bash
cd ${HOME}/environment
```

<br>

🧲 (COPY)

```bash
kubectl apply -f test-pod.yaml
```

✔ **(수행코드/결과 예시)**
```bash
mspuser:~/environment $ kubectl apply -f test-pod.yaml
pod/test-pod created
```

<br>

2-1-3. POD가 생성하는 로그를 확인해본다.

- Cloud9에서 kubectl 명령어를 통해 `test-pod`의 로그를 확인한다.

🧲 (COPY)
```bash
kubectl logs test-pod
```

✔ **(수행코드/결과 예시)**
```bash
mspuser:~/environment $ kubectl logs test-pod
0: Thu Jan 25 04:39:46 UTC 2024
1: Thu Jan 25 04:39:47 UTC 2024
2: Thu Jan 25 04:39:48 UTC 2024
3: Thu Jan 25 04:39:49 UTC 2024
4: Thu Jan 25 04:39:50 UTC 2024
5: Thu Jan 25 04:39:51 UTC 2024
6: Thu Jan 25 04:39:52 UTC 2024
7: Thu Jan 25 04:39:53 UTC 2024
8: Thu Jan 25 04:39:54 UTC 2024
9: Thu Jan 25 04:39:55 UTC 2024
10: Thu Jan 25 04:39:56 UTC 2024
11: Thu Jan 25 04:39:57 UTC 2024
```

<br>

2-1-4. 로그가 기록되는 경로를 확인해본다.

- Cloud9에서 kubectl 명령어를 통해 `test-pod`에 접속해본다.

🧲 (COPY)
```bash
kubectl exec -it test-pod /bin/bash
```

✔ **(수행코드/결과 예시)**
```bash
mspuser:~/environment $ kubectl exec -it test-pod /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@test-pod:/#
```

<br>

- cd 명령어를 통해 `/var/log/containers/` 경로로 이동한다.

🧲 (COPY)
```bash
cd /var/log/containers/
```

✔ **(수행코드/결과 예시)**
```bash
root@test-pod:/# cd /var/log/containers
root@test-pod:/var/log/containers#
```

<br>

- ls 명령어를 통해 host에 기록되는 로그파일을 확인한다.

🧲 (COPY)
```bash
ls
```

✔ **(수행코드/결과 예시)**
```bash
root@test-pod:/var/log/containers# ls
aws-node-fl652_kube-system_aws-eks-nodeagent-4563a12654479bd5be21146cb26a72f2355faaf6b15c780866939e0a373fc4c1.log
aws-node-fl652_kube-system_aws-node-4c8e10ac55e3c94f54cda6bd9a2935ebcec29ca03d70ad35269823e602ce4c9f.log
aws-node-fl652_kube-system_aws-vpc-cni-init-20e9d7dc5f906326b2479085fcb3e44676e1ed016cc0ed2389d2a68095534e1c.log
efs-csi-node-fh74d_kube-system_csi-driver-registrar-d3f62faf57e55f3586fb75397d7848400a2e7b2bf33bf403a6d4d82600a1c909.log
efs-csi-node-fh74d_kube-system_efs-plugin-69ecbcb6b902753d9d6d1592b8398d5a1f898d35207fc1d212b2b7538ca440c3.log
efs-csi-node-fh74d_kube-system_liveness-probe-83a5447d01c7ef4753dc4a856de599cd698585277ac21aea374d2f08f5aa9cd4.log
... 중략 ...
```

<br>

> 👉 일반적으로 `{POD}_{Namespace}_{Container}-{Random values}.log` 의 형태로 기록되는 것을 확인할 수 있다.

<br>
<br>

---
### 2-2. 로그 수집 대상 추가하기

▶ 수집하고자 하는 로그 대상을 추가하여 로그를 수집해본다.

<br>

- GitOps Console에 접속하여 수강생 개인별 Workspace를 선택한다.

![](../images/4-2/4-2-4.svg)

<br>

- `Logging-stack`을 리스트에서 클릭한다.

![](../images/4-2/4-2-5.svg)

<br>

- `Source Code` 탭을 선택하고, 좌측 디렉토리 View에서 `Values.yaml` 파일을 선택한다.

![](../images/4-2/4-2-6.svg)

<br>

- 우측 Text Edit 창의 아무곳이나 클릭한 후, `CTRL + G`를 누르면 line 이동 팝업이 나온다. `13`를 입력 후 `Go to line 13.`을 클릭하여 해당 라인으로 이동한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Line 이동 팝업 |`13`|🧲복사 & 📋붙여넣기|

![](../images/4-2/4-2-7.svg)

<br>

2-2-1. Multiline Parser를 위한 정규식 작성하기

- 정규식 Test URL : https://rubular.com/r/NDuyKwlTGOvq2g

![](../images/4-2/4-2-8.svg)
<br>

> 👉 `(\d+):\s(.+)`의 정규식을 사용해본다.

<br>

- Multiline Parser를 별도의 Config 파일로 추가하기 위해 아래와 같이 작성할 수 있다.

```yaml
    extraFiles:
      parsers_multiline.conf: |
        [MULTILINE_PARSER]
          name          multiline-regex
          type          regex
          flush_timeout 1000
          rule "start_state" "/(\d+):\s(.+)/" "cont"
          rule "cont"        "/^\s+at.*/"     "cont"
```

<br>

- `75` Line에서 `extraFiles: {}` 대신 위의 값을 입력한다.

![](../images/4-2/4-2-9.svg)

<br>

2-2-2. 수집대상 추가하기

- `41` Line에서 `INPUT - systemd` 밑에 위의 값을 추가한다.

```yaml
      [INPUT]
          Name tail
          Path /var/log/containers/test-pod*.log
          multiline.parser docker, cri, multiline-regex
          Tag test.*
          Mem_Buf_Limit 5MB
          Skip_Long_Lines On
```

<br>

![](../images/4-2/4-2-10.svg)

<br>

2-2-3. Parser 파일 추가하기

- `24` Line에 `Parsers_File /fluent-bit/etc/conf/parsers_multiline.conf`를 추가한다.

```yaml
    service: |
      [SERVICE]
          Daemon Off
          Flush {{ .Values.flush }}
          Log_Level {{ .Values.logLevel }}
          Parsers_File /fluent-bit/etc/parsers.conf
          HTTP_Server On
          HTTP_Listen 0.0.0.0
          HTTP_Port {{ .Values.metricsPort }}
          Health_Check On
          Parsers_File /fluent-bit/etc/conf/parsers_multiline.conf
```

![](../images/4-2/4-2-11.svg)

<br>

2-2-4. 변경사항을 모두 입력하고 `Save` 버튼을 클릭한다.

![](../images/4-2/4-2-12.svg)

<br>

2-2-5. 변경사항을 Github Repository에 반영할 수 있는 Commit 팝업을 확인할 수 있다. 아래와 같이 입력하고 `Commit` 버튼을 클릭한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Add Comment|`add log collection rule`|🧲복사 & 📋붙여넣기|

![](../images/4-2/4-2-13.svg)

<br>

2-2-6. 화면 하단의 `Sync` 버튼을 클릭한다.

![](../images/4-2/4-2-14.svg)

<br>
<br>

---

### 2-3. 수집 결과 확인하기

▶ 수집하고자 하는 로그 대상이 정상적으로 수집되었는지 Opensearch를 통해 확인해본다.

<br>

2-3-1. Opensearch-Dashboards에 접속하여 로그인한다.

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```yaml
접속 URL : https://www.<<YOUR_DOMAIN>>/opensearch
```

<br>

- 아래와 같은 로그인 화면이 나타난다. Username/Password를 입력 후 `Log in` 버튼을 클릭한다.

> |항목|내용|
> |---|---|
> ➕ Username | `admin` |
> ➕ Password | `admin` |

<br>

![](../images/4-2/4-2-15.svg)

<br>

- 로그인 후 아래와 같이 팝업 화면이 나타나면 `Dismiss` 버튼을 클릭한다.

![](../images/4-2/4-2-16.svg)

<br>

- 로그인 후 아래와 같이 테넌트 선택 화면이 나타나면 `Cancel`을 클릭한다.

![](../images/4-2/4-2-17.svg)

<br>

2-3-2. `Discover` 메뉴를 통한 Log 검색

- 좌측 메뉴바에서 `Discover`를 클릭한다.

![](../images/4-2/4-2-18.svg)

<br>

- 검색바(DQL)에 `kubernetes.pod_name: test-pod`를 입력하고 `Update` 버튼을 클릭한다.

![](../images/4-2/4-2-19.svg)

<br>

- 조회결과에 돋보기 버튼을 클릭하여 수집된 로그의 상세정보를 확인한다.

![](../images/4-2/4-2-20.svg)

![](../images/4-2/4-2-21.svg)

<br>

2-3-3. `test-pod`를 삭제한다.

```bash
kubectl delete -f test-pod.yaml
```

<br>
<br>

---
### 2-4. JSON 포맷의 로그 수집하기

▶ JSON형태의 로그를 수집하여 `Opensearch`를 통해 수집된 로그를 조회해본다.

<br>

2-4-1. Istio-Ingressgateway가 생성하는 로그를 확인해본다.

- Cloud9에서 kubectl 명령어를 통해 `Istio-Ingressgateway`의 로그를 확인한다.

🧲 (COPY)
```bash
kubectl logs -n istio-system $(kubectl get pods -n istio-system -l app=istio-ingressgateway -o jsonpath={.items[0]..metadata.name})
```

✔ **(수행코드/결과 예시)**
```bash
mspuser:~/environment $ kubectl logs -n istio-system $(kubectl get pods -n istio-system -l app=istio-ingressgateway -o jsonpath={.items[0]..metadata.name})
{"level":"info","time":"2024-01-24T04:32:34.872966Z","msg":"FLAG: --concurrency=\"0\""}
{"level":"info","time":"2024-01-24T04:32:34.872990Z","msg":"FLAG: --domain=\"istio-system.svc.cluster.local\""}
{"level":"info","time":"2024-01-24T04:32:34.872994Z","msg":"FLAG: --help=\"false\""}
{"level":"info","time":"2024-01-24T04:32:34.872997Z","msg":"FLAG: --log_as_json=\"true\""}
{"level":"info","time":"2024-01-24T04:32:34.872999Z","msg":"FLAG: --log_caller=\"\""}
{"level":"info","time":"2024-01-24T04:32:34.873002Z","msg":"FLAG: --log_output_level=\"default:info\""}
{"level":"info","time":"2024-01-24T04:32:34.873004Z","msg":"FLAG: --log_rotate=\"\""}
...중략...
{"x_forwarded_for":"27.147.136.161","downstream_remote_address":"27.147.136.161:49468","upstream_transport_failure_reason":null,"user_agent":"python-requests/2.27.1","response_flags":"NR","response_code":404,"downstream_local_address":"10.0.30.69:8080","bytes_sent":0,"upstream_service_time":null,"response_code_details":"route_not_found","path":"//libs/js/iframe.js","connection_termination_details":null,"upstream_host":null,"upstream_cluster":null,"request_id":"2e253afd-9c38-9e97-9443-9d54e3dc5f91","method":"GET","start_time":"2024-01-25T06:55:47.529Z","requested_server_name":null,"route_name":null,"bytes_received":0,"authority":"43.203.97.57","upstream_local_address":null,"protocol":"HTTP/1.1","duration":0}
{"requested_server_name":null,"upstream_cluster":null,"bytes_sent":0,"upstream_local_address":null,"downstream_local_address":"10.0.30.69:8080","connection_termination_details":null,"duration":0,"bytes_received":0,"method":"GET","downstream_remote_address":"135.125.218.67:33688","response_flags":"NR","upstream_service_time":null,"start_time":"2024-01-25T06:58:44.547Z","x_forwarded_for":"135.125.218.67","route_name":null,"user_agent":"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.129 Safari/537.36","upstream_transport_failure_reason":null,"request_id":"ca8d5faf-2c37-95b8-9c2b-e7d5bd6b3051","path":"/.env","response_code_details":"route_not_found","upstream_host":null,"authority":"54.180.199.250","response_code":404,"protocol":"HTTP/1.1"}
{"response_code":404,"bytes_received":0,"authority":"54.180.199.250","upstream_transport_failure_reason":null,"x_forwarded_for":"135.125.218.67","response_flags":"NR","upstream_cluster":null,"protocol":"HTTP/1.1","path":"/","bytes_sent":0,"method":"POST","connection_termination_details":null,"downstream_local_address":"10.0.30.69:8080","route_name":null,"duration":0,"upstream_service_time":null,"upstream_host":null,"response_code_details":"route_not_found","request_id":"f7fc89a9-e0ed-9e71-841c-dea2a8456f7e","downstream_remote_address":"135.125.218.67:34364","upstream_local_address":null,"requested_server_name":null,"start_time":"2024-01-25T06:58:45.826Z","user_agent":"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.129 Safari/537.36"}
```
<br>

> 👉 `Istio-Ingressgateway`의 로그는 JSON 형태로 기록되는 것을 확인할 수 있다.

<br>

2-4-2. 로그 수집 대상 추가하기

- GitOps Console에 접속하여 수강생 개인별 Workspace를 선택한다.

![](../images/4-2/4-2-4.svg)

<br>

- `Logging-stack`을 리스트에서 클릭한다.

![](../images/4-2/4-2-5.svg)

<br>

- `Source Code` 탭을 선택하고, 좌측 디렉토리 View에서 `Values.yaml` 파일을 선택한다.

![](../images/4-2/4-2-6.svg)

<br>

- 우측 Text Edit 창의 아무곳이나 클릭한 후, `CTRL + G`를 누르면 line 이동 팝업이 나온다. `13`를 입력 후 `Go to line 13.`을 클릭하여 해당 라인으로 이동한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Line 이동 팝업 |`13`|🧲복사 & 📋붙여넣기|

![](../images/4-2/4-2-7.svg)
<br>

> 👉 `config` 필드 하위의 값을 수정해본다.

<br>

2-4-3. JSON 포맷을 위한 `PARSER` 정의하기

- Fluent-bit는 기본 Parser외에 Custom Parser를 아래와 같이 추가할 수 있다.

```yaml
    customParsers: |
      [PARSER]
          Name docker
          Format json
          Time_Keep Off
          Time_Key time
          Time_Format %Y-%m-%dT%H:%M:%S.%L
```

<br>

> 👉 `88` Line에서 `customParsers: ""` 대신 위의 값을 입력한다.

![](../images/4-2/4-2-22.svg)

<br>

2-4-4. 수집대상 추가하기

```yaml
      [INPUT]
          Name tail
          Path /var/log/containers/istio-ingress*.log
          multiline.parser docker, cri
          Tag istio.*
          Mem_Buf_Limit 5MB
          Skip_Long_Lines On
```

<br>

> 👉 `50` Line에서 `INPUT - tail` 밑에 위의 값을 추가한다.

![](../images/4-2/4-2-23.svg)

<br>

2-4-5. PARSER 등록하기
```yaml
    service: |
      [SERVICE]
          Daemon Off
          Flush {{ .Values.flush }}
          Log_Level {{ .Values.logLevel }}
          Parsers_File /fluent-bit/etc/parsers.conf
          HTTP_Server On
          HTTP_Listen 0.0.0.0
          HTTP_Port {{ .Values.metricsPort }}
          Health_Check On
          Parsers_File /fluent-bit/etc/conf/parsers_multiline.conf
          Parsers_File /fluent-bit/etc/conf/custom_parsers.conf
```

<br>

> 👉 `25` Line에 `Parsers_File /fluent-bit/etc/conf/custom_parsers.conf`를 추가한다.

![](../images/4-2/4-2-24.svg)

<br>

2-4-6. 변경사항을 모두 입력하고 `Save` 버튼을 클릭한다.

![](../images/4-2/4-2-25.svg)

<br>

2-4-7. 변경사항을 Github Repository에 반영할 수 있는 Commit 팝업을 확인할 수 있다. 아래와 같이 입력하고 `Commit` 버튼을 클릭한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Add Comment|`add istio log config`|🧲복사 & 📋붙여넣기|

![](../images/4-2/4-2-26.svg)

<br>

2-4-8. 화면 하단의 `Sync` 버튼을 클릭한다.

![](../images/4-2/4-2-27.svg)

<br>

2-4-9. 수집 결과 확인하기

- 좌측 메뉴바에서 `Discover`를 클릭한다.

![](../images/4-2/4-2-18.svg)

<br>

- 검색바(DQL)에 `kubernetes.labels.app: istio-ingressgateway`를 입력하고 `Update` 버튼을 클릭한다.

![](../images/4-2/4-2-28.svg)

<br>

- 조회결과에 돋보기 버튼을 클릭하여 수집된 로그의 상세정보를 확인한다.

![](../images/4-2/4-2-29.svg)

![](../images/4-2/4-2-30.svg)
<br>

> 👉 JSON Field들이 모두 Key-Value 형태로 Parsing되어있음을 확인할 수 있다.

<br>
<br>

---
### 2-5. Discover 활용하여 TableView 생성하기

▶ Opensearch-Dashboards의 `Discover`기능을 활용하여 View를 만들어본다.

✔️ **이전 실습에서 적용한 Filter는 초기화한다**

<br>

2-5-1. Table View 적용해보기

- 좌측 Field View에서 테이블의 컬럼으로 고정시키길 원하는 field를 추가해보자. 아래 그림과 같이 고정하길 원하는 field에 마우스 오버하면 `+` 버튼이 나타나며, 이를 클릭하여 해당 Field를 고정시킬 수 있다. 다음의 총 6개 Field를 고정해보자.

> |항목|
> |---|
> |➕ `kubernetes.container_name` |
> |➕ `kubernetes.host` |
> |➕ `kubernetes.labels.app` |
> |➕ `kubernetes.namespace_name` |
> |➕ `log` |
> |➕ `stream` |

<br>

- column 고정 Field 추가 방법

![](../images/4-2/4-2-31.svg)

<br>

- `Selected fields`에서 고정된 filed 확인 가능

![](../images/4-2/4-2-32.svg)

<br>

- 우측 리스트 View가 고정된 Field로 컬럼이 분리되어 Table View로 변경된 것을 확인할 수 있다.

![](../images/4-2/4-2-33.svg)

<br>

- Opensearch-Dashboards에는 이렇게 고정된 Field 나 혹은 지정된 검색 Filter 등을 Search 객체로 저장하는 기능이 제공된다. 상단의 `Save` 버튼을 클릭한다.

![](../images/4-2/4-2-34.svg)

<br>

- `cta logs`를 입력 후 `save` 버튼 클릭

>|항목|내용|액션|
>|---|---|---|
>➕ Title|`cta logs`|🧲복사 & 📋붙여넣기|

![](../images/4-2/4-2-35.svg)

<br>

- 이렇게 저장된 Search 객체는 추후 상단의 `Open` 버튼을 클릭하여 언제든지 불러올 수 있다.

![](../images/4-2/4-2-36.svg)

<br>

2-5-2. DQL을 활용한 Filter 적용해보기

- 상단의 DQL 입력바에 아래와 같이 간단한 Term Query를 입력해보자.

```JAVASCRIPT
kubernetes.labels.app: "istio-ingressgateway"
```
<br>

![](../images/4-2/4-2-37.svg)
<br>

> 👉 kubernetes의 label이 `app=istio-ingressgateway` 로 되어 있는 pod에서 생성된 로그 데이터만 검색할 수 있다.

<br>

- (실습)고정 Field를 추가해보자.

> |항목|
> |---|
> |➕ `upstream_cluster` |
> |➕ `response_code` |
> |➕ `upstream_service_time` |

<br>

2-5-3. `+ Add filter`를 활용한 Filter 적용해보기

- DQL 입력바 바로 아래의 `+ Add filter`를 클릭하여 Custom filter를 추가해보자. 아래와 같이 선택 및 입력 후 하단의 `Save` 버튼을 클릭한다.

> |항목|내용|액션|
> |---|---|---|
> |➕ Field | `response_code` | 👆🏻셀렉트박스 선택 |
> |➕ Operator | `is not` | 👆🏻셀렉트박스 선택 |
> |➕ Value | `200` | 🧲복사 & 📋붙여넣기 |

<br>

![](../images/4-2/4-2-38.svg)
<br>

![](../images/4-2/4-2-39.svg)
<br>

> 👉 필터가 적용되면 response_code 레코드의 값이 200이 아닌 데이터만 검색된다.

<br>

2-5-4. `Time Filter`를 활용한 Filter 적용해보기

- 오른쪽 상단의 Time Filter를 통해 검색 시간을 조정해보자. 시간은 기본 적으로 `Last 15 minutes`로 선택되어 있는데, 이를 Commonly used 항목에서 `Last 30 minutes`를 선택하고 `Apply` 버튼을 클릭하여 변경해본다.

![](../images/4-2/4-2-40.svg)

<br>

- 오른쪽 상단의 Time Filter 아래에는 검색 Refresh 설정 기능이 있다. `60` 을 입력하고 `seconds`를 선택한 후 `Start` 버튼을 클릭한다. 이제 1분에 한번씩 검색이 Refresh 되도록 설정되었다.

![](../images/4-2/4-2-41.svg)

<br>

- Discover 메뉴에서는 이와 같이 다양한 방법을 적용하여, 원하는 서비스의 로그를 정확히 검색할 수 있다.

![](../images/4-2/4-2-42.svg)

<br>
<br>

---
### 2-6. Dashboards 활용하기

▶ `Visualize` 메뉴에서 Panel을 생성해보고, 이를 `Dashboard`로 활용해보는 실습을 진행한다.

<br>

2-6-1. 좌측 메뉴에서 `Visualize` 를 클릭한다.

![](../images/4-2/4-2-43.svg)

<br>

2-6-2. Visualize 메뉴에서는 다양한 Panel을 생성할 수 있다. 화면의 `Create new visulization`을 클릭한다.

![](../images/4-2/4-2-44.svg)

<br>

2-6-3. 첫 번째 생성해볼 Panel은 TSVB(Time Series Visual Builder)를 활용할 예정이다. Ingress로 들어오는 요청에 대한 응답을 시간 단위로 관찰해보자.

- New Visualization 패널에서 `TSVB`를 선택한다.

![](../images/4-2/4-2-45.svg)

<br>

- `TSVB`의 초기 화면이 나타난다.

![](../images/4-2/4-2-46.svg)

<br>

- `Panel options`탭을 선택하여 아래의 항목을 입력한다.

> |항목|내용|액션|
> |---|---|---|
> |➕ Index pattern | `fluentbit*` | 🧲복사 & 📋붙여넣기 |
> |➕ Time field | `@timestamp` | 👆🏻셀렉트박스 선택 |
> |➕ Panel filter | `kubernetes.labels.app: istio*` | 🧲복사 & 📋붙여넣기 |

![](../images/4-2/4-2-47.svg)

<br>

- `Data`탭을 선택하여 아래의 항목을 입력한다.

> |항목|내용|액션|
> |---|---|---|
> |➕ Group by | `Terms` | 👆🏻셀렉트박스 선택 |
> |➕ By | `response_code` | 👆🏻셀렉트박스 선택 |

![](../images/4-2/4-2-48.svg)

<br>

- 상단의 그래프가 반영해준 데이터에 따라 `Response code`와 함께 변경된다.

![](../images/4-2/4-2-49.svg)

<br>

2-6-4. 생성한 Panel을 저장해보자.

- 상단의 `Save` 버튼을 클릭한다.

![](../images/4-2/4-2-50.svg)

<br>

- Save Visualization 패널에 아래와 같이 입력 후 `Save` 버튼을 클릭하여 저장한다.

> |항목|내용|액션|
> |---|---|---|
> |➕ Title | `TSVB by response code` | 🧲복사 & 📋붙여넣기 |
> |➕ Description | `TSVB by response code` | 🧲복사 & 📋붙여넣기 |

![](../images/4-2/4-2-51.svg)

<br>

2-6-5. 두 번째 생성해볼 Panel은 `Horizontal Bar`를 활용할 예정이다. Ingress로 들어오는 요청의 처리시간을 표시해보자.

- Visualize 메뉴에 접속하여 `Create visualization` 버튼을 클릭한다.

![](../images/4-2/4-2-52.svg)

<br>

- New Visualization 패널에서 `Horizontal Bar`를 선택한다.

![](../images/4-2/4-2-53.svg)

<br>

- Panel을 생성하기 위한 데이터를 지정하는 화면이 나타난다. `fluentbit*`을 클릭한다.

![](../images/4-2/4-2-54.svg)

<br>

- `Horizontal Bar`의 초기 화면이 나타난다.

![](../images/4-2/4-2-55.svg)

<br>

- DQL영역에 `kubernetes.labels.app: istio*`를 입력하여 Filter를 적용한다.

> |항목|내용|액션|
> |---|---|---|
> |➕ DQL | `kubernetes.labels.app: istio*` | 🧲복사 & 📋붙여넣기 |

![](../images/4-2/4-2-56.svg)

<br>

- 우측 Metrics 항목을 아래와 같이 입력 및 선택한다.

> |항목|내용|액션|
> |---|---|---|
> |➕ Aggregation | `Average` | 👆🏻셀렉트박스 선택 |
> |➕ Field | `duration` | 👆🏻셀렉트박스 선택 |

![](../images/4-2/4-2-57.svg)

<br>

- 우측 Buckets 항목의 `Add` 버튼을 클릭하고 `X-axis` 를 선택한다.

![](../images/4-2/4-2-58.svg)

<br>

- 하단에 상세 설정 창이 나타난다. 아래와 같이 입력 및 선택 후 `Update`를 클릭하여 반영한다.

> |항목|내용|액션|
> |---|---|---|
> |➕ Aggregation | `Terms` | 👆🏻셀렉트박스 선택 |
> |➕ Field | `upstream_cluster.keyword` | 👆🏻셀렉트박스 선택 |

![](../images/4-2/4-2-59.svg)

<br>

- 좌측의 그래프가 반영해준 데이터에 따라 host별 평균 응답시간이 함께 변경된다.

![](../images/4-2/4-2-60.svg)

<br>

2-6-6. 생성한 Panel을 저장해보자.

- 상단의 `Save` 버튼을 클릭한다.

![](../images/4-2/4-2-50.svg)

<br>

- Save Visualization 패널에 아래와 같이 입력 후 `Save` 버튼을 클릭하여 저장한다.

> |항목|내용|액션|
> |---|---|---|
> |➕ Title | `response time by host` | 🧲복사 & 📋붙여넣기 |
> |➕ Description | `response time by host` | 🧲복사 & 📋붙여넣기 |

![](../images/4-2/4-2-61.svg)

<br>

2-6-7. 이제 생성한 Panel을 이용하여 Dashboard를 생성해보자.

- 좌측 메뉴에서 Dashboard를 클릭하여 Dashboard 메뉴에 접속한다.

![](../images/4-2/4-2-62.svg)

<br>

- `Create new dashboard` 버튼을 클릭한다.

![](../images/4-2/4-2-63.svg)

<br>

- Editing New Dashboard 화면이 나타난다. 이 화면에서 기존에 생성된 Panel을 추가해주거나 새로운 Panel을 만들어줄 수 있다. 화면의 `Add an existing` 을 클릭한다.

![](../images/4-2/4-2-64.svg)

<br>

2-6-8. 우측 화면에 Add Panels 메뉴가 나타나고, 생성해줬던 Panel과 Search 객체의 리스트를 볼 수 있다. 하나씩 추가해보자.

- 우선 `response time by host`를 클릭한다.

![](../images/4-2/4-2-65.svg)

<br>

- 아래와 같이 Dashboard에 패널이 추가된 것을 확인할 수 있다. 이런식으로 다른 Panel도 추가해주자.

![](../images/4-2/4-2-66.svg)

<br>

- 상단의 `Add`를 클릭하면 다시 한번 우측에 Add panels 메뉴가 나타난다. `TSVM by response code`를 클릭하여 Dashboard에 적용해준다.

> |항목|
> |---|
> |➕ `World Map Web Reqeust Count` (이미 선택 완료) |
> |➕ `TSVM by response code` 선택 |

![](../images/4-2/4-2-67.svg)

<br>

- 선택된 2개의 객체가 아래와 같이 Dashboard에 추가된 것을 확인할 수 있다.

![](../images/4-2/4-2-68.svg)

<br>

2-6-9. 이제 최종적으로 생성한 Dashboard를 저장해보자.

- 상단의 `Save`를 클릭하고 아래와 같이 입력해준다.

> |항목|내용|액션|
> |---|---|---|
> |➕ Title | `Ingress Log Dashboard` | 🧲복사 & 📋붙여넣기 |
> |➕ Description | `Ingress Log Dashboard` | 🧲복사 & 📋붙여넣기 |

![](../images/4-2/4-2-69.svg)

<br>
<br>

😃 **Lab 2 완료!!!**