# Task 2: Create a StorageClass, PersistentVolume, and PersistentVolumeClaim

This task demonstrates how to configure a **StorageClass** with custom settings, create a **PersistentVolume (PV)** with node affinity, and bind it to a **PersistentVolumeClaim (PVC)**. It also verifies PV retention after PVC deletion.

---

## Objective

A Kubernetes administrator needs to provide local persistent storage for workloads with strict node placement. The following requirements must be fulfilled:

- Create a **StorageClass** with:
  - Name: `fast-storage`
  - Provisioner: `rancher.io/local-path`
  - Reclaim policy: `Retain`
  - Volume binding mode: `Immediate` (default)

- Create a **PersistentVolume (PV)** that:
  - Uses the `fast-storage` StorageClass
  - Has a capacity of `1Gi`
  - Access mode: `ReadWriteOnce`
  - Uses a local path (e.g., `/opt/data`) on a specific node
  - Includes **node affinity** so Kubernetes schedules Pods using this PV on the correct node

- Create a **PersistentVolumeClaim (PVC)** that:
  - Requests `1Gi` of storage
  - Access mode: `ReadWriteOnce`
  - Uses the `fast-storage` StorageClass
  - Binds to the created PV

- Verify that:
  - The PVC successfully binds to the PV
  - When the PVC is deleted, the PV remains in `Released` state (due to the `Retain` reclaim policy)

---
