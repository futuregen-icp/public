

## kubernetes drain
----
### 1.  node check
```
# kubectl get nodes
AME            STATUS     ROLES    AGE      VERSION
10.1.20.35     Ready      node     4d3h     v1.18.10
10.1.20.45     Ready      node     4d3h     v1.18.10
10.1.40.56     Ready      node     4d3h     v1.18.10
10.1.40.76     Ready      node     4d3h     v1.18.10
```
### 2.  SchedulingDisabled the nodes
```
# kubectl cordon 10.1.20.35
# kubectl cordon 10.1.40.56
# kubectl cordon 10.1.40.76
```
target : 10.1.20.35    drain: 10.1.20.45


### 3.  node check
```
# kubectl get nodes
AME            STATUS                        ROLES    AGE      VERSION
10.1.20.35     Ready,SchedulingDisabled      node     4d3h     v1.18.10
10.1.20.45     Ready                         node     4d3h     v1.18.10
10.1.40.56     Ready.SchedulingDisabled      node     4d3h     v1.18.10
10.1.40.76     Ready.SchedulingDisabled      node     4d3h     v1.18.10
```

### 4. node drain
```
# kubectl drain 10.1.20.35
*** SchedulingDisabled 되어 있지 않은 node 로 drain 됨 ***
```
### 5.  repeat

target 과 drain 할 노드를 돌아가며 반복
 


