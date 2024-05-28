# 4일차 Observability
## Lab1. Logging Architecture Installation

<br>

---
- [4일차 Observability](#4일차-observability)
  - [Lab1. Logging Architecture Installation](#lab1-logging-architecture-installation)
    - [1-1. Logging Stack 설치하기](#1-1-logging-stack-설치하기)
    - [1-2. Logging Stack 둘러보기](#1-2-logging-stack-둘러보기)
    - [1-3. Index Pattern 지정 및 Log 검색](#1-3-index-pattern-지정-및-log-검색)
    - [1-4. Collector(Fluent-bit) Configuration](#1-4-collectorfluent-bit-configuration)
---

ⓘ 실습목표 : Logging Architecture를 설치하여 환경을 구성해본다.

---
### 1-1. Logging Stack 설치하기

▶ GitOps Console의 PaC Catalog 기능을 통해 Logging Stack을 설치한다.

<br>

✔️ **Logging Stack Catalog**

Logging Stack Catalog는 Helm Chart로 구성되어 있으며, 다음과 같은 sub chart들이 dependency 설정에 의해 함께 설치된다.

 - Fluent-bit
 - Opensearch
 - Opensearch-dashboards

<br>

![](../images/4-1/4-1-1.svg)

<br>

1-1-1. GitOps Console에 접속하여 3일차 교육에서 생성했던 수강생 개인별 Workspace를 선택한다.

![](../images/4-1/4-1-2.svg)

<br>

1-1-2. 우측 상단의 `Create App` 버튼을 클릭한다.

![](../images/4-1/4-1-3.svg)

<br>

1-1-3. PaC Source Type에 `Catalog`를 선택하고, `Search Catalog` 버튼을 클릭한다.

![](../images/4-1/4-1-4.svg)

<br>

1-1-4. `Logging-stack`을 선택하고, `Select` 버튼을 클릭한다.

![](../images/4-1/4-1-5.svg)

<br>

1-1-5. 이름과 경로를 확인하고, `Create` 버튼을 클릭한다.

![](../images/4-1/4-1-6.svg)

<br>

1-1-6. 개인 Github Repository에 PaC Catalog 코드가 복사된다. `REPO_CREATED` 상태를 확인하고, `Logging-stack`을 리스트에서 클릭한다.

![](../images/4-1/4-1-7.svg)

<br>

1-1-7. Target Namespace에 `logging`을 입력하고 Select File 콤보박스에서 `values.yaml`을 선택한다.

![](../images/4-1/4-1-8.svg)

<br>

1-1-8. 입력값을 확인 후, 하단의 `Register` 버튼을 클릭한다.

![](../images/4-1/4-1-9.svg)

<br>

1-1-9. Source Code 탭을 선택하여 코드를 확인 후, 하단의 `Sync` 버튼을 클릭한다.

![](../images/4-1/4-1-10.svg)

<br>

1-1-10. 2~3분 후 Status가 `Synced`로 바뀌면 배포가 완료된 것이다.

![](../images/4-1/4-1-11.svg)

<br>

1-1-11. Logging-stack이 정상적으로 설치되었는지를 확인한다.

- Cloud9에서 kubectl 명령어를 통해 logging namespace에 POD가 정상적으로 설치되었는지 확인한다.

🧲 (COPY)
```bash
kubectl get pods -n logging
```

✔ **(수행코드/결과 예시)**
```bash
mspuser:~/environment $ kubectl get pods -n logging
NAME                                     READY   STATUS    RESTARTS   AGE
fluent-bit-662zj                         1/1     Running   0          4m56s
fluent-bit-66spx                         1/1     Running   0          4m56s
fluent-bit-gkpqb                         1/1     Running   0          4m56s
fluent-bit-mpmtv                         1/1     Running   0          4m56s
fluent-bit-rbrrt                         1/1     Running   0          4m56s
opensearch-0                             1/1     Running   0          4m56s
opensearch-dashboards-58f769f4f8-8xz7h   1/1     Running   0          4m56s
```

<br>

- 브라우저에서 아래 URL로 접속해본다.

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```yaml
접속 URL : https://www.<<YOUR_DOMAIN>>/opensearch
```

<br>

- 아래와 같이 Opensearch 로그인 UI가 보이면 정상적으로 설치된 것이다.

![](../images/4-1/4-1-12.svg)

<br>
<br>

---
### 1-2. Logging Stack 둘러보기

▶ 각 Component의 기본 사용법을 익히고, Logging Stack과 어떻게 연계되어 있는지 확인한다.

<br>

✔️ **Opensearch-Dashboards**

Official : https://opensearch.org/docs/2.11/dashboards/

<br>

1-2-1. Opensearch-Dashboards에 접속하여 로그인한다.

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

![](../images/4-1/4-1-13.svg)

<br>

- 로그인 후 아래와 같이 팝업 화면이 나타나면 `Dismiss` 버튼을 클릭한다.

![](../images/4-1/4-1-14.svg)

<br>

- 로그인 후 아래와 같이 테넌트 선택 화면이 나타나면 `Cancel`을 클릭한다.

![](../images/4-1/4-1-15.svg)

<br>

1-2-2. Opensearch-Dashboards의 기본 메인화면에 접속하였다. 좌측 상단의 `≡`부분을 클릭하여 메뉴를 확인해보자. 메뉴바 하단의 Management의 `Dev Tools` 메뉴를 클릭한다.

- Opensearch-Dashboards 메인 화면

![](../images/4-1/4-1-16.svg)

<br>

- Opensearch-Dashboards 메뉴 구성

![](../images/4-1/4-1-17.svg)

<br>

1-2-3. 아래와 같은 `Dev Tools` 화면에 접속했다. 본 메뉴는 Opensearch-Dashboards와 연결된 Opensearch의 다양한 API를 콘솔 화면에서 손쉽게 호출하고 결과를 확인할 수 있는 기능을 제공한다.

![](../images/4-1/4-1-18.svg)

<br>

1-2-4. Opensearch API를 호출하여 Fluent-bit이 수집하는 로그 데이터가 잘 수집되고 있는지 확인해보자. 아래 API Command를 복사하여 좌측 Console 창에 붙여넣는다. 그 후 우측의 `▷` 표시를 클릭하거나, 해당 line에 커서를 위치하고 `ctrl + enter`를 입력하면 우측에 결과가 표시된다.

🧲 (COPY)

```JAVASCRIPT
GET _cat/indices?v
```

✔ **(수행코드/결과 예시)**
```JAVASCRIPT
health status index                        uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .plugins-ml-config           H1w2Qq1OS-qNEEf6xDXCbg   1   0          1            0      3.9kb          3.9kb
green  open   .opensearch-observability    XjDyozFlQzSQpH9OdKSjjg   1   0          0            0       208b           208b
yellow open   security-auditlog-2024.01.24 BDfRLgw9Q8qOWTaI4u-xBw   1   1        593            0    757.7kb        757.7kb
green  open   .ql-datasources              HbVDGPBcQ-ul2VjKKP5JSg   1   0          0            0       208b           208b
green  open   .kibana_92668751_admin_1     -P1sx0hUR1KIDyQCsvorow   1   0          2            0     16.1kb         16.1kb
yellow open   fluentbit-2024.01.24         zhT5jm2BSMew0_0SEJp_Dg   1   1      63712            0     60.3mb         60.3mb
green  open   .kibana_1                    ZdLu0BoRQtGiQb2UQ7Ezog   1   0          0            0       208b           208b
green  open   .opendistro_security         hEDoZBPNTiW3aolaDUIzYw   1   0         10            0     76.4kb         76.4kb
```

<br>

![](../images/4-1/4-1-19.svg)

<br>

1-2-5. `_cat/indices` API는 현재 Opensearch에 저장된 Index의 리스트를 확인할 수 있는 기능을 제공하며, 조회된 index를 살펴보면 `fluentbit-{DATE}` 이름의 index가 있음을 확인할 수 있다. 바로 이 index가 fluent-bit에 의해 수집된 eMarket의 로그 데이터가 저장되는 index이다.

- `docs.count` 컬럼의 값을 보면 API 조회 기준 index에 저장된 document의 개수를 확인 할 수 있다.

- `pri`, `rep` 컬럼의 값이 각각 `1`, `1`로 설정되어 있는데, 이는 Primary Shard의 개수와 Replica Shard의 개수를 의미한다.

![](../images/4-1/4-1-20.svg)

<br>

1-2-5. `fluentbit-{DATE}` 이름의 index의 mapping 정보를 확인해보자. 아래와 같이 좌측 Console 창에 입력하여 API를 호출한다. 입력시 {DATE} 부분은 오늘 날짜로 치환해준다. (예시 2024.01.24)

🧲 (COPY)
```JAVASCRIPT
GET fluentbit-{DATE}/_mapping
```

![](../images/4-1/4-1-21.svg)

<br>

✔ **(수행코드/결과 예시)**
```JSON
{
  "fluentbit-2024.01.24": {
    "mappings": {
      "properties": {
        "@level": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "@message": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "@module": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "@timestamp": {
          "type": "date"
        },
...중략...
```

![](../images/4-1/4-1-22.svg)

<br>

> 👉 `properties` 하위의 field들은 fluent-bit이 수집한 정보를 나타낸다. 이것은 수집하는 로그의 형태에 따라 달라질 수 있다.

<br>
<br>

---
### 1-3. Index Pattern 지정 및 Log 검색

▶ Opensearch-Dashboards의 Discover 기능을 활용하여 로그를 검색해보는 실습을 진행한다.

<br>

1-3-1. 좌측 메뉴바에서 Management 하단의 `Dashboards Management`를 클릭한다.

![](../images/4-1/4-1-23.svg)

<br>

1-3-2. 아래와 같은 화면이 나타난다. Dashboards Managememt는 Opensearch-Dashboards의 관리용 화면으로 저장된 Object(대시보드, 패널, 테이블 데이터 등)를 확인하거나, 설정을 수정할 수 있는 기능을 제공한다. Opensearch-Dashboards에서 Opensearch의 데이터를 조회하기 위해서는 우선 `Index Pattern`을 지정해주어 어떤 index의 데이터를 검색할 건지 결정해줘야 한다. 이를 위해 좌측 메뉴에서 `Index Patterns`를 클릭한다.

![](../images/4-1/4-1-24.svg)

<br>

1-3-3. 화면의 `Create index pattern` 버튼을 클릭한다.

![](../images/4-1/4-1-25.svg)

<br>

1-3-4. 저장되어 있는 index의 리스트와 함께 pattern을 입력할 수 있는 입력바가 보인다. index pattern은 하나의 index를 지정해줄 수도 있고, `*` 문자를 활용하여 복수의 index를 지정해줄 수도 있다.

- index pattern 생성 화면

![](../images/4-1/4-1-26.svg)

<br>

- 입력바에 `fluentbit`를 입력해주면 아래와 같이 1개의 index가 matching 된다.(뒤의 *은 자동 입력됨) `Next step >` 버튼을 클릭한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Index pattern name|`fluentbit`|🧲복사 & 📋붙여넣기|

![](../images/4-1/4-1-27.svg)

<br>

1-3-5. 검색에서 time filter로 사용할 field를 선택해주는 화면이 나타난다. Time field를 클릭하고 `@timestamp`를 선택한 다음 하단의 `Create index pattern`을 클릭한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Time field|`@timestamp`|👆🏻셀렉트박스 선택|

![](../images/4-1/4-1-28.svg)

<br>

1-3-6. 아래와 같이 index pattern이 정상적으로 생성되었으며, 맵핑된 Field의 정보를 확인할 수 있다.

![](../images/4-1/4-1-29.svg)

<br>

1-3-7. 좌측 메뉴바에서 `Discover`를 클릭한다.

![](../images/4-1/4-1-30.svg)

<br>

1-3-8. 아래와 같이 수집된 로그 데이터를 리스트 형식과 바 그래프로 확인할 수 있다. 특히 좌측 상단에 방금 생성한 `fluentbit*` index pattern이 지정되어 있는 것을 볼 수 있다.

- Discover 메뉴 전체 화면

![](../images/4-1/4-1-31.svg)

<br>

- `fluentbit*` index pattern이 선택되어 있음 확인 가능

![](../images/4-1/4-1-32.svg)

<br>

1-3-9. Discover 메뉴의 각 항목별 간략한 설명은 아래와 같다.

![](../images/4-1/4-1-33.svg)

  ① DQL(Data Query Language)을 통해 데이터 검색   
  ② 수집된 데이터의 Timestamp 기준으로 검색 시간 Filter 설정   
  ③ 데이터의 세부 Field 정보   
  ④ 수집된 데이터(Document)의 Raw Data로 _source 항목에서 Serialized JSON object 정보 확인 가능.   

<br>
<br>

---
### 1-4. Collector(Fluent-bit) Configuration

▶ 설치된 Fluent-bit의 Configuration을 확인해보고 로그를 수집하는데 필요한 설정을 알아본다.

<br>

1-4-1. 현재 설치된 Fluent-bit의 Configuration을 확인해본다.

- GitOps Console에 접속하여 수강생 개인별 Workspace를 선택한다.

![](../images/4-1/4-1-2.svg)

<br>

- `Logging-stack`을 리스트에서 클릭한다.

![](../images/4-1/4-1-34.svg)

<br>

- `Source Code` 탭을 선택하고, 좌측 디렉토리 View에서 `Values.yaml` 파일을 선택한다.

![](../images/4-1/4-1-35.svg)

<br>

- 우측 Text Edit 창의 아무곳이나 클릭한 후, `CTRL + G`를 누르면 line 이동 팝업이 나온다. `28`를 입력 후 `Go to line 28.`을 클릭하여 해당 라인으로 이동한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Line 이동 팝업 |`28`|🧲복사 & 📋붙여넣기|

![](../images/4-1/4-1-36.svg)

<br>

1-4-2. `[INPUT]` Configuration 설정 부분은 아래와 같다.

- `tail` INPUT 플러그인을 사용하여 Worker Node(EC2 Instance)의 `/var/log/containers/` 경로에 위치한 로그 파일 중 `fluent-bit`으로 시작하고 있는 파일의 내용을 수집한다. 수집된 Log를 cri 타입의 포멧으로 Parsing 처리할 수 있도록 Parser 옵션을 활용하고 있다.

```yaml
config:
  ## https://docs.fluentbit.io/manual/pipeline/inputs
  inputs: |
    [INPUT]
        Name tail
        Path /var/log/containers/fluent-bit*.log
        multiline.parser docker, cri
        Tag kube.*
        Mem_Buf_Limit 5MB
        Skip_Long_Lines On
```

✔️ **CRI(Container Runtime Interface) Logging Format**
<br>
https://github.com/kubernetes/design-proposals-archive/blob/main/node/kubelet-cri-logging.md#proposed-solution

<br>

1-4-3. `[FILTER]` Configuration 설정 부분은 아래와 같다.

- `kubernetes` FILTER 플러그인을 사용하여 다양한 Kubernetes Meta 정보를 record에 추가해주도록 설정한다. 이 필터를 통해 kubernetes의 pod, label, annotation과 같은 정보를 추가로 확인 가능하다.

```yaml
  ## https://docs.fluentbit.io/manual/pipeline/filters
  filters: |
    [FILTER]
        Name kubernetes
        Match *
        Merge_Log On
        Keep_Log Off
        K8S-Logging.Parser On
        K8S-Logging.Exclude On
```

<br>

1-4-4. `[OUTPUT]` Configuration 설정 부분은 아래와 같다.

- `opensearch` OUTPUT 플러그인을 사용하여 수집 및 처리된 로그 데이터를 elasticsearch로 전송한다.

```yaml
  ## https://docs.fluentbit.io/manual/pipeline/outputs
  outputs: |
    [OUTPUT]
        Name opensearch
        Match *
        Host opensearch-cluster-master
        Port 9200
        HTTP_User admin
        HTTP_Passwd admin
        tls On
        tls.verify Off
        Suppress_Type_Name On
        Logstash_Format On
        Logstash_Prefix fluentbit
        Replace_Dots On
        Generate_ID On
        Write_Operation upsert
        Trace_Error On
```

✔️ **Fluent-bit Output plugin List**
<br>
https://docs.fluentbit.io/manual/pipeline/outputs

<br>
<br>

😃 **Lab 1 완료!!!**

<br>

⏩ 다음 실습으로 [이동](4-2-Log_Collection_Visualization.md)합니다.