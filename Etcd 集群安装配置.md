# Etcd 集群安装配置
在 Kubernetes 集群中不论是 k8s 还是 flannel 都依赖于 etcd 来存储容器的状态和网络的分配，因此在构建 flannel 网络和 k8s 集群之前应该先配置好 etcd 集群。

| IP地址 | 主机名 | 角色 |
| :-: | :-: | :-: |
| 10.100.4.211 | k8s-node01-bjqw.bd-yg.com | etcd01 |
| 10.100.4.212 | k8s-node02-bjqw.bd-yg.com | etcd02 |
| 10.100.4.214 | office-bjqw.bd-yg.com | etcd03 |

#### 1.1 配置三台服务器 hosts 解析

```shell
# vim /etc/hosts
10.100.4.211 k8s-node01-bjqw.bd-yg.com etcd01
10.100.4.212 k8s-node02-bjqw.bd-yg.com etcd02
10.100.4.214 k8s-master-bjqw.bd-yg.com etcd03
```
#### 1.2 安装 etcd 

```shell
# yum -y install etcd
```
#### 1.3 修改 etcd 配置文件
修改 10.100.4.211 服务器

```shell
# cd /etc/etcd/
# cp etcd.conf{,.bak}
# vim etcd.conf
ETCD_DATA_DIR="/data/etcd/etcd01.etcd"
ETCD_LISTEN_PEER_URLS="http://10.100.4.211:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001"
ETCD_NAME="etcd1"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.100.4.211:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://10.100.4.211:2379,http://10.100.4.211:4001"
ETCD_INITIAL_CLUSTER="etcd1=http://etcd01:2380,etcd2=http://etcd02:2380,etcd3=http://etcd03:2380"
ETCD_INITIAL_CLUSTER_TOKEN="bdyg-etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```
修改 10.100.4.212 服务器

```shell
# cd /etc/etcd/
# cp etcd.conf{,.bak}
# vim etcd.conf
ETCD_DATA_DIR="/data/etcd/etcd02.etcd"
ETCD_LISTEN_PEER_URLS="http://10.100.4.212:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001"
ETCD_NAME="etcd2"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.100.4.212:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://10.100.4.212:2379,http://10.100.4.212:4001"
ETCD_INITIAL_CLUSTER="etcd1=http://etcd01:2380,etcd2=http://etcd02:2380,etcd3=http://etcd03:2380"
ETCD_INITIAL_CLUSTER_TOKEN="bdyg-etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```
修改 10.100.4.214 服务器

```shell
# cd /etc/etcd/
# cp etcd.conf{,.bak}
# vim etcd.conf
ETCD_DATA_DIR="/data/etcd/etcd03.etcd"
ETCD_LISTEN_PEER_URLS="http://10.100.4.214:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001"
ETCD_NAME="etcd3"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.100.4.214:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://10.100.4.214:2379,http://10.100.4.214:4001"
ETCD_INITIAL_CLUSTER="etcd1=http://etcd01:2380,etcd2=http://etcd02:2380,etcd3=http://etcd03:2380"
ETCD_INITIAL_CLUSTER_TOKEN="bdyg-etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```
#### 1.4 启动 etcd 服务
启动 etcd 时要按照顺序启动先启动 etcd03 然后依次是 etcd02 、etcd01。

```shell
root@office-bjqw:~ # systemctl enable etcd.service
root@office-bjqw:~ # systemctl start etcd.service
```
#### 1.5 etcd 配置文件说明

```shell
# etcd 数据存储目录
ETCD_DATA_DIR="/data/etcd/etcd03.etcd"
# 集群内部通信使用的 URL
ETCD_LISTEN_PEER_URLS="http://10.100.4.214:2380"
# 供外部客户端使用的 URL
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001"
# etcd 实例名称
ETCD_NAME="etcd3" 
# 广播给集群内其它成员使用的 URL
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.100.4.214:2380"
# 广播给外部客户端使用的 URL
ETCD_ADVERTISE_CLIENT_URLS="http://10.100.4.214:2379,http://10.100.4.214:4001"
# 初始集群成员列表
ETCD_INITIAL_CLUSTER="etcd1=http://etcd01:2380,etcd2=http://etcd02:2380,etcd3=http://etcd03:2380"
# 集群的名称
ETCD_INITIAL_CLUSTER_TOKEN="bdyg-etcd-cluster"
# 初始集群状态
ETCD_INITIAL_CLUSTER_STATE="new"
```


