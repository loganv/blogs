# K8s 常见问题及解决方案

1. 我已经通过k8s官方提供的解决方案安装的docker，并且docker可以成功运行。 启动minikube的时候出现的问题 

```bash
xiaoqu@k8s2:~$ sudo minikube start --driver=none
[sudo] password for xiaoqu:
Sorry, try again.
[sudo] password for xiaoqu:
😄  minikube v1.12.1 on Ubuntu 16.04
✨  Using the none driver based on user configuration
💣  Sorry, Kubernetes 1.18.3 requires conntrack to be installed in root's path
```

> Sorry, Kubernetes 1.18.3 requires conntrack to be installed in root's path

之前在另一台虚拟机上安装minikube 就没有出现这个问题。

解决方法：

安装`conntract`，之后在此尝试启动minikube。

```shell
 sudo apt-get install conntract -y
```
参考 ：
[https://github.com/kubernetes/minikube/issues/7179](https://github.com/kubernetes/minikube/issues/7179)


----   


2. docker 的Cgroup driver 改为 systemd 

```
# (Install Docker CE)
## Set up the repository:
### Install packages to allow apt to use a repository over HTTPS
apt-get update && apt-get install -y \
  apt-transport-https ca-certificates curl software-properties-common gnupg2
```

```
# Add Docker’s official GPG key:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
```

```
# Add the Docker apt repository:
add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
```

```
apt-get update && apt-get install -y \
  containerd.io=1.2.13-2 \
  docker-ce=5:19.03.11~3-0~ubuntu-$(lsb_release -cs) \
  docker-ce-cli=5:19.03.11~3-0~ubuntu-$(lsb_release -cs)
```

##### 重点来了

```
# Set up the Docker daemon
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"], ## 这里改为了systemd
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

```
mkdir -p /etc/systemd/system/docker.service.d

# Restart Docker
systemctl daemon-reload
systemctl restart docker

# enable start on boot
sudo systemctl enable docker

```

参考： 
https://kubernetes.io/zh/docs/setup/production-environment/container-runtimes/

--- 

sudo kubeadm init 出现下列错误   

> [ERROR Swap]: running with swap on is not supported. Please disable swap.

解决方法：

```
sudo swapoff -a
```

参考

[https://github.com/kubernetes/kubeadm/issues/610](https://github.com/kubernetes/kubeadm/issues/610)

--- 

问题：

``` 
sudo kubelet 
```

> ] failed to run Kubelet: misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"


解决 
目前也没有找到正确的姿势，唯一的方式就是重新安装docker 和 kubectl

---  

### 问题： k8s 集群搭好了 `kubectl get nodes` role是空的，如何指定节点的role

```bash
root@k8s2:~# kubectl get nodes
NAME   STATUS   ROLES    AGE   VERSION
k8s1   Ready    <none>   14h   v1.18.6
k8s2   Ready    master   15h   v1.18.6
```

答案： 
 暂时没记录 。

---

### 问题 如何查看K8s集群的版本号

答案：我是用kube admin 构建的k8s集群，用minikube构建的可能会不一样

> kubeadm version

```bash
root@k8s2:~/k8s# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.6", GitCommit:"dff82dc0de47299ab66c83c626e08b245ab19037", GitTreeState:"clean", BuildDate:"2020-07-15T16:56:34Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```

这里是1.18.6


### 如何删除加入集群中的节点

下线节点

```bash
root@k8s2:~# kubectl drain k8s1 --ignore-daemonsets --delete-local-data
node/k8s1 already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-bw8mn
node/k8s1 drained
```

确认下线

```bash 
root@k8s2:~# kubectl get nodes
NAME   STATUS                        ROLES    AGE   VERSION
k8s1   NotReady,SchedulingDisabled   <none>   23h   v1.18.6
k8s2   Ready                         master   23h   v1.18.6
```

删除节点

```bash
root@k8s2:~# kubectl delete node k8s1
node "k8s1" deleted
root@k8s2:~# kubectl get nodes
NAME   STATUS   ROLES    AGE   VERSION
k8s2   Ready    master   23h   v1.18.6
root@k8s2:~#
```

删除成功