


## 이슈 사항

### Cert-manager 설치 후 http01에서 Challenges가 동작 하지 않았습니다.
해당 이슈 내용

```
When using HTTP(s) protocols for your Load Balancer, it can intercept the challenge URL to replace the response’s verification hash with their hash.

In this case, cert-manager will fail did not get expected response when querying endpoint, expected 'xxxx' but got: yyyy (truncated).
```

### 해결 방안
```
- DNS webhook을 사용 하여 해당 도메인 챌린저가 승인, 하지만 인증서 생성은 되지 않았습니다.

->  위의 이슈사항에 대한 판단이 서지 않아 SR 지원으로 답변을 듣고자 합니다.

- 위의 상황에서 ' acme ' 설정 복구 후 해당 도메인에 챌린저를 실행 하지 않고 오더 승인 되어 인증서 생성에 성공 했습니다.
```


### 참고 문서 URL

[https://gitlab.com/dn13/cert-manager-webhook-oci/-/tree/develop](https://gitlab.com/dn13/cert-manager-webhook-oci/-/tree/develop)

Cluster Issuer ,OCI profile Guide

[https://cert-manager.io/docs/installation/kubernetes/](https://cert-manager.io/docs/installation/kubernetes/)

Cert manager install docs
