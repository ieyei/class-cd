# 6일차 Security
## Lab6. Kubernetes Container 보안

<br>

---
- [6일차 Security](#6일차-security)
  - [Lab6. Kubernetes Container 보안](#lab6-kubernetes-container-보안)
    - [6-1. 컨테이너 이미지 취약점 점검 실습](#6-1-컨테이너-이미지-취약점-점검-실습)
    - [6-2. baseImage를 생성해보고 취약점 점검해보기](#6-2-baseimage를-생성해보고-취약점-점검해보기)
    - [6-3. 컨테이너 레지스트리를 활용하여 이미지 관리해보기](#6-3-컨테이너-레지스트리를-활용하여-이미지-관리해보기)
    - [6-4. ImagePullSecret 실습](#6-4-imagepullsecret-실습)
---

<br>

![](../images/6-6/6-6-1.svg)

ⓘ 실습목표 : 컨테이너 이미지의 보안에 대해 이해하고, 리파지토리를 통해 관리하는 방법을 실습해본다.

---

### 6-1. 컨테이너 이미지 취약점 점검 실습

6-1-1. Cloud9에서 실행하여 터미널 접속

```bash
cd ${HOME}/environment
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $
```

<br>

6-1-2. Trivy 설치

▶ Official : https://aquasecurity.github.io/trivy/v0.49/

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
```

```bash
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
```

```bash
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
```

```bash
sudo apt-get update
```

```bash
sudo apt-get install trivy
```

<br>

- 아래의 명령으로 설치여부 확인

```bash
trivy -v
Version: 0.49.1
```

<br>

6-1-3. Public Image Pull

```bash
docker pull python:3.6-alpine
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ docker pull python:3.6-alpine
3.6-alpine: Pulling from library/python
59bf1c3509f3: Pull complete
8786870f2876: Pull complete
acb0e804800e: Pull complete
52bedcb3e853: Pull complete
b064415ed3d7: Pull complete
Digest: sha256:579978dec4602646fe1262f02b96371779bfb0294e92c91392707fa999c0c989
Status: Downloaded newer image for python:3.6-alpine
docker.io/library/python:3.6-alpine
```

<br>

6-1-4. Container Image scan

```bash
trivy image python:3.6-alpine
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ trivy image python:3.6-alpine
2024-02-07T02:57:35.007Z        INFO    Need to update DB
2024-02-07T02:57:35.007Z        INFO    DB Repository: ghcr.io/aquasecurity/trivy-db
2024-02-07T02:57:35.007Z        INFO    Downloading DB...
42.75 MiB / 42.75 MiB [----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------] 100.00% 19.34 MiB p/s 2.4s
2024-02-07T02:57:38.314Z        INFO    Vulnerability scanning is enabled
2024-02-07T03:02:33.691Z        INFO    Secret scanning is enabled
2024-02-07T03:02:33.691Z        INFO    If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2024-02-07T03:02:33.692Z        INFO    Please see also https://aquasecurity.github.io/trivy/v0.49/docs/scanner/secret/#recommendation for faster secret detection
2024-02-07T03:02:35.709Z        INFO    Detected OS: alpine
...중략...
Python (python-pkg)

Total: 3 (UNKNOWN: 0, LOW: 0, MEDIUM: 1, HIGH: 2, CRITICAL: 0)

┌───────────────────────┬────────────────┬──────────┬────────┬───────────────────┬───────────────┬───────────────────────────────────────────────────────────┐
│        Library        │ Vulnerability  │ Severity │ Status │ Installed Version │ Fixed Version │                           Title                           │
├───────────────────────┼────────────────┼──────────┼────────┼───────────────────┼───────────────┼───────────────────────────────────────────────────────────┤
│ pip (METADATA)        │ CVE-2023-5752  │ MEDIUM   │ fixed  │ 21.2.4            │ 23.3          │ pip: Mercurial configuration injectable in repo revision  │
│                       │                │          │        │                   │               │ when installing via pip                                   │
│                       │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2023-5752                 │
├───────────────────────┼────────────────┼──────────┤        ├───────────────────┼───────────────┼───────────────────────────────────────────────────────────┤
│ setuptools (METADATA) │ CVE-2022-40897 │ HIGH     │        │ 57.5.0            │ 65.5.1        │ pypa-setuptools: Regular Expression Denial of Service     │
│                       │                │          │        │                   │               │ (ReDoS) in package_index.py                               │
│                       │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2022-40897                │
├───────────────────────┼────────────────┤          │        ├───────────────────┼───────────────┼───────────────────────────────────────────────────────────┤
│ wheel (METADATA)      │ CVE-2022-40898 │          │        │ 0.37.0            │ 0.38.1        │ remote attackers can cause denial of service via attacker │
│                       │                │          │        │                   │               │ controlled input to...                                    │
│                       │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2022-40898                │
└───────────────────────┴────────────────┴──────────┴────────┴───────────────────┴───────────────┴───────────────────────────────────────────────────────────┘
```

<br>

- 실행결과를 파일로 저장해보자.

```bash
trivy image python:3.6-alpine > scan-results-python_3.6-alpine
```

<br>

- Cloud9 왼쪽의 EXPLORER 에서 `scan-results-python_3.6-alpine` 파일을 더블클릭하여 Editor 화면에서 이미지 스캔 결과를 확인한다.

![](../images/6-6/6-6-2.svg)

👉 CRITICAL, HIGH, MEDIUM, LOW, UNKNOWN 등 위험 레벨에 따른 점검결과를 확인할 수 있다.

<br>

6-1-5. python의 다른 버전을 Pull 해보자.

▶ Official : https://hub.docker.com/_/python/tags

```bash
docker pull python:3.12.1-alpine
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ docker pull python:3.6-alpine
3.12.1-alpine: Pulling from library/python
4abcf2066143: Pull complete
dca80dc46cec: Pull complete
3324090550b3: Pull complete
f5c92aa967a6: Pull complete
34eb5116f7f2: Pull complete
Digest: sha256:14cfc61fc2404da8adc7b1cb1fcb299aefafab22ae571f652527184fbb21ce69
Status: Downloaded newer image for python:3.12.1-alpine
docker.io/library/python:3.12.1-alpine
```

<br>

- docker container image 목록을 확인한다.

```bash
docker images
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ docker images
REPOSITORY   TAG             IMAGE ID       CREATED       SIZE
python       3.12.1-alpine   b32f0c9b815c   7 weeks ago   51.7MB
python       3.6-alpine      3a9e80fa4606   2 years ago   40.7MB
```

<br>

- Container Image scan

```bash
trivy image python:3.12.1-alpine
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ trivy image python:3.12.1-alpine
2024-02-07T04:52:59.601Z        INFO    Vulnerability scanning is enabled
2024-02-07T04:52:59.601Z        INFO    Secret scanning is enabled
2024-02-07T04:52:59.601Z        INFO    If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2024-02-07T04:52:59.602Z        INFO    Please see also https://aquasecurity.github.io/trivy/v0.49/docs/scanner/secret/#recommendation for faster secret detection
2024-02-07T04:53:02.247Z        INFO    Detected OS: alpine
2024-02-07T04:53:02.247Z        INFO    Detecting Alpine vulnerabilities...
2024-02-07T04:53:02.253Z        INFO    Number of language-specific files: 1
2024-02-07T04:53:02.253Z        INFO    Detecting python-pkg vulnerabilities...

python:3.12.1-alpine (alpine 3.19.1)

Total: 0 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 0)

2024-02-07T04:53:02.275Z        INFO    Table result includes only package filenames. Use '--format json' option to get the full path to the package file.

Python (python-pkg)

Total: 1 (UNKNOWN: 0, LOW: 0, MEDIUM: 1, HIGH: 0, CRITICAL: 0)

┌────────────────┬───────────────┬──────────┬────────┬───────────────────┬───────────────┬──────────────────────────────────────────────────────────┐
│    Library     │ Vulnerability │ Severity │ Status │ Installed Version │ Fixed Version │                          Title                           │
├────────────────┼───────────────┼──────────┼────────┼───────────────────┼───────────────┼──────────────────────────────────────────────────────────┤
│ pip (METADATA) │ CVE-2023-5752 │ MEDIUM   │ fixed  │ 23.2.1            │ 23.3          │ pip: Mercurial configuration injectable in repo revision │
│                │               │          │        │                   │               │ when installing via pip                                  │
│                │               │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2023-5752                │
└────────────────┴───────────────┴──────────┴────────┴───────────────────┴───────────────┴──────────────────────────────────────────────────────────┘
```

👉 버전 업데이트만으로도 많은 취약점이 조치된 것을 확인할 수 있다.

<br>
<br>

---

### 6-2. baseImage를 생성해보고 취약점 점검해보기

▶ 일반적으로 OS의 취약점은 지속적으로 업데이트가 되기 때문에 취약점 실습을 진행하기 어렵다.

▶ 본 Lab에서는 오랜기간 업데이트를 하지 않은 것으로 간주하기 위해, 취약점이 다수 있는 과거 버전의 이미지를 base로 사용한다.

<br>

6-2-1. baseImage를 만들기 위한 Dockerfile 정의

- Cloud9에서 실행하여 터미널 접속하여 `Dockerflie`을 생성한다.

![](../images/6-6/6-6-3.svg)

<br>

- 아래 내용을 복사하여 파일 내용에 붙여넣고, `Ctrl + S`를 눌러 저장한다.

🧲 (COPY)
```yaml
FROM python:3.6-alpine

RUN pip install flask
```

👉 `flask`는 Python의 웹프레임워크 중 하나이다.

<br>

6-2-2. Docker Image 생성

```bash
docker build . -t python:latest
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ docker build . -t python:latest
[+] Building 5.7s (6/6) FINISHED                                                                                                                                                                                      docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                                                            0.0s
 => => transferring dockerfile: 117B                                                                                                                                                                                            0.0s
 => [internal] load metadata for docker.io/library/python:3.6-alpine                                                                                                                                                            0.0s
 => [internal] load .dockerignore                                                                                                                                                                                               0.0s
 => => transferring context: 2B                                                                                                                                                                                                 0.0s
 => CACHED [1/2] FROM docker.io/library/python:3.6-alpine                                                                                                                                                                       0.0s
 => [2/2] RUN pip install flask                                                                                                                                                                                                 5.4s
 => exporting to image                                                                                                                                                                                                          0.2s
 => => exporting layers                                                                                                                                                                                                         0.2s
 => => writing image sha256:f3b92c29e5fb89dc33538431be77c20e9b2c0582a23a0cdb732d86bffd4bce58                                                                                                                                    0.0s
 => => naming to docker.io/library/python:latest
```

<br>

- 컨테이너 이미지 확인

```bash
docker images
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ docker images
REPOSITORY   TAG             IMAGE ID       CREATED         SIZE
python       latest          f3b92c29e5fb   5 minutes ago   51.9MB
python       3.12.1-alpine   b32f0c9b815c   7 weeks ago     51.7MB
python       3.6-alpine      3a9e80fa4606   2 years ago     40.7MB
```

<br>

6-2-3. Container Image Scan

- 이미지 스캔 결과 저장

```bash
trivy image python:latest > scan-results-python_latest
```

✔ **(수행코드/결과 예시)**

- Cloud9 왼쪽의 EXPLORER 에서 `scan-results-python_latest` 파일을 더블클릭하여 Editor 화면에서 이미지 스캔 결과를 확인한다.

```bash
python:latest (alpine 3.15.0)
=============================
Total: 56 (UNKNOWN: 0, LOW: 0, MEDIUM: 16, HIGH: 32, CRITICAL: 8)
...중략...
```

👉 여전히 CRITICAL, HIGH, MEDIUM, LOW, UNKNOWN 등 위험 레벨의 취약점이 존재하는 것을 확인할 수 있다.

<br>

6-2-4. 기본 OS에 설치된 패키지들을 업그레이드해보자.

- `Dockerfile`을 열어 아래와 같이 수정하고, `Ctrl + S`를 눌러 저장한다.

🧲 (COPY)
```yaml
FROM python:3.6-alpine

RUN apk update && apk upgrade

RUN pip install flask
```

<br>

- Docker Image 생성

```bash
docker build . -t python:latest
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ docker build . -t python:latest
[+] Building 7.9s (7/7) FINISHED                                                                                                                                                                                      docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                                                            0.0s
 => => transferring dockerfile: 113B                                                                                                                                                                                            0.0s
 => [internal] load metadata for docker.io/library/python:3.6-alpine                                                                                                                                                            0.0s
 => [internal] load .dockerignore                                                                                                                                                                                               0.0s
 => => transferring context: 2B                                                                                                                                                                                                 0.0s
 => CACHED [1/3] FROM docker.io/library/python:3.6-alpine                                                                                                                                                                       0.0s
 => [2/3] RUN apk update && apk upgrade                                                                                                                                                                                         2.1s
 => [3/3] RUN pip install flask                                                                                                                                                                                                 5.5s
 => exporting to image                                                                                                                                                                                                          0.3s
 => => exporting layers                                                                                                                                                                                                         0.3s
 => => writing image sha256:13cf79e82232dacba4b300c1e5067b26c1d713aca8df84e629f269b81badff6b                                                                                                                                    0.0s
 => => naming to docker.io/library/python:latest                                                                                                                                                                                0.0s
```

<br>

6-2-5. Container Image Scan

- 이미지 스캔 결과 저장

```bash
trivy image python:latest > scan-results-python_latest-upgrade
```

✔ **(수행코드/결과 예시)**

- Cloud9 왼쪽의 EXPLORER 에서 `scan-results-python_latest-upgrade` 파일을 더블클릭하여 Editor 화면에서 이미지 스캔 결과를 확인한다.

```bash
python:latest (alpine 3.15.0)
=============================
Total: 0 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 0)


Python (python-pkg)
===================
Total: 8 (UNKNOWN: 0, LOW: 1, MEDIUM: 3, HIGH: 4, CRITICAL: 0)
...중략...
```

👉 `alpine 3.15.0`에 대한 취약점이 조치된 것을 확인할 수 있다.

<br>
<br>

---

### 6-3. 컨테이너 레지스트리를 활용하여 이미지 관리해보기

6-3-1. ECR(Elastic Container Registry) Repository 생성

- AWS Console에서 `ECR`메뉴로 접속한다.

![](../images/6-6/6-6-4.svg)

<br>

- `Private registry` 메뉴에서 `Repositories`를 선택하고 `Create repository` 버튼을 클릭한다.

![](../images/6-6/6-6-5.svg)

<br>

- Visibility settings를 `Private`으로 선택하고, Repository name은 `cta`로 입력한다.

![](../images/6-6/6-6-6.svg)

<br>

- `Scan on push`옵션을 `Enabled`로 체크한다.

![](../images/6-6/6-6-7.svg)

<br>

- 하단의 `Create repository`버튼을 클릭하여 리파지토리를 생성한다.

![](../images/6-6/6-6-8.svg)

<br>

6-3-2. `Ubuntu:20.04` Image를 ECR에 등록

- Public Image pull

```bash
docker pull ubuntu:20.04
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ docker pull ubuntu:20.04
20.04: Pulling from library/ubuntu
8ee087424735: Pull complete
Digest: sha256:bb1c41682308d7040f74d103022816d41c50d7b0c89e9d706a74b4e548636e54
Status: Downloaded newer image for ubuntu:20.04
docker.io/library/ubuntu:20.04
```

<br>

- Change Tag

❗ `<<YOUR_ACCOUNT_ID>>`의 값은 수강생 개인별 Account ID(12자리)로 수정한다.

❗ `<<YOUR_REGION>>`의 값은 수강생이 생성한 ECR의 Region을 입력한다.

🧲 (COPY & Modify)
```bash
docker tag ubuntu:20.04 <<YOUR_ACCOUNT_ID>>.dkr.ecr.<<YOUR_REGION>>.amazonaws.com/cta:latest
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ docker tag ubuntu:20.04 948135709814.dkr.ecr.ap-northeast-2.amazonaws.com/cta:latest
mspuser:~/environment $
```

<br>

- docker container image 목록을 확인한다.

```bash
docker images
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ docker images
REPOSITORY                                              TAG             IMAGE ID       CREATED          SIZE
python                                                  latest          13cf79e82232   45 minutes ago   62.7MB
948135709814.dkr.ecr.ap-northeast-2.amazonaws.com/cta   latest          18ca3f4297e7   2 weeks ago      72.8MB
ubuntu                                                  20.04           18ca3f4297e7   2 weeks ago      72.8MB
python                                                  3.12.1-alpine   b32f0c9b815c   7 weeks ago      51.7MB
python                                                  3.6-alpine      3a9e80fa4606   2 years ago      40.7MB
```

<br>

- ECR 리파지토리 로그인

❗ `<<YOUR_ACCOUNT_ID>>`의 값은 수강생 개인별 Account ID(12자리)로 수정한다.

❗ `<<YOUR_REGION>>`의 값은 수강생이 생성한 ECR의 Region을 입력한다.

🧲 (COPY & Modify)
```bash
aws ecr get-login-password --region <<YOUR_REGION>> | docker login --username AWS --password-stdin <<YOUR_ACCOUNT_ID>>.dkr.ecr.<<YOUR_REGION>>.amazonaws.com
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 948135709814.dkr.ecr.ap-northeast-2.amazonaws.com
WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

<br>

- 컨테이너 이미지 ECR로 푸시

❗ `<<YOUR_ACCOUNT_ID>>`의 값은 수강생 개인별 Account ID(12자리)로 수정한다.

❗ `<<YOUR_REGION>>`의 값은 수강생이 생성한 ECR의 Region을 입력한다.

🧲 (COPY & Modify)
```bash
docker push <<YOUR_ACCOUNT_ID>>.dkr.ecr.<<YOUR_REGION>>.amazonaws.com/cta:latest
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ docker push 948135709814.dkr.ecr.ap-northeast-2.amazonaws.com/cta:latest
The push refers to repository [948135709814.dkr.ecr.ap-northeast-2.amazonaws.com/cta]
28da0445c449: Pushed
latest: digest: sha256:1b22587263d9ebc3c8c5789d9749f09b3edd012da5c4a3cb8bb8dac711cd8eb0 size: 529
```

<br>

- ECR에서 이미지를 확인한다.

![](../images/6-6/6-6-10.svg)

👉 리파지토리에서 제공하는 취약점 점검기능을 사용할 수 있다.

<br>
<br>

---

### 6-4. ImagePullSecret 실습

6-4-1. Docker 자격증명 확인

```bash
cat ${HOME}/.docker/config.json
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ cat ${HOME}/.docker/config.json
{
        "auths": {
                "948135709814.dkr.ecr.ap-northeast-2.amazonaws.com": {
                        "auth": "QVdTOmV5SndZWGxzYjJGa0lqb2lNMFIyYmtkblJWaDFTbVk0V...중략...
```

<br>

6-4-2. Kubernetes Secret 생성

```bash
kubectl create secret generic cta-regcred \
    --from-file=.dockerconfigjson=${HOME}/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ kubectl create secret generic cta-regcred \
    --from-file=.dockerconfigjson=${HOME}/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson
secret/cta-regcred created
```

<br>

6-4-3. 테스트를 위한 test-pod.yaml 정의

▶ 4일차에 실습했던 test-pod.yaml을 수정한다.

▶ 파일이 없다면 같은 이름으로 새로 생성한다.

![](../images/6-6/6-6-12.svg)

<br>

- 아래 내용을 복사하여 파일 내용에 붙여넣고, `Ctrl + S`를 눌러 저장한다.

🧲 (COPY)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  imagePullSecrets:
  - name: cta-regcred
  containers:
  - name: app
    image: 948135709814.dkr.ecr.ap-northeast-2.amazonaws.com/cta:latest
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

<br>

6-4-4. test-pod.yaml을 배포한다.

```bash
kubectl apply -f test-pod.yaml
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ kubectl apply -f test-pod.yaml
pod/test-pod created
```

<br>

- 배포된 pod 상태 확인

```bash
kubectl get pods
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
test-pod   2/2     Running   0          29s
```
<br>

- pod의 detail 정보를 확인해본다.

```bash
kubectl describe pods test-pod
```

✔ **(수행코드/결과 예시)**

```bash
mspuser:~/environment $ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
test-pod   2/2     Running   0          29s
```

![](../images/6-6/6-6-11.svg)

👉 ECR의 Private Repository에서 이미지를 정상적으로 다운로드했음을 확인할 수 있다.
