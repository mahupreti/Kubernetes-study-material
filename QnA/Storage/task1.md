# Task 1: Create a Persistent Volume Claim and Mount it to a Pod

This task demonstrates how to provision persistent storage using a **PersistentVolumeClaim (PVC)** and mount it into a Pod using the `local-path` storage class.

---

## Objective

A developer needs persistent storage for an application. The following requirements must be fulfilled:

- Create a **PersistentVolumeClaim** with:
  - Storage size: `500Mi`
  - Access mode: `ReadWriteOnce`
  - Storage class: `local-path`
- Create a **Pod** that mounts the volume at path `/data`.
- Ensure the volume is automatically provisioned and properly mounted.

---
