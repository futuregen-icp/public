

### 이슈 사항
url


###  Cert manager architecture
![enter image description here](https://github.com/futuregen-icp/public/blob/main/cert_manger/cert-manager%20architecture.jpg)

1.	cert-manager 발급자는 Let 's Encrypt 서비스에 등록합니다.
2.	등록이 완료되면 cert-manager가 도메인 목록을 보냅니다. 암호화 하는 URL 경로와 키를 cert-manager 인증서로 보냅니다.
3.	이후 인증 개체는 ACME 유효성을 검사 하기위해 세 개의 임시 항목을 만듭니다.
	1)	포트 8089에서  ACME POD
	2)	POD에 액세스하는 ACME 서비스
	3)	포트 8090을 가리키는 수신 컨트롤러의 Backend항목
4.	세 개의 임시 개체가 완전히 작동하면 Let 's Encrypt ACME 서비스가 도메인 및 URL의 POD에 액세스 할 수 있으며 POD에서 KEY를 받습니다. 이를 통해 Let 's Encrypt는 인증서 서명을 요청한 사람이 인증서에 나열된 도메인을 완전히 제어 할 수 있는지 확인했습니다.
5.	따라서 유효한 도메인과 ACME 서비스는 서명 된 인증서를 인증서 개체로 보냅니다.
6.	서명 된 인증서를 받은 후 cert-manager 인증서 개체는 인증서와 개인 키를 TLS 비밀로 encapsulate합니다.
7.	수신 컨트롤러는 이제 인증서와 키에 액세스 하고, 이를 사용하여 인터넷에서 HTTPS 세션을 종료 할 수 있습니다.
8.	인증서는 서명 된 인증서를 받으면 수신 컨트롤러의 POD, 서비스 및 Backend 항목을 삭제합니다.




## Cert_manager install

[enter link description here](https://github.com/futuregen-icp/public/blob/main/cert_manger/Cert_manager%20install.md)


## 인증서 생성 된다면 인증서 사용법

### Edit Ingres
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
	annotations:
	*** add an annotation indicating the issuer to use.
	*** secretName__으로 인증서 자동 생성_
	name: myIngress
	  namespace: myIngress
	spec:
	  rules:
		- host: example.com
			http:
				paths:
				- backend:
					serviceName: myservice
					servicePort: 80
tls:  *** < placing a host in the TLS config will indicate a certificate should be created
- hosts:
	- example.com
	secretName: myingress-cert 
	*** < cert-manager will store the created certificate in this secret
```

### ClusterIssuer 재 배포
```
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
	name: letsencrypt-prod
spec:
	acme:
		#preferredChain: "DST Root CA X3"
		server: https://acme-v02.api.letsencrypt.org/directory
		email: example@naver.com
		privateKeySecretRef:
			name: letsencrypt-prod
		solvers:
		- http01:
			ingress:
				class: nginx
```
- Solving Challenges 하지 않음, Order 생성 성공

### 인증서 배포

[enter link description here](https://github.com/futuregen-icp/public/blob/main/cert_manger/%EC%9D%B8%EC%A6%9D%EC%84%9C%20%EB%B0%B0%ED%8F%AC.md)

### check
```
[opc@bastion-instance ~]$ kubectl get Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,
Challenges --all-namespaces
NAMESPACE  	NAME	 										 	READY  AGE
			clusterissuer.cert-manager.io/letsencrypt-prod 		True  2d15h

NAMESPACE 		 NAME 									READY  SECRET 		 AGE
smartwork-dev  certificate.cert-manager.io/sm-dev-tls  	True  sm-dev-tls 	 2d15h

NAMESPACE 		 NAME 												 READY	  AGE
smartwork-dev  certificaterequest.cert-manager.io/sm-dev-tls-mldnc 	 True	  2d15h
smartwork-dev  certificaterequest.cert-manager.io/sm-dev-tls-pzt8j   True	  2d15h

NAMESPACE  		 NAME													  STATE    AGE
smartwork-dev  order.acme.cert-manager.io/sm-dev-tls-mldnc-1094753720	  valid   2d15h
smartwork-dev  order.acme.cert-manager.io/sm-dev-tls-pzt8j-1094753720 	  valid   2d15h
```

