# Q1: Backup and Restore Etcd

`All question is used to kubernetes-admin@kubernetes context on cluster, please don't forget it before you are to do test`

`You have to do this test first, if not, you might get a zero at score`

First, create a snapshot of the existing etcd instance running at https://127.0.0.1:2379, saving the snapshot to /srv/etcd-snapshot.db.
Next, restore an existing, previous snapshot localted at /srv/etcd_exam_backup.db

The following TLS certificates/key are supplied for connecting to the server with etcdctl:

CA certificate: /etc/kubernetes/pki/etcd/ca.crt

Client certificate: /etc/kubernetes/pki/etcd/server.crt

Client key: /etc/kubernetes/pki/etcd/server.key

---

Q1 中文题目：

`所有题都需要切换context 集群，每个题都有切换的命令，做题之前一定要复制它的命令并执行，不然你可能会直接得0分`

`你必须首先做这个题，才能做其他的，不然你可能会直接得0分`

针对etcd实例 https://127.0.0.1:2379 创建一个快照，保存到/srv/etcd-snapshot.db，然后恢复一个已经存在的快照：/srv/etcd_exam_backup.db

执行etcdctl命令的证书存放在：

ca证书：/etc/kubernetes/pki/etcd/ca.crt

客户端证书：/etc/kubernetes/pki/etcd/server.crt

客户端密钥：/etc/kubernetes/pki/etcd/server.key

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
考试的时候可能需要安装etcd-client，具体以到时候为准，这里记得安装命令

```bash
root@cka-master:~# apt install etcd-client -y
Reading package lists... Done
Building dependency tree
Reading state information... Done
```
备份数据
```bash
ETCDCTL_API=3 etcdctl \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /srv/etcd-snapshot.db
```

在恢复数据之前，需要停止使用和写入新数据
```bash
root@cka-master:~# mv /etc/kubernetes/manifests /etc/kubernetes/manifests.bak

root@cka-master:~# mv /var/lib/etcd /var/lib/etcd.bak
```
完成数据恢复
```bash
root@cka-master:~# ETCDCTL_API=3 etcdctl \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
--data-dir /var/lib/etcd \
snapshot restore /srv/etcd_exam_backup.db
```

恢复服务正常运行
```bash
mv /etc/kubernetes/manifests.bak /etc/kubernetes/manifests

systemctl restart kubelet.service

root@cka-master:~# kubectl get nodes
NAME          STATUS                        ROLES           AGE   VERSION
cka-master    Ready                         control-plane   11h   v1.27.2
cka-worker1   NotReady,SchedulingDisabled   worker          11h   v1.27.2
cka-worker2   Ready                         worker          11h   v1.27.2
```
# Q2: RBAC

You have been asked to create a new ClusterRole for a deployment pipeline and bind it to a specific ServiceAccount scoped to a specific namespace.
Task
Create a new ClusterRole named deployment-clusterrole, which only allows to create the following resource types:

1. Deployment

2. StatefulSet

3. DaemonSet

Create a new ServiceAccount named cicd-token in the existing namespace app-team1

Bind the new ClusterRole to the new ServiceAccount cicd-token, limited to the namespace app-team1

---

Q2中文题目：

创建一个新的 ClusterRole 并将其绑定到特定 namespace 的特定 ServiceAccount

创建一个名为deployment-clusterrole的clusterrole，该clusterrole只允许创建以下资源

1. Deployment

2. Daemonset

3. Statefulset

在名字为app-team1的namespace下创建一个名为cicd-token的serviceAccount，并且将上一步创建clusterrole的权限绑定到该serviceAccount

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
创建新的clusterrole，并使其具有相应的权限
```bash
root@cka-master:~# kubectl create clusterrole deployment-clusterrole \
--verb=create \
--resource=deployments,statefulsets,daemonsets
```
创建服务账号
```bash
root@cka-master:~# kubectl -n app-team1 create serviceaccount cicd-token
```
将服务账号和集群角色绑定
```bash
root@cka-master:~# kubectl -n app-team1 create rolebinding bind-cicd-token \
--clusterrole=deployment-clusterrole \
--serviceaccount=app-team1:cicd-token

root@cka-master:~# kubectl -n app-team1 describe rolebindings.rbac.authorization.k8s.io bind-cicd-token
Name:         bind-cicd-token
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  deployment-clusterrole
Subjects:
  Kind            Name        Namespace
  ----            ----        ---------
  ServiceAccount  cicd-token  app-team1
```
验证服务账号权限,经过验证，可以创建deployments，但是不能创建secret

```bash
root@cka-master:~# kubectl -n app-team1 auth can-i create deployments --as system:serviceaccount:app-team1:cicd-token
yes
root@cka-master:~# kubectl -n app-team1 auth can-i create secret --as system:serviceaccount:app-team1:cicd-token
no
```

# Q3: Node Maintenance

Set the node named cka-master as unavailiable and reschedule all the pods running on it

---

Q3 中文题目：

将cka-master节点设置为不可用，然后重新调度该节点上的所有Pod

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
禁用新的调度
```bash
root@cka-master:~# kubectl cordon cka-master

root@cka-master:~# kubectl get nodes
NAME          STATUS                        ROLES           AGE   VERSION
cka-master    Ready,SchedulingDisabled      control-plane   11h   v1.27.2
cka-worker1   NotReady,SchedulingDisabled   worker          11h   v1.27.2
cka-worker2   Ready                         worker          11h   v1.27.2
```
将现有工作负载驱赶到其他节点
```bash
root@cka-master:~# kubectl drain cka-master --delete-emptydir-data --ignore-daemonsets
```

# Q4: Upgrading kubeadm clusters

Given an existing kubernetes cluster running version 1.27.2,upgrade all of the Kubernetes control plane and node components on the master node only to version 1.27.4,Please do not upgrade etcd database.

You are also expected to upgrade kubelete and kubectl on the master node.

---

Q4 中文题目：

现有的 Kubernetes 集群正在运行的版本是 1.27.2，仅将主节点上的所有 kubernetes 控制面板和组件升级到版本 1.27.4，请勿升级etcd数据库， 另外，在主节点上升级 kubelet 和 kubectl

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```

查询现有集群版本和软件版本
```bash
root@cka-master:~# kubectl get nodes
NAME          STATUS                        ROLES           AGE   VERSION
cka-master    Ready,SchedulingDisabled      control-plane   11h   v1.27.2
cka-worker1   NotReady,SchedulingDisabled   worker          11h   v1.27.2
cka-worker2   Ready                         worker          11h   v1.27.2

root@cka-master:~# kubelet --version
Kubernetes v1.27.2

root@cka-master:~# kubectl version
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.2"}
Server Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.2"}

root@cka-master:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.2"}
```
禁用新的调度
```bash
root@cka-master:~# kubectl cordon cka-master
```
将现有工作负载驱赶到其他节点
```bash
root@cka-master:~# kubectl drain cka-master --delete-emptydir-data --ignore-daemonsets
```
安装指定版本的kubeadm
```bash
root@cka-master:~# apt-mark unhold kubeadm
root@cka-master:~# apt update
root@cka-master:~# apt install kubeadm=1.27.4-00 -y
Reading package lists... Done
Setting up kubeadm (1.27.4-00)
...
root@cka-master:~# apt-mark hold kubeadm

root@cka-master:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.4"}
```
查询现有可升级的版本
```bash
root@cka-master:~# kubeadm upgrade plan
...
You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.27.4
```
完成版本升级，记得指定不升级etcd的参数
```bash
root@cka-master:~# apt-mark unhold kubeadm
root@cka-master:~# kubeadm upgrade apply v1.27.4 --etcd-upgrade=false
...
[upgrade] Are you sure you want to proceed? [y/N]: y
...
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.27.4". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```
升级完成后，把kubelet、kubectl升级到指定版本

```bash
root@cka-master:~# apt-mark unhold kubelet
root@cka-master:~# apt-mark unhold kubectl
root@cka-master:~# apt install kubelet=1.27.4-00 kubectl=1.27.4-00 -y
Reading package lists... Done
Building dependency tree
Reading state information... Done
root@cka-master:~# apt-mark hold kubelet
root@cka-master:~# apt-mark hold kubectl
root@cka-master:~# apt-mark hold kubeadm
```
重启kuelet服务，并查询状态
```bash
root@cka-master:~# systemctl daemon-reload
root@cka-master:~# systemctl restart kubelet.service
root@cka-master:~# kubectl get nodes
NAME          STATUS                        ROLES           AGE   VERSION
cka-master    Ready,SchedulingDisabled      control-plane   11h   v1.27.4
cka-worker1   NotReady,SchedulingDisabled   worker          11h   v1.27.2
cka-worker2   Ready                         worker          11h   v1.27.2
```
这里要格外注意，正式考试的时候一定要对你刚升级的节点执行uncordon，不然本题不会得分
但是我们的模拟考试为了照顾大家的计算机性能，没有额外部署节点，所以在我们模拟考试的时候，不要执行uncordon，不然本地模拟考试时会且只会损失1分
```bash
root@cka-master:~# kubectl uncordon cka-master
root@cka-master:~# kubectl get nodes
NAME          STATUS                        ROLES           AGE   VERSION
cka-master    Ready                         control-plane   19m   v1.27.4
cka-worker1   Ready                         worker          17m   v1.27.2
cka-worker2   Ready                         worker          16m   v1.27.2
```

# Q5: Create NetworkPolicy

Create a new NetworkPolicy named allow-port-from-namespace that allows Pods in namespace corp to connect to port 80 of other Pods in the internal namespace.

Ensure that the new NetworkPolicy:

1. does not allow access to Pods not listening on port 80

2. does not allow access from Pods not in namespace corp

---

Q5 中文题目：

创建一个名为allow-port-from-namespace的NetworkPolicy， 这个NetworkPolicy允许corp命名空间下的Pod访问internal命名空间下pod的80端口

确保新的 NetworkPolicy：

1. 不允许访问没有监听80端口的Pod

2. 并且只允许corp命令空间的下的Pod访问

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```

创建networkpolicy

```bash
vim networkpolicy.yml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-from-namespace
  namespace: internal
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: corp
    ports:
    - protocol: TCP
      port: 80
```

创建并验证策略是否正确

```bash
root@cka-master:~# kubectl create -f networkpolicy.yml 

root@cka-master:~# kubectl describe networkpolicies.networking.k8s.io -n internal allow-port-from-namespace
Name:         allow-port-from-namespace
Namespace:    internal
Created on:   2022-12-01 07:01:53 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    To Port: 80/TCP
    From:
      NamespaceSelector: kubernetes.io/metadata.name=corp
  Not affecting egress traffic
  Policy Types: Ingress

```
确认internal pod的IP地址

```bash
root@cka-master:~# kubectl get pod -n internal -o wide
NAME         READY   STATUS    RESTARTS   AGE     IP             NODE          NOMINATED NODE   READINESS GATES
internlpod   1/1     Running   0          5m21s   172.16.245.1   cka-worker2   <none>           <none>
```

从宿主机上无法访问
```bash
root@cka-master:~# curl 172.16.245.1
```
从corp namespace中可以访问
```bash
root@cka-master:~# kubectl get pod -n corp
NAME      READY   STATUS    RESTARTS   AGE
corppod   1/1     Running   0          33m

root@cka-master:~# kubectl exec -it -n corp corppod -- curl 172.16.245.1

<title>Welcome to nginx!</title>
```
# Q6: Create service

Reconfigure the existing deployment front-end and add a port specification named http exposing port 80/tcp of the existing container nginx

Create a new service named front-end-svc exposing the container port http.

Configure the new service to also expose the individual Pods via a NoedPort on the nodes on which they are scheduled.

---

Q6 中文题目:

重新配置一个已经存在的deployment front-end，在名字为nginx的容器里面添加一个端口配置，名字为http，暴露端口号为80，然后创建一个service，名字为front-end-svc的服务用于暴露容器的http端口，并且service的类型为NodePort。

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```

编辑deployment，在容器下面添加端口，为了优化速度，镜像用了阿里云，考试用它指定的镜像

```bash
root@cka-master:~# kubectl edit deployments.apps front-end
...
containers:
      - image: registry.cn-shanghai.aliyuncs.com/cnlxh/nginx
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
...
```

以NodePort方式暴露端口并验证
```bash
root@cka-master:~# kubectl expose deployment front-end --name=front-end-svc --port=80 --target-port=80 --type=NodePort

root@cka-master:~# kubectl get service
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
front-end-svc   NodePort    10.104.139.137   <none>        80:32402/TCP   74s
kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP        13h

root@cka-master:~# curl 10.104.139.137
...
<title>Welcome to nginx!</title>

root@cka-master:~# curl cka-worker2:32402
...
<title>Welcome to nginx!</title>
```

# Q7: Create ingress

Create a new nginx ingress resource as follows:

1. Name: pong

2. Namespace: ing-internal

3. Exposing service hi on path /hi using service port 5678

Tips: 

The availability of service hi can be checked using the following commands,which should retun hi: 
curl -KL <INTERNAL_IP>/hi

---

Q7 中文题目：

在ing-internal 命名空间下创建一个nginx ingress，名字为pong，代理的service为hi，端口为5678，配置路径/hi。

验证：访问curl -kL <INTERNAL_IP>/hi会返回hi

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
创建ingress
```bash
vim ingress.yml 
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pong
  namespace: ing-internal
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /hi
        pathType: Prefix
        backend:
          service:
            name: hi
            port:
              number: 5678
```
```bash
root@cka-master:~# kubectl create -f ingress.yml 
```
验证是否可访问
```bash
root@cka-master:~# kubectl get ingress -n ing-internal
NAME   CLASS   HOSTS   ADDRESS          PORTS   AGE
pong   nginx   *       192.168.30.132   80      116s

root@cka-master:~# curl -kL 192.168.30.132/hi
hi
```

# Q8: Scale deployment

Scale the deployment loadbalancer to 6 pods and record it

---

Q8 中文题目：

扩容名字为loadbalancer的deployment的副本数为6

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
调整副本数为6并验证

```bash
root@cka-master:~# kubectl scale deployment loadbalancer --replicas=6 --record

root@cka-master:~# kubectl get deployments.apps loadbalancer
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
loadbalancer   6/6     6            6           43m
```

# Q9: Assigning Pods to Nodes

Schedule a pod as follows:

1. Name: nginx-kusc00401

2. Image: nginx

3. Node selector: disk=spinning

---

Q9 中文题目：

创建一个Pod，名字为nginx-kusc00401，镜像是nginx，调度到具有disk=spinning标签的节点上

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
创建pod，为了优化速度，镜像用了阿里云，考试用它指定的镜像

```bash
vim assignpod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-kusc00401
spec:
  containers:
  - name: nginx
    image: registry.cn-shanghai.aliyuncs.com/cnlxh/nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disk: spinning 
```

```bash
kubectl create -f assignpod.yml
```
验证nodes标签
```bash
root@cka-master:~# kubectl get nodes --show-labels
NAME          STATUS     ROLES           AGE   VERSION   LABELS
cka-master    Ready      control-plane   14h   v1.27.4   ...
cka-worker1   NotReady   worker          14h   v1.27.2   ...
cka-worker2   Ready      worker          14h   v1.27.2   disk=spinning
```
查询pod是否如期调度
```bash
root@cka-master:~# kubectl get pod -o wide
NAME                                      READY   STATUS    RESTARTS   AGE    IP              NODE          NOMINATED NODE   READINESS GATES
nginx-kusc00401                           1/1     Running   0          22s    172.16.245.18   cka-worker2   <none>           <none>
```

# Q10: Find how many health node

Check to see how many nodes are ready (not including nodes tainted NoSchedule) and write the number to /opt/kusc00402.txt

---

Q10 中文题目：

检查集群中有多少节点为Ready状态，并且去除包含NoSchedule污点的节点。之后将数字写到/opt/kusc00402.txt

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
查询出不包含noschedule和unreachable的节点数量

```bash
root@cka-master:~# kubectl describe node | grep -i taints|grep -v -i -e noschedule -e unreachable | wc -l
1
```
以你实际情况为准
```bash
root@cka-master:~# echo 1 > /opt/kusc00402.txt
root@cka-master:~# cat /opt/kusc00402.txt
1
```

# Q11: Create multi container in pod

Create a pod named kucc1 with a single app container for each of the following images running inside(there may be between 1 and 4 images specified): nginx+redis+memcached+consul.

---

Q11 中文题目：

创建一个Pod，名字为kucc1，这个Pod可能包含1-4容器，该题为四个：nginx+redis+memcached+consul

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
创建kucc1 pod，为了优化速度，镜像用了阿里云，考试用它指定的镜像
```bash
vim multipod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kucc1
spec:
  containers:
  - name: nginx
    image: registry.cn-shanghai.aliyuncs.com/cnlxh/nginx
  - name: redis
    image: registry.cn-shanghai.aliyuncs.com/cnlxh/redis
  - name: memcached
    image: registry.cn-shanghai.aliyuncs.com/cnlxh/memcached
  - name: consul
    image: registry.cn-shanghai.aliyuncs.com/cnlxh/consul
```

```bash
root@cka-master:~# kubectl create -f multipod.yml 
```
验证是否是4/4 running
```bash
root@cka-master:~# kubectl get -f multipod.yml 
NAME    READY   STATUS    RESTARTS   AGE
kucc1   4/4     Running   0          2m3s
```

# Q12: Create PersistentVolume

Create a persistent volume with name app-config, of capacity 2Gi and access mode ReadWriteMany. the type of volume is hostPath and its location is /srv/app-config

---

Q12 中文题目:

创建一个pv，名字为app-config，大小为2Gi，访问权限为ReadWriteMany。Volume的类型为hostPath，路径为/srv/app-config

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
创建PV

```bash
vim pv.yml 
```
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-config
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/srv/app-config"
    type: DirectoryOrCreate 
```

```bash
root@cka-master:~# kubectl create -f pv.yml 

root@cka-master:~# kubectl get -f pv.yml 
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
app-config   2Gi        RWO            Retain           Available                                   2m
```

# Q13: Create PVC

Create a new PersistentVolumeClaim:

1. Name: pv-volume

2. Class: csi-hostpath-sc

3. Capacity: 10Mi

Create a new Pod which mounts the persistentVolumeClaim as a volume:

1. Name: web-server

2. Image: nginx

3. Mount path: /usr/share/nginx/html

Configure the new Pod to have ReadWriteOnce access on the volume.

Finally, using kubectl edit or kubectl patch expand the PersistentVolumeClaim to a capacity of 70Mi and record that change.

---

Q13 中文题目：

创建一个名字为pv-volume的pvc，指定storageClass为csi-hostpath-sc，大小为10Mi
然后创建一个Pod，名字为web-server，镜像为nginx，并且挂载该PVC至/usr/share/nginx/html，挂载的权限为ReadWriteOnce。之后通过kubectl edit或者kubectl path将pvc改成70Mi，并且记录修改记录。

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
创建PVC，注意storageClassName

```bash
vim pvc.yml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-volume
spec:
  storageClassName: csi-hostpath-sc
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
```

```bash
root@cka-master:~# kubectl create -f pvc.yml 
```
创建pod来使用pvc，为了优化速度，镜像用了阿里云，考试用它指定的镜像

```bash
vim pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server
spec:
  containers:
    - name: nginx
      image: registry.cn-shanghai.aliyuncs.com/cnlxh/nginx
      volumeMounts:
      - mountPath: "/usr/share/nginx/html"
        name: pv-volume
  volumes:
    - name: pv-volume
      persistentVolumeClaim:
        claimName: pv-volume
```

```bash
root@cka-master:~# kubectl create -f pod.yml 

root@cka-master:~# kubectl get pod web-server
NAME         READY   STATUS    RESTARTS   AGE
web-server   1/1     Running   0          31s
```

扩容至70Mi

确认存储类是否允许扩容，注意下面的ALLOWVOLUMEEXPANSION参数是否为true

```bash
root@cka-master:~# kubectl get storageclasses csi-hostpath-sc
NAME                        PROVISIONER         RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
csi-hostpath-sc (default)   cnlxh/nfs-storage   Delete          Immediate           true                   51m
```
如果ALLOWVOLUMEEXPANSION参数是False，是不能扩容的，采取下方的方法修改一下，改成true
```bash
kubectl edit storageclasses csi-hostpath-sc
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-hostpath-sc
provisioner: cnlxh/nfs-storage
...
allowVolumeExpansion: true
```
第一种方法，模拟环境中由于NFS特性，不支持扩容，考试中可以的
```bash
root@cka-master:~# kubectl patch pvc pv-volume  -p '{"spec":{"resources":{"requests":{"storage": "70Mi"}}}}' --record
```
第二种方法，模拟环境中由于NFS特性，不支持扩容，考试中可以的
```bash
root@cka-master:~# kubectl  edit pvc pv-volume --record=true
...
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 70Mi
```

# Q14: Monitor pod logs

Monitor the logs of pod foobar and :

1. Extract log lines corresponding to error unable-to-access-website

2. Write them to /opt/foobar.txt

---

Q14 中文题目：

监控名为foobar的Pod的日志，并过滤出具有unable-to-access-website 信息的行，然后将写入到 /opt/foobar.txt

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```

```bash
root@cka-master:~# kubectl logs foobar  |grep unable-to-access-website > /opt/foobar.txt
```

# Q15: Add sidecar container

Without changing its existing containers, an existing Pod needs to be integrated into Kubernetes's build-in logging architecture(e.g kubectl logs).Adding a streaming sidecar container is a good and common way accomplish this requirement.

Task
Add a busybox sidecar container to the existing Pod legacy-app. The new sidecar container has to run the following command:

```bash
/bin/sh -c tail -n+1 -f /var/log/legacy-app.log
```
Use a volume mount named logs to make the file /var/log/legacy-app.log available to the sidecar container.

**TIPS**

1. Don't modify the existing container.

2. Don't modify the path of the log file, both containers

3. must access it at /var/log/legacy-app.log

---

Q15 中文题目：

在不更改现有容器的情况下，添加一个名为busybox且镜像为busybox的sidecar到一个已经存在的名为legacy-app的Pod上，这个sidecar的启动命令为/bin/sh, -c, 'tail -n+1 -f /var/log/legacy-app.log'，并且这个sidecar和原有的镜像挂载一个名为logs的volume，挂载的目录为/var/log/

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
导出现有容器yaml

```bash
root@cka-master:~# kubectl get pod legacy-app -o yaml > sidecar.yaml
```
修改导出的yaml文件，在现有容器上添加volumeMounts参数来挂载logs到/var/log，并添加新容器同样挂载这个位置，不要忘了创建出logs卷，为了优化速度，镜像用了阿里云，考试用它指定的镜像
```bash
vim sidecar.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: legacy-app
....
spec:
  containers:
  - image: registry.cn-shanghai.aliyuncs.com/cnlxh/busybox
    ...
    # 找到第一个容器下面本来就存在的volumeMounts参数，添加下面的logs卷
    volumeMounts:
    - name: logs
      mountPath: /var/log
      ...
    # 新加一个容器
  - name: busybox
    image: registry.cn-shanghai.aliyuncs.com/cnlxh/busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/legacy-app.log']
    volumeMounts:
    - name: logs
      mountPath: /var/log
      ...
  # 创建出这个卷，需要注意的是，导出的文件中，本来就有volumes参数，请在它本来的volumes参数下面写logs卷的创建，不然无法创建pod，提示没有卷
  volumes:
  - name: logs
    emptyDir: {}
```
pod不支持直接修改，确认没问题之后，直接删除原有pod并重建

```bash
root@cka-master:~# kubectl delete pod legacy-app
root@cka-master:~# kubectl create -f sidecar.yaml
root@cka-master:~# kubectl get pod legacy-app
NAME         READY   STATUS    RESTARTS      AGE
legacy-app   2/2     Running      0         2m7s

```

# Q16: Find pod with high cpu usage

From the pod label name=cpu-user, find pods running high CPU workloads and write the name of the pod consuming most CPU to the file /opt/findhighcpu.txt

---

Q16 中文题目：

找出具有name=cpu-user标签的Pod，并过滤出使用CPU最高的Pod，然后把它的名字写在/opt/findhighcpu.txt文件里

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
考试时直接用top pod来看谁的CPU用的更高，填写即可，用肉眼分析也可以的

```bash
root@cka-master:~# kubectl top pod -l name=cpu-user -A --sort-by cpu
NAMESPACE   NAME   CPU(cores)   MEMORY(bytes)
default     foobar    1m           5Mi

root@cka-master:~# echo 'foobar' > /opt/findhighcpu.txt
```

# Q17: Fixing kubernetes node state

A kubernetes worker node, named CKA-Worker1 is in state NotReady. Investigate why this is the case, and perform any appropriate steps to bring the node to a Ready state,ensuring that any changes are made permanent.

Tips:

1、you can ssh to the failed node using:

ssh CKA-Worker1

2、you can assume elevated privileges on the node with the following command：

sudo  -i

---

Q17 中文题目：

一个名为CKA-Worker1的节点状态为NotReady，让其恢复至正常状态，并确认所有的更改重新开机依旧有效

---

切换集群，考试时基本每道题都需要切换context集群，一定不要忘了复制它给的切换命令并执行

```bash
root@cka-master:~# kubectl config use-context kubernetes-admin@kubernetes
```
发现cka-worker1节点是NotReady
```bash
root@cka-master:~# kubectl get nodes
NAME          STATUS                        ROLES           AGE    VERSION
cka-master    Ready,SchedulingDisabled      control-plane   2d1h   v1.27.4
cka-worker1   NotReady,SchedulingDisabled   worker          2d1h   v1.27.2
cka-worker2   Ready                         worker          2d1h   v1.27.2
```
描述一下节点，看看事件，发现kubelet服务停止了
```bash
root@cka-master:~# kubectl describe nodes cka-worker1
...
Conditions:
  Type                 Reason                       Message
  ----                 ------                       ----------------- 
  NetworkUnavailable   CalicoIsUp                   Calico is running on this node
  MemoryPressure       NodeStatusUnknown            Kubelet stopped posting node status.
  DiskPressure         NodeStatusUnknown            Kubelet stopped posting node status.
  PIDPressure          NodeStatusUnknown            Kubelet stopped posting node status.
  Ready                NodeStatusUnknown            Kubelet stopped posting node status.

```
一般kubelet服务不启动，也有可能的runtime的问题，顺便查询docker服务，发现也没启动
```bash
root@cka-worker1:~# systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; disabled; vendor preset: enabled)
     Active: inactive (dead) since Tue 2022-12-06 14:08:47 UTC; 1s ago
```
修复两个服务状态，记得enable
```bash
root@cka-master:~# ssh root@cka-worker1
root@cka-worker1:~# systemctl start docker 
root@cka-worker1:~# systemctl start kubelet 
root@cka-worker1:~# systemctl enable kubelet docker
```
**另外请注意，有同学反馈考试时有遇到runtime不是docker，而是containerd的情况，所以考试时，灵活一些，看看到底是docker还是containerd，具体可以docker images看看是否有考试过程中的镜像，或者以下方式查询：**

```bash
root@cka-master:~# kubectl describe nodes cka-worker1 | grep -A 10 'System Info'
System Info:
  Machine ID:                 77b031532486421d9571f82739654f48
  System UUID:                2dff4d56-861e-193b-d91f-a975eb9f0d12
  Boot ID:                    728e4c89-791e-422d-9b1a-bca1c1c4f345
  Kernel Version:             5.4.0-125-generic
  OS Image:                   Ubuntu 20.04.5 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://23.0.1
  Kubelet Version:            v1.27.4
  Kube-Proxy Version:         v1.27.4
```

Container Runtime Version:  docker://23.0.1

可以看出是docker的23.0.1版本，而非containerd，当然，本题目中具体是哪个节点不正常，就去describe哪个节点