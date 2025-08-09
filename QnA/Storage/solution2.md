## Solution to Task 2

### Step 1: Create the StorageClass

We create a `StorageClass` named `fast-storage` using the `rancher.io/local-path` provisioner, with a **Retain** reclaim policy and **Immediate** volume binding mode.

`vim storage-class-fast.yaml` 
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: rancher.io/local-path
reclaimPolicy: Retain
volumeBindingMode: Immediate
```

Apply the command `kubectl apply -f storageclass-fast.yaml`

### Step 2: Prepare the Node Directory
On the node where the PV will be located; run the following command.

```bash
sudo mkdir -p /opt/data
sudo chmod 777 /opt/data
```

###

### Step 3  Create the PV

We define a PV that uses the fast-storage StorageClass, stores data in /opt/data, and is bound to a specific node via node affinity.

See the labels of the nodes

`kubectl get nodes --show-labels`

`vim pv.yaml`
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fast-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-storage
  local:
    path: /opt/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - node01
```

Apply the pv file: `kubectl apply -f pv.yaml`

### Step 4 Create the PVC

We request storage from the fast-storage class. Since the PV already exists and matches our request, the PVC will bind to it.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fast-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast-storage
```

Apply: `kubectl apply -f pvc.yaml`

### Step 5: Verify Binding
Check the PVC and PV status:
```bash
kubectl get pvc fast-pvc
kubectl get pv fast-pv
```

Expected Output:

- PVC status should be Bound
- PV status should be Bound and show the PVC name in the CLAIM column

### Step 6:  Delete PVC and Check PV State
Delete the PVC:
`kubectl delete pvc fast-pvc`
Check PV state:

`kubectl get pv fast-pv`

Expected Output:

- PV remains with status Released
- Data in /opt/data remains intact on the node

### How It Works ?

- StorageClass defines how volumes are provisioned and reclaimed.
- PersistentVolume is created manually with a local path and node affinity so Kubernetes knows where to schedule pods using it.
- PersistentVolumeClaim requests storage from the StorageClass and binds to the matching PV.
- When the PVC is deleted, because the reclaim policy is Retain, Kubernetes does not delete the PV or its data; instead, the PV status changes to Released.