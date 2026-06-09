# Istio Sidecar Canary Deployments

## Install Sidecar mode
```code
istioctl install

awasson@pi4b-node1:~/istio-1.30.1$ istioctl install
        |\          
        | \         
        |  \        
        |   \       
      /||    \      
     / ||     \     
    /  ||      \    
   /   ||       \   
  /    ||        \  
 /     ||         \ 
/______||__________\
____________________
  \__       _____/  
     \_____/        

This will install the Istio 1.30.1 profile "default" into the cluster. Proceed? (y/N) y
✔ Istio core installed ⛵️                                                                                                                                                        
✔ Istiod installed 🧠                                                                                                                                                            
✔ Ingress gateways installed 🛬                                                                                                                                                  
✔ Installation complete            
```

## Create the Helm Charts
`helm create canary`

### Update the values.yaml
1) Add the code for canary
```code
canary: true
canaryVersion: "latest"
```
2) Update the resources to allow for autoscaling
```code
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
```
3) Update the volumes for the configmap - Optional
```code
# Additional volumes on the output Deployment definition.
volumes:
  - configMap:
      name: nginx-conf
    name: default-conf

# Additional volumeMounts on the output Deployment definition.
volumeMounts:
  - mountPath: /etc/nginx/conf.d/default.conf
    name: default-conf
    subPath: default.conf
```
### _helpers.tpl
1) Create the v2 common labels
```code
{{/*
v2 Common labels
*/}}
{{- define "v2canary.labels" -}}
helm.sh/chart: {{ include "canary.chart" . }}
{{ include "canary.v2selectorLabels" . }}
{{- if .Values.canary }}
app.kubernetes.io/version: {{ .Values.canaryVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}
```
2) Create the v2 Selector Labels
```code
{{/*
v2 Selector labels
*/}}
{{- define "canary.v2selectorLabels" -}}
app.kubernetes.io/name: {{ include "canary.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
version: v2
{{- end }}
```

### deployment.yaml
1) Add the following to `metadata.labels`
```code
    {{- if .Values.canary }}
    version: v1
    {{- end }}
```
2) Add the following to `spec.selector.matchLabels`
```code
    {{- if .Values.canary }}
    version: v1
    {{- end }}
```
3) Add the following to the bottom
```code
{{- if .Values.canary }}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "canary.fullname" . }}-v2
  labels:
    {{- include "v2canary.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "canary.v2selectorLabels" . | nindent 6 }}
  template:
    metadata:
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      labels:
        {{- include "v2canary.labels" . | nindent 8 }}
        {{- with .Values.podLabels }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "canary.serviceAccountName" . }}
      {{- with .Values.podSecurityContext }}
      securityContext:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      containers:
        - name: {{ .Chart.Name }}-v2
          {{- with .Values.securityContext }}
          securityContext:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Values.canaryVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.port }}
              protocol: TCP
          {{- with .Values.livenessProbe }}
          livenessProbe:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          {{- with .Values.readinessProbe }}
          readinessProbe:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          {{- with .Values.resources }}
          resources:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          {{- with .Values.volumeMounts }}
          volumeMounts:
            {{- toYaml . | nindent 12 }}
          {{- end }}
      {{- with .Values.volumes }}
      volumes:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
{{- end }}
```

### hpa.yaml
Add the following before the final `{{- end }}`
```code
{{- if .Values.canary }}
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "canary.fullname" . }}-v2
  labels:
    {{- include "canary.labels" . | nindent 4 }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "canary.fullname" . }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  metrics:
    {{- if .Values.autoscaling.targetCPUUtilizationPercentage }}
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
    {{- end }}
    {{- if .Values.autoscaling.targetMemoryUtilizationPercentage }}
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetMemoryUtilizationPercentage }}
    {{- end }}
{{- end }}
```

## Package the Helm Chart
`helm package canary`

## Deploy the Helm Chart
`helm install canary <chart version> -n canary --create-namespace`

### You should see the following if deployment worked
```code
(base) awasson@Aarons-MacBook-Pro Istio Sidecar % kubectl get all -n canary
NAME                             READY   STATUS    RESTARTS   AGE
pod/canary-5788c85778-n2kxv      1/1     Running   0          12m
pod/canary-v2-74d677576b-bwnzs   1/1     Running   0          12m

NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/canary   ClusterIP   10.152.183.210   <none>        80/TCP    43m

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/canary      1/1     1            1           43m
deployment.apps/canary-v2   1/1     1            1           43m

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/canary-5484f4bd7       0         0         0       43m
replicaset.apps/canary-5788c85778      1         1         1       12m
replicaset.apps/canary-v2-6cbf4784c7   0         0         0       43m
replicaset.apps/canary-v2-74d677576b   1         1         1       12m

NAME                                            REFERENCE           TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/canary      Deployment/canary   cpu: <unknown>/80%   1         3         1          43m
horizontalpodautoscaler.autoscaling/canary-v2   Deployment/canary   cpu: <unknown>/80%   1         3         1          43m
```

## Create the Istio Configuration
1) Create gateway.yaml
```code
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: canary
  namespace: istio-ingress
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https-gateway
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: homelab
    hosts:
    - nginx.aaronwasson.net
```

2) Create virtualservice.yaml
```code
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: canary
  namespace: canary
spec: 
  hosts:
  - nginx.aaronwasson.net
  gateways:
  - istio-ingress/canary
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        port:
          number: 80
        host: canary
        subset: stable
      weight: 80
    - destination:
        port:
          number: 80
        host: canary
        subset: canary
      weight: 20
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,reset,connect-failure
```
3) Create destinationrule.yaml
```code
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
    name: canary-destinationrule
    namespace: canary
spec:
    host: canary
    trafficPolicy:
        loadBalancer:
            simple: ROUND_ROBIN
    subsets:
    - name: stable
      labels:
        version: v1
    - name: canary
      labels:
        version: v2
```