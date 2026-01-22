##### 1.13 ipvs部署(功能配置)
- 在Kubernetes中Service有两种带来模型，一种是基于iptables的，一种是基于ipvs的两者比较的话，ipvs的性能明显要高一些，但是如果要使用它，需要手动载入ipvs模块
- 下面操作在所有节点上都要做:
```shell
# 1.安装ipset和ipvsadm
[root@master ~]# yum install ipset ipvsadm -y
# 2.添加需要加载的模块写入脚本文件
[root@master ~]# cat <<EOF> /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
EOF
# 3.为脚本添加执行权限
[root@master ~]# chmod +x /etc/sysconfig/modules/ipvs.modules
# 4.执行脚本文件
[root@master ~]# /bin/bash /etc/sysconfig/modules/ipvs.modules
# 5.查看对应的模块是否加载成功
[root@master ~]# lsmod | grep -e ip_vs -e nf_conntrack
```
- 下面操作只要在主节点上做:
```shell
# 1.编辑kube-proxy配置文件： 
[root@master ~]# kubectl edit configmap -n kube-system kube-proxy
# 2.将配置文件中mode=""参数改为mode="ipvs",配置完成后保存重启。修改如下图：
```

![imgx](../pics/ipvs.png)

```shell
# 3.查看当前的kube-proxy组件
[root@master ~]# kubectl get pod -n kube-system | grep proxy
kube-proxy-b4n8g                 1/1     Running   0          17m
kube-proxy-b652f                 1/1     Running   0          16m
kube-proxy-cthmc                 1/1     Running   0          16m
# 4.重启kube-proxy，因为kube-proxy有期望副本数，所以这里直接删除后集群会自动拉起新的pod，达到重启的效果
[root@master ~]# kubectl delete  pod -n kube-system kube-proxy-b4n8g
[root@master ~]# kubectl delete  pod -n kube-system kube-proxy-b652f
[root@master ~]# kubectl delete  pod -n kube-system kube-proxy-cthmc
# 5.然后使用ipvsadm命令查看是否成功
[root@master ~]# ipvsadm -ln    #有输出表示成功切换为ipvs
```


