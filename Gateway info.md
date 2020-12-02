### 서브넷의 주소를 미리 준비한다.
|10.0.55.0/24|퍼블릭|
|10.0.74.0/24| 프라이빗


### service_gateway 구성정보
|이름|서비스|
|:--:|:--:|
|service_gateway|All ICN Services In Oracle Services Network|


### Internet_gateway 구성정보
```
Internet_gateway 의 이름 지정 후 사용 생성가능
```


### 경로테이블
private	
|대상|대상 유형|대상|
|:--:|:--:|:--:|
|0.0.0.0/0|NAT 게이트웨이NAT_gateway_dev|	
|All ICN Services In Oracle Services Network|서비스 게이트웨이|service_gateway|
public
|대상|유형|대상
|0.0.0.0/0|인터넷 게이트웨이|internet_gateway_dev|


### NAT gateway 설정
```
공용IP주소를 선택하고 생성 가능
```
