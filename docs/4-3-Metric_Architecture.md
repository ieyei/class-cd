# 4일차 Observability
## Lab3. Metric Architecture Installation

<br>

---
- [4일차 Observability](#4일차-observability)
  - [Lab3. Metric Architecture Installation](#lab3-metric-architecture-installation)
    - [3-1. Monitoring Stack 설치하기](#3-1-monitoring-stack-설치하기)
    - [3-2. Monitoring Stack 둘러보기](#3-2-monitoring-stack-둘러보기)
---

ⓘ 실습목표 : Metric Architecture를 설치하여 환경을 구성해본다.

---

### 3-1. Monitoring Stack 설치하기

▶ GitOps Console의 PaC Catalog 기능을 통해 Monitoring Stack을 설치한다.

<br>

✔️ **Monitoring Stack Catalog**

Monitoring Stack Catalog는 Helm Chart로 구성되어 있으며, 다음과 같은 sub chart들이 dependency 설정에 의해 함께 설치된다.

 - node-exporter
 - kube-state-metrics
 - metrics-server(kube-api)
 - prometheus
 - grafana

<br>

![](../images/4-3/4-3-1.svg)

<br>

3-1-1. GitOps Console에 접속하여 수강생 개인별 Workspace를 선택한다.

![](../images/4-3/4-3-2.svg)

<br>

3-1-2. 우측 상단의 `Create App` 버튼을 클릭한다.

![](../images/4-3/4-3-3.svg)

<br>

3-1-3. PaC Source Type에 `Catalog`를 선택하고, `Search Catalog` 버튼을 클릭한다.

![](../images/4-3/4-3-4.svg)

<br>

3-1-4. `Monitoring-stack`을 선택하고, `Select` 버튼을 클릭한다.

![](../images/4-3/4-3-5.svg)

<br>

3-1-5. 이름과 경로를 확인하고, `Create` 버튼을 클릭한다.

![](../images/4-3/4-3-6.svg)

<br>

3-1-6. 개인 Github Repository에 PaC Catalog 코드가 복사된다. `REPO_CREATED` 상태를 확인하고, `Monitoring-stack`을 리스트에서 클릭한다.

![](../images/4-3/4-3-7.svg)

<br>

3-1-7. Target Namespace에 `monitoring`을 입력하고 Select File 콤보박스에서 `values.yaml`을 선택한다.

![](../images/4-3/4-3-8.svg)

<br>

3-1-8. 입력값을 확인 후, 하단의 `Register` 버튼을 클릭한다.

![](../images/4-3/4-3-9.svg)

<br>

3-1-9. Source Code 탭을 선택하여 코드를 확인 후, 하단의 `Sync` 버튼을 클릭한다.

![](../images/4-3/4-3-10.svg)

<br>

3-1-10. 2~3분 후 Status가 `Synced`로 바뀌면 배포가 완료된 것이다.

![](../images/4-3/4-3-11.svg)

<br>

3-1-11. Monitoring-stack이 정상적으로 설치되었는지를 확인한다.

- Cloud9에서 kubectl 명령어를 통해 monitoring namespace에 POD가 정상적으로 설치되었는지 확인한다.

🧲 (COPY)
```bash
kubectl get pods -n monitoring
```

✔ **(수행코드/결과 예시)**
```bash
mspuser:~/environment $ kubectl get pods -n monitoring
NAME                                 READY   STATUS    RESTARTS   AGE
grafana-6b998d6d77-q8vjb             1/1     Running   0          15m
kube-state-metrics-f6df4999-zd95v    1/1     Running   0          15m
metrics-server-79598c8959-7njxv      1/1     Running   0          15m
node-exporter-6pbg4                  1/1     Running   0          15m
node-exporter-8kcnx                  1/1     Running   0          15m
node-exporter-ckwp5                  1/1     Running   0          15m
node-exporter-mr99k                  1/1     Running   0          15m
node-exporter-sjvkg                  1/1     Running   0          15m
prometheus-server-56566fcbf4-rn8tl   2/2     Running   0          15m
```

<br>

- 브라우저에서 아래 URL로 접속해본다.

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```yaml
접속 URL : https://www.<<YOUR_DOMAIN>>/grafana
```

- 아래와 같이 Grafana 로그인 UI가 보이면 정상적으로 설치된 것이다.

![](../images/4-3/4-3-12.svg)

<br>
<br>

---
### 3-2. Monitoring Stack 둘러보기

▶ Grafana에 접속하여 기본 사용법을 익히고, Monitoring Stack이 어떻게 연계되어 있는지 확인한다.

<br>

✔️ **Metrics Server 확인**

Official : https://github.com/kubernetes-sigs/metrics-server/tree/master/charts/metrics-server

<br>

3-2-1. Metrics-server가 정상적으로 설치되었는지를 확인한다.

- Cloud9에서 kubectl 명령어를 통해 Metrics API를 정상적으로 호출하는지 확인한다.

<br>

🧲 (COPY)
```bash
kubectl top nodes
```

✔ **(수행코드/결과 예시)**
```bash
mspuser:~/environment $ kubectl top nodes
NAME                                             CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
ip-10-0-20-121.ap-northeast-2.compute.internal   284m         3%     6941Mi          22%
ip-10-0-20-163.ap-northeast-2.compute.internal   277m         3%     5034Mi          16%
ip-10-0-30-132.ap-northeast-2.compute.internal   237m         2%     5494Mi          17%
ip-10-0-30-134.ap-northeast-2.compute.internal   96m          1%     1335Mi          4%
ip-10-0-30-56.ap-northeast-2.compute.internal    56m          0%     1003Mi          3%
```

<br>

🧲 (COPY)
```bash
kubectl top pods -n monitoring
```

✔ **(수행코드/결과 예시)**
```bash
mspuser:~/environment $ kubectl top pods -n monitoring
NAME                                  CPU(cores)   MEMORY(bytes)
grafana-64d54fb9bc-mt666              1m           42Mi
kube-state-metrics-84c56fbd9b-gcpm5   2m           15Mi
metrics-server-5fdc99fd68-8rdvt       3m           21Mi
node-exporter-hq46z                   3m           8Mi
node-exporter-npflj                   3m           8Mi
node-exporter-s2cck                   2m           8Mi
node-exporter-vqc86                   4m           9Mi
node-exporter-xvdv4                   2m           9Mi
prometheus-server-6f675dd857-f8cjm    30m          473Mi
```

<br>

3-2-2. Grafana에 접속하여 로그인한다.

✔️ **Grafana Document**

Official : https://grafana.com/docs/grafana/v10.2/

<br>

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```yaml
접속 URL : https://www.<<YOUR_DOMAIN>>/grafana
```

<br>

- 아래와 같은 로그인 화면이 나타난다. Username/Password를 입력 후 `Log in` 버튼을 클릭한다.

> |항목|내용|
> |---|---|
> ➕ Username | `admin` |
> ➕ Password | `admin` |

![](../images/4-3/4-3-13.svg)

<br>

- 로그인 후 아래와 같이 비밀번호 변경 화면이 나타나면 `Skip`을 클릭한다.

![](../images/4-3/4-3-14.svg)

<br>

3-2-3. `Data Sources` 메뉴에 접속하여 Prometheus가 잘 연결되어 있는지 확인한다.

✔️ **Prometheus Document**

Official : https://prometheus.io/docs/introduction/overview/

<br>

- 좌측 상단의 메뉴 아이콘을 선택하여 `[Connections] - [Data sources]`의 순서로 클릭하여 메뉴에 접속한다.

![](../images/4-3/4-3-15.svg)

<br>

- Data sources에 Prometheus가 등록된 것을 확인할 수 있다. Prometheus를 클릭한다.

![](../images/4-3/4-3-16.svg)

<br>

- Prometheus의 설정정보를 확인 후, 화면 하단의 `Save & test` 버튼을 클릭하여 연결이 잘 되었는지 확인한다.

![](../images/4-3/4-3-17.svg)
![](../images/4-3/4-3-18.svg)

<br>

- 아래의 그림과 같이 `Successfully queried the Prometheus API`라는 메시지가 보이면 성공적으로 연결된 것이다.

![](../images/4-3/4-3-19.svg)

<br>

3-2-4. `Node Exporter`의 Metrics을 조회하여 데이터를 잘 수집하는지 확인한다.

✔️ **Node-Exporter Github**

Official : https://github.com/prometheus/node_exporter/tree/v1.7.0?tab=readme-ov-file

<br>

- Cloud9에서 Curl 명령어를 통해 Metrics API를 정상적으로 호출하는지 확인한다.
- Node Exporter 서비스로 직접 Curl 명령을 요청할 수 없으므로 Grafana 파드를 활용한다.

<br>

🧲 (COPY)
```bash
kubectl -n monitoring exec -it $(kubectl -n monitoring get pods -l app.kubernetes.io/name=grafana -o jsonpath={.items..metadata.name}) -c grafana -- curl http://node-exporter.monitoring:9100/metrics
```

<br>

✔ **(수행코드/결과 예시)**
<details>
<summary>Click</summary>
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.<br>
# TYPE go_gc_duration_seconds summary<br>
go_gc_duration_seconds{quantile="0"} 1.4522e-05<br>
go_gc_duration_seconds{quantile="0.25"} 1.8951e-05<br>
go_gc_duration_seconds{quantile="0.5"} 2.345e-05<br>
go_gc_duration_seconds{quantile="0.75"} 3.6813e-05<br>
go_gc_duration_seconds{quantile="1"} 5.0363e-05<br>
go_gc_duration_seconds_sum 0.066665811<br>
go_gc_duration_seconds_count 2436<br>
# HELP go_goroutines Number of goroutines that currently exist.<br>
# TYPE go_goroutines gauge<br>
go_goroutines 8<br>
...(중략)<br>
# HELP node_cpu_seconds_total Seconds the CPUs spent in each mode.<br>
# TYPE node_cpu_seconds_total counter<br>
node_cpu_seconds_total{cpu="0",mode="idle"} 573032.3<br>
node_cpu_seconds_total{cpu="0",mode="iowait"} 132.12<br>
node_cpu_seconds_total{cpu="0",mode="irq"} 0<br>
node_cpu_seconds_total{cpu="0",mode="nice"} 0.2<br>
node_cpu_seconds_total{cpu="0",mode="softirq"} 251.73<br>
node_cpu_seconds_total{cpu="0",mode="steal"} 24.44<br>
node_cpu_seconds_total{cpu="0",mode="system"} 3377.1<br>
node_cpu_seconds_total{cpu="0",mode="user"} 7925.06<br>
...(중략)<br>
# HELP process_resident_memory_bytes Resident memory size in bytes.<br>
# TYPE process_resident_memory_bytes gauge<br>
process_resident_memory_bytes 2.0475904e+07<br>
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.<br>
# TYPE process_start_time_seconds gauge<br>
process_start_time_seconds 1.70555854859e+09<br>
# HELP process_virtual_memory_bytes Virtual memory size in bytes.<br>
# TYPE process_virtual_memory_bytes gauge<br>
process_virtual_memory_bytes 1.271967744e+09<br>
# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.<br>
# TYPE process_virtual_memory_max_bytes gauge<br>
process_virtual_memory_max_bytes 1.8446744073709552e+19<br>
# HELP promhttp_metric_handler_errors_total Total number of internal errors encountered by the promhttp metric handler.<br>
# TYPE promhttp_metric_handler_errors_total counter<br>
promhttp_metric_handler_errors_total{cause="encoding"} 0<br>
promhttp_metric_handler_errors_total{cause="gathering"} 0<br>
# HELP promhttp_metric_handler_requests_in_flight Current number of scrapes being served.<br>
# TYPE promhttp_metric_handler_requests_in_flight gauge<br>
promhttp_metric_handler_requests_in_flight 1<br>
# HELP promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code.<br>
# TYPE promhttp_metric_handler_requests_total counter<br>
promhttp_metric_handler_requests_total{code="200"} 796<br>
promhttp_metric_handler_requests_total{code="500"} 0<br>
promhttp_metric_handler_requests_total{code="503"} 0<br>
</details>

<br>

3-2-5. Prometheus에서 `Node Exporter`의 Metrics을 조회해본다.

- Grafana의 좌측 상단의 메뉴 아이콘을 선택하여 `Explore`메뉴에 접속한다.

![](../images/4-3/4-3-20.svg)

<br>

- `Metrics browser`에 node라고 입력했을 때, 아래와 같이 node_ 로 시작되는 metric의 리스트가 보인다면 node-exporter의 Metric 수집이 정상적으로 되고 있는 것이다.

![](../images/4-3/4-3-21.svg)

<br>

👉 실습화면과 다르게 `Metric browser`가 보이지 않는 경우, 화면 오른쪽에 `builder`로 선택되어있는 지 확인한다. `builder`로 선택되어 있다면 `Code`를 클릭한다.

![](../images/4-3/4-3-22.svg)

<br>

3-2-6. `Data Sources` 메뉴에 접속하여 Prometheus가 잘 연결되어 있는지 확인한다.

✔️ **Kube-State-Metrics Github**

Official : https://github.com/kubernetes/kube-state-metrics/tree/v2.10.1/docs

<br>

- `Metrics browser`에 kube라고 입력했을 때, 아래와 같이 kube_ 로 시작되는 metric의 리스트가 보인다면 kube-state-metrics의 Metric 수집이 정상적으로 되고 있는 것이다.

![](../images/4-3/4-3-23.svg)

<br>
<br>

---

### 3-3. PromQL

3-3-1. Basic

- Instant Vector

> Instant Vector란, 특정 시간에 하나의 시계열 데이터가 갖는 정보

```bash
kube_pod_info
```

<br>

- Label Matcher를 활용한 쿼리

> |Matcher|내용|
> |---|---|
> ➕ = | Label 값이 정확히 일치 |
> ➕ != | Label 값이 불일치 |
> ➕ =~ | Label 값에 대해서 정규표현식과 일치 |
> ➕ !~ | Label 값에 대해서 정규표현식과 불일치 |

```bash
kube_pod_info{namespace="monitoring"}
```

```bash
kube_pod_info{namespace="monitoring", pod=~"node-exporter.*"}
```

<br>

- Range Vector

> 지정된 시간 범위 내에 있는 정보의 집합

```bash
kube_pod_info{namespace="monitoring", pod=~"node-exporter.*"}[5m]
```

👉 지난 5분간 Pod Info의 평균 수 계산

<br>

3-3-2. Operator

```bash
node_cpu_seconds_total
```

<br>

- sum

```bash
sum(node_cpu_seconds_total)
```

<br>

- count

```bash
count(node_cpu_seconds_total)
```

<br>

- average

```bash
avg(node_cpu_seconds_total)
```

<br>

- min/max

```bash
min(node_cpu_seconds_total)
```

```bash
max(node_cpu_seconds_total)
```

<br>

3-3-3. Grouping

```bash
count(kube_pod_info) by (namespace)
```

<br>

3-3-4. Functions

- round

```bash
round(node_cpu_seconds_total)
```

```bash
round(node_cpu_seconds_total, 10)
```

```bash
round(node_cpu_seconds_total, 0.01)
```

<br>

- sort

```bash
sort(node_cpu_seconds_total)
```

```bash
sort_desc(node_cpu_seconds_total)
```

<br>

- rate

> 주어진 Range Vector에서 시간에 따른 평균 변화율을 계산

```bash
rate(node_cpu_seconds_total[5m])
```

<br>

- irate

> 주어진 Range Vector에서 가장 최근 두 데이터 포인트를 사용하여 순간 변화율 계산

```bash
irate(node_cpu_seconds_total[5m])
```

👉 시스템 전반적인 성능 추세를 모니터링하고자 한다면 `rate`, 짧은 시간 동안의 급격한 변화를 감지하고자 한다면 `irate`

<br>
<br>

😃 **Lab 3 완료!!!**

<br>

⏩ 다음 실습으로 [이동](4-4-Metric_Configuration.md)합니다.