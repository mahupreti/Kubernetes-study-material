
## Solution to Task 1

This solution includes:

- Creating a PersistentVolumeClaim (PVC)
- Creating a Pod that mounts the PVC at `/data`
- Verifying the PVC is mounted correctly
- Optionally cleaning up the resources

---

##  Step 1: Create a PersistentVolumeClaim (PVC)

### `pvc.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: local-path
```
If the PVC is stuck in Pending, ensure the local-path storage class is installed.


- apiVersion: v1: This defines the version of the Kubernetes API to use for this object.
- kind: PersistentVolumeClaim: Specifies that we are creating a PVC resource.
- metadata.name: local-pvc: The name used to refer to this PVC.
- accessModes: Set to ReadWriteOnce, which allows the volume to be mounted as read-write by a single node.
- resources.requests.storage: Requests 500Mi of storage.
- storageClassName: local-path: Specifies the dynamic storage provisioner to use, which must be available in your cluster.

Apply the PVC `kubectl apply -f pvc.yaml`

Check the status of PVC `kubectl get pvc` and verify if it's bound. 

## Step 2: Create a Pod to Use the PVC

### `pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-test-pod
spec:
  containers:
    - name: app
      image: busybox
      command: [ "sleep", "3600" ]
      volumeMounts:
        - mountPath: "/data"
          name: storage-volume
  volumes:
    - name: storage-volume
      persistentVolumeClaim:
        claimName: local-pvc
```

- image: busybox – Lightweight Linux image for testing.
- command: [ "sleep", "3600" ] – Keeps the container alive for 1 hour.
- volumeMounts.mountPath: /data – Mount the volume at /data inside the container.
- volumes.persistentVolumeClaim.claimName: local-pvc – Bind the volume from the previously created PVC.

Apply the Pod `kubectl apply -f pod.yaml`
Check Pod Status `kubectl get pods` and see if it's running

### Step 3: Verify the Volume Mount
`kubectl exec -it storage-test-pod -- sh`

Check Mounted Volume `mount | grep /data`
Or check available space: `df -h /dat`

Create and Read a Test File `echo "Hello Persistent Storage" > /data/test.txt`

`cat /data/test.txt`

Expected output:

`Hello Persistent Storage`

This confirms the volume is writable and mounted properly at /data.