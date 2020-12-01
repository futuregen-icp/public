## vcn 구성

```
구성 정보

|Host   |OS              |Instance Spec|

|Bastion|Oracle Linux 7.9|E.x|
|db01   |Oracle Linux 7.9|E.x|
|db02   |Oracle Linux 7.9|E.x|
|oke    |Oracle Linux 7.9|E.x|
```

### Gateway 구성순서
```
sevic gateway : oracle cloud servic 이용시 필요 
    |
    |
internet gateway : 인터넷 연결을 하는 gateway
    |   
    |
경로 테이블 : 경로 테이블을 지정하여 라우팅 해줌
    |   
    |
보안 그룹 : subnet 마다 필요로 하는 보안 규칙들을 만들어 놓을수 있음, firewall 과 같은 개념
    |   
    |
Subnet 생성 :  비공인 IP를 부여 하여 Instance 에 적용 가능함   
    |
    |
NAT gateway :  subnet에 Work node 가 외부 인터넷을 필요로 할때 사용

```
