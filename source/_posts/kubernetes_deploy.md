---
title: Kubernetes系列：kubeadm部署K8S集群v1.16.4
date:  2019-12-01
categories:
  - 云原生
tags:
  - Kubernetes
  - kubeadm
---
# 环境准备工作

## 环境角色

| IP           | 角色       | 软件安装                                                     |
| ------------ | ---------- | ------------------------------------------------------------ |
| 192.168.0.16 | K8S Master | kube-apiserver kube-schduler kube-controller-manager kube-proxy docker flannel kubelet etcd |
| 192.168.0.17 | K8s Node   | kubelet kube-proxy docker flannel                            |
| 192.168.0.18 | K8s Node   | kubelet kube-proxy docker flannel                            |

> 以下所有1.x操作,在三台节点都执行

## 统一设置主机名

```
$ hostnamectl set-hostname test01
$ hostnamectl set-hostname test02
$ hostnamectl set-hostname test03
```

## 部署机免密登录到test01、test02、test03

> 前提是目标主机需要生成公钥以及密钥 ssh-keygen -t rsa

```
[root@harbor ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@test02
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'test02 (192.168.0.17)' can't be established.
ECDSA key fingerprint is SHA256:Omg7RAlyXPvDcwNyJdufEmAMHnwcS3eh/gsaHPZVP6I.
ECDSA key fingerprint is MD5:5d:55:0f:b2:75:4d:39:ee:47:c1:a8:3f:0f:12:96:30.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@test02's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@test02'"
and check to make sure that only the key(s) you wanted were added.
```



## 关闭防火墙及selinux

> docker selinux-enabled作用和原理看介绍

https://www.cnblogs.com/elnino/p/10845449.html

> 防火墙的文档介绍

https://wangchujiang.com/linux-command/c/firewall-cmd.html

```
# 关闭selinux
$ systemctl stop firewalld && systemctl disable firewalld
$ sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux && setenforce 0
# 关闭防火墙
[root@test01 ~]# systemctl start firewalld
[root@test01 ~]# firewall-cmd --set-default-zone=trusted
success
[root@test01 ~]# firewall-cmd --complete-reload
success
```

## 关闭 swap 分区

> 对于禁用swap内存，具体原因可以查看Github上的Issue：https://github.com/kubernetes/kubernetes/issues/53533

```
$ swapoff -a # 临时
$ sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab #永久
```

## 内核调整,将桥接的IPv4流量传递到iptables的链

> https://kubernetes.io/zh/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/
>
> ```
> 开启Linux的路由转发功能:
> sysctl -w net.ipv4.ip_forward=1
> 
> cni插件确保简单的配置（如带网桥的 Docker ）与 iptables 代理正常工作:
> sysctl -w net.bridge.bridge-nf-call-iptables=1
> sysctl -w net.bridge.bridge-nf-call-ip6tables=1
> ```

```
[root@test01 ~]# cat /proc/sys/net/ipv4/ip_forward
1
[root@test01 ~]# cat /proc/sys/net/bridge/bridge-nf-call-iptables
1
[root@test01 ~]# cat /proc/sys/net/bridge/bridge-nf-call-ip6tables
1
```

## 设置系统时区并同步时间服务器

```
$ yum install -y ntpdate
$ ntpdate time.windows.com
```

# 安装dokcer

```
[root@test01 ~]# yum install docker -y
[root@test01 ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
[root@test01 ~]# systemctl start docker
[root@test01 ~]# docker --version
Docker version 1.13.1, build 4ef4b30/1.13.1
```

# 安装K8s

## 添加kubernetes YUM软件源

```
$ cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

## 安装kubeadm,kubelet和kubectl

```
$ yum install -y kubelet-1.16.4 kubeadm-1.16.4 kubectl-1.16.4
$ systemctl enable kubelet
```

## 部署Kubernetes Master

> 只需要在Master 节点执行，这里的apiserver需要修改成自己的master地址。
> 由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址。
>
> ```
> [root@test01 ~]# kubeadm init --apiserver-advertise-address=192.168.0.16 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.16.4 --service-cidr=10.1.0.0/16 --pod-network-cidr=10.244.0.0/16
> [init] Using Kubernetes version: v1.16.4
> [preflight] Running pre-flight checks
> 	[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
> 	[WARNING Hostname]: hostname "test01" could not be reached
> 	[WARNING Hostname]: hostname "test01": lookup test01 on 10.16.140.4:53: no such host
> [preflight] Pulling images required for setting up a Kubernetes cluster
> [preflight] This might take a minute or two, depending on the speed of your internet connection
> [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
> [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
> [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
> [kubelet-start] Activating the kubelet service
> [certs] Using certificateDir folder "/etc/kubernetes/pki"
> [certs] Generating "ca" certificate and key
> [certs] Generating "apiserver" certificate and key
> [certs] apiserver serving cert is signed for DNS names [test01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.1.0.1 192.168.0.16]
> [certs] Generating "apiserver-kubelet-client" certificate and key
> [certs] Generating "front-proxy-ca" certificate and key
> [certs] Generating "front-proxy-client" certificate and key
> [certs] Generating "etcd/ca" certificate and key
> [certs] Generating "etcd/server" certificate and key
> [certs] etcd/server serving cert is signed for DNS names [test01 localhost] and IPs [192.168.0.16 127.0.0.1 ::1]
> [certs] Generating "etcd/peer" certificate and key
> [certs] etcd/peer serving cert is signed for DNS names [test01 localhost] and IPs [192.168.0.16 127.0.0.1 ::1]
> [certs] Generating "etcd/healthcheck-client" certificate and key
> [certs] Generating "apiserver-etcd-client" certificate and key
> [certs] Generating "sa" key and public key
> [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
> [kubeconfig] Writing "admin.conf" kubeconfig file
> [kubeconfig] Writing "kubelet.conf" kubeconfig file
> [kubeconfig] Writing "controller-manager.conf" kubeconfig file
> [kubeconfig] Writing "scheduler.conf" kubeconfig file
> [control-plane] Using manifest folder "/etc/kubernetes/manifests"
> [control-plane] Creating static Pod manifest for "kube-apiserver"
> [control-plane] Creating static Pod manifest for "kube-controller-manager"
> [control-plane] Creating static Pod manifest for "kube-scheduler"
> [etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
> [wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
> [apiclient] All control plane components are healthy after 33.002262 seconds
> [upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
> [kubelet] Creating a ConfigMap "kubelet-config-1.16" in namespace kube-system with the configuration for the kubelets in the cluster
> [upload-certs] Skipping phase. Please see --upload-certs1
> [mark-control-plane] Marking the node test01 as control-plane by adding the label "node-role.kubernetes.io/master=''"
> [mark-control-plane] Marking the node test01 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
> [bootstrap-token] Using token: 7e169l.60ykxxr8md7sb8ak
> [bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
> [bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
> [bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
> [bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
> [bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
> [addons] Applied essential addon: CoreDNS
> [addons] Applied essential addon: kube-proxy
> 
> Your Kubernetes control-plane has initialized successfully!
> 
> To start using your cluster, you need to run the following as a regular user:
> 
>   mkdir -p $HOME/.kube
>   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
>   sudo chown $(id -u):$(id -g) $HOME/.kube/config
> 
> You should now deploy a pod network to the cluster.
> Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
>   https://kubernetes.io/docs/concepts/cluster-administration/addons/
> 
> Then you can join any number of worker nodes by running the following on each as root:
> 
> kubeadm join 192.168.0.16:6443 --token 7e169l.60ykxxr8md7sb8ak \
>     --discovery-token-ca-cert-hash sha256:c0b04a34ac7bb55fdf5b04a70233c14c15a11341a2bde0ea33b30579d84c0ce4
> ```

> 根据kubeadm日志输出提示操作：
>
> ```
> [root@test01 ~]# mkdir -p $HOME/.kube
> [root@test01 ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
> [root@test01 ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
> [root@test01 ~]# kubectl get po --all-namespaces
> NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
> kube-system   coredns-58cc8c89f4-jrfqj         0/1     Pending   0          12m
> kube-system   coredns-58cc8c89f4-qh5bg         0/1     Pending   0          12m
> kube-system   etcd-test01                      1/1     Running   0          11m
> kube-system   kube-apiserver-test01            1/1     Running   0          11m
> kube-system   kube-controller-manager-test01   1/1     Running   0          11m
> kube-system   kube-proxy-m6hp2                 1/1     Running   0          12m
> kube-system   kube-scheduler-test01            1/1     Running   0          11m
> ```

> 继续安装网络插件
> kube-flannel.yml下载地址如下
> https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
>
> ```
> [root@test01 ~]# kubectl apply -f kube-flannel.yml
> podsecuritypolicy.policy/psp.flannel.unprivileged created
> clusterrole.rbac.authorization.k8s.io/flannel created
> clusterrolebinding.rbac.authorization.k8s.io/flannel created
> serviceaccount/flannel created
> configmap/kube-flannel-cfg created
> daemonset.apps/kube-flannel-ds-amd64 created
> daemonset.apps/kube-flannel-ds-arm64 created
> daemonset.apps/kube-flannel-ds-arm created
> daemonset.apps/kube-flannel-ds-ppc64le created
> daemonset.apps/kube-flannel-ds-s390x created
> ```

## 添加worknode节点

```
[root@test02 yum.repos.d]# kubeadm join 192.168.0.16:6443 --token 7e169l.60ykxxr8md7sb8ak \
>     --discovery-token-ca-cert-hash sha256:c0b04a34ac7bb55fdf5b04a70233c14c15a11341a2bde0ea33b30579d84c0ce4
[preflight] Running pre-flight checks
	[WARNING Hostname]: hostname "test02" could not be reached
	[WARNING Hostname]: hostname "test02": lookup test02 on 10.16.140.4:53: no such host
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.16" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

[root@test01 ~]# kubectl get po --all-namespaces -o wide
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE     IP             NODE     NOMINATED NODE   READINESS GATES
kube-system   coredns-58cc8c89f4-jrfqj         1/1     Running   0          46m     10.244.0.3     test01   <none>           <none>
kube-system   coredns-58cc8c89f4-qh5bg         1/1     Running   0          46m     10.244.0.2     test01   <none>           <none>
kube-system   etcd-test01                      1/1     Running   0          45m     192.168.0.16   test01   <none>           <none>
kube-system   kube-apiserver-test01            1/1     Running   0          45m     192.168.0.16   test01   <none>           <none>
kube-system   kube-controller-manager-test01   1/1     Running   0          45m     192.168.0.16   test01   <none>           <none>
kube-system   kube-flannel-ds-amd64-7cwtb      1/1     Running   0          5m40s   192.168.0.16   test01   <none>           <none>
kube-system   kube-flannel-ds-amd64-gcw85      1/1     Running   1          3m47s   192.168.0.18   test03   <none>           <none>
kube-system   kube-flannel-ds-amd64-sqj75      1/1     Running   0          3m50s   192.168.0.17   test02   <none>           <none>
kube-system   kube-proxy-fr7wk                 1/1     Running   0          3m47s   192.168.0.18   test03   <none>           <none>
kube-system   kube-proxy-m6hp2                 1/1     Running   0          46m     192.168.0.16   test01   <none>           <none>
kube-system   kube-proxy-qpv2g                 1/1     Running   0          3m50s   192.168.0.17   test02   <none>           <none>
kube-system   kube-scheduler-test01            1/1     Running   0          45m     192.168.0.16   test01   <none>           <none>
[root@test01 ~]# kubectl get no
NAME     STATUS   ROLES    AGE     VERSION
test01   Ready    master   46m     v1.16.4
test02   Ready    <none>   4m1s    v1.16.4
test03   Ready    <none>   3m58s   v1.16.4
```

## 测试K8s集群

```
[root@test01 ~]# kubectl create deployment nginx --image=nginx
deployment.apps/nginx created
[root@test01 ~]# kubectl expose deployment nginx --port=80 --type=NodePort
service/nginx exposed
[root@test01 ~]# kubectl get pods,svc
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-86c57db685-8mmgz   1/1     Running   0          21s

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.1.0.1      <none>        443/TCP        55m
service/nginx        NodePort    10.1.210.90   <none>        80:30580/TCP   13s

[root@test01 ~]# curl -I 10.1.210.90
HTTP/1.1 200 OK
Server: nginx/1.17.8
Date: Thu, 13 Feb 2020 15:04:16 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 21 Jan 2020 13:36:08 GMT
Connection: keep-alive
ETag: "5e26fe48-264"
Accept-Ranges: bytes

[root@test01 ~]# curl -I 192.168.0.18:30580
HTTP/1.1 200 OK
Server: nginx/1.17.8
Date: Thu, 13 Feb 2020 15:04:25 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 21 Jan 2020 13:36:08 GMT
Connection: keep-alive
ETag: "5e26fe48-264"
Accept-Ranges: bytes
```

## 部署Dashboard

> 安装参考官方https://github.com/kubernetes/dashboard

> 1、预先在$HOME/certs目录下放置apiserver证书

```
[root@test01 ~]# ll
-rw-------. 1 root root  1348 8月  13 2018 anaconda-ks.cfg
drwxr-xr-x  2 root root  4096 2月  23 04:36 certs
-rw-r--r--  1 root root  7682 2月  23 05:01 dashboard-recommended.yaml
-rw-r--r--  1 root root 14428 2月  13 22:46 kube-flannel.yml
[root@test01 ~]# kubectl create secret generic kubernetes-dashboard-certs --from-file=$HOME/certs -n kubernetes-dashboard
secret/kubernetes-dashboard-certs created
```

> 2、部署dashboard-recommended.yaml
>
> ```
> [root@test01 ~]# kubectl apply -f dashboard-recommended.yaml
> namespace/kubernetes-dashboard created
> serviceaccount/kubernetes-dashboard created
> service/kubernetes-dashboard created
> secret/kubernetes-dashboard-certs created
> secret/kubernetes-dashboard-csrf created
> secret/kubernetes-dashboard-key-holder created
> configmap/kubernetes-dashboard-settings created
> role.rbac.authorization.k8s.io/kubernetes-dashboard created
> clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
> rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
> clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
> deployment.apps/kubernetes-dashboard created
> service/dashboard-metrics-scraper created
> deployment.apps/dashboard-metrics-scraper created
> 
> [root@test01 ~]# kubectl get po -n kubernetes-dashboard
> NAME                                         READY   STATUS    RESTARTS   AGE
> dashboard-metrics-scraper-7b8b58dc8b-x5p9p   1/1     Running   0          18m
> kubernetes-dashboard-557f4b4587-zjglq        1/1     Running   0          18m
> [root@test01 ~]#
> [root@test01 ~]# kubectl get svc -n kubernetes-dashboard
> NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
> dashboard-metrics-scraper   ClusterIP   10.1.80.221    <none>        8000/TCP        21m
> kubernetes-dashboard        NodePort    10.1.241.233   <none>        443:30001/TCP   21m
> ```

> 3、重新创建serviceaccount并绑定默认cluster-admin管理员集群角色
>
> ```
> [root@test01 ~]# kubectl delete clusterrolebinding kubernetes-dashboard
> clusterrolebinding.rbac.authorization.k8s.io "kubernetes-dashboard" deleted
> [root@test01 ~]# kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:kubernetes-dashboard
> clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
> ```

> 4、获取token，访问30001端口输入token即可成功登录
>
> ```
> [root@test01 ~]# kubectl get secret -n kubernetes-dashboard
> NAME                               TYPE                                  DATA   AGE
> default-token-vzv7l                kubernetes.io/service-account-token   3      4m26s
> kubernetes-dashboard-certs         Opaque                                2      3m24s
> kubernetes-dashboard-csrf          Opaque                                1      4m26s
> kubernetes-dashboard-key-holder    Opaque                                2      4m26s
> kubernetes-dashboard-token-hpff2   kubernetes.io/service-account-token   3      4m26s
> 
> [root@test01 ~]# kubectl describe secret kubernetes-dashboard-token-hpff2 -n kubernetes-dashboard |grep token:
> token:      eyJhbGciOiJSUzI1NiIsImtpZCI6InBYV1Utc2Yxc0JvWE5zQkFpTW1lX2hpRmJKY3ZWOWhkSDhWWHFscHBSMTAifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi1ocGZmMiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjkyOWI3NTczLWU5OGQtNGU1NC04NTBiLWQyZmYyODIxYWM3ZSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.gK7EsvOnaeCUBcIoDTwsH323ZhQ_tmtYFuC9v8DRlKiX_nMbzBiMzmrvCxGRvXeD-ntlmLBE56R9s2yrUEt-3cx0_QFWYBZGmayVMqY7I-sj8fwSwJOYizq6SE4KZO8Ek3bc7qpV2BxKktWVZl77v7jU7upbtMtNRogFCKEAJWdGztpYoSD9F8s-osvcPJMYhHAYhLs002d84ux-CZUTadTMu9Y-mJ-0M5oulNLXB8m3TueAva-Tav6Hgw-1_ABPtpnOLbPJoaHqIzBXhMAgiKKpF6gs0GCui0-8sKGTFlY9jBNuUZSzIF4K9yKEHeF82ZNj9PxlBVajeej7pbCmOg
> ```

# 部署遇到的问题记录（待确认）

> 问题1、kubeadm join时出现warn信息
>
> ```
> [WARNING Hostname]: hostname "test02" could not be reached
> [WARNING Hostname]: hostname "test02": lookup test02 on 10.16.140.4:53: no such host
> ```

> 问题2、kube-proxy没有开启ipvs，默认用了iptables Proxier
>
> ```
> [root@test01 ~]# kubectl logs kube-proxy-fr7wk  -n kube-system
> W0213 14:48:50.804540       1 proxier.go:592] Failed to load kernel module ip_vs with modprobe. You can ignore this message when kube-proxy is running inside container without mounting /lib/modules
> W0213 14:48:50.805641       1 proxier.go:592] Failed to load kernel module ip_vs_rr with modprobe. You can ignore this message when kube-proxy is running inside container without mounting /lib/modules
> W0213 14:48:50.806607       1 proxier.go:592] Failed to load kernel module ip_vs_wrr with modprobe. You can ignore this message when kube-proxy is running inside container without mounting /lib/modules
> W0213 14:48:50.807514       1 proxier.go:592] Failed to load kernel module ip_vs_sh with modprobe. You can ignore this message when kube-proxy is running inside container without mounting /lib/modules
> W0213 14:48:50.811786       1 server_others.go:330] Flag proxy-mode="" unknown, assuming iptables proxy
> I0213 14:48:50.819927       1 node.go:135] Successfully retrieved node IP: 192.168.0.18
> I0213 14:48:50.819966       1 server_others.go:150] Using iptables Proxier.
> I0213 14:48:50.820484       1 server.go:529] Version: v1.16.4
> ```

> 问题3、etcd有异常日志出现
>
> ```
> 2020-02-13 14:48:04.732974 W | wal: sync duration of 1.126809415s, expected less than 1s
> 2020-02-13 14:48:04.733537 W | etcdserver: read-only range request "key:\"/registry/services/endpoints/kube-system/kube-scheduler\" " with result "range_response_count:1 size:429" took too long (760.032104ms) to execute
> 2020-02-13 14:48:04.733800 W | etcdserver: read-only range request "key:\"/registry/minions/test02\" " with result "range_response_count:0 size:5" took too long (975.286916ms) to execute
> 2020-02-13 14:48:04.733918 W | etcdserver: read-only range request "key:\"/registry/jobs/\" range_end:\"/registry/jobs0\" limit:500 " with result "range_response_count:0 size:5" took too long (332.81178ms) to execute
> 2020-02-13 14:48:04.734030 W | etcdserver: read-only range request "key:\"/registry/pods\" range_end:\"/registry/podt\" count_only:true " with result "range_response_count:0 size:7" took too long (377.827782ms) to execute
> 2020-02-13 14:48:04.734145 W | etcdserver: read-only range request "key:\"/registry/services/endpoints/kube-system/kube-controller-manager\" " with result "range_response_count:1 size:447" took too long (431.157766ms) to execute
> 2020-02-13 14:48:04.734246 W | etcdserver: read-only range request "key:\"/registry/minions/test03\" " with result "range_response_count:0 size:5" took too long (1.05179271s) to execute
> 2020-02-13 14:48:04.734349 W | etcdserver: read-only range request "key:\"/registry/pods/\" range_end:\"/registry/pods0\" limit:500 " with result "range_response_count:8 size:18170" took too long (292.020624ms) to execute
> ```

> 问题4、api-server报错
>
> ```
> I0213 16:31:39.167022       1 log.go:172] http: TLS handshake error from 192.168.0.10:59610: remote error: tls: bad certificate
> ```

> 问题5、kube-scheduler报错
>
> ```
> User "system:kube-scheduler" cannot list resource "replicationcontrollers" in API group "" at the cluster scope
> ```

> 问题6、kubectl get cs问题（上游问题，影响版本从1.16之后。）
>
> ```
> [root@test01 ~]# kubectl get cs
> NAME                 AGE
> scheduler            <unknown>
> controller-manager   <unknown>
> etcd-0               <unknown>
> #workround方案：
> [root@test01 ~]# kubectl get cs -o=go-template='{{printf "|NAME|STATUS|MESSAGE|\n"}}{{range .items}}{{$name := .metadata.name}}{{range .conditions}}{{printf "|%s|%s|%s|\n" $name .status .message}}{{end}}{{end}}'
> |NAME|STATUS|MESSAGE|
> |controller-manager|True|ok|
> |scheduler|True|ok|
> |etcd-0|True|{"health":"true"}|
> ```

参考链接：
https://cloud.tencent.com/developer/article/1509412
https://www.kubernetes.org.cn/6632.html