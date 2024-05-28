# 5일차 Availability
## Lab6. Kubernetes에서 Volume 관리 실습

<br>

---
- [5일차 Availability](#5일차-availability)
  - [Lab6. Kubernetes에서 Volume 관리 실습](#lab6-kubernetes에서-volume-관리-실습)
    - [6-1. Volume 설정 없이 POD 사용](#6-1-volume-설정-없이-pod-사용)
    - [6-2. hostPath를 활용하여 POD 사용](#6-2-hostpath를-활용하여-pod-사용)
    - [6-3. PV를 활용한 POD 사용](#6-3-pv를-활용한-pod-사용)
    - [6-4. EFS를 활용한 Multi-AZ 가용성 확보](#6-4-efs를-활용한-multi-az-가용성-확보5-4-readinessprobe-실습)
---

ⓘ 실습목표 : Kubernetes에서 Volume을 사용해보고, 가용성을 위한 전략을 실습해본다.

---

### 6-1. Volume 설정 없이 POD 사용

<br>

![](../images/5-6/5-6-1.svg)

👉 별도의 Volume설정을 하지 않을 경우, Pod는 Node의 리소스 안에서 격리된 Volume을 사용하게된다. 이 Volume은 Pod의 생명주기와 연결된다.

<br>

6-1-1. Cloud9에서 실행하여 실습용 디렉토리 생성

- Terminal에서 실습용 디렉토리 생성

```bash
cd ${HOME}/environment
```

```bash
mkdir k8s-volume
```

```bash
cd k8s-volume
```

<br>

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ mkdir k8s-volume
mspuser:~/environment $ cd k8s-volume
mspuser:~/environment/k8s-volume $
```

<br>

6-1-2. 테스트를 위한 nginx yaml 정의

- Cloud9 왼쪽의 EXPLORER 에서 k8s-volume 폴더 우클릭 후 > New Files를 누른다.

![](../images/5-6/5-6-2.svg)

<br>

- 파일명은 `nginx.yaml`로 입력하고 엔터를 누른다.

![](../images/5-6/5-6-3.svg)

<br>

- 아래 내용을 그대로 복사하여 파일 내용에 붙여넣고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    imagePullPolicy: IfNotPresent
```

<br>

6-1-3. nginx.yaml을 배포한다.

- nginx.yaml 배포

```bash
kubectl apply -f nginx.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl apply -f nginx.yaml
pod/nginx created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
nginx                   1/1     Running   0          2m50s
```

<br>

6-1-4. nginx pod에 파일을 아래와 같이 생성한다.

```bash
kubectl exec nginx -- /bin/bash -c 'echo "Hello world" > ${HOME}/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'echo "Hello world" > ${HOME}/vol-test.txt'
mspuser:~/environment/k8s-volume $
```

<br>

- 생성한 파일을 조회해본다.

```bash
kubectl exec nginx -- /bin/bash -c 'cat ${HOME}/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'cat ${HOME}/vol-test.txt'
Hello world
mspuser:~/environment/k8s-volume $
```

<br>

6-1-5. nginx pod를 삭제한 뒤, 다시 생성해본다.

- nginx pod 삭제

```bash
kubectl delete -f nginx.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl delete -f nginx.yaml
pod "nginx" deleted
```

<br>

- nginx pod 재배포

```bash
kubectl apply -f nginx.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl apply -f nginx.yaml
pod "nginx" created
```

<br>


6-1-6. `6-1-4`에서 생성한 파일을 조회해본다.

```bash
kubectl exec nginx -- /bin/bash -c 'cat ${HOME}/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'cat ${HOME}/vol-test.txt'
cat: /root/vol-test.txt: No such file or directory
command terminated with exit code 1
```

👉 Pod가 삭제됨에 따라 생성했던 파일도 같이 삭제된 것을 확인할 수 있다.

<br>

6-1-7. 테스트에 사용한 리소스를 삭제한다.

```bash
kubectl delete -f nginx.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl delete -f nginx.yaml
pod "nginx" deleted
```

<br>
<br>

---

### 6-2. hostPath를 활용하여 POD 사용

<br>

![](../images/5-6/5-6-4.svg)

👉 `hostPath`란 노드의 스토리지를 공유하는 볼륨으로 Pod가 공유된 Path에 직접적인 읽기/쓰기가 가능한 형태이다.

<br>

6-2-1. 테스트를 위한 nginx yaml 정의

- Cloud9 왼쪽의 EXPLORER 에서 k8s-volume 폴더 우클릭 후 > New Files를 누른다.

![](../images/5-6/5-6-2.svg)

<br>

- 파일명은 `nginx-hostvol.yaml`로 입력하고 엔터를 누른다.

![](../images/5-6/5-6-5.svg)

<br>

- 아래 내용을 그대로 복사하여 파일 내용에 붙여넣고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /volume-test
      name: host-volume
  volumes:
  - name: host-volume
    hostPath:
      path: /data
      type: DirectoryOrCreate
```

<br>

6-2-2. nginx-hostvol.yaml을 배포한다.

- nginx-hostvol.yaml 배포

```bash
kubectl apply -f nginx-hostvol.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl apply -f nginx-hostvol.yaml
pod/nginx created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods -o wide
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE                                             NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          56s   10.0.30.101   ip-10-0-30-134.ap-northeast-2.compute.internal   <none>           <none>
```

👉 메모장을 열어 NODE명을 기록해둔다.

<br>

6-2-3. nginx pod에 파일을 아래와 같이 생성한다.

```bash
kubectl exec nginx -- /bin/bash -c 'echo "Hello world" > /volume-test/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'echo "Hello world" > /volume-test/vol-test.txt'
mspuser:~/environment/k8s-volume $
```

<br>

- 생성한 파일을 조회해본다.

```bash
kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
Hello world
mspuser:~/environment/k8s-volume $
```

<br>

6-2-4. nginx pod를 삭제한 뒤, 다시 생성해본다.

- nginx pod 삭제

```bash
kubectl delete -f nginx-hostvol.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl delete -f nginx-hostvol.yaml
pod "nginx" deleted
```

<br>

- nginx pod 재배포

```bash
kubectl apply -f nginx-hostvol.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl apply -f nginx-hostvol.yaml
pod "nginx" created
```

<br>

6-2-5. `6-2-3`에서 생성한 파일을 조회해본다.

```bash
kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
Hello world
mspuser:~/environment/k8s-volume $
```

👉 Pod를 삭제한 뒤 재생성해도 파일이 유지된 것을 확인할 수 있다.

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods -o wide
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE                                             NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          27s   10.0.30.101   ip-10-0-30-134.ap-northeast-2.compute.internal   <none>           <none>
```

<br>

6-2-6. 테스트에 사용한 리소스를 삭제한다.

```bash
kubectl delete -f nginx-hostvol.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl delete -f nginx-hostvol.yaml
pod "nginx" deleted
```

<br>

6-2-7. 이번엔 다른 노드에 nginx를 배포해보자.

- 노드 목록을 조회해본다.

```bash
kubectl get nodes
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl get nodes
NAME                                             STATUS   ROLES    AGE   VERSION
ip-10-0-20-121.ap-northeast-2.compute.internal   Ready    <none>   20d   v1.28.3-eks-e71965b
ip-10-0-20-163.ap-northeast-2.compute.internal   Ready    <none>   20d   v1.28.3-eks-e71965b
ip-10-0-30-132.ap-northeast-2.compute.internal   Ready    <none>   20d   v1.28.3-eks-e71965b
```

<br>

- `nginx-hostval.yaml` 파일을 열어 아래와 같이 `nodeName` 항목을 추가한다.

❗ `<<ANOTHER_NODE>>`의 값은 `6-2-2`에서 기록한 Node와는 다른 노드를 입력한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /volume-test
      name: host-volume
  volumes:
  - name: host-volume
    hostPath:
      path: /data
      type: DirectoryOrCreate
  nodeName: <<ANOTHER_NODE>>
```

<br>

- nginx-hostvol.yaml 배포

```bash
kubectl apply -f nginx-hostvol.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl apply -f nginx-hostvol.yaml
pod/nginx created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods -o wide
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE                                             NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          56s   10.0.30.101   ip-10-0-30-132.ap-northeast-2.compute.internal   <none>           <none>
```

<br>

6-2-8. `6-2-3`에서 생성한 파일을 조회해본다.

```bash
kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
cat: /volume-test/vol-test.txt: No such file or directory
command terminated with exit code 1
```

👉 hostPath는 Node의 볼륨을 사용하는 것이므로 다른 노드에 생성된 Pod에서는 데이터를 조회할 수 없다.

<br>
<br>

---

### 6-3. PV를 활용한 POD 사용

<br>

![](../images/5-6/5-6-6.svg)

👉 EBS Volume을 활용하여 노드와는 별개로 네트워크로 연결되며 영속적인 스토리지 사용이 가능하다.

<br>

6-3-1. AWS Console에서 EBS Volume 생성

- AWS 콘솔(https://console.aws.amazon.com/)에 로그인 한 후, `EC2` 서비스로 이동한다.

![](../images/5-6/5-6-8.svg)

<br>

- EC2 서비스 좌측의 `Volumes`메뉴를 클릭한 후, 우측 상단의 `Create Volume` 버튼을 클릭한다.

![](../images/5-6/5-6-9.svg)

<br>

- Volume type을 `General Purpose SSD(gp2)`로 선택하고, Size는 `1`을 입력한다.

![](../images/5-6/5-6-10.svg)

<br>

- `Encrypt this volume`을 체크하고 KMS key를 `(default)aws/ebs`으로 선택한다.

![](../images/5-6/5-6-11.svg)

<br>

- 하단의 `Create volume` 버튼을 클릭하여 EBS 볼륨을 생성한다.

![](../images/5-6/5-6-12.svg)

<br>

- 생성된 볼륨의 ID를 기록한다.

![](../images/5-6/5-6-13.svg)

<br>

6-3-2. 테스트를 위한 nginx-ebs.yaml 정의

- Cloud9 왼쪽의 EXPLORER 에서 k8s-volume 폴더 우클릭 후 > New Files를 누른다.

![](../images/5-6/5-6-2.svg)

<br>

- 파일명은 `nginx-ebs.yaml`로 입력하고 엔터를 누른다.

![](../images/5-6/5-6-7.svg)

<br>

- 아래 내용을 그대로 복사하여 파일 내용에 붙여넣고 "Ctrl+S"를 눌러 저장한다.

❗ `<<YOUR_VOLUME_ID>>`의 값은 `6-3-1`에서 생성한 volume id 값으로 수정한다.

❗ `<<YOUR_AZ>>`의 값은 노드가 2개이상 배포된 AZ를 선택한다.<br>

```bash
kubectl get nodes
```

✔ **(수행코드/결과 예시)**
```bash
NAME                                            STATUS   ROLES    AGE   VERSION
ip-10-0-3-121.ap-northeast-2.compute.internal   Ready    <none>   21d   v1.28.3-eks-e71965b
ip-10-0-3-163.ap-northeast-2.compute.internal   Ready    <none>   21d   v1.28.3-eks-e71965b
ip-10-0-4-132.ap-northeast-2.compute.internal   Ready    <none>   21d   v1.28.3-eks-e71965b
```

<br>

❗ `ip-10-0-3`으로 시작하는 노드가 2개 이상이면 `ap-northeast-2a`, `ip-10-0-4`으로 시작하는 노드가 2개 이상이면 `ap-northeast-2c`를 입력한다.<br>

🧲 (COPY & Modify)
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-retain
parameters:
  fsType: ext4
  type: gp2
provisioner: ebs.csi.aws.com
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /volume-test
      name: host-volume
  volumes:
  - name: host-volume
    persistentVolumeClaim:
      claimName: nginx
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ebs-retain
  volumeName: nginx
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nginx
spec:
  storageClassName: ebs-retain
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  capacity:
    storage: 1Gi
  csi:
    driver: ebs.csi.aws.com
    fsType: ext4
    volumeHandle: <<YOUR_VOLUME_ID>>
  persistentVolumeReclaimPolicy: Retain
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: topology.ebs.csi.aws.com/zone
              operator: In
              values:
                - <<YOUR_AZ>>
```

<br>

6-3-3. nginx-ebs.yaml을 배포한다.

- nginx-ebs.yaml 배포

```bash
kubectl apply -f nginx-ebs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl apply -f nginx-ebs.yaml
storageclass.storage.k8s.io/ebs-retain created
pod/nginx created
persistentvolumeclaim/nginx created
persistentvolume/nginx created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods -o wide
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE                                             NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          56s   10.0.30.101   ip-10-0-30-134.ap-northeast-2.compute.internal   <none>           <none>
```

👉 메모장을 열어 NODE명을 기록해둔다.

<br>

6-3-4. nginx pod에 파일을 아래와 같이 생성한다.

```bash
kubectl exec nginx -- /bin/bash -c 'echo "Hello world" > /volume-test/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'echo "Hello world" > /volume-test/vol-test.txt'
mspuser:~/environment/k8s-volume $
```

<br>

- 생성한 파일을 조회해본다.

```bash
kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
Hello world
mspuser:~/environment/k8s-volume $
```

<br>

6-3-5. nginx pod를 삭제한 뒤, 다른 노드에 다시 생성해본다.

- nginx 삭제

```bash
kubectl delete -f nginx-ebs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl delete -f nginx-ebs.yaml
storageclass.storage.k8s.io "ebs-retain" deleted
pod "nginx" deleted
persistentvolumeclaim "nginx" deleted
persistentvolume "nginx" deleted
```

<br>

- 노드 목록을 조회해본다.

```bash
kubectl get nodes
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl get nodes
NAME                                             STATUS   ROLES    AGE   VERSION
ip-10-0-20-121.ap-northeast-2.compute.internal   Ready    <none>   20d   v1.28.3-eks-e71965b
ip-10-0-20-163.ap-northeast-2.compute.internal   Ready    <none>   20d   v1.28.3-eks-e71965b
ip-10-0-30-132.ap-northeast-2.compute.internal   Ready    <none>   20d   v1.28.3-eks-e71965b
```

<br>

- `nginx-ebs.yaml` 파일을 열어 아래와 같이 `nodeName` 항목을 추가한다.

❗ `<<ANOTHER_NODE>>`의 값은 `6-3-2`에서 기록한 Node와는 다른 노드를 입력한다.<br>
❗ 단, ip 대역이 같은 node를 선택한다.<br>
❗ 예) 기록한 노드의 정보가 `ip-10-0-30-134.ap-northeast-2.compute.internal`이면, `ip-10-0-30`으로 시작하는 노드를 선택한다.

🧲 (COPY)
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-retain
parameters:
  fsType: ext4
  type: gp2
provisioner: ebs.csi.aws.com
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /volume-test
      name: host-volume
  volumes:
  - name: host-volume
    persistentVolumeClaim:
      claimName: nginx
  nodeName: <<ANOTHER_NODE>>
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ebs-retain
  volumeName: nginx
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nginx
spec:
  storageClassName: ebs-retain
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  capacity:
    storage: 1Gi
  csi:
    driver: ebs.csi.aws.com
    fsType: ext4
    volumeHandle: <<YOUR_VOLUME_ID>>
  persistentVolumeReclaimPolicy: Retain
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: topology.ebs.csi.aws.com/zone
              operator: In
              values:
                - <<YOUR_AZ>>
```

<br>

- nginx-ebs.yaml 배포

```bash
kubectl apply -f nginx-ebs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl apply -f nginx-ebs.yaml
storageclass.storage.k8s.io/ebs-retain created
pod/nginx created
persistentvolumeclaim/nginx created
persistentvolume/nginx created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods -o wide
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE                                             NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          56s   10.0.30.101   ip-10-0-30-132.ap-northeast-2.compute.internal   <none>           <none>
```

<br>

6-3-6. `6-3-4`에서 생성한 파일을 조회해본다.

```bash
kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
Hello world
mspuser:~/environment/k8s-volume $
```

👉 다른 Node에서 실행된 Pods에서도 파일이 유지된 것을 확인할 수 있다.

<br>

6-3-7. 테스트에 사용한 리소스를 삭제한다.

```bash
kubectl delete -f nginx-ebs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl delete -f nginx-ebs.yaml
storageclass.storage.k8s.io "ebs-retain" deleted
pod "nginx" deleted
persistentvolumeclaim "nginx" deleted
persistentvolume "nginx" deleted
```

<br>
<br>

---

### 6-4. EFS를 활용한 Multi-AZ 가용성 확보

![](../images/5-6/5-6-14.svg)

👉 EFS Volume을 활용하여 Multi-AZ를 지원하는 영속적인 스토리지 사용이 가능하다.

<br>

6-4-1. 테스트를 위한 nginx-efs.yaml 정의

- Cloud9 왼쪽의 EXPLORER 에서 k8s-volume 폴더 우클릭 후 > New Files를 누른다.

![](../images/5-6/5-6-2.svg)

<br>

- 파일명은 `nginx-efs.yaml`로 입력하고 엔터를 누른다.

![](../images/5-6/5-6-15.svg)

<br>

- 아래 내용을 그대로 복사하여 파일 내용에 붙여넣고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY & Modify)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /volume-test
      name: host-volume
  volumes:
  - name: host-volume
    persistentVolumeClaim:
      claimName: nginx
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-retain-app
```

<br>

6-4-2. nginx-efs.yaml을 배포한다.

- nginx-efs.yaml 배포

```bash
kubectl apply -f nginx-efs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl apply -f nginx-efs.yamln
pod/nginx created
persistentvolumeclaim/nginx created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods -o wide
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE                                             NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          53s   10.0.30.172   ip-10-0-30-134.ap-northeast-2.compute.internal   <none>           <none>
```

👉 메모장을 열어 NODE명을 기록해둔다.

<br>

6-4-3. nginx pod에 파일을 아래와 같이 생성한다.

```bash
kubectl exec nginx -- /bin/bash -c 'echo "Hello world" > /volume-test/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'echo "Hello world" > /volume-test/vol-test.txt'
mspuser:~/environment/k8s-volume $
```

<br>

- 생성한 파일을 조회해본다.

```bash
kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
Hello world
mspuser:~/environment/k8s-volume $
```

<br>

6-4-4. nginx pod를 삭제한 뒤, 다른 노드에 다시 생성해본다.

- nginx 삭제

```bash
kubectl delete pod nginx
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl delete pod nginx
pod "nginx" deleted
```

<br>

- 노드 목록을 조회해본다.

```bash
kubectl get nodes
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl get nodes
NAME                                             STATUS   ROLES    AGE   VERSION
ip-10-0-3-121.ap-northeast-2.compute.internal   Ready    <none>   20d   v1.28.3-eks-e71965b
ip-10-0-3-163.ap-northeast-2.compute.internal   Ready    <none>   20d   v1.28.3-eks-e71965b
ip-10-0-4-132.ap-northeast-2.compute.internal   Ready    <none>   20d   v1.28.3-eks-e71965b
```

<br>

- `nginx-efs.yaml` 파일을 열어 아래와 같이 `nodeName` 항목을 추가한다.

❗ `<<ANOTHER_NODE>>`의 값은 `6-4-2`에서 기록한 Node와는 다른 노드를 입력한다.<br>
❗ 단, ip 대역이 다른 node를 선택한다.<br>
❗ 예) 기록한 노드의 정보가 `ip-10-0-3-134.ap-northeast-2.compute.internal`이면, `ip-10-0-4`으로 시작하는 노드를 선택한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /volume-test
      name: host-volume
  volumes:
  - name: host-volume
    persistentVolumeClaim:
      claimName: nginx
  nodeName: <<ANOTHER_NODE>>
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-retain-app
```

<br>

- nginx-efs.yaml 배포

```bash
kubectl apply -f nginx-efs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl apply -f nginx-efs.yaml
pod/nginx created
persistentvolumeclaim/nginx unchanged
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods -o wide
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE                                             NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          56s   10.0.4.101   ip-10-0-4-132.ap-northeast-2.compute.internal   <none>           <none>
```

<br>

6-4-5. `6-4-3`에서 생성한 파일을 조회해본다.

```bash
kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl exec nginx -- /bin/bash -c 'cat /volume-test/vol-test.txt'
Hello world
mspuser:~/environment/k8s-volume $
```

👉 다른 AZ의 노드에서 실행된 Pods에서도 파일이 유지된 것을 확인할 수 있다.

<br>

6-4-6. 테스트에 사용한 리소스를 삭제한다.

```bash
kubectl delete -f nginx-efs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-volume $ kubectl delete -f nginx-efs.yaml
pod "nginx" deleted
persistentvolumeclaim "nginx" deleted
```

<br>
<br>

😃 **Lab 6 완료!!!**