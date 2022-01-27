#### 使用socket连接一个无效IP时成功
在使用sidecar注入时发现,在进行ip探测时候,采用icmp协议探测失败(对端可能禁ping)后采用java,InetAddress.getByName(ip).isReachable()方法，默认实现采用的tcp方式:telnet ip 7判断是否返回rst包。

 [github issue地址](https://github.com/istio/istio/issues/34739)
 
 验证: 在注入sidecar容器内进行telnet ip 端口；
 ```
 [zpaas@zcm-portal-202211291716-7bf4559f9c-jfxj6 ~]$ telnet 10.50.33.22 22
 Trying 10.50.33.22...
 Connected to 10.50.33.22.
 Escape character is '^]'.
 ```

```
Every 3.0s: netstat -anp|grep 10.50.33.22                                                                                                                                              Fri Jan 14 11:36:50 2022

tcp        0      1 172.23.235.165:41422    10.50.33.22:22          SYN_SENT    -
tcp        0      0 172.23.235.165:41420    10.50.33.22:22          ESTABLISHED 82276/telnet
```

实质先跟envoy已经建立了链接,envoy直接返回链接正常建立。
官方提供解决方案: 官方支持通过配置ip段,在alpha版本提供端口段配置，支持绕过envoy代理。

具体配置:

```
# 编辑deploy资源。
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      ...
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      annotations:
        proxy.istio.io/config: |
          concurrency: "16"
        sidecar.istio.io/inject: "true"
        traffic.sidecar.istio.io/excludeOutboundPorts: "7"

```