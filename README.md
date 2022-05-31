# Doraemon_monitoring
Doraemon监控告警


## k8s部署配置

```
1、https://github.com/Qihoo360/doraemon/blob/master/README-CN.md （github源码地址）

2、git clone https://github.com/Qihoo360/doraemon.git  （下载）

3、cd /data/doraemon/deployments/kubernetes/   (进入到该目录下，编辑doraemon.yml)
```
#### doraemon.yml
```

apiVersion: v1
kind: ConfigMap
metadata:
labels:
app: doraemon-api
name: doraemon-api
namespace: default
data:
app.conf: |-
appname = doraemon
httpport = 8080
runmode = dev
autorender = false
copyrequestbody = true
sessionon = true
EnableDocs = true
# mysql
DBName = "doraemon"
DBTns = "tcp(172.28.19.151:3306)"
DBUser = "itsm_sqladmin"
DBPasswd = "-s7$w-_Na8R*xYKjj4=7"
DBLoc = "Asia%2FShanghai"
DBConnTTL = 30

SmsUrl="http://smsurl:8000/api/v1/sms"
LanxinUrl="http://lanxinurl:8000/api/v1/lanxin/text"
CallUrl="http://callurl:8000/api/v1/lanxin/call"
DutyGroupUrl="http://dutygroupurl:8000/Api/getDutyUser"
BrokenUrl="http://brokenurl:8000/api/hosts/broken"
WebUrl="http://localhost:32000"

# oauth2
[auth.oauth2]
auth_url="https://oauth2/v1/authorize"
client_id="xxxxxxxxxxxx"
client_secret="xxxxxxxxxxxx"
token_url = "https://oauth2/v1/token"
api_url ="https://oauth2/v1/userinfo"

# ldap config
[auth.ldap]
enabled = false
ldap_url = ldap://127.0.0.1
ldap_search_dn = "cn=admin,dc=example,dc=com"
ldap_search_password = admin
ldap_base_dn = "dc=example,dc=com"
ldap_filter =
ldap_uid = cn
ldap_scope = 2
ldap_connection_timeout = 30
---

apiVersion: v1
kind: ConfigMap
metadata:
labels:
app: doraemon-ui
name: doraemon-ui
namespace: default
data:
config.js: |-
window.CONFIG = {
baseURL: 'http://doraemon-web.default.svc.hk8s.zhizh.com:80',
};

---

apiVersion: apps/v1
kind: Deployment
metadata:
name: doraemon-web
labels:
app: doraemon-web
namespace: default
spec:
replicas: 1
selector:
matchLabels:
app: doraemon-web
template:
metadata:
labels:
app: doraemon-web
spec:
volumes:
- name: config
configMap:
name: doraemon-api
- name: ui-config
configMap:
name: doraemon-ui
items:
- key: config.js
path: config.js
containers:
- resources:
limits:
memory: 1Gi
cpu: '1'
requests:
memory: 1Gi
cpu: '0.5'
env: []
envFrom: []
imagePullPolicy: Always
name: alertgateway
image: harbor.hk.batmobi.cn/library/alert-gateway:latest
command:
- ./doraemon
volumeMounts:
- name: config
mountPath: /opt/doraemon/conf/
- resources:
limits:
memory: 1Gi
cpu: '1'
requests:
memory: 1Gi
cpu: '0.5'
env: []
envFrom: []
imagePullPolicy: Always
name: alertgateway-fe
image: harbor.hk.batmobi.cn/library/doraemon-frontend:latest
volumeMounts:
- name: ui-config
mountPath: /usr/local/openresty/nginx/html/config.js
subPath: config.js
- resources:
limits:
memory: 1Gi
cpu: '1'
requests:
memory: 1Gi
cpu: '0.5'
env: []
envFrom: []
imagePullPolicy: Always
name: ruleengine
image: harbor.hk.batmobi.cn/library/rule-engine:latest
args:
- --gateway.url=http://localhost:8080
- --log.level=info

---

apiVersion: v1
kind: Service
metadata:
labels:
app: doraemon-web
name: doraemon-web
namespace: default
spec:
type: NodePort
ports:
- name: tcp
nodePort: 32000
protocol: TCP
port: 8080
targetPort: 80
- name: http
nodePort: 32001
port: 80
protocol: TCP
targetPort: 80
selector:
app: doraemon-web

```

 - 4、加载yml配置文件
```
kubectl apply -f doraemon.yml
kubectl get pods
```
 
