## ***cluster config***

**구성 순서**

## 1. vcn 구성 정보
### 구성 정보

|Host|OS|Instance Spec|
|:--:|:--:|:--:|
|Bastion|Oracle Linux 7.9|E.x|
|db01|Oracle Linux 7.9|E.x|
|db02|Oracle Linux 7.9|E.x|
|oke|Oracle Linux 7.9|E.x|
 
## 2. Gateway 구성 

 |순서|항목|역활|
|:--:|:--:|:---|
|1|service gateway|oracle cloud service 이용 시 필요| 
|2|internet gateway|인터넷 연결을 하는 gateway|
|3|경로 테이블|경로 테이블을 지정하여 라우팅 해줌|
|4|보안 그룹|subnet 마다 필요로 하는 보안 규칙들을 만들어 놓을수 있음, firewall 과 같은 개념|
|5|subnet 생성|비공인 IP를 부여 하여 Instance 에 적용 가능함|   
|6|NAT gateway|subnet에 Work node 가 외부 인터넷을 필요로 할 때 사용|
 
 
   ### Gateway 구성 정보
### [gateway.md](https://github.com/futuregen-icp/public/blob/main/gatway.md.md)


### 보안 그룹 구성 정보
### [security group.md](https://github.com/futuregen-icp/public/blob/main/security%20group.md.md)

## 3. 서브넷 구성 

|서브넷 이름|CIDR|보안그룹|
|:--:|:--:|:--:|
|public|10.0.55.0/24|public|
|private|10.0.74.0/24|private| 

	위의 내용으로 구성

## 4. 인스턴스 구성
- bastion instance 구성
 
 bastion instance 구성 시 공용키 를 다운 받아 관리 필요
 
```
 
- Private key (ssh-key-2020-11-26.key)를 사용하여 접속
 bastion instance 의 기본 접속 계정
  user ID: opc  
   
 # ssh -i ssh-key-2020-11-26.key opc@<bastion_IP>
 
 - window putty 접속 방법
 1. putty gen 을 사용 하여 인증서를 생성
 
 2. Private key (ssh-key-2020-11-26.key) 를 사용하여 접속한 뒤 
 
 # vi $HOME/.ssh/authorized_keys 에 생성된 ppk 의 코드를 붙여넣습니다.
 
 3. 생성한 인증서로 putty 접속 시도합니다.
 
 ```

- ocl 를 이용하기 위한 인증 

[enter link description here](https://drive.google.com/file/d/1-ww7Nh2Gco8HkSXMPxzN91r-0GR38hpz/view?usp=sharing)

위의 파일을 다운 받아 참고하여 진행

## 5. OKE cluster install

[OKE cluster](https://github.com/futuregen-icp/public/blob/main/oracle/OKE%20cluster.md)

