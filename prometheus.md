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
