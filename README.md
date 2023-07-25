## Ref
[link](https://1week.tistory.com/101)

## Easy Access

```
git clone https://github.com/clucle/mongodb-kubernetes-operator.git
cd mongodb-kubernetes-operator

kubectl apply -f config/crd/bases/mongodbcommunity.mongodb.com_mongodbcommunity.yaml
# clucle@APSEO-D-1LS87X3:/mnt/d/repo/mongodb-kubernetes-operator$ kubectl apply -f config/crd/bases/mongodbcommunity.mongodb.com_mongodbcommunity.yaml
# customresourcedefinition.apiextensions.k8s.io/mongodbcommunity.mongodbcommunity.mongodb.com configured

kubectl get crd/mongodbcommunity.mongodbcommunity.mongodb.com
# clucle@APSEO-D-1LS87X3:/mnt/d/repo/mongodb-kubernetes-operator$ kubectl get crd/mongodbcommunity.mongodbcommunity.mongodb.com
# NAME                                            CREATED AT
# mongodbcommunity.mongodbcommunity.mongodb.com   2023-07-25T06:49:19Z

kubectl create ns mongodb-operator
# clucle@APSEO-D-1LS87X3:/mnt/d/repo/mongodb-kubernetes-operator$ kubectl create ns mongodb-operator
# namespace/mongodb-operator created

kubectl apply -k config/rbac/ -n mongodb-operator
# clucle@APSEO-D-1LS87X3:/mnt/d/repo/mongodb-kubernetes-operator$ kubectl apply -k config/rbac/ -n mongodb-operator
# serviceaccount/mongodb-database created
# serviceaccount/mongodb-kubernetes-operator created
# role.rbac.authorization.k8s.io/mongodb-database created
# role.rbac.authorization.k8s.io/mongodb-kubernetes-operator created
# rolebinding.rbac.authorization.k8s.io/mongodb-database created
# rolebinding.rbac.authorization.k8s.io/mongodb-kubernetes-operator created

kubectl apply -f deploy/clusterwide
# clucle@APSEO-D-1LS87X3:/mnt/d/repo/mongodb-kubernetes-operator$ kubectl apply -f deploy/clusterwide
# clusterrole.rbac.authorization.k8s.io/mongodb-kubernetes-operator created
# clusterrolebinding.rbac.authorization.k8s.io/mongodb-kubernetes-operator created
# clusterrole.rbac.authorization.k8s.io/read-access-for-service-binding created

kubectl apply -f config/manager/manager.yaml -n mongodb-operator
# clucle@APSEO-D-1LS87X3:/mnt/d/repo/mongodb-kubernetes-operator$ kubectl apply -f config/manager/manager.yaml -n mongodb-operator
# deployment.apps/mongodb-kubernetes-operator created

kubectl create namespace mongodb-test
# clucle@APSEO-D-1LS87X3:/mnt/d/repo/mongodb-kubernetes-operator$ kubectl create namespace mongodb-test
# namespace/mongodb-test created

kubectl apply -k config/rbac -n mongodb-test
# clucle@APSEO-D-1LS87X3:/mnt/d/repo/mongodb-kubernetes-operator$ kubectl apply -k config/rbac -n mongodb-test
# serviceaccount/mongodb-database created
# serviceaccount/mongodb-kubernetes-operator created
# role.rbac.authorization.k8s.io/mongodb-database created
# role.rbac.authorization.k8s.io/mongodb-kubernetes-operator created
# rolebinding.rbac.authorization.k8s.io/mongodb-database created    
# rolebinding.rbac.authorization.k8s.io/mongodb-kubernetes-operator created

kubectl apply -f sample/mongodb.yaml -n mongodb-test
# clucle@APSEO-D-1LS87X3:/mnt/d/repo/mongodb-kubernetes-operator$ kubectl apply -f sample/mongodb.yaml -n mongodb-test
# mongodbcommunity.mongodbcommunity.mongodb.com/example-mongodb created
# secret/user1-password created
```

Pending 상태가 오래 지속되다가 Running 으로 변경됨
상태 확인 (pod 3개가 정상적으로 Running)
```
k get all -n mongodb-test
# clucle@APSEO-D-1LS87X3:/mnt/d/repo/mongodb-kubernetes-operator$ k get all -n mongodb-test
# NAME                    READY   STATUS    RESTARTS   AGE
# pod/example-mongodb-0   2/2     Running   0          28m
# pod/example-mongodb-1   2/2     Running   0          27m
# pod/example-mongodb-2   2/2     Running   0          26m
# 
# NAME                          TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)     AGE
# service/example-mongodb-svc   ClusterIP   None         <none>        27017/TCP   28m
# 
# NAME                                   READY   AGE
# statefulset.apps/example-mongodb-arb   0/0     28m
# statefulset.apps/example-mongodb       3/3     28m
```


접속 테스트
```
k exec -n mongodb-test example-mongodb-0 -it bash
mongosh "mongodb+srv://user1:somepassword@example-mongodb-svc.mongodb-test.svc.cluster.local/admin?ssl=false"
```

외부 포트 열기
```
kubectl port-forward -n mongodb-test svc/example-mongodb-svc 27017:27017
```

name 으로 ip 찾기 위해 host 에 등록
linux : /etc/hosts
window : C:\Windows\System32\drivers\etc\hosts
```
127.0.0.1   example-mongodb-0.example-mongodb-svc.mongodb-test.svc.cluster.local
127.0.0.1   example-mongodb-1.example-mongodb-svc.mongodb-test.svc.cluster.local
127.0.0.1   example-mongodb-2.example-mongodb-svc.mongodb-test.svc.cluster.local
```

테스트 계정
- name : user1
- password : somepassword

권한 수정이 필요하다면 sample/mongodb.yaml 의 role 을 설정하고 다시 실행합니다.
기본적 상태에서는 admin, test db 에만 권한을 부여했습니다

```
vi sample/mogodb.yaml # 수정..
kubectl delete -f sample/mongodb.yaml -n mongodb-test
kubectl apply -f sample/mongodb.yaml -n mongodb-test
```