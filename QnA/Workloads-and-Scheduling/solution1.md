## Solution to Task 1

### Step 1: Create Deployment cpu-demo with CPU requests

`vim cpu-demo.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cpu-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cpu-demo
  template:
    metadata:
      labels:
        app: cpu-demo
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: 200m
          limits:
            cpu: 500m
```
Apply: `kubectl apply -f cpu-demp.yaml`

### Step 2: Create Horizontal Pod Autoscaler (HPA)
You can create the HPA with this command:

`kubectl autoscale deployment cpu-demo --min=1 --max=5 --cpu-percent=50`

Alternatively, create a YAML file `cpu-demo-hpa.yaml`:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: cpu-demo
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cpu-demo
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

Apply it: `kubectl apply -f cpu-demo-hpa.yaml`

### Step 3: Verify

Check deployment: `kubectl get deployments cpu-demo`

Check pods: `kubectl get pods -l app=cpu-demo`

Check HPA status: `kubectl get hpa cpu-demo`

### Step 4: Generate CPU load inside the pod to trigger scaling
Get a shell into the pod:
`kubectl exec -it <pod-name> -- /bin/sh`

Run CPU load: `while true; do :; done &`

Check HPA scaling with: `kubectl get hpa cpu-demo -w`

You should see replicas increase as CPU usage exceeds 50%.


