# Task 2: Create a StorageClass, PersistentVolume, and PersistentVolumeClaim

This task demonstrates how to configure a StorageClass for local storage using the rancher.io/local-path dynamic provisioner, create a PersistentVolumeClaim (PVC) that requests storage from this StorageClass, and verify PVC binding and PV retention behavior.

---

## Objective

A Kubernetes administrator needs to provide local persistent storage for workloads with strict node placement. The following requirements must be fulfilled:

- Create a **StorageClass** with:
  - Name: `fast-storage`
  - Provisioner: `rancher.io/local-path`
  - Reclaim policy: `Retain`
  - Volume binding mode: `Immediate` (default)

- Create a **PersistentVolumeClaim (PVC)** that:
  - Requests `1Gi` of storage
  - Access mode: `ReadWriteOnce`
  - Uses the `fast-storage` StorageClass
  - Binds to the created PV

- Verify that:
  - The PVC successfully binds to the PV
  - When the PVC is deleted, the PV remains in `Released` state (due to the `Retain` reclaim policy)

---
