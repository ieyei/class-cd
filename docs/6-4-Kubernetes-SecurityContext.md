# 6일차 Security
## Lab4. Kubernetes SecurityContext 실습

<br>

---
- [6일차 Security](#6일차-security)
  - [Lab4. Kubernetes SecurityContext 실습](#lab4-kubernetes-securitycontext-실습)
    - [4-1. runAsUser, runAsNonRoot 실습해보기](#4-1-runasuser-runasnonroot-실습해보기)
    - [4-2. fsGroup 실습해보기](#4-2-fsgroup-실습해보기)
    - [4-3. allowPrivilegeEscalation 실습해보기](#4-3-allowprivilegeescalation-실습해보기)
    - [4-4. capabilities 실습해보기](#4-4-capabilities-실습해보기)
    - [4-5. capabilities 추가 실습(필요한 capabilities만 허용)](#4-5-capabilities-추가-실습필요한-capabilities만-허용)
---

ⓘ 실습목표 : Kubernetes의 SecurityContext를 실습해보고 이해한다.

---

### 4-1. runAsUser, runAsNonRoot 실습해보기

4-1-1. Cloud9에서 실행하여 실습용 디렉토리 생성

- Terminal에서 실습용 디렉토리 생성

```bash
cd ${HOME}/environment
```

```bash
mkdir k8s-sc
```

```bash
cd k8s-sc
```

<br>

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ mkdir k8s-sc
mspuser:~/environment $ cd k8s-sc
mspuser:~/environment/k8s-sc $
```

<br>

4-1-2. 테스트를 위한 pod yaml 정의

- Cloud9 왼쪽의 EXPLORER 에서 k8s-sc 폴더 우클릭 후 > New Files를 누른다.

![](../images/6-4/6-4-1.svg)

<br>

- 파일명은 `test-pod.yaml`로 입력하고 엔터를 누른다.

![](../images/6-4/6-4-2.svg)

<br>

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
    command: ["bin/bash", "-c"]
    args:
      - 'sleep 36000'
```

<br>

4-1-3. test-pod.yaml을 배포한다.

```bash
kubectl apply -f test-pod.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-pod.yaml
pod/test-pod created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
test-pod   1/1     Running   0          3s
```

<br>

4-1-4. test-pod에 접속해본다.

```bash
kubectl exec -it test-pod /bin/bash
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec -it test-pod /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@test-pod:/#
```

👉 Pod에 root user로 접속된 것을 확인할 수 있다.

<br>

4-1-5. test-pod의 프로세스를 확인해본다.

```bash
ps -eo uid,gid,comm
```

✔ **(수행코드/결과 예시)**

```bash
root@test-pod:/# ps -eo uid,gid,comm
  UID   GID COMMAND
    0     0 sleep
    0     0 bash
    0     0 ps
root@test-pod:/#
```

👉 User, Group 모두 root로 접속된 것을 확인할 수 있다.

<br>

4-1-6. root directory에 파일을 생성하고 삭제해본다.

- 파일 생성

```bash
echo 'hello world' > /test.txt
```

✔ **(수행코드/결과 예시)**

```bash
root@test-pod:/# echo 'hello world' > /test.txt
root@test-pod:/#
```

<br>

- 파일 조회

```bash
cat /test.txt
ls /test.txt -al
```

✔ **(수행코드/결과 예시)**

```bash
root@test-pod:/# cat /test.txt
ls /test.txt -al
hello world
-rw-r--r-- 1 root root 12 Feb  5 00:37 /test.txt
root@test-pod:/#
```

👉 생성한 파일도 User, Group 모두 root 생성되었다.

<br>

- 파일 삭제

```bash
rm /test.txt
```

✔ **(수행코드/결과 예시)**

```bash
root@test-pod:/# rm /test.txt
root@test-pod:/#
```

<br>

4-1-7. Pod에 설정되어있는 User 목록 확인

```bash
cat /etc/passwd
```

✔ **(수행코드/결과 예시)**

```bash
root@test-pod:/# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
```

<br>

4-1-8. SecurityContext를 활용한 UID 적용해보기

- `test-pod` 삭제

```bash
kubectl delete -f test-pod.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl delete -f test-pod.yaml
pod "test-pod" deleted
```

<br>

- 아래와 같이 SecurityContext를 추가하고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY & Modify)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: app
    image: ubuntu
    command: ["bin/bash", "-c"]
    args:
      - 'sleep 36000'
    securityContext:
      runAsUser: 65534
```

<br>

- `test-pod`를 배포한다.

```bash
kubectl apply -f test-pod.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-pod.yaml
pod/test-pod created
```

<br>

- `test-pod`에 접속한다.

```bash
kubectl exec -it test-pod /bin/bash
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec -it test-pod /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
nobody@test-pod:/$
```

👉 Pod에 nobody user로 접속된 것을 확인할 수 있다.

<br>

- test-pod의 프로세스를 확인해본다.

```bash
ps -eo uid,gid,comm
```

✔ **(수행코드/결과 예시)**

```bash
nobody@test-pod:/$ ps -eo uid,gid,comm
  UID   GID COMMAND
65534 65534 sleep
65534 65534 bash
65534 65534 ps
nobody@test-pod:/$
```

👉 User, Group 모두 nobody로 접속된 것을 확인할 수 있다.

<br>

- root directory에 파일을 생성해본다

```bash
echo 'hello world' > /test.txt
```

✔ **(수행코드/결과 예시)**

```bash
nobody@test-pod:/$ echo 'hello world' > /test.txt
bash: /test.txt: Permission denied
nobody@test-pod:/$
```

👉 권한이 없어 파일 생성이 제한되는 것을 확인할 수 있다.

<br>

4-1-9. RunAsNonRoot 적용해보기

- `test-pod` 삭제

```bash
kubectl delete -f test-pod.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl delete -f test-pod.yaml
pod "test-pod" deleted
```

<br>

- 아래와 같이 RunAsNonRoot를 추가하고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY & Modify)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: app
    image: ubuntu
    command: ["bin/bash", "-c"]
    args:
      - 'sleep 36000'
    securityContext:
      runAsUser: 65534
      runAsNonRoot: true
```

<br>

- `test-pod`를 배포한다.

```bash
kubectl apply -f test-pod.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-pod.yaml
pod/test-pod created
```

<br>

- `test-pod`에 접속한다.

```bash
kubectl exec -it test-pod /bin/bash
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec -it test-pod /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
nobody@test-pod:/$
```

👉 Pod에 nobody user로 접속된 것을 확인할 수 있다.

<br>

- `test-pod` 삭제

```bash
kubectl delete -f test-pod.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl delete -f test-pod.yaml
pod "test-pod" deleted
```

<br>

- 아래와 같이 RunAsUser를 삭제하고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY & Modify)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: app
    image: ubuntu
    command: ["bin/bash", "-c"]
    args:
      - 'sleep 36000'
    securityContext:
      runAsNonRoot: true
```

<br>

- `test-pod`를 배포한다.

```bash
kubectl apply -f test-pod.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-pod.yaml
pod/test-pod created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl get pods
NAME       READY   STATUS                       RESTARTS   AGE
test-pod   0/1     CreateContainerConfigError   0          5m57s
```

<br>

- describe 명령어를 활용하여 pod의 상태를 확인해본다.

```bash
kubectl describe pods test-pod
```

✔ **(수행코드/결과 예시)**

![](../images/6-4/6-4-3.svg)

👉 Container가 Root User로 실행하려고하여 에러가 발생했음을 확인할 수 있다.

<br>

- `test-pod` 삭제

```bash
kubectl delete -f test-pod.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl delete -f test-pod.yaml
pod "test-pod" deleted
```

<br>

4-1-10. Grafana Pod에 접속해서 UID, GID 확인해보기

```bash
kubectl exec -it -n monitoring $(kubectl get pods -n monitoring -l app.kubernetes.io/name=grafana -o jsonpath={.items[]..metadata.name}) /bin/bash
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec -it -n monitoring $(kubectl get pods -n monitoring -l app.kubernetes.io/name=grafana -o jsonpath={.items[]..metadata.name}) /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
grafana-74c874dfcb-bnfz9:/usr/share/grafana$
```

<br>

- grafana의 프로세스를 확인해본다.

```bash
ps -eo user,group,comm
```

✔ **(수행코드/결과 예시)**

```bash
grafana-74c874dfcb-bnfz9:/usr/share/grafana$ ps -eo user,group,comm
USER     GROUP    COMMAND
grafana  472      grafana
grafana  472      gpx_opensearch-
grafana  472      bash
grafana  472      bash
grafana  472      ps
grafana-74c874dfcb-bnfz9:/usr/share/grafana$
```

👉 Grafana 고유의 계정과 GROUP을 사용한 것을 확인할 수 있다.

<br>
<br>

---

### 4-2. fsGroup 실습해보기

4-2-1. 테스트를 위한 pod yaml 정의

- Cloud9 왼쪽의 EXPLORER 에서 k8s-sc 폴더 우클릭 후 > New Files를 누른다.

![](../images/6-4/6-4-1.svg)

<br>

- 파일명은 `test-fs.yaml`로 입력하고 엔터를 누른다.

![](../images/6-4/6-4-4.svg)

<br>

- 아래 내용을 복사하여 파일 내용에 붙여넣고, `Ctrl + S`를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-fs
spec:
  containers:
  - name: app
    image: ubuntu
    command: ["bin/bash", "-c"]
    args:
      - 'sleep 36000'
    volumeMounts:
    - mountPath: /volume-test
      name: host-volume
  volumes:
  - name: host-volume
    emptyDir: {}
```

<br>

4-2-2. test-fs.yaml을 배포한다.

- test-fs.yaml 배포

```bash
kubectl apply -f test-fs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-fs.yaml
pod/test-fs created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
test-fs    1/1     Running   0          3s
```

<br>

4-2-3. test-fs pod에 접속하여 마운트된 볼륨의 권한을 확인해본다.

- pod 접속

```bash
kubectl exec -it test-fs /bin/bash
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec -it test-fs /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@test-fs:/#
```

<br>

- volume 권한 확인

```bash
ls -al | grep volume-test
```

✔ **(수행코드/결과 예시)**

```bash
root@test-fs:/# ls -al | grep volume-test
drwxrwxrwx   2 root root    6 Feb  5 03:00 volume-test
root@test-fs:/#
```

👉 volume이 root 권한으로 마운트된 것을 확인할 수 있다.

<br>

4-2-4. fsGroup 적용해보기

- pod 삭제

```bash
kubectl delete -f test-fs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl delete -f test-fs.yaml
pod "test-fs" deleted
```

<br>

- 아래와 같이 fsGroup을 추가하고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY & Modify)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-fs
spec:
  securityContext:
    fsGroup: 2000
  containers:
  - name: app
    image: ubuntu
    command: ["bin/bash", "-c"]
    args:
      - 'sleep 36000'
    volumeMounts:
    - mountPath: /volume-test
      name: host-volume
  volumes:
  - name: host-volume
    emptyDir: {}
```

<br>

- `test-fs` Pod를 배포한다.

```bash
kubectl apply -f test-fs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-fs.yaml
pod/test-fs created
```

<br>

- test-fs pod에 접속하여 마운트된 볼륨의 권한을 확인해본다.

```bash
kubectl exec -it test-fs /bin/bash
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec -it test-fs /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@test-fs:/#
```

<br>

```bash
ls -al | grep volume-test
```

✔ **(수행코드/결과 예시)**

```bash
root@test-fs:/# ls -al | grep volume-test
drwxrwsrwx   2 root 2000    6 Feb  5 03:38 volume-test
root@test-fs:/#
```

👉 emptyDir 볼륨은 777권한을 부여한다.

<br>

4-2-5. EBS를 활용한 볼륨 마운트

- pod 삭제

```bash
kubectl delete -f test-fs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl delete -f test-fs.yaml
pod "test-fs" deleted
```

<br>

- 아래와 같이 볼륨을 변경하고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY & Modify)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-fs
spec:
  securityContext:
    fsGroup: 2000
  containers:
  - name: app
    image: ubuntu
    command: ["bin/bash", "-c"]
    args:
      - 'sleep 36000'
    volumeMounts:
    - mountPath: /volume-test
      name: host-volume
  volumes:
  - name: host-volume
    persistentVolumeClaim:
      claimName: test
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ebs-retain
---
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
```

<br>

- `test-fs` Pod를 배포한다.

```bash
kubectl apply -f test-fs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-fs.yaml
pod/test-fs created
persistentvolumeclaim/test created
storageclass.storage.k8s.io/ebs-retain created
```

<br>

- test-fs pod에 접속하여 마운트된 볼륨의 권한을 확인해본다.

```bash
kubectl exec -it test-fs /bin/bash
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec -it test-fs /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
groups: cannot find name for group ID 2000
root@test-fs:/#
```

<br>

```bash
ls -al | grep volume-test
```

✔ **(수행코드/결과 예시)**

```bash
root@test-fs:/# ls -al | grep volume-test
drwxrwsr-x   3 root 2000 4096 Feb  5 04:27 volume-test
root@test-fs:/#
```

👉 EBS 볼륨은 775권한을 부여한다.

<br>

4-2-6. 다른 사용자로 볼륨에 파일 생성해보기

- 아래의 명령어를 통해 임의의 사용자를 생성한다.

🧲 (COPY)
```bash
adduser temp
```

✔ **(수행코드/결과 예시)**

![](../images/6-4/6-4-5.svg)

👉 password는 user와 동일하게 `temp`로 입력하고 나머지는 `Enter`를 입력해서 Skip한다.

👉 마지막 메시지인 `Is the information correct? [Y/n]` 메시지가 나타나면 `y`를 입력한다.

<br>

4-2-7. `temp` 사용자로 로그인 하고 `/volume-test` 디렉토리에 파일을 생성해본다.

- 아래의 명령어로 사용자를 변경한다.

🧲 (COPY)
```bash
su temp
```

✔ **(수행코드/결과 예시)**

```bash
root@test-fs:/# su temp
temp@test-fs:/#
```

<br>

- `/volume-test` 디렉토리에 파일을 생성해본다.

🧲 (COPY)
```bash
echo 'hello world' > /volume-test/test.txt
```

✔ **(수행코드/결과 예시)**

```bash
temp@test-fs:/$ echo 'hello world' > /volume-test/test.txt
bash: /volume-test/test.txt: Permission denied
temp@test-fs:/$
```

👉 `volume-test`는 775 권한을 갖고있으므로 사용자나 Group이 아니면 파일을 생성할 수 없다.

<br>

4-2-8. Grafana Pod를 Describe 해본다.

🧲 (COPY)
```bash
kubectl describe pod -n monitoring $(kubectl get pods -n monitoring -l app.kubernetes.io/name=grafana -o jsonpath={.items[]..metadata.name})
```

✔ **(수행코드/결과 예시)**

![](../images/6-4/6-4-6.svg)

👉 EFS의 경우, fsGroup이 적용되지 않는다. NFS의 경우 CSP 정책에 따라 다를 수 있기 때문에, 이와 같이 `init container`를 활용할 수 있다.

<br>

4-2-9. 리소스 삭제

```bash
kubectl delete -f test-fs.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl delete -f test-fs.yaml
pod "test-fs" deleted
persistentvolumeclaim "test" deleted
storageclass.storage.k8s.io "ebs-retain" deleted
```

<br>
<br>

---

### 4-3. allowPrivilegeEscalation 실습해보기

4-3-1. 테스트를 위한 pod yaml 정의

- Cloud9 왼쪽의 EXPLORER 에서 k8s-sc 폴더 우클릭 후 > New Files를 누른다.

![](../images/6-4/6-4-1.svg)

<br>

- 파일명은 `test-pe.yaml`로 입력하고 엔터를 누른다.

![](../images/6-4/6-4-7.svg)

<br>

- 아래 내용을 복사하여 파일 내용에 붙여넣고, `Ctrl + S`를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pe
spec:
  containers:
  - name: app
    image: eclipse/alpine_jdk8
```

👉 https://hub.docker.com/r/eclipse/alpine_jdk8

<br>

4-3-2. test Pod를 배포한다.

```bash
kubectl apply -f test-pe.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-pe.yaml
pod/test-pe created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
test-pe    1/1     Running   0          3s
```

<br>

4-3-3. Pod를 통해 다양한 커맨드를 실행해본다.

- 현재 path 확인

```bash
kubectl exec test-pe -- pwd
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test-pe -- pwd
/projects
mspuser:~/environment/k8s-sc $
```

<br>

- `/` 디렉토리 확인

```bash
kubectl exec test-pe -- ls / -al
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test-pe -- ls / -al
total 24
drwxr-xr-x    1 root     root            39 Feb  5 05:45 .
drwxr-xr-x    1 root     root            39 Feb  5 05:45 ..
drwxr-xr-x    1 root     root            70 Jul 18  2018 bin
drwxr-xr-x    5 root     root           360 Feb  5 05:45 dev
drwxr-xr-x    1 root     root            36 Feb  5 05:45 etc
drwxr-xr-x    1 root     root            18 Jul 18  2018 home
drwxr-xr-x    1 root     root           174 Jul 18  2018 lib
lrwxrwxrwx    1 root     root            12 Mar  3  2017 linuxrc -> /bin/busybox
drwxr-xr-x    5 root     root            44 Mar  3  2017 media
drwxr-xr-x    2 root     root             6 Mar  3  2017 mnt
-rw-r--r--    1 root     root         20562 Jul 18  2018 open-jdk-source-file-location
dr-xr-xr-x  312 root     root             0 Feb  5 05:45 proc
drwxr-xr-x    2 root     root             6 Jul 18  2018 projects
drwxrwx---    1 root     root             6 Mar  3  2017 root
drwxr-xr-x    1 root     root            22 Feb  5 05:45 run
drwxr-xr-x    1 root     root           130 Jul 18  2018 sbin
drwxr-xr-x    2 root     root             6 Mar  3  2017 srv
dr-xr-xr-x   13 root     root             0 Feb  5 05:44 sys
drwxrwxrwt    1 root     root             6 Jul 18  2018 tmp
drwxr-xr-x    1 root     root            53 Mar  3  2017 usr
drwxr-xr-x    1 root     root            17 Jul 18  2018 var
mspuser:~/environment/k8s-sc $
```

<br>

- 현재 접속한 user 확인하기

```bash
kubectl exec test-pe -- sudo id
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test-pe -- id
uid=1000(user) gid=0(root) groups=0(root)
```

<br>

- sudo(superuser)를 사용한 명령어 실행

```bash
kubectl exec test-pe -- sudo touch temp.txt
kubectl exec test-pe -- sudo ls -al
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test-pe -- sudo touch temp.txt
kubectl exec test-pe -- sudo ls -al
total 0
drwxr-xr-x    1 root     root            22 Feb  5 05:55 .
drwxr-xr-x    1 root     root            67 Feb  5 05:45 ..
-rw-r--r--    1 root     root             0 Feb  5 05:55 temp.txt
```

👉 사용자(1000)보다 더 높은 권한(sudo)을 사용할 수 있다.

<br>

4-3-4. allowPrivilegeEscalation를 활용하여 권한제어 하기

- `test-pod` 삭제

```bash
kubectl delete -f test-pe.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl delete -f test-pe.yaml
pod "test-pe" deleted
```

<br>

- 아래와 같이 SecurityContext를 추가하고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY & Modify)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pe
spec:
  containers:
  - name: app
    image: eclipse/alpine_jdk8
    securityContext:
      allowPrivilegeEscalation: false
```

<br>

- `test-pe`를 배포한다.

```bash
kubectl apply -f test-pe.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-pe.yaml
pod/test-pe created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl get pods
NAME      READY   STATUS   RESTARTS   AGE
test-pe   0/1     Error    0          4s
```

<br>

- pod의 로그를 확인해본다.

```bash
kubectl logs test-pe
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl logs test-pe
sudo: effective uid is not 0, is /usr/bin/sudo on a file system with the 'nosuid' option set or an NFS file system without root privileges?
mspuser:~/environment/k8s-sc $
```

👉 sudo에 대한 권한이 없어 오류가 발생한 것을 확인할 수 있다.

<br>

4-3-5. 권한 조정 후, 재배포

- `test-pod` 삭제

```bash
kubectl delete -f test-pe.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl delete -f test-pe.yaml
pod "test-pe" deleted
```

<br>

- 아래와 같이 SecurityContext를 추가하고 "Ctrl+S"를 눌러 저장한다.

🧲 (COPY & Modify)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pe
spec:
  containers:
  - name: app
    image: eclipse/alpine_jdk8
    securityContext:
      allowPrivilegeEscalation: false
      runAsUser: 0
```

<br>

- `test-pe`를 배포한다.

```bash
kubectl apply -f test-pe.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-pe.yaml
pod/test-pe created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl get pods
NAME      READY   STATUS   RESTARTS   AGE
test-pe   1/1     Running  0          4s
```

👉 Root User는 최상위 권한자로 Pod가 정상실행된 것을 확인할 수 있다.

<br>
<br>

---

### 4-4. capabilities 실습해보기

4-4-1. `test-pe` pod에 파일을 생성하여 소유자를 변경해본다.

- pod에 파일을 생성해본다.

🧲 (COPY)
```bash
kubectl exec test-pe -- sudo touch temp.txt
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test-pe -- sudo touch temp.txt
mspuser:~/environment/k8s-sc $
```

<br>

- 생성한 파일을 조회해본다.

🧲 (COPY)
```bash
kubectl exec test-pe -- sudo ls -al
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test-pe -- sudo ls al
total 0
drwxr-xr-x    1 root     root            22 Feb  5 06:47 .
drwxr-xr-x    1 root     root            55 Feb  5 06:46 ..
-rw-r--r--    1 root     root             0 Feb  5 06:47 temp.txt
mspuser:~/environment/k8s-sc $
```

<br>

- 아래의 명령어로 소유자를 user(1000)으로 변경한다.

🧲 (COPY)
```bash
kubectl exec test-pe -- sudo chown 1000:1000 temp.txt
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test-pe -- sudo chown 1000:1000 temp.txt
mspuser:~/environment/k8s-sc $
```

<br>

- 파일을 조회해본다.

🧲 (COPY)
```bash
kubectl exec test-pe -- sudo ls -al
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test-pe -- sudo ls al
total 0
drwxr-xr-x    1 root     root            22 Feb  5 06:47 .
drwxr-xr-x    1 root     root            55 Feb  5 06:46 ..
-rw-r--r--    1 user     1000             0 Feb  5 06:47 temp.txt
mspuser:~/environment/k8s-sc $
```

👉 `temp.txt`파일의 소유권이 user(1000), gid(1000)으로 변경된 것을 확인할 수 있다.

<br>

4-4-2. capabilities 추가해보기

- pod 삭제

```bash
kubectl delete -f test-pe.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl delete -f test-pe.yaml
pod "test-pe" deleted
```

<br>

- 아래와 같이 `test-pe` 파일에 capabilities를 추가하고 `Ctrl + S`를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pe
spec:
  containers:
  - name: app
    image: eclipse/alpine_jdk8
    securityContext:
      allowPrivilegeEscalation: false
      runAsUser: 0
      capabilities:
        drop: ["CHOWN"]
```

<br>

- `test-pe` Pod를 배포한다.

```bash
kubectl apply -f test-pe.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-pe.yaml
pod/test-pe created
```

<br>

- pod에 파일을 생성해본다.

🧲 (COPY)
```bash
kubectl exec test-pe -- sudo touch temp.txt
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test-pe -- sudo touch temp.txt
mspuser:~/environment/k8s-sc $
```

<br>

- 생성한 파일을 조회해본다.

🧲 (COPY)
```bash
kubectl exec test-pe -- sudo ls -al
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test-pe -- sudo ls al
total 0
drwxr-xr-x    1 root     root            22 Feb  5 06:47 .
drwxr-xr-x    1 root     root            55 Feb  5 06:46 ..
-rw-r--r--    1 root     root             0 Feb  5 06:47 temp.txt
mspuser:~/environment/k8s-sc $
```

<br>

- 아래의 명령어로 소유자를 user(1000)으로 변경한다.

🧲 (COPY)
```bash
kubectl exec test-pe -- sudo chown 1000:1000 temp.txt
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test-pe -- sudo chown 1000:1000 temp.txt
chown: temp.txt: Operation not permitted
command terminated with exit code 1
```

👉 `CHOWN`권한이 없으므로 오류가 발생하는 것을 확인할 수 있다.

<br>

❗ Linux Capabilities List : https://www.man7.org/linux/man-pages/man7/capabilities.7.html

<br>

4-4-3. pod 삭제

```bash
kubectl delete -f test-pe.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl delete -f test-pe.yaml
pod "test-pe" deleted
```

<br>
<br>

---

### 4-5. capabilities 추가 실습(필요한 capabilities만 허용)

4-5-1. 테스트를 위한 pod yaml 정의

- Cloud9 왼쪽의 EXPLORER 에서 k8s-sc 폴더 우클릭 후 > New Files를 누른다.

![](../images/6-4/6-4-1.svg)

<br>

- 파일명은 `test-cap.yaml`로 입력하고 엔터를 누른다.

![](../images/6-4/6-4-8.svg)

<br>

- 아래 내용을 복사하여 파일 내용에 붙여넣고, `Ctrl + S`를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - name: app
    image: ubuntu
    command: ["/bin/bash", "-c"]
    args:
      - 'sleep 36000'
    securityContext:
      capabilities:
        drop: ["ALL"]
```

<br>

4-5-2. test-cap.yaml을 배포한다.

```bash
kubectl apply -f test-cap.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-cap.yaml
pod/test created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
test   1/1     Running   0          3s
```

<br>

4-5-3. 다음의 명령어로 시간 변경을 수행해본다.

- 현재 시간 확인

```bash
kubectl exec test -- date
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test -- date
Mon Feb  5 07:12:38 UTC 2024
```

<br>

- 시간 변경

```bash
kubectl exec test -- date +%T -s "12:00:00"
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test -- date
date: cannot set date: Operation not permitted
12:00:00
command terminated with exit code 1
```
<br>

4-5-4. `SYS_TIME` capabilities를 추가하여 재배포 한다.

- pod 삭제

```bash
kubectl delete -f test-cap.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl delete -f test-cap.yaml
pod "test" deleted
```

<br>

- 아래와 같이 `test-cap` 파일에 capabilities를 변경하고 `Ctrl + S`를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - name: app
    image: eclipse/alpine_jdk8
    securityContext:
      allowPrivilegeEscalation: false
      runAsUser: 0
      capabilities:
        drop: ["ALL"]
        add: ["SYS_TIME"]
```

<br>

- `test` Pod를 배포한다.

```bash
kubectl apply -f test-cap.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl apply -f test-cap.yaml
pod/test created
```

<br>

4-5-5. 다음의 명령어로 시간 변경을 수행해본다.

- 현재 시간 확인

```bash
kubectl exec test -- date
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test -- date
Mon Feb  5 07:12:38 UTC 2024
```

<br>

- 시간 변경

```bash
kubectl exec test -- date +%T -s "12:00:00"
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment/k8s-sc $ kubectl exec test -- date +%T -s "12:00:00"
12:00:00
```

<br>
<br>

😃 **Lab 4 완료!!!**

<br>

⏩ 다음 실습으로 [이동](6-5-Kubernetes-Network.md)합니다.