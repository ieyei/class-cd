# 3일차 - Lab 10. deleting bookinfo

ⓘ 실습목표 : bookinfo를 삭제합니다.

---

- [3일차 - Lab 10. deleting bookinfo](#3일차---lab-10.-deleting-bookinfo)
  - [1. fualt 인젝션 실습 리소스 제거](#🔴-1.-fualt-인젝션-실습-리소스-제거)
  - [2. bookinfo 설치 실습 리소스 제거](#🔴-2.-bookinfo-설치-실습-리소스-제거)
  - [3. default 네임스페이스에 사이드카 인젝션 해제](#🔴-3.-default-네임스페이스에-사이드카-인젝션-해제)

---

## 🔴 1. fualt 인젝션 실습 리소스 제거

```bash
cd ~/environment/istio/05_bookinfo/fault
```

```bash
kubectl delete -f 04_virtual-service-ratings-test-abort.yaml
```

```bash
kubectl delete -f 03_virtual-service-ratings-test-delay.yaml
```

```bash
kubectl delete -f 02_virtual-service-reviews-test-v2.yaml
```

```bash
kubectl delete -f 01_virtual-service-all-v1.yaml
```

## 🔴 2. bookinfo 설치 실습 리소스 제거

```bash
cd ~/environment/istio/05_bookinfo/
```

```bash
kubectl delete -f 03_destination-rule-all.yaml
```

```bash
kubectl delete -f 02_bookinfo-route.yaml
```

```bash
kubectl delete -f 01_bookinfo.yaml
```

## 🔴 3. default 네임스페이스에 사이드카 인젝션 해제

### ✔ 3-1. 사이드카 인젝션 해제

```bash
kubectl label namespace default istio-injection-
```

### ✔ 3-2. 확인

```bash
kubectl get ns -L istio-injection
```

- 결과예시

```
NAME              STATUS   AGE     ISTIO-INJECTION
default           Active   2d22h
istio-system      Active   2d18h
kube-node-lease   Active   2d22h
kube-public       Active   2d22h
kube-system       Active   2d22h
```
