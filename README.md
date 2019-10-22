# heapster安装

## 安装文件和镜像下载

**安装文件下载：**

```bash
[root@master ~]# git clone https://github.com/kubernetes-retired/heapster.git
```

或者

```bash
[root@master ~]# wget https://github.com/kubernetes-retired/heapster/archive/master.zip
[root@master ~]# unzip master.zip 
```

两种方式都可以下载安装文件，本文采取第二种方式

**镜像下载及打标签**

```bash
[root@node02 ~]# docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-grafana-amd64:v5.0.4
[root@node02 ~]# docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-influxdb-amd64:v1.5.2
[root@node02 ~]# docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-amd64:v1.5.4

[root@node02 ~]# docker image tag registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-amd64:v1.5.4 k8s.gcr.io/heapster-amd64:v1.5.4 
[root@node02 ~]# docker image tag registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-influxdb-amd64:v1.5.2 k8s.gcr.io/heapster-influxdb-amd64:v1.5.2
[root@node02 ~]# docker image tag registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-grafana-amd64:v5.0.4 k8s.gcr.io/heapster-grafana-amd64:v5.0.4

[root@node02 ~]# docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-amd64:v1.5.4 registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-influxdb-amd64:v1.5.2 registry.cn-hangzhou.aliyuncs.com/google_containers/heapster-grafana-amd64:v5.0.4
```

**注意**：`每个节点都需执行以上命令`

## 安装文件修改

```bash
[root@master ~]# cd heapster-master/deploy/kube-config/
[root@master kube-config]# pwd
/root/heapster-master/deploy/kube-config
[root@master kube-config]# ll
总用量 0
drwxr-xr-x 2 root root  27 11月 30 2018 google
drwxr-xr-x 2 root root  68 10月 22 15:00 influxdb
drwxr-xr-x 2 root root  32 10月 22 15:02 rbac
drwxr-xr-x 2 root root  38 11月 30 2018 standalone
drwxr-xr-x 2 root root 170 11月 30 2018 standalone-test
drwxr-xr-x 2 root root 145 11月 30 2018 standalone-with-apiserver
[root@master kube-config]# cd influxdb/
[root@master influxdb]# ll
总用量 12
-rw-r--r-- 1 root root 2294 10月 22 14:51 grafana.yaml
-rw-r--r-- 1 root root 1162 10月 22 15:00 heapster.yaml
-rw-r--r-- 1 root root  997 10月 22 14:51 influxdb.yaml
[root@master influxdb]# cd ../rbac/
[root@master rbac]# ll
总用量 4
-rw-r--r-- 1 root root 263 10月 22 15:02 heapster-rbac.yaml
```

分别修改文件**grafana.yaml**、**influxdb.yaml**、**heapster.yaml**和**heapster-rbac.yaml**

![图片.png](https://ask.qcloudimg.com/draft/6211241/r50k4yir43.png)

修改**grafana.yaml**，port类型为NodePort，nodePort为30011，可通过http://NodeIp:30011方式访问

![图片.png](https://ask.qcloudimg.com/draft/6211241/fljetjrny7.png)

修改**influxdb.yaml**，port类型为NodePort，nodePort为30012，grafana配置数据源会用到

![图片.png](https://ask.qcloudimg.com/draft/6211241/e0mvhnje8s.png)

修改**heapster.yaml**中的source和sink参数

**source**: 指定数据获取源

| source参数        | 说明                                                         |
| :---------------- | :----------------------------------------------------------- |
| inClusterConfig   | 在与heapster的命名空间关联的服务帐户中使用kube config(默认值：true) |
| kubeletPort       | 指定kubelet的使用端口，默认10255                             |
| kubeletHttps      | 是否使用https去连接kubelets(默认：false)                     |
| insecure          | 是否使用安全证书(默认：false)                                |
| auth              | 安全认证                                                     |
| useServiceAccount | 是否使用K8S的安全令牌(默认：false)                           |

**sink**: 指定后端数据存储

| sink参数                | 说明                                              |
| :---------------------- | :------------------------------------------------ |
| user                    | InfluxDB用户,默认root                             |
| pw                      | InfluxDB密码，默认root                            |
| db                      | 数据库名，默认k8s                                 |
| retention               | 默认infloxDB保留策略的持续时间，默认值0，表示无限 |
| secure                  | 安全连接到InfluxDB(默认：false)                   |
| insecuressl             | 忽略SSL证书有效性(默认值：false)                  |
| withfields              | 使用InfluxDB fields(默认：false)                  |
| cluster_name            | 不同cubernete集群的集群名称(默认：default)        |
| disable_counter_metrics | 禁用接收计数器度量以流入数据库(默认：false)       |
| concurrency             | 并发数(默认：1)                                   |

![图片.png](https://ask.qcloudimg.com/draft/6211241/ld32unhogg.png)

修改**heapster-rbac.yaml**，将权限修改为cluster-admi

## 执行安装

```bash
[root@master kube-config]# pwd
/root/heapster-master/deploy/kube-config
[root@master kube-config]# kubectl apply -f influxdb/
deployment.extensions/monitoring-grafana created
service/monitoring-grafana created
serviceaccount/heapster created
deployment.extensions/heapster created
service/heapster created
deployment.extensions/monitoring-influxdb created
service/monitoring-influxdb created
[root@master kube-config]# kubectl apply -f rbac/heapster-rbac.yaml 
clusterrolebinding.rbac.authorization.k8s.io/heapster created
```

## 资源查看

```bash
[root@master kube-config]# kubectl get all -n kube-system -o wide |grep -e monitor -e heapster    
```

![图片.png](https://ask.qcloudimg.com/draft/6211241/zn6qrd07p.png)

# Grafana配置

## 登录grafana

登陆地址： [http://172.27.9.131:30011](http://172.27.9.131:30011/) 

![图片.png](https://ask.qcloudimg.com/draft/6211241/kva9a6ioez.png)

## 配置DataSource

![图片.png](https://ask.qcloudimg.com/draft/6211241/bk1abv6g3b.png)

![图片.png](https://ask.qcloudimg.com/draft/6211241/60l3mtkm4q.png)

![图片.png](https://ask.qcloudimg.com/draft/6211241/ivmwthdbxc.png)

url为http://172.27.9.131:30012

## 导入模板

**模板下载**

下载地址：https://grafana.com/api/dashboards/3649/revisions/1/download 、

https://grafana.com/api/dashboards/3646/revisions/1/download



**导入**

![图片.png](https://ask.qcloudimg.com/draft/6211241/073y4dkuog.png)

![图片.png](https://ask.qcloudimg.com/draft/6211241/ljs3d2w5cu.png)

![图片.png](https://ask.qcloudimg.com/draft/6211241/v0e692dnwe.png)

同理导入kubernetes-node-statistics

# 查看Grafana

![图片.png](https://ask.qcloudimg.com/draft/6211241/j6v1g1pc9x.png)

![图片.png](https://ask.qcloudimg.com/draft/6211241/v7xa3ni5mg.png)



# 资源删除

```bash
[root@master ~]# kubectl delete -n kube-system  ClusterRoleBinding heapster               
[root@master ~]# kubectl get all -n kube-system -o wide |grep -e monitor -e heapster |awk '{print $1}'|xargs kubectl delete  -n kube-system
[root@master ~]# rm -rf heapster-master master.zip 
```

![图片.png](https://ask.qcloudimg.com/draft/6211241/5z375x9z7g.png)
