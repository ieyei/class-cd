# 6일차 Security
## Lab1. 정적분석도구를 활용한 인프라 보안점검

<br>

---
- [6일차 Security](#6일차-security)
  - [Lab1. 정적분석도구를 활용한 인프라 보안점검](#lab1-정적분석도구를-활용한-인프라-보안점검)
    - [1-1. 현재 인프라에 대한 보안점검 수행해보기 - VPC](#1-1-현재-인프라에-대한-보안점검-수행해보기---vpc)
    - [1-2. 현재 인프라에 대한 보안점검 수행해보기 - EKS](#1-2-현재-인프라에-대한-보안점검-수행해보기---eks)
    - [1-3. [실습]현재 인프라에 대한 보안점검 수행해보기 - Security Group / RDS 등](#1-3-실습현재-인프라에-대한-보안점검-수행해보기---security-group--rds-등)
---

ⓘ 실습목표 : 정적분석도구를 활용하여 IaC (Ex. Terraform)에 대한 보안점검을 실습해보고 이해한다.

---

### 1-1. 현재 인프라에 대한 보안점검 수행해보기 - VPC

1-1-1. GitOps Console에 로그인한다.

- GitOps Console URL : http://t3.gitopsconsole.com/
- GitOps Console 계정 : 사전에 전달받은 t3user로 시작하는 계정과 비밀번호로 접속

![](../images/6-1/6-1-1.svg)

<br>

1-1-2. Pipeline Workspace에 접속한다.

- [Workspace] - [Pipeline Workspace] - [User Workspace] 클릭

![](../images/6-1/6-1-2.svg)

<br>

1-1-3. Pipeline View를 확인한다.

![](../images/6-1/6-1-3.svg)

<br>

![](../images/6-1/6-1-4.svg)

<br>

1-1-4. VPC Task를 선택하고 Plan 버튼 클릭

![](../images/6-1/6-1-5.svg)

<br>

- 아래와 같이 Plan이 Wait 상태로 준비된다.

![](../images/6-1/6-1-6.svg)

<br>

- 잠시 후, Plan이 정상 수행되면 상태가 `Success`로 변경된다. Plan의 결과는 하단의 로그영역에서 확인할 수 있다.

![](../images/6-1/6-1-7.svg)

<br>

1-1-5. VPC Task의 Plan 결과를 확인해보자.

- Keyboard에 `Ctrl + F`를 입력하여 `tfsec task starts`를 검색한다.

![](../images/6-1/6-1-8.svg)

>👉 GitOps Console은 Task의 Plan 수행 시, 보안점검 기능을 지원한다.
<br>

>👉 보안점검 결과를 Plan 로그에서 다음과 같이 확인할 수 있다.
<br>

![](../images/6-1/6-1-9.svg)

<br>

✔ **(수행코드/결과 예시)**
```bash
Result #1 MEDIUM VPC Flow Logs is not enabled for VPC
────────────────────────────────────────────────────────────────────────────────
  main.tf:13-26
────────────────────────────────────────────────────────────────────────────────
   13  ┌ resource "aws_vpc" "main" {
   14  │   cidr_block           = var.cidr_block
   15  │   enable_dns_support   = true
   16  │   enable_dns_hostnames = true
   17  │   tags = {
   18  │     Name = format("vpc_%s", local.tag_suffix)
   19  │   }
   20  │   # kubernetes tag 때문에 추가, k8s가 추가한 tag 자동 삭제 방지용
   21  └   lifecycle {
   ..
────────────────────────────────────────────────────────────────────────────────
          ID aws-ec2-require-vpc-flow-logs-for-all-vpcs
      Impact Without VPC flow logs, you risk not having enough information about network traffic flow to investigate incidents or identify security issues.
  Resolution Enable flow logs for VPC

  More Information
  - https://aquasecurity.github.io/tfsec/v1.28.1/checks/aws/ec2/require-vpc-flow-logs-for-all-vpcs/
────────────────────────────────────────────────────────────────────────────────


  timings
  ──────────────────────────────────────────
  disk i/o             301.494µs
  parsing              80.371545ms
  adaptation           278.65µs
  checks               7.660286ms
  total                88.611975ms

  counts
  ──────────────────────────────────────────
  modules downloaded   0
  modules processed    2
  blocks processed     124
  files read           9

  results
  ──────────────────────────────────────────
  passed               11
  ignored              0
  critical             0
  high                 0
  medium               1
  low                  0

  11 passed, 1 potential problem(s) detected.
```

>👉 11개의 점검항목을 Pass했고, 1개의 잠재적 문제를 발견한 것을 확인할 수 있다.

<br>

1-1-6. 잠재적인 문제점을 조치해보자.

▶ URL : https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/flow_log#cloudwatch-logging

>👉 tfsec의 이슈사항으로 aws flow log에 대한 가이드 페이지 접근이 불가하다. terraform의 매뉴얼을 대신 확인해보자.

<br>

- Pipeline의 VPC Task를 더블클릭하여 `enable_flowlog` 속성을 `true`로 변경하고 저장한다.

![](../images/6-1/6-1-10.svg)

<br>

- Plan 버튼을 클릭하여 보안점검 결과를 다시 확인해보자.

![](../images/6-1/6-1-11.svg)

<br>

✔ **(수행코드/결과 예시)**
```bash
Result #1 HIGH IAM policy document uses sensitive action 'logs:CreateLogGroup' on wildcarded resource '*'
────────────────────────────────────────────────────────────────────────────────
  main.tf:263
────────────────────────────────────────────────────────────────────────────────
  246    resource "aws_iam_role_policy" "main" {
  ...
  263  [               "Resource": "*"
  ...
  268    }
────────────────────────────────────────────────────────────────────────────────
          ID aws-iam-no-policy-wildcards
      Impact Overly permissive policies may grant access to sensitive resources
  Resolution Specify the exact permissions required, and to which resources they should apply instead of using wildcards.

  More Information
  - https://aquasecurity.github.io/tfsec/v1.28.1/checks/aws/iam/no-policy-wildcards/
  - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document
────────────────────────────────────────────────────────────────────────────────


Result #2 LOW Log group is not encrypted.
────────────────────────────────────────────────────────────────────────────────
  main.tf:214-221
────────────────────────────────────────────────────────────────────────────────
  214    resource "aws_cloudwatch_log_group" "main" {
  215      for_each          = var.enable_flowlog ? toset(["flowlog"]) : toset([])
  216      name              = "/aws/flowlog/${local.tag_suffix}"
  217      retention_in_days = var.flowlog_retention
  218      tags = {
  219        "Name" = "/aws/flowlog/${local.tag_suffix}"
  220      }
  221    }
────────────────────────────────────────────────────────────────────────────────
          ID aws-cloudwatch-log-group-customer-key
      Impact Log data may be leaked if the logs are compromised. No auditing of who have viewed the logs.
  Resolution Enable CMK encryption of CloudWatch Log Groups

  More Information
  - https://aquasecurity.github.io/tfsec/v1.28.1/checks/aws/cloudwatch/log-group-customer-key/
  - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_log_group#kms_key_id
────────────────────────────────────────────────────────────────────────────────


  timings
  ──────────────────────────────────────────
  disk i/o             256.662µs
  parsing              83.185265ms
  adaptation           512.436µs
  checks               10.206559ms
  total                94.160922ms

  counts
  ──────────────────────────────────────────
  modules downloaded   0
  modules processed    2
  blocks processed     124
  files read           9

  results
  ──────────────────────────────────────────
  passed               17
  ignored              0
  critical             0
  high                 1
  medium               0
  low                  1

  17 passed, 2 potential problem(s) detected.
```

>👉 17개의 점검항목을 Pass했고, 2개의 잠재적 문제를 발견한 것을 확인할 수 있다.

<br>

1-1-7. flowlog를 적용한 VPC를 Apply 해보자.

- `Apply`버튼을 클릭하여 변경사항을 적용한다.

![](../images/6-1/6-1-12.svg)

<br>

- `ongoing` 상태가 생성되고, 잠시 후에 `success`로 변경되면 정상 반영된 것이다.

![](../images/6-1/6-1-13.svg)
<br>

![](../images/6-1/6-1-14.svg)

<br>
<br>

---

### 1-2. 현재 인프라에 대한 보안점검 수행해보기 - EKS

1-2-1. EKS Task를 선택하고 Plan 버튼 클릭

![](../images/6-1/6-1-15.svg)

<br>

- 아래와 같이 Plan이 Wait 상태로 준비된다.

![](../images/6-1/6-1-6.svg)

<br>

- 잠시 후, Plan이 정상 수행되면 상태가 `Success`로 변경된다. Plan의 결과는 하단의 로그영역에서 확인할 수 있다.

![](../images/6-1/6-1-7.svg)

<br>

1-2-2. EKS Task의 Plan 결과를 확인해보자.

- Keyboard에 `Ctrl + F`를 입력하여 `tfsec task starts`를 검색한다.

![](../images/6-1/6-1-16.svg)

<br>

- 보안점검 결과를 확인해보자.

✔ **(수행코드/결과 예시)**
```bash
Result #1 CRITICAL Public cluster access is enabled.
────────────────────────────────────────────────────────────────────────────────
  terraform-aws-modules/eks/aws/gitops/workdir/.terraform/modules/eks_cluster/main.tf:37
   via main.tf:23-103 (module.eks_cluster)
────────────────────────────────────────────────────────────────────────────────
   25    resource "aws_eks_cluster" "this" {
   ..
   37  [     endpoint_public_access  = var.cluster_endpoint_public_access (true)
   ..
   91    }
────────────────────────────────────────────────────────────────────────────────
          ID aws-eks-no-public-cluster-access
      Impact EKS can be access from the internet
  Resolution Don't enable public access to EKS Clusters

  More Information
  - https://aquasecurity.github.io/tfsec/v1.28.1/checks/aws/eks/no-public-cluster-access/
  - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eks_cluster#endpoint_public_access
────────────────────────────────────────────────────────────────────────────────


Result #2 CRITICAL Cluster allows access from a public CIDR: 0.0.0.0/0.
────────────────────────────────────────────────────────────────────────────────
  terraform-aws-modules/eks/aws/gitops/workdir/.terraform/modules/eks_cluster/main.tf:38
   via main.tf:23-103 (module.eks_cluster)
────────────────────────────────────────────────────────────────────────────────
   25    resource "aws_eks_cluster" "this" {
   ..
   38  [     public_access_cidrs     = var.cluster_endpoint_public_access_cidrs
   ..
   91    }
────────────────────────────────────────────────────────────────────────────────
          ID aws-eks-no-public-cluster-access-to-cidr
      Impact EKS can be accessed from the internet
  Resolution Don't enable public access to EKS Clusters

  More Information
  - https://aquasecurity.github.io/tfsec/v1.28.1/checks/aws/eks/no-public-cluster-access-to-cidr/
  - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eks_cluster#vpc_config
────────────────────────────────────────────────────────────────────────────────


Results #3-4 MEDIUM Control plane scheduler logging is not enabled. (2 similar results)
────────────────────────────────────────────────────────────────────────────────
  terraform-aws-modules/eks/aws/gitops/workdir/.terraform/modules/eks_cluster/main.tf:25-91
   via main.tf:23-103 (module.eks_cluster)
────────────────────────────────────────────────────────────────────────────────
   25  ┌ resource "aws_eks_cluster" "this" {
   26  │   count = local.create ? 1 : 0
   27  │
   28  │   name                      = var.cluster_name
   29  │   role_arn                  = local.cluster_role
   30  │   version                   = var.cluster_version
   31  │   enabled_cluster_log_types = var.cluster_enabled_log_types
   32  │
   33  └   vpc_config {
   ..
────────────────────────────────────────────────────────────────────────────────
  Individual Causes
  - terraform-aws-modules/eks/aws/gitops/workdir/.terraform/modules/eks_cluster/main.tf:23-103 (module.eks_cluster) 2 instances
────────────────────────────────────────────────────────────────────────────────
          ID aws-eks-enable-control-plane-logging
      Impact Logging provides valuable information about access and usage
  Resolution Enable logging for the EKS control plane

  More Information
  - https://aquasecurity.github.io/tfsec/v1.28.1/checks/aws/eks/enable-control-plane-logging/
  - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eks_cluster#enabled_cluster_log_types
────────────────────────────────────────────────────────────────────────────────


  timings
  ──────────────────────────────────────────
  disk i/o             636.308µs
  parsing              231.438462ms
  adaptation           1.409781ms
  checks               12.024827ms
  total                245.509378ms

  counts
  ──────────────────────────────────────────
  modules downloaded   0
  modules processed    5
  blocks processed     396
  files read           21

  results
  ──────────────────────────────────────────
  passed               24
  ignored              0
  critical             2
  high                 0
  medium               2
  low                  0

  24 passed, 4 potential problem(s) detected.
```

>👉 24개의 점검항목을 Pass했고, 4개의 잠재적 문제를 발견한 것을 확인할 수 있다.

<br>

1-2-3. 잠재적인 문제점을 조치해보자.

- Public access issue

▶ URL : https://aquasecurity.github.io/tfsec/v1.28.1/checks/aws/eks/no-public-cluster-access/

>👉 Public access를 `false`로 설정하여 조치할 수 있다.

![](../images/6-1/6-1-17.svg)

<br>

- CIDR any open(0.0.0.0/0) issue

▶ URL : https://aquasecurity.github.io/tfsec/v1.28.1/checks/aws/eks/no-public-cluster-access-to-cidr/

>👉 Public access 설정이 true인 경우, cidr을 설정하라는 의미이다.

<br>

- Control plane scheduler logging is not enabled issue

▶ URL : https://aquasecurity.github.io/tfsec/v1.28.1/checks/aws/eks/enable-control-plane-logging/

>👉 cluster logging에 "scheduler", "controllerManager"가 활성화되어야한다는 의미이다.

![](../images/6-1/6-1-18.svg)

<br>

1-2-4. eks의 속성값을 수정해서 저장하고, Plan을 수행해보자.

- cluster_logs에 `[ "audit", "api", "authenticator", "scheduler", "controllerManager" ]` 입력
<br>
- enable_public_access를 `false`로 설정

<br>

✔ **(수행코드/결과 예시)**
```bash
tfsec task starts
  timings
  ──────────────────────────────────────────
  disk i/o             598.929µs
  parsing              223.465894ms
  adaptation           1.379438ms
  checks               10.263572ms
  total                235.707833ms

  counts
  ──────────────────────────────────────────
  modules downloaded   0
  modules processed    5
  blocks processed     396
  files read           21

  results
  ──────────────────────────────────────────
  passed               27
  ignored              0
  critical             0
  high                 0
  medium               0
  low                  0


No problems detected!
```

>👉 27개의 점검항목을 Pass했고, 잠재적인 문제가 발견되지 않은 것을 확인할 수 있다.

<br>

❗현재는 강의를 위한 환경이 설정되어있으므로 `Apply`는 수행하지 않는다.

<br>
<br>

---

### 1-3. [실습]현재 인프라에 대한 보안점검 수행해보기 - Security Group / RDS 등

❗(주의!!)현재는 강의를 위한 환경이 설정되어있으므로 `Apply`는 수행하지 않는다.

<br>

[Hint]
- S/G - Public access
- RDS - Delete protection, backup retention period

<br>
<br>

😃 **Lab 1 완료!!!**

<br>

⏩ 다음 실습으로 [이동](6-2-AWS%20Security.md)합니다.