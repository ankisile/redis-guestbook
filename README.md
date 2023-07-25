## 20. 최종 연습문제

### 20.1 PHP GuestBook with Redis

### 20.1.1 과제 개요

- 레디스 마스터 구성
- 레디스 슬레이브 구성
- Guestbook 프론트앤드 구성
- 프로트앤드 애플리케이션 서비스로 노출 시키기
- 리포지토리 만들고 ArgoCD 연동하기

### 20.1.2 레디스 마스터 Deployment 로 구성

| 항목 | 값 |
| --- | --- |
| 파일명 | redis-master-deployment.yaml |
| Deployment name | redis-master |
| Deployment Label | app: redis |
| deployment Selector / pod metadata | app: redis / role: master / tier: backend |
| replica | 1 |
| Image | k8s.gcr.io/redis:e2e |
| Port | 6379 |

> kubectl logs -f POD-NAME
> 

### 20.1.3 레디스 마스터 서비스 구성

| 항목 | 값 |
| --- | --- |
| Type | ClusterIP |
| Service 이름 | redis-master |
| Service Label | app: redis / role: master / tier: backend |
| port / targetPort | 6379 / 6379 |
| selector | app: redis / role: master / tier: backend |

### 20.1.4 레디스 슬레이브 Deployment 구성

| 항목 | 값 |
| --- | --- |
| 파일명 | redis-slave-deployment.yaml |
| Deployment Name | redis-slave |
| Deployment Label | app: redis |
| Deployment matchLabel | app: redis / role: slave / tier: backend |
| Replicas : 2 |  |
| Pod Label | app: redis / role: slave / tier: backend |
| Container name | slave |
| Image | gcr.io/google_samples/gb-redis-follower:v2 |
| 환경변수 : GET_HOSTS_FROM | 마스터 노드의 주소로 지정 하되 value=dns 로 설정 |
| containerPort | 6379 |
- 아래와 같이 소스 코드를 보면 GET_HOSTS_FROM 에는 master 의 주소가 들어가야함

```
 $host = 'redis-master';
  if (getenv('GET_HOSTS_FROM') == 'env') {
    $host = getenv('REDIS_MASTER_SERVICE_HOST');
  }
```

### 20.1.5 레디스 슬레이브 서비스 구성

| 항목 | 값 |
| --- | --- |
| Type | ClusterIP |
| Service 이름 | redis-slave |
| Service Label | app: redis / role: slave / tier: backend |
| port / targetPort | 6379 / 6379 |
| selector | app: redis / role: slaver / tier: backend |

### 20.1.6 GuestBook 애플리케이션 Deployment 생성

| 항목 | 값 |
| --- | --- |
| 파일명 | frontend-deployment.yaml |
| Deployment Name | frontend |
| Deployment Label | app: guestbook |
| Deployment matchLabel | app: guestbook / tier: frontend |
| Replicas : 3 |  |
| Pod Label | app: guestbook / tier: frontend |
| Container name | php-redis |
| Image | gcr.io/google-samples/gb-frontend:v4 |
| 환경변수 : GET_HOSTS_FROM | 마스터 노드의 주소로 지정 하되 value=dns 로 설정 |
| containerPort | 80 |

### 20.1.7 GuestBook 애플리케이션 LoadBalancer 서비스 구성

| 항목 | 값 |
| --- | --- |
| Service name | frontend |
| service Label | app: guestbook / tier: frontend |
| Type | LoadBalancer |
| Port | 80 |
| selector | app: guestbook / tier: frontend |

### 20.1.8 FrontEnd 앱 스케일링 해보기

replica 를 5개로 스케일링 해보기