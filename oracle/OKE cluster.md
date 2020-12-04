### kubernetes install

OKE 구성후 생성된 클러스터에 ' 클러스터 엑세스 ' 버튼을 누른 후 로컬접속을 선택하면 나오는 내용 입니다. 

```
oracle cloud 에서는 kubernetes를 설치 하기 위해 oci 명령어를 사용
#oci -v
```
```
kube 디렉터리 생성
#mkdir -p $HOME/.kube
```
```
oic 를 이용 하여 config 파일 install
oci ce cluster create-kubeconfig --cluster-id ocid1.cluster.oc1.ap-seoul-1.aaaaaaaaae.......................qt --file $HOME/.kube/config --region ap-seoul-1 --token-version 2.0.0 

클러스터 구축과 생성자의 따라 값이 바뀜니다. 여기까지 의 차례는 본인이 콘솔에 직접 들어가 확인 해주세요
```

위의 명령어를 사용하면 권한에 대한 문제와 함께 아래에 명령어가 제시됩니다.
순차적으로 입력후 다시 위의 명령어를 입력 해야 합니다.
```
 oci setup repair-file-permissions --file /home/opc/.oci/config
 export OCI_CLI_SUPPRESS_FILE_PERMISSIONS_WARNING=True
```
```
export KUBECONFIG=$HOME/.kube/config
```
kube config 를 export 시킵니다.


