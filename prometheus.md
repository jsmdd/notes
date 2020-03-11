# Prometheus+Grafana #
> by yassor 20.3.11
<br>

## prometheus的概述 ##
<br>

**略**

<br>

## Prometheus 部署 ##
<br>

这里部署的服务是由两台云服务器:`ip-A`，`ip-B`以及一台本地虚拟机`test1`完成的，后续根据需求来改变架构
<br>

直接从官网下载tar包，[官网地址](https://github.com/prometheus/prometheus/releases/download)
```shell
[root@test1 ~]# wget https://github.com/prometheus/prometheus/releases/download/v2.10.0/prometheus-2.10.0.linux-amd64.tar.gz
[root@test1 ~]# tar zxf prometheus-2.10.0.linux-amd64.tar.gz 
[root@test1 ~]# mv prometheus-2.10.0.linux-amd64 /usr/local/prometheus
```
<br>

`这里要说下，默认的安装目录这里是/usr/local/，根据具体需要去调整`
<br>

`prometheus`默认的配置文件是`prometheus.yml`
```shell
[root@test1 ~]# cd /usr/local/prometheus/ 
[root@test1 /usr/local/prometheus]# ll
total 117908
drwxr-xr-x. 2 3434 3434       38 May 25  2019 console_libraries
drwxr-xr-x. 2 3434 3434      173 May 25  2019 consoles
drwxr-xr-x. 2 root root       25 Mar  4 21:26 files_sd_configs
-rw-r--r--. 1 3434 3434    11357 May 25  2019 LICENSE
-rw-r--r--. 1 3434 3434     2770 May 25  2019 NOTICE
-rwxr-xr-x. 1 3434 3434 74561203 May 25  2019 prometheus
-rw-r--r--  1 3434 3434     1616 Mar  8 22:37 prometheus.yml
-rwxr-xr-x. 1 3434 3434 46152675 May 25  2019 promtool
```
<br>

默认的配置文件已经配置好监控本机Prometheus了，就是这段
```shell
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
```
> 至于`prometheus.yml`配置文件的用法参考下[官网手册](https://prometheus.io/docs/introduction/overview/),里面有详细的说明，这里就不再说明
<br>

`Prometheus`服务已经安装好了，接下来建议把`Prometheus`交给`systemctl`去管理
```shell
[root@test1 /usr/local/prometheus]# cat /usr/lib/systemd/system/prometheus.service 
[Unit]
Description=prometheus server daemon

[Service]
Restart=on-failure
ExecStart=/usr/local/prometheus/prometheus --config.file=/usr/local/prometheus/prometheus.yml

[Install]
WantedBy=multi-user.target
[root@test1 /usr/local/prometheus]#systemctl daemon-reload && systemctl start prometheus.service
```
<br>

这样就启动服务了，访问下看看
![alt text](https://res.rj-bai.com/201907/20190702163712.png?imageView2/2/w/1920/q/75)
<br>

这页面能看到的东西很多，自己点点看下，`Status`下的`Targets`能看到你所监控的项目的信息,`Alerts`是报警规则，`Graph`是使用`Promsql`去查看的
<br>

### 配置文件的介绍 ###
<br>

贴下我的配置文件
```shell
[root@test1 /usr/local/prometheus]# cat prometheus.yml 
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
       - 192.168.159.100:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "rules/*.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'master'
    file_sd_configs:
      - files: ['/usr/local/prometheus/files_sd_configs/*.yml']
        refresh_interval: 3s

  - job_name: 'nodes'
    file_sd_configs:
      - files: ['/usr/local/prometheus/nodes_sd_configs/*.yml']
        refresh_interval: 3s

  - job_name: 'mysql'
    file_sd_configs:
      - files: ['/usr/local/prometheus/mysql_sd_configs/*.yml']
        refresh_interval: 3s
    
  - job_name: 'nginx'
    file_sd_configs:
      - files: ['/usr/local/prometheus/nginx_sd_configs/*.yml']
        refresh_interval: 3s
```
<br>

我这里是把每一个要监控的项目，分别建立了子目录去管理
```shell
[root@test1 /usr/local/prometheus]# ll
total 117908
drwxr-xr-x. 2 3434 3434       38 May 25  2019 console_libraries
drwxr-xr-x. 2 3434 3434      173 May 25  2019 consoles
drwxr-xr-x. 2 root root       25 Mar  4 21:26 files_sd_configs
-rw-r--r--. 1 3434 3434    11357 May 25  2019 LICENSE
drwxr-xr-x  2 root root       23 Mar  8 22:38 mysql_sd_configs
drwxr-xr-x  2 root root       23 Mar  9 14:33 nginx_sd_configs
drwxr-xr-x. 2 root root       23 Mar  7 16:54 nodes_sd_configs
-rw-r--r--. 1 3434 3434     2770 May 25  2019 NOTICE
-rwxr-xr-x. 1 3434 3434 74561203 May 25  2019 prometheus
-rw-r--r--  1 3434 3434     1616 Mar  8 22:37 prometheus.yml
-rwxr-xr-x. 1 3434 3434 46152675 May 25  2019 promtool
drwxr-xr-x  2 root root       29 Mar  9 17:43 rules
[root@test1 /usr/local/prometheus]# cat files_sd_configs/configs.yml 
- targets: ['localhost:9090'] 
  labels:
    name: server100
[root@test1 /usr/local/prometheus]# cat nodes_sd_configs/nodes.yml 
- targets: ['192.168.159.100:9100'] 
  labels:
    name: server100
- targets: ['IP-A:9100'] 
  labels:
    name: server001
- targets: ['IP-B:9100'] 
  labels:
    name: server002
[root@test1 /usr/local/prometheus]# cat mysql_sd_configs/mysql.yml 
- targets: ['IP-A:9104'] 
  labels:
    name: server001
- targets: ['IP-B:9104'] 
  labels:
    name: server002
[root@test1 /usr/local/prometheus]# cat nginx_sd_configs/nginx.yml 
- targets: ['IP-A:9913'] 
  labels:
    name: server001
- targets: ['IP-B:9913'] 
  labels:
    name: server002
```
<br>

上述只是基于文件的服务发现，还可以基于`kubernetes`的服务发现,这里就不再表述了，好，现在只是完成了`prometheus`服务的部署，要监控服务器的话需要额外的组件
<br>

## 监控服务器 ##
<br>

在被监控端需要装一个名为 `node_exporter` 的导出器，他会帮你收集系统指标和一些软件运行的指标，把指标暴露出去，这样 `prometheus` 就可以去采集了，具体 `node_exporter` 能采集哪些东西，看官方吧，还是蛮多的，现在随便找个服务器下载一下 `node_exporter` 运行起来就行了，启动端口为`9100`。
```shell
[root@IP-A ~]# wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
[root@IP-A ~]# tar zxf node_exporter-0.18.1.linux-amd64.tar.gz 
[root@IP-A ~]# mv node_exporter-0.18.1.linux-amd64 /usr/local/node_exporter
```
<br>

同理，交给`systemctl`去管理
```shell
[root@IP-A /usr/local/node_exporter]# cat /usr/lib/systemd/system/node_exporter.service 
[Unit]
Description=node_exporter

[Service]
Restart=on-failure
ExecStart=/usr/local/node_exporter/node_exporter --collector.systemd --collector.systemd.unit-whitelist=(docker|sshd).service

[Install]
WantedBy=multi-user.target
[root@IP-A /usr/local/node_exporter]# systemctl daemon-reload 
[root@IP-A /usr/local/node_exporter]# systemctl start node_exporter.service
[root@IP-A /usr/local/node_exporter]# curl -s 127.0.0.1:9100/metrics
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 44880    0 44880    0     0  1760k      0 --:--:-- --:--:-- --:--:-- 1826k
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 1.0319e-05
go_gc_duration_seconds{quantile="0.25"} 1.2262e-05
go_gc_duration_seconds{quantile="0.5"} 1.336e-05
go_gc_duration_seconds{quantile="0.75"} 1.4901e-05
go_gc_duration_seconds{quantile="1"} 2.754e-05
go_gc_duration_seconds_sum 0.072176643
go_gc_duration_seconds_count 5176
# HELP go_goroutines Number of goroutines that currently exist.
......
```

出现上述情况说明`node_exporter`已经正常启动了，`IP-B`重复上述操作，这里要注意下`node`节点与`Prometheus`的通信问题，也可以在`Prometheus`的web上查看这个信息，顺便查下数据
![alt text](https://res.rj-bai.com/201907/20190709111843.png?imageView2/2/w/1920/q/75)
![alt text](https://res.rj-bai.com/201907/20190709112040.png?imageView2/2/w/1920/q/75)
<br>

这样就可以了，没问题了，使用下` PromSQL`去查写基本信息
nodes CPU 利用率:
![alt text](https://res.rj-bai.com/201907/20190709112714.png?imageView2/2/w/1920/q/75)
nodes 内存使用率:
![alt text](https://res.rj-bai.com/201907/20190709134504.png?imageView2/2/w/1920/q/75)
等等这些吧，更多的看看官网文档
<br>

现在把这个加到普罗米修斯中，这里就不贴了，看下上面我贴出来的文件，修改完后记得重启`prometheus`
<br>

## 部署 grafana ##
<br>

`grafana` 就是一个图形展示系统，他主要是对度量指标进行分析可视化，他本身不会存储任何数据，他只会展示数据库里面的数据,说白了只是调用，话不多说，直接下载，[官网地址](https://grafana.com/grafana/download)
```shell
[root@test1 ~] wget https://dl.grafana.com/oss/release/grafana-6.6.2-1.x86_64.rpm
[root@test1 ~] yum install grafana-6.6.2-1.x86_64.rpm
[root@test1 ~] systemctl start grafana-server
```
<br>

`grafana`服务默认是3000端口，也可以使用`docker`启动，命令如下
```shell
[root@test1 ~]# docker run \
> -u 0 \
> -d \
> -p 3000:3000  \
> --name=grafana   \
> -v /var/lib/grafana:/var/lib/grafana \
> grafana/grafana
```
<br>

这样服务就启动了，直接访问看下,用户名密码默认 `admin/admin`，初次登陆会让你修改密码，就可以看到主页了，然后直接添加数据源，把 `prometheus`加进去，保存就行了
![alt text](http://www.yassor.xyz:81/photo/3.png)
<br>

直接导入一个仪表盘进来吧，ID 是 9276
![alt text](http://www.yassor.xyz:81/photo/1.png)
输入9276，等几秒就添加数据源了，选择`Prometheus`
![alt text](http://www.yassor.xyz:81/photo/2.png)
<br>

## 监控 mysql ##
<br>

在被监控端需要装一个名为 `mysqld_exporter`的组件，启动端口为`9104`,直接来下载吧
```
[root@IP-A ~]# wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.12.1/mysqld_exporter-0.12.1.linux-amd64.tar.gz
[root@IP-A ~]# tar xf mysqld_exporter-0.12.1.linux-amd64.tar.gz
[root@IP-A ~]# mv mysqld_exporter-0.12.1.linux-amd64 /usr/local/mysqld_exporter
```
<br>

这里的还要`mysql`给`mysqld_exporter`组件授一个用户
```shell
mysql> CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'exporter';
Query OK, 0 rows affected (0.02 sec)

mysql> GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```
<br>

> 这里要注意监控的mysql版本最好是5.7，要不然有些数据显示不出来，还有就是授权的用户要试下能否登陆，要不然查询不到数据的
<br>

接下来就是配置`mysqld_exporter`的配置文件了,再将`mysqld_exporter`交给`systemctl`去管理
```shell
[root@IP-A ~] cat /usr/local/mysqld_exporter/.my.cnf
[client]
user=exporter
password=exporter
port=33000
[root@IP-A /usr/lib/systemd/system]# cat /usr/local/mysqld_exporter/.my.cnf
[client]
user=exporter
password=exporter
port=33000
[root@IP-A /usr/lib/systemd/system]# cat mysqld_exporter.service 
[Unit]
Description=mysqld_exporter

[Service]
Restart=on-failure
ExecStart=/usr/local/mysqld_exporter/mysqld_exporter --config.my-cnf /usr/local/mysqld_exporter/.my.cnf \
  --collect.slave_status \
  --collect.slave_hosts \
  --log.level=error \
  --collect.info_schema.processlist \
  --collect.info_schema.innodb_metrics \
  --collect.info_schema.innodb_tablespaces \
  --collect.info_schema.innodb_cmp \
  --collect.info_schema.innodb_cmpmem 

[Install]
WantedBy=multi-user.targe
```
<br>

>这里有一点还是值得说的，要监控的mysql服务如果启动端口不是默认的3306，是需要如上述加一行port参数的
<br>

这样就好了，现在把这个加到普罗米修斯中，这里就不贴了，看下上面我贴出来的文件，修改完后记得重启`prometheus`,IP-B重复上述操作
<br>

再就是，在Grafana中导入id为7362，看看效果
![alt text](http://www.yassor.xyz:81/photo/4.png)
刚启动的话要等一会才有数据显示出来，主要还是看http://IP-A:9104/metrics 有没有数据
<br>

## 监控nginx ##
<br>

监控nginx有两种方法，一个是在openresty上监控，另一个直接在nginx上监控，
这里就直接在nginx监控吧，前提条件是 nginx 要编译进 nginx-module-vts 模块，先下载吧
```shell
[root@IP-A ~]# wget https://github.com/vozlt/nginx-module-vts/archive/v0.1.18.tar.gz
[root@IP-A ~]# tar xf v0.1.18.tar.gz
```
<br>

在编译nginx的时候加一条`--add-module=PATH/nginx-module-vts-0.1.18`,如果是在已经有nginx的服务器上的，不要`make install`，具体操作，贴一下地址吧,[这里](https://www.jianshu.com/p/695af00ee857),然后就是修改nginx的配置文件了，具体操作不细说，贴个地址，[这里](https://blog.csdn.net/hxpjava1/article/details/80451101)
<br>

接下来就是在被监控端需要装一个名为 `nginx-vts-exporter `的组件，启动端口为`9913`,直接来下载吧
```shell
[root@IP-A ~]# wget https://github.com/hnlq715/nginx-vts-exporter/releases/download/v0.10.3/nginx-vts-exporter-0.10.3.linux-amd64.tar.gz
[root@IP-A ~]# tar xf nginx-vts-exporter-0.10.3.linux-amd64.tar.gz
[root@IP-A ~]# mv nginx-vts-exporter-0.10.3.linux-amd64 /usr/local/nginx-vts-exporter
[root@IP-A ~]# cat /usr/lib/systemd/system/nginx-vts-exporter.service 
[Unit]
Description=nginx-vts-exporter
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/nginx-vts-exporter/nginx-vts-exporter -nginx.scrape_uri=http://127.0.0.1:6666/vts_status/format/json
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
> 我这里nginx启动的是一个6666端口来提供JSON格式的数据
<br>

修改`prometheus`的配置文件，重启服务，在`grafana`导入一个id为`2949`，看看图
![alt text](http://www.yassor.xyz:81/photo/5.png)
<br>

这样就可以了，IP-B重复上述操作
