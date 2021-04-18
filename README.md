# openshift-monitor-external-service
Demo code for article "Utilize OpenShift to manage external services metrics"

## 1. enable openshift-user-workload-monitoring
cluster-monitorinng-config.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    enableUserWorkload: true

```

```
oc apply -f cluster-monitoring-config.yaml
```
## 2. Grant monitor-edit permission for your external-service project

```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: AppMonitoring
  namespace: external-service
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: monitoring-edit
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: developer
```
```bash
oc apply -f AppMonitoring-User-RoleBinding.yaml
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: user-workload-monitoring-admin
  namespace: openshift-user-workload-monitoring
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: user-workload-monitoring-config-edit
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: developer
```
```bash
oc apply -f user-workload-monitoring-admin-rolebindings.yaml
```
同时也可以在 openshift-user-workload-monitoring项目赋予

**user-workload-monitoring-config-edit**

角色权限

查看: **user-workload-monitoring-admin-rolebindings.yaml**

## 3. 部署自己的应用
注意一定需要是/metrics rest endpoint

oc apply -f prometheus-example-app.yaml
## 部署 servicemonitor
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    k8s-app: prometheus-example-monitor
  name: prometheus-example-monitor
  namespace: sandbox
spec:
  endpoints:
  - interval: 30s
    port: web <-注意这里是service 的port的名字，不是端口号
    scheme: http
  selector:
    matchLabels:
      app: prometheus-example-app <-注意这里是service的label 而不是Pod的label

```
它就是通过service进行Label match来查找metrics;


