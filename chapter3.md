# Chapter 3 
   

kubernetes 는 FlyingCube 2.0 ( OKD 4.7 ) 로 실습을 진행하다.  

1. kubernetes 설명

2. Kubernetes 의 API 서버 보안 ( Service Account )

3. Kubernetes 의 Downward API   

참고 : https://subicura.com/k8s/


 
<br/>

##  kubernetes

### 쿠버네티스 시작하기

개발 환경에서 당연하게 사용해왔던 쿠버네티스에 대해 이해하고, 왜 쿠버네티스를 사용하는지 알아보자.  

<br/>

개요
- 컨테이너 오케스트레이션의 개념과 사용하는 이유 그리고 특징에 대해 이해한다.

소개
- 쿠버네티스는 컨테이너 오케스트레이션 툴의 한 종류이며 엄청난 인기로 사실상 표준으로 사용된다.
- 컨테이너 오케스트레이션은 복잡한 컨테이너 환경을 효과적으로 관리하기 위한 도구이다.  

배경  
- 서버의 상태를 편하게 관리하기 위한 노력으로 도커 컨테이너가 등장했다.
- 그러나 관리하는 서버 컨테이너 수가 점점 증가하며 관리가 힘들다는 문제가 생겼다.
- 그래서 등장한 것이 컨테이너 오케스트레이션이다.  

컨테이너의 특징  

- 가상머신과 비교하여 컨테이너 생성이 쉽고 효율적
- 컨테이너 이미지를 이용한 배포와 롤백이 간단
- 언어나 프레임워크에 상관없이 애플리케이션을 동일한 방식으로 관리
- 개발, 테스팅, 운영 환경을 물론 로컬 피시와 클라우드까지 동일한 환경을 구축
- 특정 클라우드 벤더에 종속적이지 않음
  

<br/>

### 컨테이너 오케스트레이션

<br/>

컨테이너 오케스트레이션을 사용하지 않는다면

- 배포(Deployment)

    <img src="./assets/k8s_overview_1.png" style="width: 80%; height: auto;"/>  

    - 각 서버의 ip를 찾고 각 서버에 ssh로 접속해서 docker 명령어로 컨테이너를 실행 및 종료하는 수고가 든다.
    - 만약 새로운 컨테이너를 실행하려면 빈 서버(여유있는)에 실행 하는것이 리소스 절약을 위해 좋으나, 이를 확인하기 위한 모니터링 툴이 없으면 빈 서버를 일일히 찾기도 힘들다.
    - 서버를 배포할 때(version upgrade 또는 rollback) 모든 서버를 한번에 배포하는 방법이 필요하다.  

<br/>

- 서비스 검색(Service Discovery)  

    <img src="./assets/k8s_overview_2.png" style="width: 80%; height: auto;"/>  

    - 보통의 구조라면 프록시 서버가 있고 로드밸런서를 통해 서버에 적절히 부하를 분산한다.
    - 그러나 로드밸런서와 서버의 ip 설정과 같은 부분이 관리자가 직접 관리해야하는 포인트였다.
    - 마이크로서비스 환경이 유행처럼 등장하며 서버가 점점 많아지고 서버의 ip가 업데이트로 변경되고 하면서 관리자가 이를 모두 관리하는 것은 쉽지 않았다.

<br/>

- 서비스 노출(Gateway)

    <img src="./assets/k8s_overview_3.png" style="width: 80%; height: auto;"/>  

    - NginX와 같이 외부에 노출된 프록시 서버를 두고, 프록시 서버로 들어오는 host 요청에 따라 내부 컨테이너(서버)에 연결하는데 이 과정에 자동화가 필요했다.
 
<br/>

- 서비스 이상과 부하 모니터링

    <img src="./assets/k8s_overview_4.png" style="width: 80%; height: auto;"/>  

    - 갑자기 컨테이너가 죽은 경우에 이전에는 일일히 로그 확인해서 다시 서버를 띄워야 했다.
        - 같은 서버 컨테이너가 3개 돌다가 하나의 컨테이너가 죽으면 남은 2개의 서버에 부하가 생긴다.
    - 서버가 죽지는 않았는데 트래픽이 많아지면 부하가 걸려 느려졌다.
        - 트래픽에 따라 적절하게 서버를 늘려야 함
    - 위와 같은 상황으로 자동화가 필요했다.

 
<br/>

컨테이너 오케스트레이션  

- 위와같은 문제로 많은 컨테이너를 효율적으로 관리하기 위한 기술이 컨테이너 오케스트레이션이다.
- 컨테이너 오케스트레이션 복잡한 컨테이너 환경을 효과적으로 관리하기 위한 도구이다.

<br/>

컨테이너 오케스트레이션 특징  

- 클러스터(Cluster)  

    <img src="./assets/k8s_overview_5.png" style="width: 80%; height: auto;"/>  

    - 중앙 제어
        - 이전에는 서로 다른 노드의 CPU와 RAM 상태를 각각 관리했었다.
        - 그러나 노드 수가 증가하면 힘들기 때문에, 컨테이너 오케스트레이션에서는 합쳐서 추상화하여 클러스트 단위로 관리한다.
        - 클러스터 하나하나의 노드에 ssh로 직접 접속하기 힘들기때문에 프록시처럼 앞에 마스터 서버를 두고 마스터서버가 각 노드에 알아서 명령을 보낸다.  

    - 네트워킹
        - 클러스터 내 노드끼리는 서로 통신이 잘되어야 한다.
    - 노드 스케일
        - 노드 스케일이 커지더라도 잘 돌아가기 위해서는 정교한 설계가  필요하다.  

<br/>

- 상태 관리(State)  

    <img src="./assets/k8s_overview_6.png" style="width: 80%; height: auto;"/>  

    - 트래픽이 증가해 서버를 새로 늘리거나, 하나의 서버에 장애가 발생했을 때 감지하고 자동으로 서버를 늘려준다.
    - 오토 스케일링

<br/>

- 배포 관리(Scheduling)  

    <img src="./assets/k8s_overview_7.png" style="width: 80%; height: auto;"/>  

    - 자원이 여유가 있는 서버에 알아서 적절하게 띄워준다.

<br/>

- 배포 버전관리(Rollout & Rollback)  

    <img src="./assets/k8s_overview_8.png" style="width: 80%; height: auto;"/>  

    - 전체 컨테이너에 대한 롤아웃과 롤백을 중앙에서 관리한다.

<br/>

- 서비스 등록 및 조회(Service Discovery)  

    <img src="./assets/k8s_overview_9.png" style="width: 80%; height: auto;"/>  

    - 새로 등록된 서비스 ip나 변경된 ip를 자동으로 관리해줘서 관리자가 하나하나 수정할필요가 없다.


<br/>

- 볼륨 스토리지(Volume)  

    <img src="./assets/k8s_overview_10.png" style="width: 80%; height: auto;"/>  

    - 각 서버에 필요한 스토리지(AWS EBS, GCE 등)를 설정으로 연결할 수 있다.
    - AWS에서 EC2에 스토리지 일일히 하나하나 붙이고 그럴 필요가 없다는 말이다.  

<br/>

왜 쿠버네티스인가  

- 컨테이너 관리도구가 많이 생기고 했으나, 쿠버네티스가 표준처럼 등장했다.
- 오픈소스  

    - 컨테이너를 쉽고 빠르게 배포/확장하고 관리를 자동화해주는 오픈소스 플랫폼
    - 구글에서 만듬 (구글은 1주일에 20억개 컨테이너 생성한다.)  

- 엄청난 인기
    - 점유율이 높고 그렇기에 라이브러리 또는 레퍼런스가 많다.  

- 무한한 확장성
    - 쿠버네티스 위에서 머신러닝, CI/CD, 서비스. 서버리스 등 다양한 서비스가 동작
    - 쿠버네티스만 설치되어 있으면 거기에 서비스를 올리기 쉬움  

- 사실상 표준(de facto)
    - 많은 오케스트레이션이 있지만 사실상 표준이 되어가고 있음
    - 도커도 자체 오케스트레이션이 있지만 쿠버네티스의 인기로 인해 어쩔수없이 쿠버네티스 지원
    - AWS(Elastic Kubernetes Service), Azure(Azure Kubernetes Service), Google(Google Kubernetes Engine)와 같이 대표적인 클라우드 기업들이 쿠버네티스를 매니지드 서비스로 제공하고 있음  

<br/>


### kubernetes API 서버 보안

<br/>

쿠버네티스에서 API 서버 보안은 왜 필요할까?  

쿠버네티스에서는 이론적으로 파드 외부 또는 내부에서 API 서버로 적절한 요청을 하면 어떤 리소스던 생성, 삭제, 수정, 조회가 가능하다.  

그런데 만약 개발자의 코딩 실수로 어떤 파드에서 아무 관련이 없는 다른 파드를 삭제해버릴 수 있다면 큰 문제가 될 것이다.  

그렇기 때문에 사용자 또는 파드에게 적절하게 권한을 부여하는 기능은 보안과 안정적인 운영을 위해 필수적이다.  

회사에서 여러개의 팀이 하나의 공용 클러스터를 함께 사용하는 멀티테넌트 환경에서는 이러한 권한의 분리가 더욱 중요하다.

쿠버네티스 클러스터는  API 서버가 사용자나 Pod 의 요청을 받을 때 명시적으로 설정된 권한만 허용하고, 그 이외의 모든 권한은 허용하지 않도록 동작한다.

<br/>

#### Service Account 란?  

<br/>

서비스 어카운트(Service Account) 는 Kubernetes 의 파드에서 API 서버에 요청을 보냈을 때 이 "파드"를 식별하기 위한 리소스다. (사용자를 식별하는데 사용되지는  않는다)  

파드에서 API 서버에 요청을 보내면 이 파드의 정체가 무엇인지 알아야 어떤 권한을 가지고 있는지도 알 수 있고,  

이를 기반으로 파드의 요청이 권한에 맞는지를 확인하여 요청을 처리해줄지 말지를 결정할 것이다. 

실제로 권한을 정의하고, 설정하는 부분은 이후에 설명할 Role, ClusterRole, RoleBinding, ClusterRoleBinding 의 역할이다.

<br/>

#### ServiceAccount 의 특징  

<br/>

모든 파드는 무조건 하나의 ServiceAccount 와 매핑이 되어야 실행될 수가 있다.  

그런데 ServiceAccount 를 만들지 않고, 파드의 매니페스트에 ServiceAccount 를 명시적으로 적어주지 않아도 파드가 잘 생성이 되고 실행 되는 것을 보고 의아할 수도 있다.  

사실 이것은, 쿠버네티스의 ServiceAccount Controller 가 모든 네임스페이스에 default 라는 이름의 서비스어카운트가 있도록 자동 생성해주며,  

ServiceAccount `Admission Controller` 가 파드의 매니페스트에 명시적으로 서비스어카운트를 정의하지 않으면 default 서비스어카운트를 매핑해주기 때문에 가능한것이다.  

뿐만 아니라, 서비스어카운트는 mountable secrets 에 지정한 시크릿만 파드에 마운트할 수 있도록 강제하는 기능과,  

Image pull secrets 기능을 통해 프라이빗 이미지 레지스트리에서 이미지를 가져올 수 있도록 하기 위한 시크릿을 이 서비스어카운트를 사용하는 파드에 자동으로 마운트시켜주는 기능도 가지고 있는데  

image pull secrets 기능도 ServiceAccount Admission Controller 가 수행한다.  

<br/>

#### ServiceAccount 의 동작 방식  

<br/>

kubectl create sa <name> 명령어로 서비스어카운트를 생성할 수 있다.  


`Token Controller` 는 서비스어카운트가 생성될 때마다 자동으로 kubernetes.io/service-account-token 타입의 Secret 을 생성하여 매핑시켜준다.  

```bash
root@jakelee:~# kubectl get sa
NAME      SECRETS   AGE
default   1         18d
root@jakelee:~# kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-p9fpr   kubernetes.io/service-account-token   3      18d
root@jakelee:~# kubectl describe secret default-token-p9fpr
Name:         default-token-p9fpr
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: ef9e4103-6663-4fcb-a12f-e2626d7e9666

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImhvbzBnMGhOX1VBMFp6MVN4ejFoX0RoVEtBX25lZUNkX291d3BnLUNWWVEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tcDlmcHIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImVmOWU0MTAzLTY2NjMtNGZjYi1hMTJmLWUyNjI2ZDdlOTY2NiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.wHfW2CGCePikcfSumoM9GvUIQS_8gI_wqKRnz9XzRFJ7rBl2iZNM7h47rPhHcA-uR61G6C6mzFHzb0zVYfo_PQAuddYF1PB3FZ9DmlylX3J1ne2KBQhh_f88FXmRCQZbiK5DS_GiR1Ys2UWzykLQFXtHDrQ7n_oy3Bq3poi4lDObg_TIH2rSyrhADebDqRXHOUZ5GGNV3Y7qwTvl9hFhbPT6Z8VO0mGgpHvv52XmNzHORUhwZRsMBLlMh19DNVTizvRlhemTZJMM71d8wJ9KG9rFhDwodWzuWgCYNxklWDvhDCsuJSj1QHei-9QSdqy2YTp5Qddcux7nMgyufQZT9Q
ca.crt:     570 bytes
namespace:  7 bytes
```  


Token Controller 에 의해 생성된 Secret 에는 아래와 같은 3가지 데이터가 base64 로 인코딩되어 들어있다.  

- ca.crt - API 서버와 통신 시, SSL 인증을 위한 증명서
- token - 서비스어카운트 이름, Secret 이름 등의 정보에 서명한 JWT.   
           API 서버에 요청시 Bearer 토큰으로 사용됨
- namespace - 네임스페이스  

파드 내 애플리케이션은 위의 데이터를 사용하여 API 서버와 통신한다.

API 서버는 요청의 Authorization 헤더에 있는 Bearer token 을 디코딩하여 어느 서비스어카운트를 사용해 보낸 요청인지 식별하게 된다.

<br/>  

#### RBAC 란?

<br/>  

어떤 사람, 혹은 파드가 API 서버에 요청을 하면 API 서버는 인증(Authentication)과 인가(Authorization)를 수행한다.  

인증은 접근 가능 여부를 확인하는 것이고, 인가는 접근 가능한 요청에 대해 요청된 자원에 접근할 수 있는지를 확인하는 것이다.  

RBAC(Role-Based Access Control) 는 API 서버가 인가를 수행하는 여러 방법 중 하나다.  

쿠버네티스는 다음과 같은 인가 방식을 제공한다.  

- Node
    - Kubelet 에 의한 요청에 대한 인가를 위한 방식
- ABAC(Attribute-Based Access Control)
    - 리소스의 속성에 따라 인가를 하는 방식
- RBAC
    - Role 을 기반으로 인가를 하는 방식
- Webhook
    - 외부 API 를 통해 인가를 하는 방식  

이 중에서 RBAC가 표준이며, kubernetes 1.8.0 부터는 대부분의 클러스터에서 기본적으로 사용하는 방식이다.  

RBAC 는 `특정 주체(subject) 가 특정 대상(url, resource 타입, 혹은 특정 resource) 에 대해 특정 행위(verb)` 를 할 수 있는지를 지정하는 방식이다.  

<br/>  

#### Role 과 RoleBinding  


<br/>  

어떤 사람, 혹은 파드가 API 서버에 요청을 하면 API 서버는 인증

Role 은 대상(resource 등) 과 행위(verb) 를 지정하며, RoleBinding 은 주체(subject) 를 지정한다.  

여기서 주체는 3가지(User, Group, ServiceAccount) 중 한가지가 된다.  

대상은 보통 resource 의 타입을 정하는데, resourceName 으로 특정 리소스를 지정할 수도 있다.   

이후에 설명할 ClusterRole 에서는 리소스가 아닌 URL 을 지정할 수도 있다. /healthz 와 같이 특정 리소스에 대한 요청이 아닌 경우도 있기 때문이다.  

행위는 아래 표와같이 API 요청에 사용된 HTTP 메서드에 따라 특정 행위에 매핑되는데,  


<img src="./assets/role_rolebinding.png" style="width: 80%; height: auto;"/>   

행위에 대한 대상이 리소스라면 행위를 verb 로 적어주고, URL 이라면 HTTP 메소드로 적어준다.  

이름 그대로 Role 은 역할이고, RoleBinding 은 이러한 역할을, 역할을 수행하는 주체에 연결시켜 주는 것이다.  

<br/>

#### Role 과 RoleBinding 의 특징  

<br/>

Role 과 RoleBinding 은 특정 네임스페이스에 종속된 개념이다. 그렇기 때문에 RoleBinding 은 다른 네임스페이스의 Role 을 바인딩해줄 수는 없다.  

하지만, RoleBinding 이 같은 네임스페이스의 Role 을 다른 네임스페이스의 subject 에게 바인딩해 줄 수는 있다.  

그래서 RoleBinding 에 subject 들을 명시할 때는 name 과 namespace, 그리고 kind(user/group/serviceaccount) 를 함께 명시한다.  

하나의 Role 은 여러개의 RoleBinding 에 의해 바인드될 수 있고, 하나의 RoleBinding 은 하나의 Role 만 참조할 수 있다.  

즉, `Role 과 RoleBinding 은 일대다(one-to-many)` 관계다.  

반면 하나의 RoleBinding 은 하나의 Role 을 여러 주체에 연결시켜 줄 수 있고, 하나의 주체는 여러개의 RoleBinding 에 의해 권한이 부여될 수 있다.  

즉, `RoleBinding 과 Subject(ServiceAccount 등) 는 다대다(many-to-many)` 관계다.  

<br/>

#### Role 과 RoleBinding 만들기

<br/>

Role 과 RoleBinidng 을 만드는 방법은 크게 두가지가 있다.  

하나는 매니페스트를 통해 만드는 방법이고, 나머지 하나는 kubectl create 명령어를 통해 만드는 방법이다.  

다음과 같이 YAML 파일을 작성하지 않고도 직접 verb, resource, role, serviceaccount 등을 인자로 주어 Role 과 RoleBinding 을 생성할 수 있다.  

먼저 아래 소스를 사용하여 role을 생성합니다.  

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: service-reader
rules:
  - apiGroups: [""]
    verbs: ["get", "list"]
    resources: ["services"]
```

### kuKubernetes 의 Downward API
  
<br/>

#### Downward API 란?

<br/>

애플리케이션이 실행되기 전에 이미 알고있는 속성이나 설정 값들은 ConfigMap 이나 Secret 으로 파드에 전달할 수 있지만,  

파드의 이름, 파드의 IP, 파드가 실행되는 노드의 이름 등 실제로 파드가 생성 및 실행이 되기전에는 알 수 없는 속성들도 존재한다.  

물론 파드의 레이블이나 어노테이션과 같은 일부 속성들은 파드 생성 이전에도 알 수 있지만,  

파드 내에서 정보를 사용하고 싶다는 이유로 이미 설정되어 있는 속성을 ConfigMap 등을 통해 중복하여 정의하고 싶지는 않을 것이다.  

이런 속성들을 컨테이너에서 실행 중인 애플리케이션에서 알아내려면 어떻게 해야할까?  

이 때 사용되는 것이 Downward API 이다.  

Downward API 는 단순히 환경변수, 또는 파일(downwardAPI 볼륨을 통해) 로 위와 같은 속성들을 컨테이너에서 손쉽게 사용할 수 있도록 하는 기능일 뿐이다.  

Downward API 를 통해 전달할 수 있는 정보는 다음과 같다.  

- 파드의 이름
- 파드의 IP 주소
- 파드가 속한 네임스페이스
- 파드가 실행중인 노드의 이름
- 파드가 실행 중인 서비스 어카운트 이름
- 각 컨테이너의 CPU와 메모리 request
- 각 컨테이너의 CPU와 메모리 limit
- 파드의 label
- 파드의 annotation

k8s 에서는 파드가 API server 와 통신할 수 있도록 하기 위해  
각 파드마다 기본적으로 Default token 시크릿 볼륨을 만들어  

파드 내 컨테이너의 /var/run/secrets/kubernetes.io/serviceaccount/에 마운트해 주는데,  

이 곳에 namespace 라는 파일에 네임스페이스가 적혀있기 때문이다.  

```bash
root@jakelee:~# kubectl get po
NAME                                  READY   STATUS    RESTARTS   AGE
flask-edu4-app-74788b6479-nmcvv       1/1     Running   0          16d
flask-edu4-app-74788b6479-rlght       1/1     Running   0          16d
hpa-example-deploy-59bf97fcc6-6nxjs   1/1     Running   0          16d
inspekt-deployment-c8d9f5dcf-2slpx    1/1     Running   0          11d
mynginx-69d586ff67-bmh6m              1/1     Running   0          10d
web2-5d47994f45-dmrwr                 1/1     Running   0          9d
pod-test-app                          1/1     Running   0          6d20h
root@jakelee:~# kubectl exec -it pod-test-app sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# ls /var/run/secrets/kubernetes.io/serviceaccount/
ca.crt	namespace  token
# cat /var/run/secrets/kubernetes.io/serviceaccount/namespace
default
```  

<br/>

#### 환경 변수로 전달하기 vs 볼륨으로 전달하기

<br/>

Downward API 를 통해 데이터를 전달하기 위한 방법으로는  

환경 변수를 통한 방법과 볼륨을 통한 방법, 이렇게 크게 두가지가 있다.  

대부분의 경우 환경변수를 통한 방법과 볼륨을 통한 방법 중 어떤 방법을 사용해도 크게 문제가 없지만  

약간의 차이점이 있다. 우선 일부 정보들은 둘 중 한가지 방법으로만 얻을 수 있다.  

예를 들어 Pod 의 label 과 annotation 은 downwardAPI 볼륨을 통해서만 전달할 수 있다.   

그 이유는, Pod 의 label 과 annotation 은 Pod 가 실행되는 동안 수정될 수가 있는데,  

이 때 Pod 가 변경된 데이터를 볼 수 있도록 해야 한다.  

파드가 실행중인 노드의 이름과 IP 는 환경 변수를 통한 방법으로만 얻을 수 있다.  

<br/>

#### 환경변수로 전달하기   

<br/>


vi에디터로 downward-env.yaml 화일을 만들어 보자.   

```bash
root@jakelee:~# vi downward-env.yaml
```

아래 소스를 사용한다.  


```bash
apiVersion: v1
kind: Pod
metadata:
  name: downward-env
spec:
  containers:
  - name: main
    image: busybox
    command: ["sleep", "99999"]
    resources:
      requests:
        cpu: 15m
        memory: 100Ki
      limits:
        cpu: 100m
        memory: 20Mi
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: SERVICE_ACCOUNT
      valueFrom:
        fieldRef:
          fieldPath: spec.serviceAccountName
    - name: CONTAINER_CPU_REQUEST_MILLICORES
      valueFrom:
        resourceFieldRef:
          resource: requests.cpu
          divisor: 1m
    - name: CONTAINER_MEMORY_LIMIT_KIBIBYTES
      valueFrom:
        resourceFieldRef:
          resource: limits.memory
          divisor: 1Ki
```  

pod를 생성한다.  

```bash
root@jakelee:~# kubectl apply -f downward-env.yaml
pod/downward-env created
```

```bash
root@jakelee:~# kubectl get po
NAME                                  READY   STATUS    RESTARTS   AGE
pod-test-app                          1/1     Running   0          6d20h
downward-env                          1/1     Running   0          12s
```  

pod가 생성이 되면 환경 변수를 조회해 본다.  

기본 환경 변수에 POD_NAME , POD_NAMESPACE , POD_IP , POD_NAME이 추가 된 것을 확인 할 수 있다.  

```bash
root@jakelee:~# kubectl exec downward-env env
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=downward-env
SERVICE_ACCOUNT=default
CONTAINER_CPU_REQUEST_MILLICORES=15
CONTAINER_MEMORY_LIMIT_KIBIBYTES=20480
POD_NAME=downward-env
POD_NAMESPACE=default
POD_IP=10.42.0.130
NODE_NAME=jakelee
```  

<img src="./assets/downward_api_pod.png" style="width: 80%; height: auto;"/> 

<br/>

#### 볼륨으로 전달하기  

<br/>

vi에디터로 downward-volume.yaml 화일을 만들어 보자.   

```bash
root@jakelee:~# vi downward-volume.yaml
```

아래 소스를 사용한다.  


```bash
apiVersion: v1
kind: Pod
metadata:
  name: downward-volume
  labels:
    foo: bar
  annotations:
    key1: value1
    key2: |
      multi
      line
      value
spec:
  containers:
  - name: main
    image: busybox
    command: ["sleep", "9999999"]
    resources:
      requests:
        cpu: 15m
        memory: 100Ki
      limits:
        cpu: 100m
        memory: 40Mi
    volumeMounts:
    - name: downward
      mountPath: /etc/downward
  volumes:
  - name: downward
    downwardAPI:
      items:
      - path: "podName"
        fieldRef:
          fieldPath: metadata.name
      - path: "podNamespace"
        fieldRef:
          fieldPath: metadata.namespace
      - path: "labels"
        fieldRef:
          fieldPath: metadata.labels
      - path: "annotations"
        fieldRef:
          fieldPath: metadata.annotations
      - path: "containerCpuRequestMilliCores"
        resourceFieldRef:
          containerName: main
          resource: requests.cpu
          divisor: 1m
      - path: "containerMemoryLimitBytes"
        resourceFieldRef:
          containerName: main
          resource: limits.memory
          divisor: 1
```  

pod가 생성이 되면 volume을 조회해 본다.  


```bash
root@jakelee:~# kubectl get po
NAME                                  READY   STATUS    RESTARTS   AGE
pod-test-app                          1/1     Running   0          6d20h
downward-env                          1/1     Running   0          14m
downward-volume                       1/1     Running   0          68s
root@jakelee:~# kubectl exec downward-volume  ls /etc/downward
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
annotations
containerCpuRequestMilliCores
containerMemoryLimitBytes
labels
podName
podNamespace
root@jakelee:~# kubectl exec downward-volume  cat /etc/downward/podName
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
downward-volume
```  

<br/>
