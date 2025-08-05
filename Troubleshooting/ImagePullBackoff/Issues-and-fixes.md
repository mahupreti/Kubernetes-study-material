# Kubernetes `ImagePullBackOff` Error

## What is `ImagePullBackOff` ? 

`ImagePullBackOff` is a **Kubernetes Pod status** that indicates a container **failed to start because the image could not be pulled** from the container registry. This is not an issue with your Pod's spec itself but with accessing the image required to run your container.

---

## Common Causes

### 1. Incorrect Image Name or Tag

- Typo in image name or version tag.
- Example:
  ```yaml
  image: nginx:latset  # Wrong tag (should be 'latest')
## 2. Private Image Registry Without Credentials
You are trying to pull from a private registry (e.g., Docker Hub, ECR, GitHub Container Registry) without providing authentication.

**Fix:** Create a Kubernetes Secret of type `docker-registry` and reference it in your Pod spec.

### How to Create Image Pull Secret

```bash
kubectl create secret docker-registry my-secret-name \
  --docker-username=<your-username> \
  --docker-password=<your-password> \
  --docker-server=<registry-url> \
  --docker-email=<your-email>
```

After creating secret, add that secret to your yaml file and apply the file again.

`vim nginx-deploy.yaml`

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:    
      containers:
      - name: nginx
        image: mahesh/nginx-image-pull-demo:v1 #this is my private repo in docker hub
        ports:
        - containerPort: 80  
      imagePullSecrets:
      - name: my-secret-name 
```
## 3. Network/DNS Issues
Kubernetes nodes can't reach the image registry due to:
- Network firewall rules
- Incorrect DNS resolution
- Proxy issues

### How to Test ?
- Try ping or curl to the registry from the node.
- Use nslookup or dig to test DNS resolution.