#### K8S调整

配置apiserver添加参数来支持第三方令牌（Third-party-jwt）

在/etc/kubernetes/manifests/kube-apiserver.yaml里添加：  
```
- --service-account-signing-key-file=/etc/kubernetes/pki/sa.key  
- --service-account-key-file=/etc/kubernetes/pki/sa.pub  
- --service-account-issuer=api  
- --service-account-api-audiences=api,vault,factors  
- --service-node-port-range=25000-52999
```
#### ISTIO部署
[配置参考](https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/)
```
#istio安装包下载  
curl -L https://istio.io/downloadIstio | sh -  
​  
#helm部署istio基础组件  
#cd 到istio部署包目录下  
kubectl create namespace istio-system  
helm install istio-base manifests/charts/base -n istio-system  


#设置额外参数--set meshConfig.ingressService=ingress-nginx 

helm install istiod manifests/charts/istio-control/istio-discovery \      
                                --set global.hub="docker.io/istio" \      
                                --set global.tag="1.12.1" \      
                                -n istio-system
```



