# 3일차 - Lab 3. Gitops Console Pac (2)

ⓘ 실습목표 : Gitops console을 이용해 prometheus와 kiali를 설치합니다.

---

- [3일차 - Lab 3. Gitops Console Pac (2)](<#3일차---lab-3.-gitops-console-pac-(2)>)
  - [1. prometheus 설치](#🔴-1.-prometheus-설치)
  - [2. kiali](#🔴-2.-kiali)

---

## 🔴 1. prometheus 설치

- 생성되어있는 PaC Workspace를 이용하여 prometheus를 설치합니다.

### ✔ 1-1. App 설치 화면 진입

- `Gitops Console` > `Workspace` > `PaC Workspace` > 생성한 `cta-2024-pac` workspace > `Create App`

**📌 [입력]**

> | 항목               | 내용                                 | 액션              |
> | ------------------ | ------------------------------------ | ----------------- |
> | ➕ PaC Source Type | `Catalog`                            | 👆🏻라디오버튼 선택 |
> | ➕ Title           | `Search Catalog` > `prometheus` 선택 |                   |

- 입력후 Create 클릭

### ✔ 1-2. 생성된 PaC App 확인

![](../images/175.png)

### ✔ 1-3. prometheus Register

- `cta-2024-pac` workspace의 `prometheus`를 클릭하여 pac detail 페이지로 진입합니다.

**📌 [입력]**

> | 항목                | 내용                       | 액션                |
> | ------------------- | -------------------------- | ------------------- |
> | ➕ Target Namespace | `default`                  | 🧲복사 & 📋붙여넣기 |
> | ➕ Select File      | `override-vaules-sds.yaml` | 👆🏻셀렉트박스 선택   |

- `Register` 버튼 클릭

### ✔ 1-4. prometheus sync

- `Sync` 버튼 클릭

### ✔ 1-5. PaC workspace Page에서 prometheus 확인

- 몇분정도 기다린후
- PaC workspace Page에서 새로고침을 눌러 App Health가 `Healthy`로 되는 것을 확인한다.

![](../images/176.png)

### ✔ 1-6. kubectl을 이용한 prometheus 확인

- Cloud9에서 아래 명령어를 수행

```bash
kubectl get all | grep prometheus
```

- 결과 예시

```bash
mspmanager:~/environment/manifests $ kubectl get all | grep prometheus
pod/prometheus-server-774b9b4964-chlfg    2/2     Running   0          89s
service/prometheus-server    ClusterIP   172.20.157.79   <none>        80/TCP     89s
deployment.apps/prometheus-server    1/1     1            1           89s
replicaset.apps/prometheus-server-774b9b4964    1         1         1       89s
```

<br>

## 🔴 2. kiali

- 이번에는 PaC Workspace를 이용하여 kiali 설치합니다.

### ✔ 2-1. App 설치 화면 진입

- `Gitops Console` > `Workspace` > `PaC Workspace` > 생성한 `cta-2024-pac` workspace > `Create App`

**📌 [입력]**

> | 항목               | 내용                                   | 액션              |
> | ------------------ | -------------------------------------- | ----------------- |
> | ➕ PaC Source Type | `Catalog`                              | 👆🏻라디오버튼 선택 |
> | ➕ Title           | `Search Catalog` > `kiali-server` 선택 |                   |

- 입력후 Create 클릭

### ✔ 2-2. 생성된 PaC App 확인

![](../images/177.png)

### ✔ 2-3. kiali Register

- `cta-2024-pac` workspace의 `kiali`를 클릭하여 pac detail 페이지로 진입합니다.

**📌 [입력]**

> | 항목                | 내용                       | 액션                |
> | ------------------- | -------------------------- | ------------------- |
> | ➕ Target Namespace | `istio-system`             | 🧲복사 & 📋붙여넣기 |
> | ➕ Select File      | `override-vaules-sds.yaml` | 👆🏻셀렉트박스 선택   |

- `Register` 버튼 클릭

### ✔ 2-4. kiali sync

- `Sync` 버튼 클릭

### ✔ 2-5. PaC workspace Page에서 kiali 확인

- 몇분정도 기다린후
- PaC workspace Page에서 새로고침을 눌러 App Health가 `Healthy`로 되는 것을 확인한다.

![](../images/178.png)

### ✔ 2-6. kubectl을 이용한 kiali 확인

- Cloud9에서 아래 명령어를 수행

```bash
kubectl get all -n istio-system | grep kiali
```

- 결과 예시

```bash
mspmanager:~/environment/manifests $ kubectl get all -n istio-system | grep kiali
pod/kiali-5ffd65595d-gqv6m                  1/1     Running   0          87s
service/kiali                  ClusterIP      172.20.74.71     <none>                                                                               20001/TCP                                    87s
deployment.apps/kiali                  1/1     1            1           87s
replicaset.apps/kiali-5ffd65595d                  1         1         1       87s
```

### ✔ 2-7. 브라우저에서 kiali 확인

- https://www.`<나의도메인>`/kiali 을 통해 접속합니다.

![](../images/179.png)

### ✔ 2-8. 로그인을 위한 token 생성

- cloud9에서 아래 명령어를 수행합니다.

```bash
kubectl -n istio-system create token kiali
```

- 결과예시

```bash
mspmanager:~/environment/manifests $ kubectl -n istio-system create token kiali
eyJhbGciOiJSUzI1NiIsImtpZCI6IjAwZjJkMzAxMjQ1ZGJmNDc3NjU3YTMzZmVjMTk0MjljMzhhMTg3YWQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjIl0sImV4cCI6MTcwNjg4NDQ0NCwiaWF0IjoxNzA2ODgwODQ0LCJpc3MiOiJodHRwczovL29pZGMuZWtzLmFwLW5vcnRoZWFzdC0yLmFtYXpvbmF3cy5jb20vaWQvRkQzRDc1QjhFMEE5MEU0NDc2MEUyMzFBNUFDN0FGMEMiLCJrdWJlcm5ldGVzLmlvIjp7Im5hbWVzcGFjZSI6ImlzdGlvLXN5c3RlbSIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJraWFsaSIsInVpZCI6ImIMzLTcwYjMtNDc0ZC05YWRmLTViM2JjODk1YmVkMyJ9fSwibmJmIjoxNzA2ODgwODQ0LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6aXN0aW8tc3lzdGVtOmtpYWxpIn0.aHHLbki2Kxi7oyFkvFefcjvDj83phDmcVAYg5lt83MIj7AwW26a_bSPHuzAMGC0RitdLUJq_7XQCqLQPBN8Yxxi0nAJOhO5JlbgIp4Wnyv2pp0YWgOLmj3mJUcw92F8inZ150UyQQ9oW_IHL8PTvol1eFfTeNJ1NjVkdnaPcBnk1UlhKuCURwHdkn7OiaSJwPnwWiUwHKHus6nXSwEceL8bTWHolVqUiQ2B2kF1oPFgow--eqxIh0w7kEExdberCt2gO5ZZ9YV0-9KFvwO5KnXjeyV0PORjwKuT3Gw5V-n_UFdPILbtWKjteOdtKQwpBgHeD3VleMV8xQ4cA
```

### ✔ 2-9. 생성한 token을 이용하여 로그인

![](../images/180.png)
