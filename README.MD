PrometheusAlert 简介
-----------------

PrometheusAlert是开源的运维告警中心消息转发系统,支持主流的监控系统Prometheus,日志系统Graylog和数据可视化系统Grafana发出的预警消息,支持将收到的这些消息发送到钉钉,短信和语音提醒等

--------------------------------------

PrometheusAlert具备如下特性
---------------------
 - 支持多种消息来源,目前主要有prometheus,graylog,grafana
 - 支持多种类型的发送目标,支持钉钉,短信,语音
 - 针对Prometheus增加了告警级别,并且支持按照不同级别发送消息到不同目标对象
 - 简化Prometheus分组配置,支持按照具体消息发送到单个或多个接收方
 - 增加手机号码配置项,和号码自动轮询配置,可固定发送给单一个人告警信息,也可以通过自动轮询的方式发送到多个人员且支持按照不同日期发送到不同人员

--------------------------------------
部署方式
----

PrometheusAlert可以部署在本地和云平台上，支持windows、linux、公有云、私有云、混合云、容器和kubernetes。你可以根据实际场景或需求，选择相应的方式来部署PrometheusAlert：

 - 使用容器部署
```
docker run -d -p 8080:8080 --name prometheusalert-center feiyu563/prometheus-alert:latest
```
 - 在linux系统中部署
```
git clone https://github.com/feiyu563/PrometheusAlert.git
cd PrometheusAlert/example/linux/
./PrometheusAlert #后台运行请执行nohup ./PrometheusAlert &
```
- 在windows系统中运行
```
git clone https://github.com/feiyu563/PrometheusAlert.git
cd PrometheusAlert/example/windows/
双击运行 PrometheusAlert.exe即可
```
- 在kubernetes中运行
```
kubectl app -f https://raw.githubusercontent.com/feiyu563/PrometheusAlert/master/example/kubernetes/PrometheusAlert-Deployment.yaml
```

配置说明
----
--------------------------------------

PrometheusAlert 暂提供以下三个接口,分别对应各自接入端

 - `prometheus接口`  `/prometheus/alert`
 - `grafana接口`     `/grafana/alert`
 - `graylog接口`     `/graylog/alert`
 
--------------------------------------

 **1. Prometheus 接入配置**

在 Prometheus Alertmanager 中启用 Webhook，可参考如下模板：

```
global:
  resolve_timeout: 5m
route:
  group_by: ['instance']
  group_wait: 10m
  group_interval: 10s
  repeat_interval: 10m
  receiver: 'web.hook.prometheusalert'
receivers:
- name: 'web.hook.prometheusalert'
  webhook_configs:
  - url: 'http://[prometheusalert_url]:8080/prometheus/alert'
```

Prometheus Server 的告警rules配置，可参考如下模板：

```
groups:
 1. name: node_alert
  rules:
 2. alert: 主机CPU告警
    expr: node_load1 > 3
    labels:
      severity: warning
    annotations:
      description: "{{ $labels.instance }} CPU load占用过高"  #告警信息
      summary: "{{ $labels.instance }} CPU load占用过高已经恢复"  #告警恢复信息
      level: 3   #告警级别,告警级别定义 0 信息,1 警告,2 一般严重,3 严重,4 灾难
      mobile: 15888888881,15888888882,15888888883  #告警发送目标手机号(需要设置电话和短信告警级别)
      ddurl: "https://oapi.dingtalk.com/robot/send?access_token=f259e295a8f63c93b44498c53aba170882f948b2e967ac6027,https://oapi.dingtalk.com/robot/send?access_token=914129ff5cd8199072773d736c6bbfabe421f19a9af9511374d687ee0" #支持添加多个钉钉告警,用,号分割即可,如果留空或者未填写,则默认发送到配置文件中填写的钉钉地址
```
最终告警效果:

![prometheus1](doc/prometheus.png)

--------------------------------------
 **2. Grafana 接入配置**
 
打开grafana管理页面,登录后进入notification channels配置

![grafana1](doc/addchannel.png)
注意这里的url地址填写上自己部署所在的url
![grafana2](doc/addchannel2.png)
配置完成后保存即可.继续进行告警消息配置,选择任意一个折线图,点击编辑,进入aler配置,配置参考下图:
![grafana3](doc/grafanaalert1.png)
![grafana4](doc/grafanaalert2.png)

Notifications配置格式参考,支持配置多个钉钉机器人url:
```
告警消息内容&&ddurl[钉钉机器人url,钉钉机器人url....]
```

最终告警效果:

![grafana5](doc/grafana.png)

--------------------------------------

 **3. Graylog 接入配置**
打开Graylog管理页面,登录后进入Alerts配置

![graylog1](doc/graylog1.png)
点击```Add new notification```创建新的告警通道,选择如下图配置:
![graylog2](doc/graylog2.png)
在弹出的窗口中填入名称和对应的PrometheusAlert的地址即可,配置参考下图:
![graylog3](doc/graylog3.png)
配置完成后,点击```Test```测试下是否能够正常接收告警消息即可

最终告警效果:

![graylog4](doc/graylog4.png)

--------------------------------------

**4. 配置文件解析**

短信告警和语音告警均使用的是腾讯云的短信和语音提醒接口,具体参数获取可去腾讯云开通相关服务即可

```
appname = PrometheusAlert
#监听端口
httpport = 8080
runmode = dev
#开启JSON请求
copyrequestbody = true
#钉钉机器人地址
ddurl=https://oapi.dingtalk.com/robot/send?access_token=914129ff5cd8199072773d736c6bbfabe421f19a9af9
#告警消息标题
title=NB云平台
#点击告警消息后链接到告警平台地址
alerturl=http://prometheus.haima.me
#告警消息中显示的logo图标地址
logourl=https://raw.githubusercontent.com/feiyu563/PrometheusAlert/master/doc/haima.png
#腾讯短信接口key
appkey=e710521dad8e545d4df964513f
#腾讯短信模版ID
tpl_id=14386
#腾讯短信sdk app id
sdkappid=1400027785
#短信告警级别(等于3就进行短信告警) 告警级别定义 0 信息,1 警告,2 一般严重,3 严重,4 灾难
messagelevel=3
#腾讯电话接口key
phonecallappkey=c24bbacb6bea9d3ab306475f55
#腾讯电话模版ID
phonecalltpl_id=2920
#腾讯电话sdk app id
phonecallsdkappid=1400191
#电话告警级别(等于4就进行语音告警) 告警级别定义 0 信息,1 警告,2 一般严重,3 严重,4 灾难
phonecalllevel=4
#默认拨打号码,默认不配置,如果配置了此项,那么按照user.csv文件轮询的方式将自动失效
#defaultphone=
```

另外 PrometheusAlert 同时支持按照日期发送告警到不同号码,只需新建user.csv文件,并将文件放到程序运行目录下即可自动加载,csv文件格式如下:
```
2019年4月10日,15888888881,a
2019年4月11日,15888888882,b
2019年4月12日,15888888883,c
2019年4月13日,15888888884,d
2019年4月14日,15888888885,e
2019年4月15日,15888888886,f
2019年4月16日,15888888887,g
2019年4月17日,15888888888,h
2019年4月18日,15888888889,i
2019年4月19日,15888888810,g
2019年4月20日,15888888811,k
2019年4月21日,15888888812,l
2019年4月22日,15888888813,zhangsan
2019年4月23日,15888888814,lilei
```

--------------------------------------


项目源码
----

 - [PrometheusAlert][1]


  [1]: https://github.com/feiyu563/PrometheusAlert

FOR HELP
----
Email: 244217140@qq.com