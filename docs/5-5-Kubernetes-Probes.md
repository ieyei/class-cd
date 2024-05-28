# 5일차 Availability
## Lab5. Kubernetes Probes를 통한 Health check 실습

<br>

---
- [5일차 Availability](#5일차-availability)
  - [Lab5. Kubernetes Probes를 통한 Health check 실습](#lab5-kubernetes-probes를-통한-health-check-실습)
    - [5-1. command 기반의 LivenessProbe 실습](#5-1-command-기반의-livenessprobe-실습)
    - [5-2. httpGet 기반의 LivenessProbe 실습](#5-2-httpget-기반의-livenessprobe-실습)
    - [5-3. StartupProbe 실습](#5-3-startupprobe-실습)
    - [5-4. ReadinessProbe 실습](#5-4-readinessprobe-실습)
---

ⓘ 실습목표 : Kubernetes의 LivenessProbe, ReadinessProbe, StartupProbe 사용법을 실습해본다.

---

### 5-1. command 기반의 LivenessProbe 실습

![](../images/5-5/5-5-3.svg)

👉 LivenessProbe는 Pod 내 컨테이너가 정상적으로 작동하고 있는지 확인하기 위한 설정으로, Probe 검증 실패 시, 컨테이너를 재시작한다.

<br>

5-1-1. Cloud9에서 실행하여 실습용 디렉토리 생성

- Terminal에서 실습용 디렉토리 생성

```bash
cd ${HOME}/environment
```

```bash
mkdir k8s-probe
```

```bash
cd k8s-probe
```

<br>

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ mkdir k8s-probe
mspuser:~/environment $ cd k8s-probe
mspuser:~/environment/k8s-probe $
```

<br>

5-1-2. 테스트를 위한 liveness yaml 정의

- Cloud9 왼쪽의 EXPLORER 에서 k8s-probe 폴더 우클릭 후 > New Files를 누른다.

![](../images/5-5/5-5-1.svg)

<br>

- 파일명은 `liveness.yaml`로 입력하고 엔터를 누른다.

![](../images/5-5/5-5-2.svg)

<br>

- 아래 내용을 그대로 복사하여 파일 내용에 붙여넣고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

<br>

5-1-3. liveness pod를 배포하고, 확인 후 삭제

- liveness pod 배포

```bash
kubectl apply -f liveness.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl apply -f liveness.yaml
pod/liveness created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
liveness                1/1     Running   0          30s
```

<br>

- 잠시 후 pod 상태를 다시 확인한다.

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get pods
NAME                    READY   STATUS    RESTARTS      AGE
liveness                1/1     Running   1 (19s ago)   94s
```

<br>

- pod의 detail 정보를 확인해본다.

```bash
kubectl describe pods liveness
```

✔ **(수행코드/결과 예시)**

![](../images/5-5/5-5-4.svg)

👉 Container가 종료된 이력과, Restarting된 Event를 확인할 수 있다.

<br>

- liveness pod를 삭제한다.

```bash
kubectl delete -f liveness.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe$ kubectl delete -f liveness.yaml
pod "liveness" deleted
```

<br>
<br>

---

### 5-2. httpGet 기반의 LivenessProbe 실습

5-2-1. 테스트를 위한 liveness yaml 정의

- Cloud9 왼쪽의 EXPLORER 에서 k8s-probe 폴더 우클릭 후 > New Files를 누른다.

![](../images/5-5/5-5-1.svg)

<br>

- 파일명은 `liveness-http.yaml`로 입력하고 엔터를 누른다.

![](../images/5-5/5-5-5.svg)

<br>

- 아래 내용을 그대로 복사하여 파일 내용에 붙여넣고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

<br>

👉 liveness 컨테이너는 `healthz` api에 대해 최초 10초 동안 200을 응답하고, 그 이후로는 500을 리턴한다.

```java
http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
    duration := time.Now().Sub(started)
    if duration.Seconds() > 10 {
        w.WriteHeader(500)
        w.Write([]byte(fmt.Sprintf("error: %v", duration.Seconds())))
    } else {
        w.WriteHeader(200)
        w.Write([]byte("ok"))
    }
})
```

<br>

5-2-2. liveness-http pod를 배포하고, 확인 후 삭제

- liveness pod 배포

```bash
kubectl apply -f liveness-http.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl apply -f liveness-http.yaml
pod/liveness-http created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
liveness-http           1/1     Running   0          30s
```

<br>

- 잠시 후 pod 상태를 다시 확인한다.

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get pods
NAME                    READY   STATUS    RESTARTS      AGE
liveness-http           1/1     Running   1 (19s ago)   94s
```

<br>

- pod의 detail 정보를 확인해본다.

```bash
kubectl describe pods liveness-http
```

✔ **(수행코드/결과 예시)**

![](../images/5-5/5-5-6.svg)

👉 Container가 종료된 이력과, Restarting된 Event를 확인할 수 있다.

<br>

- liveness-http pod를 삭제한다.

```bash
kubectl delete -f liveness-http.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl delete -f liveness-http.yaml
pod "liveness-http" deleted
```

<br>
<br>

---

### 5-3. StartupProbe 실습

<br>

![](../images/5-5/5-5-8.svg)

👉 LivenessProbe는 Container가 기동된 후에 정상 응답을 얻을 수 있다. 만약 Container가 Running 상태가 되기 전에 LivenessProbe 체크가 이뤄진다면 컨테이너가 계속 재기동하는 상황이 발생하게 된다.

<br>

5-3-1. 테스트를 위한 startup yaml 정의

- Cloud9 왼쪽의 EXPLORER 에서 k8s-probe 폴더 우클릭 후 > New Files를 누른다.

![](../images/5-5/5-5-1.svg)

<br>

- 파일명은 `startup.yaml`로 입력하고 엔터를 누른다.

![](../images/5-5/5-5-7.svg)

<br>

- 아래 내용을 그대로 복사하여 파일 내용에 붙여넣고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: spring-petclinic
spec:
  hosts:
  - "*"
  gateways:
  - istio-system/istio-gateway
  http:
  - match:
    - uri:
        prefix: /
    rewrite:
      uri: /
    route:
    - destination:
        host: spring-petclinic
        port:
          number: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: spring-petclinic
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    app: petclinic
---
apiVersion: v1
kind: Pod
metadata:
  name: spring-petclinic
  labels:
    app: petclinic
spec:
  containers:
  - name: petclinic
    image: springcommunity/spring-framework-petclinic
    ports:
    - containerPort: 8080
    env:
      - name: profile
        value: mysql
    livenessProbe:
      httpGet:
        path: /  # 또는 애플리케이션의 특정 경로
        port: 8080
      periodSeconds: 10
      failureThreshold: 1
  - name: mysql
    image: mysql:8.2
    env:
      - name: MYSQL_USER
        value: petclinic
      - name: MYSQL_PASSWORD
        value: petclinic
      - name: MYSQL_ROOT_PASSWORD
        value: root
      - name: MYSQL_DATABASE
        value: petclinic
    ports:
    - containerPort: 3306
```

<br>

5-3-2. startup.yaml을 배포하여 pod의 상태를 확인한다.

- startup.yaml 배포

```bash
kubectl apply -f startup.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl apply -f startup.yaml
virtualservice.networking.istio.io/spring-petclinic created
service/spring-petclinic created
pod/spring-petclinic created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
spring-petclinic        2/2     Running   0          6m43s
```

<br>

5-3-3. Sample Application이 잘 작동하는지 웹페이지를 통해 접속해본다.

❗ `<<YOUR_DOMAIN>>`의 값은 수강생 개인별 Route53 click 도메인으로 수정한다.

🧲 (COPY & Modify)
```yaml
접속 URL : https://www.<<YOUR_DOMAIN>>/
```

<br>

- 아래와 같은 화면이 나타나면 Application이 정상 작동하는 것이다.

![](../images/5-5/5-5-15.svg)

<br>

5-3-4. pod를 삭제하고, `startup.yaml`에서 LivenessProbe의 `periodSeconds`값을 `1`로 수정해본다.

```bash
kubectl delete -f startup.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl delete -f startup.yaml
virtualservice.networking.istio.io "spring-petclinic" deleted
service "spring-petclinic" deleted
pod "spring-petclinic" deleted
```

<br>

🧲 (COPY)
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: spring-petclinic
spec:
  hosts:
  - "*"
  gateways:
  - istio-system/istio-gateway
  http:
  - match:
    - uri:
        prefix: /
    rewrite:
      uri: /
    route:
    - destination:
        host: spring-petclinic
        port:
          number: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: spring-petclinic
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    app: petclinic
---
apiVersion: v1
kind: Pod
metadata:
  name: spring-petclinic
  labels:
    app: petclinic
spec:
  containers:
  - name: petclinic
    image: springcommunity/spring-framework-petclinic
    ports:
    - containerPort: 8080
    env:
      - name: profile
        value: mysql
    livenessProbe:
      httpGet:
        path: /  # 또는 애플리케이션의 특정 경로
        port: 8080
      periodSeconds: 1
      failureThreshold: 1
  - name: mysql
    image: mysql:8.2
    env:
      - name: MYSQL_USER
        value: petclinic
      - name: MYSQL_PASSWORD
        value: petclinic
      - name: MYSQL_ROOT_PASSWORD
        value: root
      - name: MYSQL_DATABASE
        value: petclinic
    ports:
    - containerPort: 3306
```

<br>

5-3-5. startup.yaml을 배포하여 pod의 상태를 확인한다.

- startup.yaml 배포

```bash
kubectl apply -f startup.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl apply -f startup.yaml
virtualservice.networking.istio.io/spring-petclinic created
service/spring-petclinic created
pod/spring-petclinic created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
spring-petclinic        2/2     Running   0          3s
```

<br>

- 잠시 후 pod 상태를 다시 확인한다.

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get pods
NAME                    READY   STATUS    RESTARTS      AGE
spring-petclinic        2/2     Running   1 (3s ago)    10s
```

<br>

- pod의 detail 정보를 확인해본다.

```bash
kubectl describe pods spring-petclinic
```

✔ **(수행코드/결과 예시)**

![](../images/5-5/5-5-9.svg)

👉 Container가 종료된 이력과, Restarting된 Event를 확인할 수 있다.

<br>

5-3-6. pod를 삭제하고, `startup.yaml`에서 startupProbe를 추가하여 배포해본다.

```bash
kubectl delete -f startup.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl delete -f startup.yaml
virtualservice.networking.istio.io "spring-petclinic" deleted
service "spring-petclinic" deleted
pod "spring-petclinic" deleted
```

<br>

🧲 (COPY)
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: spring-petclinic
spec:
  hosts:
  - "*"
  gateways:
  - istio-system/istio-gateway
  http:
  - match:
    - uri:
        prefix: /
    rewrite:
      uri: /
    route:
    - destination:
        host: spring-petclinic
        port:
          number: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: spring-petclinic
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    app: petclinic
---
apiVersion: v1
kind: Pod
metadata:
  name: spring-petclinic
  labels:
    app: petclinic
spec:
  containers:
  - name: petclinic
    image: springcommunity/spring-framework-petclinic
    ports:
    - containerPort: 8080
    env:
      - name: profile
        value: mysql
    livenessProbe:
      httpGet:
        path: /  # 또는 애플리케이션의 특정 경로
        port: 8080
      periodSeconds: 1
      failureThreshold: 1
    startupProbe:
      httpGet:
        path: /
        port: 8080
      periodSeconds: 5
  - name: mysql
    image: mysql:8.2
    env:
      - name: MYSQL_USER
        value: petclinic
      - name: MYSQL_PASSWORD
        value: petclinic
      - name: MYSQL_ROOT_PASSWORD
        value: root
      - name: MYSQL_DATABASE
        value: petclinic
    ports:
    - containerPort: 3306
```

<br>

- startup.yaml 배포

```bash
kubectl apply -f startup.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl apply -f startup.yaml
virtualservice.networking.istio.io/spring-petclinic created
service/spring-petclinic created
pod/spring-petclinic created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
spring-petclinic        1/2     Running   0          3s
```

<br>

- 잠시 후 pod 상태를 다시 확인한다.

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get pods
NAME                    READY   STATUS    RESTARTS      AGE
spring-petclinic        2/2     Running   0             25s
```

<br>

- pod의 detail 정보를 확인해본다.

```bash
kubectl describe pods spring-petclinic
```

✔ **(수행코드/결과 예시)**

![](../images/5-5/5-5-10.svg)

👉 Startup probe가 failed이 발생하기도 하지만 Container는 Restart하지 않은 것을 확인할 수 있다.

<br>

- 현재 상태를 그림으로 파악해보면 다음과 같다.

<br>

![](../images/5-5/5-5-11.svg)

<br>
<br>

---

### 5-4. ReadinessProbe 실습

<br>

![](../images/5-5/5-5-13.svg)

👉 ReadinessProbe는 Pod 내 컨테이너가 트래픽을 수신할 준비가 되어있는지 확인하기 위한 설정으로, Probe 검증 실패 시, SVC와 연계되는 Enpoint는 생성되지 않는다.

<br>

5-4-1. 테스트를 위한 readiness yaml 정의

- Cloud9 왼쪽의 EXPLORER 에서 k8s-probe 폴더 우클릭 후 > New Files를 누른다.

![](../images/5-5/5-5-1.svg)

<br>

- 파일명은 `readiness.yaml`로 입력하고 엔터를 누른다.

![](../images/5-5/5-5-12.svg)

<br>

- 아래 내용을 그대로 복사하여 파일 내용에 붙여넣고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    test: readiness
  name: readiness
spec:
  replicas: 2
  selector:
    matchLabels:
      test: readiness
  template:
    metadata:
      labels:
        test: readiness
    spec:
      containers:
        - name: readiness
          image: registry.k8s.io/busybox
          args:
          - /bin/sh
          - -c
          - touch /tmp/healthy; sleep 600
          readinessProbe:
            exec:
              command:
              - cat
              - /tmp/healthy
            initialDelaySeconds: 5
            periodSeconds: 5
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  labels:
    test: readiness
  name: readiness
spec:
  ports:
    - name: http
      port: 80
  selector:
    test: readiness
  type: ClusterIP
```

<br>

5-4-2. readiness.yaml을 배포하여 pod의 상태를 확인한다.

- readiness pod 배포

```bash
kubectl apply -f readiness.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl apply -f readiness.yaml
deployment.apps/readiness created
service/readiness created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
readiness-7d94fcc5bd-c7q97   1/1     Running   0          43s
readiness-7d94fcc5bd-q78cc   1/1     Running   0          43s
```

<br>

5-4-3. readiness의 endpoint를 조회해본다.

```bash
kubectl get endpoints
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get endpoints
NAME                           ENDPOINTS                                            AGE
readiness                      10.0.30.209:80,10.0.30.221:80                        2m54s
```

<br>

- 조회된 endpoint와 pod의 ip주소가 같은지 확인한다.

```bash
kubectl get pods -o wide
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE     IP            NODE                                             NOMINATED NODE   READINESS GATES
readiness-7d94fcc5bd-c7g26   1/1     Running   0          3m51s   10.0.30.221   ip-10-0-30-56.ap-northeast-2.compute.internal    <none>           <none>
readiness-7d94fcc5bd-mcwdl   1/1     Running   0          3m51s   10.0.30.209   ip-10-0-30-134.ap-northeast-2.compute.internal   <none>           <none>
```

<br>

5-4-4. readiness pod 둘 중 하나의 pod에서 `/tmp/healthy` 파일을 삭제한다.

```bash
kubectl exec -it $(kubectl get pods -l test=readiness -o jsonpath={.items[0]..metadata.name}) -- rm /tmp/healthy
```

✔ **(수행코드/결과 예시)**

```bash
kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
readiness-7d94fcc5bd-c7g26   0/1     Running   0          6m52s
readiness-7d94fcc5bd-mcwdl   1/1     Running   0          6m52s
```

<br>

5-4-5. pod의 detail 정보를 확인해본다.

```bash
kubectl describe pods $(kubectl get pods -l test=readiness -o jsonpath={.items[0]..metadata.name})
```

✔ **(수행코드/결과 예시)**

![](../images/5-5/5-5-14.svg)

👉 컨테이너의 `Ready` 상태 확인 결과, `False`임을 확인할 수 있다. 그리고 Readiness Probe의 실패 Event도 확인할 수 있다.

<br>

5-4-6. service의 endpoint 정보를 확인해본다.

```bash
kubectl get endpoints
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl get endpoints
NAME                           ENDPOINTS                             AGE
readiness                      10.0.30.209:80                        2m54s
```

👉 서비스의 Endpoint가 삭제됨으로써 Traffic을 전달받을 수 없는 상태임을 확인할 수 있다.

<br>

- readiness pod를 삭제한다.

```bash
kubectl delete -f readiness.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-probe $ kubectl delete -f readiness.yaml
deployment.apps "readiness" deleted
service "readiness" deleted
```

<br>
<br>

---

⏩ [실습] Tomcat 컨테이너를 배포해보고, LivenessProbe를 적용해보자.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tomcat
spec:
  containers:
  - name: tomcat
    image: tomcat
    ports:
    - containerPort: 8080
      name: http
```

👍👍 hint!!
>|항목|내용|
>|---|---|
>➕ health uri|`/`|
>➕ health port|`8080`|

<br>
<br>

😃 **Lab 5 완료!!!**

<br>

⏩ 다음 실습으로 [이동](5-6-Kubernetes-Volumes.md)합니다.