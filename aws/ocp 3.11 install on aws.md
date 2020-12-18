## ocp 3.11 install on aws

### 1 AWS 사전 설정
#### 1-1 Security Group 설정
|||
|--|--|
|All OpenShift Container Platform Hosts| * tcp/22 from host running the installer/Ansible|
|etcd Security Group| * tcp/2379 from masters<br> * tcp/2380 from etcd hosts|
|Master Security Group| * tcp/8443 from 0.0.0.0/0<br> * tcp/53 from all OpenShift Container Platform hosts for environments installed prior to or upgraded to 3.2<br> * udp/53 from all OpenShift Container Platform hosts for environments installed prior to or upgraded to 3.2<br> * tcp/8053 from all OpenShift Container Platform hosts for new environments installed with 3.2<br> * udp/8053 from all OpenShift Container Platform hosts for new environments installed with 3.2|
|Node Security Group| * tcp/10250 from masters<br> * udp/4789 from nodes|
|Infrastructure Nodes (ones that can host the OpenShift Container Platform router)| * tcp/443 from 0.0.0.0/0<br> * tcp/80 from 0.0.0.0/0|
|CRI-O|If using CRIO, you must open tcp/10010 to allow `oc exec` and `oc rsh` operations.|

#### 1-2 EC2 Instance 생성
|node|ec2 type|OS|EIP|
|--|--|--|--|
|master,bastion|t3a.xlarge|RHEL-7.7|O|
|CPU Node|t3a.large|RHEL-7.7|X|
|GPU Node|p2.xlarge|RHEL-7.7|X|

#### 1-3 DNS 설정

|Record name|Type|Value/Route traffic to|description|
|-|-|-|-|
|master.ocp311.icsfuturegen.de|A|15.164.159.55(EIP)|master api|
|*.apps.icsfuturegen.de|A|15.164.159.55(EIP)|console|

### 2 OCP3.11 설치
#### 2-1 Subscription 등록(전체 서버)
```
subscription-manager register --username=qingsong1989 --password=Azwell12#$
subscription-manager refresh
subscription-manager attach --pool=8a85f9997385090b0173b342862a50b8
subscription-manager repos --enable="rhel-7-server-rpms"  --enable="rhel-7-server-extras-rpms" --enable="rhel-7-server-ose-3.11-rpms" --enable="rhel-7-server-ansible-2.9-rpms"
```
#### 2-2 Ensuring host access (master -> worker)
- ssh key 생성
```
#login to master node
ssh-keygen
```
- Distribute the key to all worker nodes
```
#마스트 노드에서 /root/.ssh/id_rsa.pub 키 복사
cat /root/.ssh/id_rsa.pub

#모든 노드에서 키 등록 (master, cpu, gpu)
마스트 노드의 public key 복사한 내용을 각 노드에 추가
vi authorized_keys
...
ctrl + v
...
```
- ssh login test
```
[root@ip-10-0-0-130 .ssh]# ssh ip-10-0-0-72
The authenticity of host 'ip-10-0-0-72 (10.0.0.72)' can't be established.
ECDSA key fingerprint is SHA256:XPv09d7jkyE9JYRD2B+3gH2n2Eg2MDYz+XIuToProS4.
ECDSA key fingerprint is MD5:bb:2b:94:18:c3:7a:fd:3a:cf:1c:97:7b:04:36:87:81.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ip-10-0-0-72,10.0.0.72' (ECDSA) to the list of known hosts.
Last login: Wed Dec 16 09:38:38 2020
[root@ip-10-0-0-72 ~]#
```

#### 2-3 Installing base packages (on master)
- Install the following base packages:
```
yum install wget git net-tools bind-utils yum-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct
```
- Update the system to the latest packages
```
yum update
```
- docker and openshift-ansible install
```
yum -y install openshift-ansible
yum -y install docker
```
#### 2-4 set selinux (all nodes)

```
/etc/sysconfig/selinux
...
SELINUX=enforcing
...
```

#### 2-5 deploy OCP 3.11 cluster (on master)
- Configuring Your Inventory File
```
vi /usr/share/ansible/openshift-ansible/hosts
[OSEv3:vars]
###########################################################################
### Ansible Vars
###########################################################################
ansible_ssh_user=root
ansible_become=true
debug_level=2


###########################################################################
### OpenShift Basic Vars
###########################################################################
openshift_deployment_type=openshift-enterprise
openshift_release=v3.11.188
openshift_image_tag=v3.11.188
openshift_pkg_version=-3.11.188

# Configuring Certificate Validity
openshift_hosted_registry_cert_expire_days=3650
openshift_ca_cert_expire_days=3650
openshift_node_cert_expire_days=3650
openshift_master_cert_expire_days=3650
etcd_ca_default_days=3650
openshift_certificate_expiry_warning_days=2920
openshift_certificate_expiry_fail_on_warn=2920

# Configuring Cluster Pre-install Checks
openshift_disable_check=memory_availability,disk_availability,docker_storage,docker_storage_driver,docker_image_availability,package_version,package_availability,package_update

# Node Groups
openshift_node_groups=[{'name': 'node-config-master', 'labels': ['node-role.kubernetes.io/master=true','node-role.kubernetes.io/infra=true']},  {'name': 'node-config-compute', 'labels': ['node-role.kubernetes.io/compute=true']}]
#osm_default_node_selector={"node-role.kubernetes.io/compute": "true"}

# Configure logrotate scripts
logrotate_scripts=[{"name": "syslog", "path": "/var/log/cron\n/var/log/maillog\n/var/log/messages\n/var/log/secure\n/var/log/spooler\n", "options": ["daily", "rotate 7","size 500M", "compress", "sharedscripts", "missingok"], "scripts": {"postrotate": "/bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true"}}]

oreg_auth_user=qingsong1989
oreg_auth_password=Azwell12#$

###########################################################################
### OpenShift Master Vars
###########################################################################
openshift_master_api_port=8443
openshift_master_console_port=8443
openshift_master_cluster_hostname=master.ocp311.icsfuturegen.de
openshift_master_cluster_public_hostname=master.ocp311.icsfuturegen.de
#openshift_master_named_certificates=[{"certfile": "/home/ebaycloud/named_certificates/STAR_ebaykorea_com.crt", "keyfile": "/home/ebaycloud/named_certificates/STAR_ebaykorea_com.key", "names": ["fusione3.ebaykorea.com"], "cafile": "/home/ebaycloud/named_certificates/STAR_ebaykorea_com.ca-bundle"}]
openshift_master_default_subdomain=apps.icsfuturegen.de
#openshift_master_overwrite_named_certificates=true
openshift_master_cluster_method=native

# Audit log
#openshift_master_audit_config={"enabled": true, "auditFilePath": "/var/log/openpaas-oscp-audit/openpaas-oscp-audit.log", "maximumFileRetentionDays": 30, "maximumFileSizeMegabytes": 500, "maximumRetainedFiles": 100}


###########################################################################
### OpenShift Network Vars
###########################################################################
openshift_portal_net=172.29.0.0/16
#osm_cluster_network_cidr=10.128.0.0/14
os_sdn_network_plugin_name='redhat/openshift-ovs-subnet'
#os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant'


###########################################################################
### OpenShift Authentication Vars
###########################################################################
# LDAP AND HTPASSWD Authentication (download ipa-ca.crt first)
#openshift_master_identity_providers=[{'name': 'ldap', 'challenge': 'true', 'login': 'true', 'kind': 'LDAPPasswordIdentityProvider','attributes': {'id': ['dn'], 'email': ['mail'], 'name': ['cn'], 'preferredUsername': ['uid']}, 'bindDN': 'uid=admin,cn=users,cn=accounts,dc=shared,dc=example,dc=opentlc,dc=com', 'bindPassword': 'r3dh4t1!', 'ca': '/etc/origin/master/ipa-ca.crt','insecure': 'false', 'url': 'ldaps://ipa.shared.example.opentlc.com:636/cn=users,cn=accounts,dc=shared,dc=example,dc=opentlc,dc=com?uid?sub?(memberOf=cn=ocp-users,cn=groups,cn=accounts,dc=shared,dc=example,dc=opentlc,dc=com)'},{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]

# Just LDAP
#openshift_master_identity_providers=[{'name': 'ldap', 'challenge': 'true', 'login': 'true', 'kind': 'LDAPPasswordIdentityProvider','attributes': {'id': ['dn'], 'email': ['mail'], 'name': ['cn'], 'preferredUsername': ['uid']}, 'bindDN': 'uid=admin,cn=users,cn=accounts,dc=shared,dc=example,dc=opentlc,dc=com', 'bindPassword': 'r3dh4t1!', 'ca': '/etc/origin/master/ipa-ca.crt','insecure': 'false', 'url': 'ldaps://ipa.shared.example.opentlc.com:636/cn=users,cn=accounts,dc=shared,dc=example,dc=opentlc,dc=com?uid?sub?(memberOf=cn=ocp-users,cn=groups,cn=accounts,dc=shared,dc=example,dc=opentlc,dc=com)'}]

# Just HTPASSWD
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]

# LDAP and HTPASSWD dependencies
#Command => htpasswd -c /root/htpasswd.openshift ocpadmin
#openshift_master_htpasswd_file=/home/ebaycloud/htpasswd.openshift
#openshift_master_ldap_ca_file=/root/ipa-ca.crt


###########################################################################
### OpenShift Router and Registry Vars
###########################################################################
# Router
openshift_router_selector='node-role.kubernetes.io/infra=true'
openshift_hosted_router_replicas=1
#openshift_hosted_router_certificate=[{"certfile": "/home/ebaycloud/STAR.e3.ebaykorea.com/STAR_e3_ebaykorea_com.crt", "keyfile": "/home/ebaycloud/STAR.e3.ebaykorea.com/STAR_e3_ebaykorea_com.key", "cafile": "/home/ebaycloud/STAR.e3.ebaykorea.com/STAR_e3_ebaykorea_com.ca-bundle"}]

# Registry
openshift_registry_selector='node-role.kubernetes.io/infra=true'
openshift_hosted_registry_replicas=1
openshift_hosted_registry_pullthrough=true
openshift_hosted_registry_acceptschema2=true
openshift_hosted_registry_enforcequota=true


###########################################################################
### OpenShift Service Catalog Vars
###########################################################################
#openshift_enable_service_catalog=true
#template_service_broker_install=true
#openshift_template_service_broker_namespaces=['openshift']
#ansible_service_broker_install=true
#ansible_service_broker_local_registry_whitelist=['.*-apb$']


openshift_enable_service_catalog=false
template_service_broker_install=false
ansible_service_broker_install=false

template_service_broker_remove=true
ansible_service_broker_install=true
openshift_service_catalog_remove=true


#########################
# Prometheus Metrics
#########################
openshift_cluster_monitoring_operator_install=false
openshift_cluster_monitoring_operator_node_selector={"node-role.kubernetes.io/monitoring": "true"}

openshift_cluster_monitoring_operator_prometheus_storage_enabled=true
openshift_cluster_monitoring_operator_prometheus_storage_capacity=10Gi

openshift_cluster_monitoring_operator_alertmanager_storage_enabled=true
openshift_cluster_monitoring_operator_alertmanager_storage_capacity=20Gi


########################
# Cluster Metrics
########################
openshift_metrics_install_metrics=false
openshift_metrics_cassandra_storage_type=pv
openshift_metrics_cassandra_pvc_size=10Gi
openshift_metrics_hawkular_nodeselector={"node-role.kubernetes.io/monitoring": "true"}
openshift_metrics_cassandra_nodeselector={"node-role.kubernetes.io/monitoring": "true"}
openshift_metrics_heapster_nodeselector={"node-role.kubernetes.io/monitoring": "true"}



###########################################################################
### OpenShift Hosts
###########################################################################
[OSEv3:children]
masters
etcd
nodes

[masters]
ip-10-0-0-130.ap-northeast-2.compute.internal
[etcd]
ip-10-0-0-130.ap-northeast-2.compute.internal
[nodes]
ip-10-0-0-130.ap-northeast-2.compute.internal openshift_node_group_name='node-config-master'
ip-10-0-0-72.ap-northeast-2.compute.internal openshift_node_group_name='node-config-compute'
```

- ansible test
```
[root@ip-10-0-0-130 openshift-ansible]# ansible -i hosts nodes -m  ping
ip-10-0-0-130.ap-northeast-2.compute.internal | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
ip-10-0-0-72.ap-northeast-2.compute.internal | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```
- 사전점검
```
cd /usr/share/ansible/openshift-ansible
ansible-playbook -i hosts playbooks/prerequisites.yml
```
- 배포 
```
cd /usr/share/ansible/openshift-ansible
ansible-playbook -i hosts playbooks/deploy_cluster.yml
```

### 3 OCP3.11 ADD CPU Nodes
#### 3-1 Subscription 등록(전체 서버)
```
subscription-manager register --username=qingsong1989 --password=Azwell12#$
subscription-manager refresh
subscription-manager attach --pool=8a85f9997385090b0173b342862a50b8
subscription-manager repos --enable="rhel-7-server-rpms"  --enable="rhel-7-server-extras-rpms" --enable="rhel-7-server-ose-3.11-rpms" --enable="rhel-7-server-ansible-2.9-rpms"
```
#### 3-2 Ensuring host access (master -> worker)
- ssh key 생성
```
#login to master node
ssh-keygen
```
- Distribute the key to all worker nodes
```
#마스트 노드에서 /root/.ssh/id_rsa.pub 키 복사
cat /root/.ssh/id_rsa.pub

#모든 노드에서 키 등록 (master, cpu, gpu)
마스트 노드의 public key 복사한 내용을 각 노드에 추가
vi authorized_keys
...
ctrl + v
...
```
- ssh login test
```
[root@ip-10-0-0-130 .ssh]# ssh ip-10-0-0-72
The authenticity of host 'ip-10-0-0-72 (10.0.0.72)' can't be established.
ECDSA key fingerprint is SHA256:XPv09d7jkyE9JYRD2B+3gH2n2Eg2MDYz+XIuToProS4.
ECDSA key fingerprint is MD5:bb:2b:94:18:c3:7a:fd:3a:cf:1c:97:7b:04:36:87:81.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ip-10-0-0-72,10.0.0.72' (ECDSA) to the list of known hosts.
Last login: Wed Dec 16 09:38:38 2020
[root@ip-10-0-0-72 ~]#
```
#### 3-3 set selinux (all nodes)
```
/etc/sysconfig/selinux
...
SELINUX=enforcing
...
```
#### 3-4 Ansible Playbooks 수행 하여 노드 추가(on master node)
- Edit your **_/etc/ansible/hosts_**
```
vi hosts
###########################################################################
### OpenShift Hosts
###########################################################################
[OSEv3:children]
masters
etcd
nodes
new_nodes # <- add

[masters]
ip-10-0-0-130.ap-northeast-2.compute.internal
[etcd]
ip-10-0-0-130.ap-northeast-2.compute.internal
[nodes]
ip-10-0-0-130.ap-northeast-2.compute.internal openshift_node_group_name='node-config-master'
ip-10-0-0-72.ap-northeast-2.compute.internal openshift_node_group_name='node-config-compute'

[new_nodes] # <- add
ip-10-0-0-183.ap-northeast-2.compute.internal openshift_node_group_name='node-config-compute'
```

- Playbooks 수행
```
ansible-playbook -i hosts playbooks/openshift-node/scaleup.yml
```

### 4 OCP3.11 ADD GPU Nodes
#### 4-1 Subscription 등록(전체 서버)
```
subscription-manager register --username=qingsong1989 --password=Azwell12#$
subscription-manager refresh
subscription-manager attach --pool=8a85f9997385090b0173b342862a50b8
subscription-manager repos --enable="rhel-7-server-rpms"  --enable="rhel-7-server-extras-rpms" --enable="rhel-7-server-ose-3.11-rpms" --enable="rhel-7-server-ansible-2.9-rpms"
yum install pciutils wget
```
#### 4-2 Ensuring host access (master -> worker)
- ssh key 생성
```
#login to master node
ssh-keygen
```
- Distribute the key to all worker nodes
```
#마스트 노드에서 /root/.ssh/id_rsa.pub 키 복사
cat /root/.ssh/id_rsa.pub

#모든 노드에서 키 등록 (master, cpu, gpu)
마스트 노드의 public key 복사한 내용을 각 노드에 추가
vi authorized_keys
...
ctrl + v
...
```
- ssh login test
```
[root@ip-10-0-0-130 .ssh]# ssh ip-10-0-0-72
The authenticity of host 'ip-10-0-0-72 (10.0.0.72)' can't be established.
ECDSA key fingerprint is SHA256:XPv09d7jkyE9JYRD2B+3gH2n2Eg2MDYz+XIuToProS4.
ECDSA key fingerprint is MD5:bb:2b:94:18:c3:7a:fd:3a:cf:1c:97:7b:04:36:87:81.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ip-10-0-0-72,10.0.0.72' (ECDSA) to the list of known hosts.
Last login: Wed Dec 16 09:38:38 2020
[root@ip-10-0-0-72 ~]#
```
#### 4-3 set selinux (all nodes)
```
/etc/sysconfig/selinux
...
SELINUX=enforcing
...
```
#### 4-4 nvidia drivers 설치 (기존 버전 동일 하게 설정)

- 참고 문서 https://docs.nvidia.com/datacenter/tesla/tesla-installation-notes/index.html#rhel7
- enable repos for RHEL 7
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
subscription-manager repos --enable="rhel-*-optional-rpms" --enable="rhel-*-extras-rpms"  --enable="rhel-ha-for-rhel-*-server-rpms"
```
- Setup the CUDA network repository
```
distribution=$(. /etc/os-release;echo $ID`rpm -E "%{?rhel}%{?fedora}"`)
ARCH=$( /bin/arch )
yum-config-manager --add-repo http://developer.download.nvidia.com/compute/cuda/repos/$distribution/${ARCH}/cuda-$distribution.repo
```
- install kernel-devel package
```
sudo yum install -y kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```
- Update the repository cache and install the driver
```
yum clean expire-cache
yum install -y nvidia-driver-latest-dkms # 버전에 맞게 설정
modprobe -r nouveau
nvidia-modprobe && nvidia-modprobe -u
reboot
```
- 확인
```
[root@ip-10-0-0-183 ec2-user]# nvidia-smi --query-gpu=gpu_name --format=csv,noheader --id=0 | sed -e 's/ /-/g'
Tesla-K80
```

#### 4-5 Ansible Playbooks 수행 하여 노드 추가(on master node)
- Edit your **_/etc/ansible/hosts_**
```
vi hosts
###########################################################################
### OpenShift Hosts
###########################################################################
[OSEv3:children]
masters
etcd
nodes
new_nodes # <- add

[masters]
ip-10-0-0-130.ap-northeast-2.compute.internal
[etcd]
ip-10-0-0-130.ap-northeast-2.compute.internal
[nodes]
ip-10-0-0-130.ap-northeast-2.compute.internal openshift_node_group_name='node-config-master'
ip-10-0-0-72.ap-northeast-2.compute.internal openshift_node_group_name='node-config-compute'

[new_nodes] # <- add
ip-10-0-0-183.ap-northeast-2.compute.internal openshift_node_group_name='node-config-compute'
```

- Playbooks 수행
```
ansible-playbook -i hosts playbooks/openshift-node/scaleup.yml
```

#### 4-6 Adding the nvidia-container-runtime-hook
- 참고 문서

repos 등록
https://nvidia.github.io/nvidia-container-runtime
container-runtime-hook 설치
https://github.com/NVIDIA/nvidia-container-runtime

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.repo | tee /etc/yum.repos.d/nvidia-container-runtime.repo


yum install nvidia-container-runtime

#oci-nvidia-hook.json 생성
[root@ip-10-0-0-183 ec2-user]# cat /usr/share/containers/oci/hooks.d/oci-nvidia-hook.json
{
    "hasbindmounts": true,
    "hook": "/usr/bin/nvidia-container-runtime-hook",
    "stage": [ "prestart" ]
}

```

#### 4-7 Adding the SELinux policy module
- First install the SELinux policy module on all GPU worker nodes
```
wget https://raw.githubusercontent.com/zvonkok/origin-ci-gpu/master/selinux/nvidia-container.pp
semodule -i nvidia-container.pp
```
- Check and restore the labels
```
nvidia-container-cli -k list | restorecon -v -f -
restorecon -Rv /dev
restorecon -Rv /var/lib/kubelet
```

#### 4-8 Verify SELinux and prestart hook functionality
```
[root@ip-10-0-0-183 ec2-user]# docker run --user 1000:1000 --security-opt=no-new-privileges --cap-drop=ALL --security-opt label=type:nvidia_container_t docker.io/mirrorgooglecontainers/cuda-vector-add:v0.1
[Vector addition of 50000 elements]
Copy input data from the host memory to the CUDA device
CUDA kernel launch with 196 blocks of 256 threads
Copy output data from the CUDA device to the host memory
Test PASSED
Done
```

#### 4.9 Deploy the NVIDIA Device Plugin Daemonset
- label the node
```
oc label node ip-10-0-0-183.ap-northeast-2.compute.internal openshift.com/gpu-accelerator=true
```
- Deploy Daemonset
```
git clone https://github.com/redhat-performance/openshift-psap.git
cd openshift-psap/blog/gpu/device-plugin/
oc create -f nvidia-device-plugin.yml
```
#### 4.10 확인
- Plugin pod 상태 확인
```
#gpu node에서만 Plugin pod 실행 됬는지 확인

[root@ip-10-0-0-130 openshift-ansible]# kubectl get pods -o wide  --selector=name=nvidia-device-plugin-ds --all-namespaces
NAMESPACE     NAME                                   READY     STATUS    RESTARTS   AGE       IP           NODE                                            NOMINATED NODE
kube-system   nvidia-device-plugin-daemonset-gpgg7   1/1       Running   0          43m       10.130.0.2   ip-10-0-0-183.ap-northeast-2.compute.internal   <none>
``` 

- Plugin pod  log 확인
```
[root@ip-10-0-0-130 openshift-ansible]# oc logs nvidia-device-plugin-daemonset-gpgg7 -n kube-system
2020/12/18 03:43:27 Loading NVML
2020/12/18 03:43:27 Fetching devices.
2020/12/18 03:43:27 Starting FS watcher.
2020/12/18 03:43:27 Starting OS watcher.
2020/12/18 03:43:27 Starting to serve on /var/lib/kubelet/device-plugins/nvidia.sock
2020/12/18 03:43:27 Registered device plugin with Kubelet
```
- node capacity 확인
```
[root@ip-10-0-0-130 openshift-ansible]# oc describe node ip-10-0-0-183.ap-northeast-2.compute.internal | egrep 'Capacity:|Allocatable:|gpu'
                    openshift.com/gpu-accelerator=true
Capacity:
 nvidia.com/gpu:  1
Allocatable:
 nvidia.com/gpu:  1
  nvidia.com/gpu  0           0
```
- Deploy a pod that requires a GPU
```
cd openshift-psap/blog/gpu/device-plugin/
oc new-project nvidia
oc create -f cuda-vector-add.yaml

[root@ip-10-0-0-130 openshift-ansible]# oc get pod -n nvidia
NAME              READY     STATUS      RESTARTS   AGE
cuda-vector-add   0/1       Completed   0          1h

[root@ip-10-0-0-130 openshift-ansible]# oc logs cuda-vector-add  -n nvidia
[Vector addition of 50000 elements]
Copy input data from the host memory to the CUDA device
CUDA kernel launch with 196 blocks of 256 threads
Copy output data from the CUDA device to the host memory
Test PASSED
Done
```
