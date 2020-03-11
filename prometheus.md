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

在被监控端需要装一个名为 `node_exporter` 的导出器，他会帮你收集系统指标和一些软件运行的指标，把指标暴露出去，这样 `prometheus` 就可以去采集了，具体 `node_exporter` 能采集哪些东西，看官方吧，还是蛮多的，现在随便找个服务器下载一下 `node_exporter` 运行起来就行了。
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

出现上述情况说明`node_exporter`已经正常启动了，这里要注意下`node`节点与`Prometheus`的通信问题，也可以在`Prometheus`的web上查看这个信息，顺便查下数据
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
![alt text](https://res.rj-bai.com/201907/20190709163321.png?imageView2/2/w/1920/q/75)
<br>

直接导入一个仪表盘进来吧，ID 是 9276
