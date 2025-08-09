# TASK 4: Autoscaling

Deploy a sample workload and configure Horizontal Pod Autoscaling for it.
- Create the deployment `cpu-demo` with nginx image having current replicas as 1
- Configure an HPA to scale this deployment from 1 up to 5 replicas when the average CPU utilization exceeds 50%.

### Before performing task, remember to check if metrics-sever is installed or not.

`kubectl get apiservices`

You should get service kube-system/metrics-server as AVAILABLE as True.