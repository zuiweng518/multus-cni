# 快速开始指南

这个指南旨在帮助你入门，使用Multus CNI创建多网络接口的Kubernetes pods。如果你已经使用过Multus CNI，需要了解更多细节，请查看how-to-use-ch.md这个文档，这个文档是快速开始指南，旨在帮助你第一次运行Multus CNI。

## 关键概念

* "Default network"--这个是你的POD到POD之间的网络，这个是你的集群中和另外POD通讯


## 要求
我们的安装方法要求你首先已经安装了Kubernets集群并且配置了默认的网络（一个网路插件，主要用于pod之间的网络通讯）。
我们支持的Kubernets社区的版本，请去Kubernets文档[Supported versions](https://kubernetes.io/releases/version-skew-policy/#supported-versions) 查询支持的版本


## 安装


### What the Multus daemonset does


### 验证你的安装


## 创建额外的接口


### CNI配置


### 以CR方式存储配置


### 创建一个含有多个网络接口的POD
我们将要创建一个POD。这和之前创建的POD很相似，但是，我们会有个`annotations`域。在这个案例当中，我们有一个叫做`k8s.v1.cni.cncf.io/networks`的annotations（注解)。

### Network Status Annotations
为了更多确认，使用此命令`kubectl describe pod samplepod`，它会返回一个类似下下面的注解单元：
```
Annotations:        k8s.v1.cni.cncf.io/networks: macvlan-conf
                    k8s.v1.cni.cncf.io/network-status:
                      [{
                          "name": "cbr0",
                          "ips": [
                              "10.244.1.73"
                          ],
                          "default": true,
                          "dns": {}
                      },{
                          "name": "macvlan-conf",
                          "interface": "net1",
                          "ips": [
                              "192.168.1.205"
                          ],
                          "mac": "86:1d:96:ff:55:0d",
                          "dns": {}
                      }]
```
这元数据告诉我们两个CNI网络插件运行成功。
### What if I want more interfaces?
你可以通过更多自定义资源，并把他们指向POD的annotation来创建更多的网络接口。你也可以重用配置，比如为了把2个macvlan附加到POD上，你可以像下边一样创建一个POD：
```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: samplepod
  annotations:
    k8s.v1.cni.cncf.io/networks: macvlan-conf,macvlan-conf
spec:
  containers:
  - name: samplepod
    command: ["/bin/ash", "-c", "trap : TERM INT; sleep infinity & wait"]
    image: alpine
EOF
```

注意：这个注解现在读取`k8s.v1.cni.cncf.io/networks: macvlan-conf,macvlan-conf`,在这里，同样的配置用逗号分隔，使用了2次。
如果你想要用`foo` 这个名字创建另外一个自定义资源，你可以这样使用：`k8s.v1.cni.cncf.io/networks: foo,macvlan-conf`