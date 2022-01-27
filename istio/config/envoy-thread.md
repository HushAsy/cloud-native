## 线程数调节
envoy通过concurrency参数调节工作线程数
默认线程数查看:
```
root@10.10.183.11[/root]# kubectl exec -it zcm-service-2022111205314-6d5f955478-f64qc -n zcm9 -c istio-proxy bash
istio-proxy@zcm-service-2022111205314-6d5f955478-f64qc:/$ ps -ef|cat
UID         PID   PPID  C STIME TTY          TIME CMD
istio-p+      1      0  0 Jan11 ?        00:05:25 /usr/local/bin/pilot-agent proxy sidecar --domain zcm9.svc.cluster.local --proxyLogLevel=warning --proxyComponentLogLevel=misc:error --log_output_level=default:info --concurrency 2
istio-p+     31      1  0 Jan11 ?        00:10:21 /usr/local/bin/envoy -c etc/istio/proxy/envoy-rev0.json --restart-epoch 0 --drain-time-s 45 --drain-strategy immediate --parent-shutdown-time-s 60 --local-address-ip-version v4 --file-flush-interval-msec 1000 --disable-hot-restart --log-format %Y-%m-%dT%T.%fZ.%l.envoy %n.%v -l warning --component-log-level misc:error --concurrency 2
istio-p+     52      0  0 02:41 pts/0    00:00:00 bash
istio-p+     59     52  0 02:41 pts/0    00:00:00 ps -ef
istio-p+     60     52  0 02:41 pts/0    00:00:00 cat
```

实际生效配置,从pilot下发:
```
curl -X POST localhost:15000/config_dump|more
 "configs": [
  {
   "@type": "type.googleapis.com/envoy.admin.v3.BootstrapConfigDump",
   "bootstrap": {
    "node": {
     "id": "sidecar~172.23.235.252~zcm-service-2022111205314-6d5f955478-f64qc.zcm9~zcm9.svc.cluster.local",
     "cluster": "zcm-service-2022111205314.zcm9",
     "metadata": {
      "OWNER": "kubernetes://apis/apps/v1/namespaces/zcm9/deployments/zcm-service-2022111205314",
      "ISTIO_VERSION": "1.12.1",
      "PROV_CERT": "var/run/secrets/istio/root-cert.pem",
      "ENVOY_PROMETHEUS_PORT": 15090,
      "NAMESPACE": "zcm9",
      "INSTANCE_IPS": "172.23.235.252",
      "LABELS": {
       "pod-template-hash": "6d5f955478",
       "app.kubernetes.io/name": "zcm-service-2022111205314",
       "security.istio.io/tlsMode": "istio",
       "app.kubernetes.io/instance": "zcm-iplatform",
       "service.istio.io/canonical-revision": "latest",
       "service.istio.io/canonical-name": "zcm-service-2022111205314",
       "zcm-app": "zcm-service-2022111205314"
      },
      "ANNOTATIONS": {
       "kubectl.kubernetes.io/default-container": "zcm-service",
       "kubernetes.io/config.source": "api",
       "prometheus.io/port": "15020",
       "kubernetes.io/config.seen": "2022-01-11T20:53:17.191289640+08:00",
       "prometheus.io/scrape": "true",
       "kubectl.kubernetes.io/default-logs-container": "zcm-service",
       "prometheus.io/path": "/stats/prometheus",
       "sidecar.istio.io/status": "{\"initContainers\":[\"istio-init\"],\"containers\":[\"istio-proxy\"],\"volumes\":[\"istio-envoy\",\"istio-data\",\"istio-podi
nfo\",\"istiod-ca-cert\"],\"imagePullSecrets\":null,\"revision\":\"default\"}"
      },
      ......
 "statNameLength": 189,
       "statusPort": 15020,
       "controlPlaneAuthPolicy": "MUTUAL_TLS",
       "serviceCluster": "istio-proxy",
       "concurrency": 2,
```

调整workload即单个服务的工作线程数: 从默认线程数2改为4
[配置链接](https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
    meta.helm.sh/release-name: zcm-iplatform
    meta.helm.sh/release-namespace: default
  creationTimestamp: "2022-01-11T12:53:17Z"
  generation: 2
  labels:
    app.kubernetes.io/instance: zcm-iplatform
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: zcm-service
    app.kubernetes.io/version: 1.16.0
    helm.sh/chart: zcm-service-0.1.0
    zcm-app: zcm-service-2022111205314
  name: zcm-service-2022111205314
  namespace: zcm9
  resourceVersion: "387539560"
  selfLink: /apis/apps/v1/namespaces/zcm9/deployments/zcm-service-2022111205314
  uid: 6ab229af-0aa8-4aae-8190-23290e69c429
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/instance: zcm-iplatform
      app.kubernetes.io/name: zcm-service-2022111205314
      zcm-app: zcm-service-2022111205314
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      annotations:
        proxy.istio.io/config: |
          concurrency: "4"
```
