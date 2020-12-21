

## Cert_manager install

### Cert-manager install

- helm 을 사용하여  install 진행
```
# Helm v3+
$ helm install \
cert-manager jetstack/cert-manager \
--namespace cert-manager \
--version v1.1.0 \

# --set installCRDs=true
# Helm v2
$ helm install \
--name cert-manager \
--namespace cert-manager \
--version v1.1.0 \
jetstack/cert-manager \
# --set installCRDs=true
```

### Verifying the installation

```
[opc@bastion-instance ~]$  kubectl get pods --namespace cert-manager

NAME 						 READY 		       STATUS   RESTARTS   AGE
cert-manager-5fd9d77768-c924k  1/1 			   Running     0 	  2d15h
cert-manager-cainjector-78cbd59555-b828k  1/1  Running     0  	  2d15h
cert-manager-webhook-756d477cc4-9csql  1/1     Running     0  	  2d15h
```

### Webhook
- Using public helm chart
```
# helm repo add cert-manager-webhook-oci https://dn13.gitlab.io/cert-manager-webhook-oci

# helm install --namespace cert-manager cert-manager-webhook-oci cert-manager-webhook-oci/cert-manager-webhook-oci
```
Cert_manager install- From local checkout
```
# helm install --namespace cert-manager cert-manager-webhook-oci deploy/cert-manager-webhook-oci
```

###  OCI Profile Secret 생성
```
apiVersion: v1
kind: Secret
metadata:
 name: oci-profile
type: Opaque
stringData:
 tenancy: "ocid1.tenancy.oc1..aaaaaaaaeq..........................."
 user: "ocid1.user.oc1....................................................a"
 region: "ap-seoul-1"
 fingerprint: "a0:ba:ad:84:73:59:f9:76:fb:77:54:3d:21:75:7c:47"
 privateKey: |
	 -----BEGIN RSA PRIVATE KEY-----
	MIIEpAIBAAKCAQEArXTxcB5fEYfs1u9qYwefUDyX5YKsqasHR087RrVA6I/hp9ra
	 ……………………… 이하 생략 ……………………………………………
	 2OWnETwe81LEhwmJ91r7eYCxqwMHMw09BkNmYRR0X3jUobMXPpA+Xw==
	 -----END RSA PRIVATE KEY-----
 privateKeyPassphrase: ""
```

### Webhook Challenges 방식으로 ClusterIssuer 생성
```
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
	name: letsencrypt-prod
spec:
	acme:
	  server: https://acme-staging-v02.api.letsencrypt.org/directory
	  email: example@naver.com
	  privateKeySecretRef:
	   name: letsencrypt-prod
	  solvers:
	  - dns01:
	      webhook:
	        groupName: acme.d-n.be
	          solverName: oci
	            config:
	  	  	      ociProfileSecretName: oci-profile
				    compartmentOCID: 
ocid1.tenancy.oc1..,,,,............................................q
```
### Set up DNS zones on oci

```
Oracle cloud console에서
Core infrastructure -> networking -> DNS management 들어가서
Manage DNS Service -> Zones 클릭
Public Zones -> Create Zone 클릭
moimstone.com zone 생성
sm-dev.moimstone.com recode 생성 (CNAME -> ingress 용 LB 이름 입력 f1813689-0bc5-48fa-972e-ef1373ab37fe)
```

### 인증서 생성

- 인증서 생성 오더 실패 대신 domain Solving Challenges 성공
```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
	name: example-cert
	namespace: smartwork-dev
spec:
	commonName: sm-dev.moimstone.com
	dnsNames:
	 - sm-dev.moimstone.com
issuerRef:
	kind: ClusterIssuer
	name: letsencrypt-prod
	secretName: example-cert
```






