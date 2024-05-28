# 2일차 - Lab 1. Draw.io 실습

## 1. 목표

- CTA 과정을 진행하며 구축하게될 인프라에 대한 다이어그램을 draw.io을 이용하여 그려봅니다.
- 아래의 설명을 토대로 다이어그램을 그립니다.

## 2. 설명

- **📌 [조건]**

> | 항목                  | 내용                                                                               |
> | --------------------- | ---------------------------------------------------------------------------------- |
> | ➕ Region             | 서울리전 (`ap-northeast-2`)                                                        |
> | ➕ 사용 AZ            | `ap-northeast-2a`, `ap-northeast-2c`                                               |
> | ➕ 서브넷             | `Public Subnet` 2개, `Private Subnet for App` 2개, `Private Subnet for DB` 2개     |
> | ➕ 인터넷 게이트 웨이 | Public Subnet을 위한 인터넷 게이트웨이 존재                                        |
> | ➕ NAT 게이트웨이     | `Private Subnet for App`를 위한 `NAT 게이트웨이` 존재, 퍼블릭 서브넷 상에 2개 존재 |
> | ➕ EKS                | App을 위한 `EKS`가 존재, `Private subnet for App`에 존재                           |
> | ➕ EKS NodeGroup      | 평시 인스턴스 3대 존재, 오토스케일링 Max값 5대                                     |
> | ➕ DB                 | `Amzon RDS(엔진 : Maria DB)`, `Priavate Subnet for DB`에 존재                      |
> | ➕ EFS                | EKS PV를 위해 `EFS` 사용                                                           |
> | ➕ 서비스노출         | K8S 서비스 외부 노출을 위해 `istio`와 `NLB`, `ACM`, `route53`을 이용               |

## 3. 제출

- 다이어그램을 그린 후 제출합니다.
