### 보안 그룹 구성
 
**public 과 private 2가지 구성**

1)보안 목록: private	(private subnet의 보안 목록) 
**수신**
|소스|IP프로토콜|소스포트 범위|대상 포트 범위|State Less|
|:---|:--:|:--:|:--:|:--:|
|10.0.55.0/24|모든|프로토콜|-|-|YES|
|10.0.0.0/16|모든|프로토콜|-|-|NO|
|0.0.0.0/0|TCP|모두|22|NO|
|0.0.0.0/0|TCP|모두|80|NO|
|0.0.0.0/0|TCP|모두|443|NO|
|0.0.0.0/0|TCP|모두|30000-32767|NO|
|0.0.0.0/0|모든프로토콜|-|-|NO|
|10.0.55.0/24|TCP|모두|32455|NO|
|10.0.55.0/24|TCP|모두|32455|NO|
|10.0.55.0/24|TCP|모두|10256|NO|
|10.0.55.0/24|TCP|모두|31423|NO|
|10.0.55.0/24|TCP|모두|32455|NO|

**송신**
|대상|IP프로토콜|소스포트 범위|대상 포트 범위|유형 및 코드|State Less|
|:---|:--:|:--:|:--:|:--:|:--:|
|10.0.10.0/24|모든 프로토콜|-|-|-|예|
|All ICN Services in Oracle Services Network|ICMP|-|-|모두|아니요|
|All ICN Services in Oracle Services Network|TCP|80|22|-|아니요|
|All ICN Services in Oracle Services Network|TCP|443|80|-|아니요|
