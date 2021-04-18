# Create my-grafana instance to host external service dashboard

## Install the grafana operator into my-grafana namespaces

##  create  my-grafana Customer Resources

```yaml
apiVersion: integreatly.org/v1alpha1
kind: Grafana
metadata:
  name: my-grafana
  namespace: my-grafana
spec:
  config:
    auth:
      disable_signout_menu: false
    auth.anonymous:
      enabled: true
    log:
      level: warn
      mode: console
    security:
      admin_password: admin <-1. this is your frist time login password
      admin_user: admin
  dashboardLabelSelector:
  - matchExpressions:
    - key: app
      operator: In
      values:
      - grafana
  dataStorage:
    labels: # Additional labels for the PVC
      app: grafana
    annotations: # Additional annotations for the PVC
      app: grafana
    accessModes: # An array of access modes, e.g. `ReadWriteOnce`
    - ReadWriteMany
    size: '10Gi'        # Requested size, e.g. `10Gi`
    class: "" <-2. you can input StorageClass Name if there is available 
  ingress:
    enabled: true
    hostname: my-grafana.apps-crc.testing <- 3. You need to input your own subdomain here.

```
```
oc apply -f my-grafana-cr.yaml
oc get po
NAME                                  READY   STATUS    RESTARTS   AGE
grafana-deployment-789d66454d-xkdwk   1/1     Running   0          23d
grafana-operator-5bddbfb9f9-x72jv     1/1     Running   0          24d
```
## Configure the cluster-monitor-view-rolebinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-monitoring-view
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-monitoring-view
subjects:
- kind: ServiceAccount
  name: grafana-serviceaccount <- Bind the grafana-serviceaccount to cluster role: cluster-monitoring-view
  namespace: my-grafana
```
```bash
oc apply -f cluster-monitoring-view-clusterrolebinding.yaml
```
