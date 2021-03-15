

## 요점

oke 구성 했던 cert-manager 1.2.1var 에서 사용된 oci web hook 이 불 안전한 방법 임으로 대안 책을 찾아 진행한 작업입니다.

## error 처리 
 kubernetes 환경에서 clusterissuer 가 fales 에 머무는 현상은  certmanager install 후 namespace kubesystem 의 coredns, ingress-controller pod 들 이 정상 가동이 되지 않은  상황이 생겨 certmanager install 이후에 위의 pod들을 재 생성 시켜야 하는 문제를 발견하였습니다.이후 다른 cluster configuration시 동일한 문제가 발견되어 pod Regeneration을 진행 하였습니다.

##  환경 구성 내용

oracle cloud instructur 
 - OKE cluster configration
    * ingress install 
    * ingress controll install
    * LB config
    * calico install


## 작업 절차

1. certmanager install
2. 설치후 10~30분 가량의 텀을 두고 coredns,ingress-controller pods check
3. 정상구동이 되지 않는 pod들을 재생성
4. clusterissuer 생성 (prod, stageing 두가지 생성) 
5. kubectl get Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,Challenges --all-namespaces 실행
6. Clusterissuer 의 상태가 ture 확인
7. ingress 생성 (certmanager anotastion 추가)
8. kubectl get Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,Challenges --all-namespaces 실행






## cert-manager install
```
#  kubectl apply  -f https://github.com/jetstack/cert-manager/releases/download/v1.1.1/cert-manager.yaml

kubectl get pod -A
cert-manager     cert-manager-857cf4745-wlk68                     1/1     Running   0          4h29m
cert-manager     cert-manager-cainjector-84997cc878-qnv2m         1/1     Running   0          4h34m
cert-manager     cert-manager-webhook-546d97788c-nsxp4            1/1     Running   0          4h34m

////certmanager install check
    pod 생성 명령어를 보면 현재 certmanager v1.1.1 사용
```
## Cluster Issuer install
### Clusterissuer prod
```

apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: ex@example.com
    preferredChain: "ISRG Root X1"
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
          
```
### Clusterissuer staging
```

apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: ex@example.com
    preferredChain: "ISRG Root X1"
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - selector: {}
      http01:
        ingress:
          class: nginx
          
```
### 생성 및 확인
```

# kubectl apply -f Clusterissuer_prod.yaml
# kubectl apply -f Clusterissuer_staging.yaml

***생성 확인
# kubectl get Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,Challenges --all-namespaces
NAME                                                READY   AGE
clusterissuer.cert-manager.io/letsencrypt-prod      True    24h
clusterissuer.cert-manager.io/letsencrypt-staging   True    24h

```

### ingress 구성 및 인증서 발급
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ex@example.com-ingress
  namespace: smart-test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/cors-allow-origin: '*'
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/service-upstream: "true"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  tls:
  - hosts:
    -  ex@example.com
    secretName: ex@example.com-tls
  rules:
    - host: ex@example.com.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              serviceName: web
              servicePort: 8080
```
### 생성 및 확인
```
# kubectl apply -f inagress.yaml


***생성 확인
# kubectl get Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,Challenges --all-namespaces

NAME                                                READY   AGE
clusterissuer.cert-manager.io/letsencrypt-prod      True    24h
clusterissuer.cert-manager.io/letsencrypt-staging   True    24h

NAMESPACE        NAME                                                       READY   SECRET                         AGE
smartwork-test   certificate.cert-manager.io/sm-test-fe.moimstone.com-tls   True    sm-test-fe.moimstone.com-tls   24h
smartwork-test   certificate.cert-manager.io/sm-test.moimstone.com-tls      True    sm-test.moimstone.com-tls      24h

NAMESPACE        NAME                                                                 READY   AGE
smartwork-test   certificaterequest.cert-manager.io/sm-test.moimstone.com-tls-d49gr   True    24h

NAMESPACE        NAME                                                                    STATE   AGE
smartwork-test   order.acme.cert-manager.io/sm-test.moimstone.com-tls-d49gr-1434215337   valid   24h

*****************
certificaterequest 가 true 값이 나오기 위해서는 대략 15분 가량이 소모됩니다.
위의 보여지는 값은 이미 인증 받은후 나오는 값입니다.
```
