1) Ingress service loadbalancer
```code
istio-ingress   LoadBalancer   10.96.170.73   172.18.0.5    15021:30345/TCP,80:30098/TCP,443:32055/TCP   160m
```
2) ingress gateway
```code
erpadsui-gateway
apiVersion: networking.istio.io/v1
kind: Gateway
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1","kind":"Gateway","metadata":{"annotations":{},"name":"erpadsui-gateway","namespace":"istio-ingress"},"spec":{"selector":{"instio":"ingressgateway"},"servers":[{"hosts":["adssubsdev.corp.chartercom.com"],"port":{"name":"http","number":80,"protocol":"HTTP"}}]}}
  creationTimestamp: "2026-06-03T17:53:11Z"
  generation: 2
  name: erpadsui-gateway
  namespace: istio-ingress
  resourceVersion: "21264"
  uid: 7af03cdd-1cfd-41bd-ab7d-3aca4f255658
spec:
  selector:
    instio: ingressgateway
  servers:
  - hosts:
    - adssubsdev.corp.chartercom.com
    port:
      name: http
      number: 80
      protocol: HTTP
```
3) virtual service
```code
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  annotations:
    meta.helm.sh/release-name: erpadsui-subs
    meta.helm.sh/release-namespace: erpadsui
  creationTimestamp: "2026-06-03T17:49:10Z"
  generation: 1
  labels:
    app.kubernetes.io/managed-by: Helm
  name: erpadsui-subs
  namespace: erpadsui
  resourceVersion: "20215"
  uid: ad8280f8-e332-4790-9a43-b68ead9c50a1
spec:
  gateways:
  - istio-ingress/erpadsui-gateway
  hosts:
  - adssubsdev.corp.chartercom.com
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: erpadsui-subs
        port:
          number: 80
```
4) service
```code
apiVersion: v1
kind: Service
metadata:
  annotations:
    meta.helm.sh/release-name: erpadsui-subs
    meta.helm.sh/release-namespace: erpadsui
  creationTimestamp: "2026-06-03T17:49:10Z"
  labels:
    app.kubernetes.io/instance: erpadsui-subs
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: erpadsui-subs
    app.kubernetes.io/version: latest
    helm.sh/chart: erpadsui-subs-1-Snapshot-Dev-123456
  name: erpadsui-subs
  namespace: erpadsui
  resourceVersion: "20199"
  uid: b45ee910-b095-4fd9-b9cc-45109442f769
spec:
  clusterIP: 10.96.21.151
  clusterIPs:
  - 10.96.21.151
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: http
  selector:
    app.kubernetes.io/instance: erpadsui-subs
    app.kubernetes.io/name: erpadsui-subs
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```