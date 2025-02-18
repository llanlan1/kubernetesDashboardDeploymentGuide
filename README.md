# Kubernetes Dashboard Deployment Guide

This guide walks through deploying a Kubernetes dashboard on **Azure Kubernetes Service (AKS)** while resolving common issues. It covers setting up your environment, deploying services, and troubleshooting potential problems.

---

## **Prerequisites**

Ensure you have the following installed:

- **Azure CLI** ([Installation Guide](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli))  
- **Kubectl** ([Installation Guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/))  
- **Docker** ([Installation Guide](https://docs.docker.com/get-docker/))  
- **Admin access to an Azure Kubernetes Service (AKS) cluster**  
  - **If you already have an AKS cluster,** retrieve its credentials:
    ```sh
    az aks list --output table  # Check if an AKS cluster exists
    az aks get-credentials --resource-group k8s-resource-group --name k8s-cluster
    ```
  - **If you don't have an AKS cluster, follow Step 0 below.**

✅ **Confirm your cluster is running**:
```sh
kubectl get nodes
```
If nodes appear as `Ready`, your AKS cluster is set up correctly.

---

## **Step 0: Create an AKS Cluster (If Needed)**

If you don't already have an AKS cluster, create one with the following commands:

```sh
az group create --name k8s-resource-group --location eastus
az aks create --resource-group k8s-resource-group --name k8s-cluster --node-count 1 --enable-managed-identity --generate-ssh-keys
```

Retrieve cluster credentials:

```sh
az aks get-credentials --resource-group k8s-resource-group --name k8s-cluster
kubectl get nodes  # Verify nodes are Ready
```

---

## **Step 1: Connect to Your AKS Cluster**

Retrieve cluster credentials to interact with Kubernetes securely.

```sh
az login  # Authenticate with Azure
az aks get-credentials --resource-group k8s-resource-group --name k8s-cluster
```

✅ **Verify the connection:**

```sh
kubectl config get-contexts
kubectl get nodes
```

If nodes are in `Ready` state, you are successfully connected.

> **Security Tip:** Avoid exposing kubeconfig paths. Instead, use:
> ```sh
> export KUBECONFIG=/tmp/kubeconfig
> az aks get-credentials --resource-group k8s-resource-group --name k8s-cluster --file /tmp/kubeconfig
> ```
> After use, remove the file:
> ```sh
> rm /tmp/kubeconfig && unset KUBECONFIG
> ```

---

## **Step 2: Build and Push the Docker Image**

### **2.1 Create a `Dockerfile`**

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### **2.2 Build and Push the Image**

Replace `YOUR-DOCKERHUB-USERNAME` with your actual Docker Hub username.

```sh
docker build -t YOUR-DOCKERHUB-USERNAME/k8s-dashboard:v1 .
docker login
docker push YOUR-DOCKERHUB-USERNAME/k8s-dashboard:v1
```

---

## **Step 3: Deploy Kubernetes Dashboard**

### **3.1 Apply Deployment Configuration**

Create a `deployment.yaml` file and apply it.

```sh
kubectl apply -f deployment.yaml --validate=false
```

### **3.2 Apply Service Configuration**

Create a `service.yaml` file and apply it to expose the dashboard.

```sh
kubectl apply -f service.yaml --validate=false
```

✅ **Check if the service is running:**

```sh
kubectl get services
```

Expected output:

```
NAME                    TYPE           CLUSTER-IP        EXTERNAL-IP     PORT(S)        AGE
k8s-dashboard-service   LoadBalancer   <your-cluster-ip> <your-external-ip> 80:32619/TCP  2m33s
```

---

## **Step 4: Access the Dashboard**

Use the external IP address of your service to access the dashboard in your browser:

```
http://<your-external-ip>
```

If successful, you should see **"Welcome to the Internal Dashboard - Deployed on Kubernetes!"** 🎉

---

## **Step 5: Verify and Debug (if necessary)**

### **5.1 Check Running Pods**

```sh
kubectl get pods -A
```

Ensure all pods are running. If any are stuck in `Pending` or `CrashLoopBackOff`, debug using:

```sh
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
```

### **5.2 Restart Pods if Needed**

```sh
kubectl delete pod <pod-name> -n <namespace>
```

### **5.3 Troubleshoot Service Issues**

If the service is not accessible:

1. **Check if the LoadBalancer has an assigned external IP:**  
   ```sh
   kubectl get services
   ```
2. **Restart the service:**  
   ```sh
   kubectl delete service k8s-dashboard-service
   kubectl apply -f service.yaml
   ```
3. **If using `ClusterIP` instead of `LoadBalancer`, manually port-forward:**  
   ```sh
   kubectl port-forward svc/k8s-dashboard-service 8080:80
   ```  
   Then access it via `http://localhost:8080`.

---

## **Summary**

✅ **Built and pushed Docker image** to Docker Hub  
✅ **Deployed Kubernetes Dashboard** using AKS  
✅ **Connected to Cluster** using `az aks get-credentials`  
✅ **Checked Nodes and Services**  
✅ **Debugged Common Issues**  

Your dashboard is now up and running! 🎉

---

## **Next Steps**

- 🔒 **Secure the dashboard** with **RBAC (Role-Based Access Control)**.  
- 🔐 **Implement HTTPS and authentication** for secure access.  
- 📊 **Monitor logs** using **Azure Monitor for Containers**.  
- 🚀 **Set up automated deployments** using **CI/CD pipelines**.  

Happy deploying! 🚀
