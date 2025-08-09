## Solution to Task 3
---
### Step 1: Crete a directory on node where you want to reside the volume

```bash
sudo mkdir -p /opt/data
sudo chmod 777 /opt/data
```
### Step 2: Create the PersistentVolume 

`pv.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: static-pv-example
spec:
  capacity:
    storage: 200Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ""  # No StorageClass for static binding
  hostPath:
    path: /opt/data
    type: Directory
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - node-1
```

Apply: `kubectl apply -f pv.yaml`

### Step 3: Create the PersistentVolumeClaim

`vim pvc.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: static-pvc-example
spec:
  volumeName: static-pv-example
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 200Mi

```