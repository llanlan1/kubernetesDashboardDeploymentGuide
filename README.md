# Kubernetes Dashboard Deployment Guide

This guide walks through deploying a Kubernetes dashboard on **Azure Kubernetes Service (AKS)** (plus resolving some common issues). 

## Prerequisites
Ensure you have the following:

- **Azure CLI** installed ([Installation Guide](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli))
- **Kubectl** installed ([Installation Guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/))
- **Azure Kubernetes Service (AKS)** cluster set up
- **Admin access to your cluster**

## Step 1: Connect to Your AKS Cluster
First, retrieve your cluster credentials to interact with Kubernetes.

```sh
az login  # Authenticate with Azure
az aks get-credentials --resource-group k8s-resource-group --name k8s-cluster --file ~/.kube/config
```

Verify the connection:

```sh
kubectl config get-contexts
kubectl get nodes
```

If nodes are in `Ready` state, you are successfully connected.

## Step 2: Deploy Kubernetes Dashboard

### 2.1 Apply Deployment Configuration

Create a `deployment.yaml` file to deploy the dashboard.

```sh
kubectl apply -f deployment.yaml --validate=false
```

### 2.2 Apply Service Configuration

Create a `service.yaml` file to expose the dashboard.

```sh
kubectl apply -f service.yaml --validate=false
```

Check if the service is running:

```sh
kubectl get services
```

You should see output similar to this:

```
NAME                    TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)        AGE
k8s-dashboard-service   LoadBalancer   10.0.68.207   57.152.101.27   80:32619/TCP   2m33s
```

## Step 3: Access the Dashboard

Use the external IP address of your service to access the dashboard in your browser:

```
http://57.152.101.27
```

If successful, you should see **"Welcome to the Internal Dashboard - Deployed on Kubernetes!"**

## Step 4: Verify and Debug (if necessary)

### Check Running Pods

```sh
kubectl get pods -A
```

Ensure all pods are running. If any are stuck in `Pending` or `CrashLoopBackOff`, debug using:

```sh
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
```

### Restart Pods if Needed

```sh
kubectl delete pod <pod-name> -n <namespace>
```

### Troubleshoot Service Issues

If the service is not accessible:

1. Check if the LoadBalancer has an assigned external IP.
   ```sh
   kubectl get services
   ```
2. Restart the service.
   ```sh
   kubectl delete service k8s-dashboard-service
   kubectl apply -f service.yaml
   ```
3. If using a **ClusterIP** instead of a **LoadBalancer**, port-forward manually:
   ```sh
   kubectl port-forward svc/k8s-dashboard-service 8080:80
   ```
   Then access it via `http://localhost:8080`.

## Summary

✅ **Deployed Kubernetes Dashboard** using AKS
✅ **Connected to Cluster** using `az aks get-credentials`
✅ **Checked Nodes and Services**
✅ **Debugged Common Issues**

Your dashboard is now up and running! 🎉

---

## Next Steps
- Secure the dashboard with **RBAC (Role-Based Access Control)**.
- Implement HTTPS and authentication.
- Monitor logs using **Azure Monitor for Containers**.
