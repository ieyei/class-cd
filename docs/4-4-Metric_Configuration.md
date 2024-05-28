# 4일차 Observability
## Lab4. Metric 수집 대상 추가하기

<br>

---
- [4일차 Observability](#4일차-observability)
  - [Lab4. Metric 수집 대상 추가하기](#lab4-metric-수집-대상-추가하기)
    - [4-1. Static 설정을 통한 Fluent-bit Metrics 수집하기](#4-1-static-설정을-통한-fluent-bit-metrics-수집하기)
    - [4-2. Fluent-bit 모니터링을 위한 Grafana 대시보드 설정하기](#4-2-fluent-bit-모니터링을-위한-grafana-대시보드-설정하기)
    - [4-3. Kubernetes Service Discovery 설정을 통한 Istio Metrics 수집하기](#4-3-kubernetes-service-discovery-설정을-통한-istio-metrics-수집하기)
    - [4-4. Istio 모니터링을 위한 Grafana 대시보드 설정하기](#4-4-istio-모니터링을-위한-grafana-대시보드-설정하기)
---

ⓘ 실습목표 : Metric을 수집하여 Grafana로 조회해보고 다양한 시각화 기능을 실습해본다.

---

✔️ **실습에 등록해볼 모니터링 대상**

- Fluent-bit Metrics
- Istio Metrics

---
### 4-1. Static 설정을 통한 Fluent-bit Metrics 수집하기

Official Documents : https://docs.fluentbit.io/manual/administration/monitoring

<br>

▶ Fluent-bit이 노출하는 Metrics을 Prometheus로 수집해보자.

![](../images/4-4/4-4-1.svg)

<br>

4-1-1. Fluent-bit POD에 접속하여 Metrics이 정상적으로 조회되는지 확인한다.

- Cloud9에서 kubectl 명령어를 통해 POD에 접속하여 API(/metrics)를 호출해본다.

🧲 (COPY)
```bash
kubectl -n monitoring exec -it $(kubectl -n monitoring get pods -l app.kubernetes.io/name=grafana -o jsonpath={.items..metadata.name}) -c grafana -- curl fluent-bit.logging:2020/api/v1/metrics/prometheus
```

<br>

✔ **(수행코드/결과 예시)**
<details>
<summary>Click</summary>
# HELP fluentbit_filter_add_records_total Fluentbit metrics.<br>
# TYPE fluentbit_filter_add_records_total counter<br>
fluentbit_filter_add_records_total{name="kubernetes.0"} 0 1706247687582<br>
# HELP fluentbit_filter_bytes_total Fluentbit metrics.<br>
# TYPE fluentbit_filter_bytes_total counter<br>
fluentbit_filter_bytes_total{name="kubernetes.0"} 18397 1706247687582<br>
# HELP fluentbit_filter_drop_records_total Fluentbit metrics.<br>
# TYPE fluentbit_filter_drop_records_total counter<br>
fluentbit_filter_drop_records_total{name="kubernetes.0"} 0 1706247687582<br>
# HELP fluentbit_filter_records_total Fluentbit metrics.<br>
# TYPE fluentbit_filter_records_total counter<br>
fluentbit_filter_records_total{name="kubernetes.0"} 32 1706247687582<br>
...중략...<br>
# HELP fluentbit_uptime Number of seconds that Fluent Bit has been running.<br>
# TYPE fluentbit_uptime counter<br>
fluentbit_uptime 79914<br>
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.<br>
# TYPE process_start_time_seconds gauge<br>
process_start_time_seconds 1706167773<br>
# HELP fluentbit_build_info Build version information.<br>
# TYPE fluentbit_build_info gauge<br>
fluentbit_build_info{version="2.2.1",edition="Community"} 1<br>
</details>

<br>

> 👉 Fluent-bit POD는 curl이 설치되어있지 않기 때문에 curl 명령어 수행이 가능한 Grafana POD로 실행해본다.

<br>

4-1-2. 이제 Fluent-bit Metrics를 Prometheus에서 수집할 수 있도록 Prometheus Scrape Config를 추가하자. GitOps Console에 접속하여 수강생 개인별 PaC Workspace 에서 monitoring-stack을 클릭한다.

![](../images/4-4/4-4-2.svg)

<br>

4-1-3. `Source code`탭을 클릭한다.

![](../images/4-4/4-4-3.svg)

<br>

4-1-4. 좌측 디렉토리 View에서 `values.yaml` 파일을 선택하면 우측 editor 창에 values.yaml의 값이 보여진다. Prometheus의 Metric Scrape Rule을 수정해보자.

- 우측 Text Edit 창의 아무곳이나 클릭한 후, `CTRL + G`를 누르면 line 이동 팝업이 나온다. `7`를 입력 후 `Go to line 7.`를 클릭하여 해당 라인으로 이동한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Line 이동 팝업 |`7`|🧲복사 & 📋붙여넣기|

![](../images/4-4/4-4-4.svg)

<br>

- `Line 7`의 `Prometheus` 하위에 아래와 같이 Scrape Config를 추가하고 `Save` 버튼을 클릭한다.

> |항목|내용|
> |---|---|
> ➕ 파일경로 | values.yaml |
> ➕ 수정위치 | 7 line 밑으로 scrape config 추가 |
> ➕ 수정내용 | 아래 Scarpe config를 🧲복사 & 📋붙여넣기 |

🧲 (COPY)
```yaml
  # Scrape Configs
  extraScrapeConfigs: |
    # scrape flunt-bit metrics
    - job_name: fluentbit
      static_configs:
        - targets:
          - fluent-bit.logging.svc.cluster.local:2020
      metrics_path: /api/v1/metrics/prometheus
```

> 👉 Fluent-bit metric을 수집하기 위한 scrape_config로 `static_configs`를 사용하였으며, target 등록시 Fluent-bit service 자원의 FQDN(Fully Qualified Domain Name)을 활용하였다.

![](../images/4-4/4-4-5.svg)

<br>

❗ scrape config 추가할 때, 반드시 indent를 함께 추가해야한다.

![](../images/4-4/4-4-6.svg)

<br>

4-1-5. 변경사항을 Github Repository에 반영할 수 있는 Commit 팝업을 확인할 수 있다. 아래와 같이 입력하고 `Commit` 버튼을 클릭한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Add Comment|`update fluent-bit scrape config`|🧲복사 & 📋붙여넣기|

![](../images/4-4/4-4-7.svg)

<br>

4-1-6. 화면 하단의 `Sync` 버튼을 클릭한다.

![](../images/4-4/4-4-8.svg)

<br>

4-1-7. `PaC Workspace` 목록으로 돌아온다.

- 잠시 뒤, `Synced` 상태로 바뀌면 정상적으로 배포가 완료된것이다.

![](../images/4-4/4-4-9.svg)

<br>

4-1-8. 잠시 뒤, Grafana UI Web 화면에서 Target 수집 여부를 확인할 수 있다.

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```yaml
접속 URL : https://www.<<YOUR_DOMAIN>>/grafana
```

<br>

- Grafana의 좌측 상단의 메뉴 아이콘을 선택하여 `Explore`메뉴에 접속한다.

![](../images/4-4/4-4-10.svg)

<br>

- `Metrics browser`에 `fluentbit`라고 입력했을 때, 아래와 같이 `fluentbit` 로 시작되는 metric의 리스트가 보인다면 Fluent-bit Metric 수집이 정상적으로 되고 있는 것이다.

![](../images/4-4/4-4-11.svg)

<br>

👉 실습화면과 다르게 `Metric browser`가 보이지 않는 경우, 화면 오른쪽에 `builder`로 선택되어있는 지 확인한다. `builder`로 선택되어 있다면 `Code`를 클릭한다.

![](../images/4-4/4-4-12.svg)

<br>
<br>

---
### 4-2. Fluent-bit 모니터링을 위한 Grafana 대시보드 설정하기

▶ Grafana는 데이터 시각화와 모니터링을 위한 오픈소스 플랫폼으로 사용자가 데이터를 쉽게 이해하고 해석할 수 있도록 다양한 Visualization 기능을 제공한다.
또한, Grafana Dashboard Community를 통해 사용자가 만든 대시보드를 공유하고 다운로드할 수 있는 플랫폼을 제공한다.

4-2-1. Grafana dashboard Community 홈페이지에 접속한다.

👉 접속 URL : <a href="https://grafana.com/grafana/dashboards/" target="_blank">https://grafana.com/grafana/dashboards/</a>

![](../images/4-4/4-4-13.svg)

<br>

4-2-2. 검색바에 `fluent`를 입력하고 엔터를 치면 관련 대시보드를 확인할 수 있다. 이중 `9104 - FluentBit`을 Grafana에 추가해보자. `9104 - FluentBit`을 클릭한다.

![](../images/4-4/4-4-14.svg)

<br>

4-2-3. Dashboard에 대한 Overview 설명과 지원하는 Grafana, Prometheus 버전 정보 등을 볼 수 있다.

- 우측에서 지원 가능한 버전 정보를 확인하고, `Copy ID to clipboard` 버튼을 클릭한다..

![](../images/4-4/4-4-15.svg)

<br>

4-2-4. Grafana에 접속하여 로그인한다.

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```yaml
접속 URL : https://www.<<YOUR_DOMAIN>>/grafana
```

- 아래와 같은 로그인 화면이 나타난다. Username/Password를 입력 후 `Log in` 버튼을 클릭한다.

> |항목|내용|
> |---|---|
> ➕ Username | `admin` |
> ➕ Password | `admin` |

![](../images/4-4/4-4-16.svg)

<br>

- 로그인 후 아래와 같이 비밀번호 변경 화면이 나타나면 `Skip`을 클릭한다.

![](../images/4-4/4-4-17.svg)

<br>

4-2-5. 아래와 같이 `Dashboard`메뉴에서 Fluent-bit Dashboard를 Import한다.

- Dashboard 메뉴로 접속한다.

![](../images/4-4/4-4-18.svg)

<br>

- `New`버튼을 클릭하고, `Import` 메뉴를 선택한다.

![](../images/4-4/4-4-19.svg)

<br>

- `Grafana.com dashboard URL or ID` 영역을 클릭하여 `Ctrl + V`를 명령어로 이전에 복사한 대시보드 ID를 입력하고, `Load` 버튼을 클릭한다.

![](../images/4-4/4-4-20.svg)

<br>

❗ 클립보드가 사라졌을 경우, 아래의 숫자를 대신 입력한다.

🧲 (COPY & Modify)
```yaml
13573
```

- `Import` 버튼을 클릭하여 대시보드를 불러온다.

![](../images/4-4/4-4-21.svg)

<br>

> 👉 아래의 같은 메시지가 발생하는 경우, Datasource를 다시 설정해야 한다.

![](../images/4-4/4-4-22.svg)

<br>

- Grafana 상단의 톱니바퀴 icon을 클릭하여 대시보드 설정 메뉴로 접속한다.

![](../images/4-4/4-4-23.svg)

<br>

- 좌측 메뉴에서 Variables를 클릭하여 `Variables` 메뉴를 접속한다.

![](../images/4-4/4-4-24.svg)

<br>

- Variables 목록 중, `database`를 클릭한다.

![](../images/4-4/4-4-25.svg)

<br>

- `Select variable type`을 콤보박스에서 `Data source`를 다시 선택하고, Data source options 항목에서 `Type` 콤보박스를 `Prometheus`로 선택한다.

![](../images/4-4/4-4-26.svg)

<br>

- `Run query`버튼을 클릭하여 설정에 대한 Preview를 확인한다.

![](../images/4-4/4-4-27.svg)

<br>

- `Apply`버튼을 클릭하여 변경사항을 저장한다.

![](../images/4-4/4-4-28.svg)

<br>

4-2-6. 만들어진 Dashboard를 확인해본다.

- Dashboard를 통해 Fluent-bit의 성능을 모니터링할 수 있다.

![](../images/4-4/4-4-29.svg)

<br>
<br>

----
### 4-3. Kubernetes Service Discovery 설정을 통한 Istio Metrics 수집하기

Official Documents : https://istio.io/latest/docs/ops/integrations/prometheus/

<br>

▶ Istio가 노출하는 Metrics을 Prometheus로 수집해보자.

![](../images/4-4/4-4-50.svg)

<br>

4-3-1. Istiod POD에 접속하여 Metrics이 정상적으로 조회되는지 확인한다.

- Cloud9에서 kubectl 명령어를 통해 POD에 접속하여 API(/metrics)를 호출해본다.

🧲 (COPY)
```bash
kubectl -n istio-system exec -it $(kubectl -n istio-system get pods -l app=istiod -o jsonpath={.items[0]..metadata.name}) -- curl localhost:15014/metrics
```

<br>

✔ **(수행코드/결과 예시)**
<details>
<summary>Click</summary>
# HELP citadel_server_csr_count The number of CSRs received by Citadel server.<br>
# TYPE citadel_server_csr_count counter<br>
citadel_server_csr_count 21<br>
# HELP citadel_server_root_cert_expiry_timestamp The unix timestamp, in seconds, when Citadel root cert will expire. A negative time indicates the cert is expired.<br>
# TYPE citadel_server_root_cert_expiry_timestamp gauge<br>
citadel_server_root_cert_expiry_timestamp 2.021266664e+09<br>
# HELP citadel_server_success_cert_issuance_count The number of certificates issuances that have succeeded.<br>
# TYPE citadel_server_success_cert_issuance_count counter<br>
citadel_server_success_cert_issuance_count 21<br>
# HELP endpoint_no_pod Endpoints without an associated pod.<br>
# TYPE endpoint_no_pod gauge<br>
endpoint_no_pod 0<br>
...중략...<br>
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.<br>
# TYPE process_start_time_seconds gauge<br>
process_start_time_seconds 1.70590666407e+09<br>
# HELP process_virtual_memory_bytes Virtual memory size in bytes.<br>
# TYPE process_virtual_memory_bytes gauge<br>
process_virtual_memory_bytes 1.344413696e+09<br>
# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.<br>
# TYPE process_virtual_memory_max_bytes gauge<br>
process_virtual_memory_max_bytes 1.8446744073709552e+19<br>
# HELP webhook_patch_attempts_total Webhook patching attempts<br>
# TYPE webhook_patch_attempts_total counter<br>
webhook_patch_attempts_total{name="sidecar-injector.istio.io"} 1<br>
</details>

<br>

4-3-2. 이제 Istio Metrics를 Prometheus에서 수집할 수 있도록 Prometheus Scrape Config를 추가하자. GitOps Console의 PaC Workspace `t3-cta` 에서 monitoring-stack을 클릭한다.

![](../images/4-4/4-4-2.svg)

<br>

4-3-3. `Source code`탭을 클릭한다.

![](../images/4-4/4-4-3.svg)

<br>

4-3-4. 좌측 디렉토리 View에서 `values.yaml` 파일을 선택하면 우측 editor 창에 values.yaml의 값이 보여진다. Prometheus의 Metric Scrape Rule을 수정해보자.

- 우측 Text Edit 창의 아무곳이나 클릭한 후, `CTRL + G`를 누르면 line 이동 팝업이 나온다. `7`를 입력 후 `Go to line 7.`를 클릭하여 해당 라인으로 이동한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Line 이동 팝업 |`7`|🧲복사 & 📋붙여넣기|

![](../images/4-4/4-4-30.svg)

<br>

- `Line 7`의 `Prometheus` 하위에 아래와 같이 Scrape Config를 추가하고 `Save` 버튼을 클릭한다.

> |항목|내용|
> |---|---|
> ➕ 파일경로 | values.yaml |
> ➕ 수정위치 | extraScrapeConfigs 하위에 scrape config 추가 |
> ➕ 수정내용 | 아래 Scarpe config를 🧲복사 & 📋붙여넣기 |

🧲 (COPY)
```yaml
    # scrape istiod metrics
    - job_name: 'istiod'
      kubernetes_sd_configs:
      - role: endpoints
        namespaces:
          names:
          - istio-system
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: istiod;http-monitoring
    # scrape envoy metrics
    - job_name: 'envoy-stats'
      metrics_path: /stats/prometheus
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_container_port_name]
        action: keep
        regex: '.*-envoy-prom'
```

<br>

> 👉 Istio metric을 수집하기 위한 scrape_config로 `kubernetes_sd_configs`를 사용하였으며, regex를 활용하여 Istio 컴포넌트들을 조회할 수 있게 설정하였다.

![](../images/4-4/4-4-31.svg)

<br>

❗ scrape config 추가할 때, 반드시 indent를 함께 추가해야한다.

![](../images/4-4/4-4-32.svg)

<br>

4-3-5. 변경사항을 Github Repository에 반영할 수 있는 Commit 팝업을 확인할 수 있다. 아래와 같이 입력하고 `Commit` 버튼을 클릭한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Add Comment|`update istio scrape config`|🧲복사 & 📋붙여넣기|

![](../images/4-4/4-4-33.svg)

<br>

4-3-6. 화면 하단의 `Sync` 버튼을 클릭한다.

![](../images/4-4/4-4-34.svg)

<br>

4-3-7. `PaC Workspace` 목록으로 돌아온다.

- 잠시 뒤, `Synced` 상태로 바뀌면 정상적으로 배포가 완료된것이다.

![](../images/4-4/4-4-9.svg)

<br>

4-3-8. 잠시 뒤, Grafana UI Web 화면에서 Target 수집 여부를 확인할 수 있다.

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```yaml
접속 URL : https://www.<<YOUR_DOMAIN>>/grafana
```

<br>

- Grafana의 좌측 상단의 메뉴 아이콘을 선택하여 `Explore`메뉴에 접속한다.

![](../images/4-4/4-4-10.svg)

<br>

- `Metrics browser`에 `istio`라고 입력했을 때, 아래와 같이 `istio` 로 시작되는 metric의 리스트가 보인다면 Fluent-bit Metric 수집이 정상적으로 되고 있는 것이다.

![](../images/4-4/4-4-35.svg)

<br>

👉 실습화면과 다르게 `Metric browser`가 보이지 않는 경우, 화면 오른쪽에 `builder`로 선택되어있는 지 확인한다. `builder`로 선택되어 있다면 `Code`를 클릭한다.

![](../images/4-4/4-4-12.svg)

<br>
<br>

---
### 4-4. Istio 모니터링을 위한 Grafana 대시보드 설정하기

▶ Community에서 공유되는 Dashboards로는 모니터링에 한계가 있을 수 있다. 대시보드를 직접 구현하여 나만의 대시보드를 구성해본다.

<br>

4-4-1. 아래와 같이 `Dashboard`메뉴에 접속하여 `+ Create Dashboard` 버튼을 클릭한다.

![](../images/4-4/4-4-18.svg)

<br>

![](../images/4-4/4-4-36.svg)

<br>

4-4-2. `+ Add visualization` 버튼을 클릭한다.

![](../images/4-4/4-4-37.svg)

<br>

4-4-3. `Select data source` 화면에서 `Prometheus`를 Datasource로 선택한다.

![](../images/4-4/4-4-38.svg)

<br>

4-4-4. `Visualization`을 작성하는 기본화면은 다움과 같다.

![](../images/4-4/4-4-39.svg)

  ① Panel Preview<br>
  ② Visualization에 사용할 원천 데이터 구성<br>
  ③ Visualization 속성<br>

<br>

4-4-5. Istio-Ingressgateway의 `CPU`를 모니터링하는 Visualization을 만들어보자.

- `Metrics Browser`에 `container_cpu_usage_seconds_total`을 입력해본다.

>|항목|내용|액션|
>|---|---|---|
>➕ Metrics Browser|`container_cpu_usage_seconds_total`|🧲복사 & 📋붙여넣기|

- `Preview`영역에 Graph가 표시되는 것을 확인할 수 있다.

- `Metrics Browser`하단에 `Selected metric is a counter`라고 메시지가 뜬 것을 확인할 수 있다. 이것은 CPU를 나타내는 지표 특성상 정적 값이 아닌, 변동량을 의미한다.

![](../images/4-4/4-4-40.svg)

<br>

- `rate()`` 함수를 사용하여 아래와 같이 쿼리를 변경한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Metrics Browser|`rate(container_cpu_usage_seconds_total[5m])`|🧲복사 & 📋붙여넣기|

![](../images/4-4/4-4-41.svg)
<br>

> 👉 Metric 관련 메시지가 사라진 것을 확인할 수 있다.

<br>

4-4-6. Filter를 적용해보자.

- Istio-Ingressgateway POD의 CPU를 측정하기 위해, `Namespace`와 `Pod`, `Container` 조건을 추가한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Metrics Browser|`rate(container_cpu_usage_seconds_total{namespace="istio-system", pod=~"istio-ingress.*", container!=""}[5m])`|🧲복사 & 📋붙여넣기|

![](../images/4-4/4-4-42.svg)

<br>

4-4-7. Alias를 적용해보자.

- `Metrics Browser`하단의 `Options`를 클릭하고, Legend의 `Auto` 영역을 클릭한다.

![](../images/4-4/4-4-43.svg)

<br>

- 팝업메뉴에서 `Custom`을 선택한다.

![](../images/4-4/4-4-44.svg)

<br>

- Legend 영역에 `pod`를 입력한다.

![](../images/4-4/4-4-45.svg)

<br>

- 범례가 `Pod`명으로 변경되었음을 확인할 수 있다.

![](../images/4-4/4-4-46.svg)

<br>

4-4-8. (실습)Istio-Ingressgateway의 `Memory`를 모니터링하는 Visualization을 만들어보자.

> 👉 `container_memory_working_set_bytes` 메트릭을 활용하여 구성해본다.

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

---

<br>

✔️ **Istio Dashboard Import 하기**

- `Dashboard`메뉴에 접속한다.

![](../images/4-4/4-4-18.svg)

<br>

- `New`버튼을 클릭하고, `Import` 메뉴를 선택한다.

![](../images/4-4/4-4-19.svg)

<br>

- 하단 링크의 JSON 파일을 복사하여 붙여넣기 하고, `Load` 버튼을 클릭한다.

[Dashboard Json](../etc/dashboards.json)

<br>

![](../images/4-4/4-4-47.svg)

<br>

- `Name`을 `CTA Monitoring`이라 입력하고 `Import`버튼을 클릭한다.

>|항목|내용|액션|
>|---|---|---|
>➕ Name|`CTA Monitoring`|🧲복사 & 📋붙여넣기|

![](../images/4-4/4-4-48.svg)

<br>

- `CTA Monitoring` 대시보드를 확인해본다.

![](../images/4-4/4-4-49.svg)

<br>
<br>

😃 **Lab 4 완료!!!**

<br>

⏩ 다음 실습으로 [이동](4-5-Notification.md)합니다.