

# `kubernetes-services-kubeshark`

This project contains the configuration files (`Deployment` and `Service`) for running a simple Python Django application on Minikube. The setup is designed to demonstrate key Kubernetes Service concepts, including **NodePort exposure**, **Service Discovery (Selectors)**, and **Load Balancing** across multiple pod replicas.

The final steps involve using **Kubeshark** to visualize the traffic flow and confirm the load balancing behavior.

## 🚀 Prerequisites

Before you begin, ensure you have the following tools installed and configured:

  * **Docker:** (Necessary for the Minikube driver)
  * **Minikube:** (Local Kubernetes cluster)
  * **kubectl:** (Kubernetes command-line tool)
  * **Kubeshark:** (Network traffic analyzer for the final demo)

## 📁 Project Structure (Updated)

The Kubernetes manifests are nested within the `Docker-Zero-to-Hero` structure:

```
kubernetes-services-kubeshark/
└── Docker-Zero-to-Hero/
    └── examples/
        └── python-web-app/
            ├── deployment.yaml      # Defines the Pods, Replicas (2), and container specs.
            ├── service.yaml         # Defines the NodePort Service and the TargetPort mapping.
            ├── Dockerfile           # Used to build the application image.
            └── devops/              # Source code directory for the Django application.
```

## 🛠️ Setup and Deployment

### 1\. Navigate to the Configuration Directory

All `kubectl` commands must be run from the `python-web-app` directory.

```bash
cd Docker-Zero-to-Hero/examples/python-web-app
```

### 2\. Start Minikube

Ensure your Minikube cluster is running.

```bash
minikube start
```

### 3\. Apply Kubernetes Configuration

Apply the Deployment and Service manifests to your Minikube cluster:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### 4\. Verify Pod and Service Status

Check that the two replicas are running and the NodePort service is active.

```bash
kubectl get pods
kubectl get svc
```

You should see the `dj-demo` service running with the NodePort (e.g., `30008`):

```
NAME      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
dj-demo   NodePort   10.103.185.41   <none>        80:30008/TCP   1m
```

## 🌐 Accessing the Application

Due to the networking layer used by the Docker driver on macOS/Linux, you need to use `minikube service` to establish a network route.

1.  **Run the Minikube Service Proxy** (Keep this process running in a dedicated terminal window):

    ```bash
    minikube service dj-demo
    ```

    This command will open a browser to a URL like `http://127.0.0.1:53829/`.

2.  **Access the Application:** The Django application is configured with a context root of `/demo`. In the browser opened by Minikube, append `/demo` to the URL:

    ```
    http://127.0.0.1:<PORT>/demo
    ```

    You should now see the "Free DevOps Course By Abhishek" welcome page.

## 📈 Load Balancing Demonstration

This service automatically load balances traffic across the two pod replicas defined in `deployment.yaml`.

### 1\. Monitor Pod Logs

Open two separate terminal windows to stream logs from your two running pods (replace the pod names with your current ones):

```bash
# Terminal 1: Monitor logs for Pod 1
kubectl logs -f dj-demo-5c57f658-xxxxx

# Terminal 2: Monitor logs for Pod 2
kubectl logs -f dj-demo-5c57f658-yyyyy
```

### 2\. Send Requests

Repeatedly send requests to your application (e.g., refreshing the browser or running the `curl` command multiple times).

```bash
curl http://127.0.0.1:<PORT>/demo
```

You will observe log entries appearing **alternatingly** between the two terminal windows, confirming that the Kubernetes Service is successfully performing **Round-Robin Load Balancing**.

## 📊 Traffic Visualization with Kubeshark (Advanced)

To visually confirm load balancing and observe all internal Kubernetes traffic, use Kubeshark.

### 1\. Install and Tap Traffic

If not already installed, run the installation script:

```bash
/bin/bash -c "$(curl -fsSL https://kubeshark.co/install)"
```

Then, run the tap command to start monitoring all namespaces:

```bash
kubeshark tap --all-namespaces
```

### 2\. Observe Traffic Flow

Kubeshark will open a Web UI (usually at `http://localhost:8899`). Send a few more requests to your application.

  * Filter the traffic for the Pod IP addresses (e.g., `10.244.0.30`).
  * You will see the request packets being routed from the NodePort entry to the two separate Pod IP addresses, providing a powerful visualization of the service's internal functionality.

## 🧹 Cleanup

When finished, from the `python-web-app` directory, run:

```bash
kubectl delete deployment dj-demo
kubectl delete service dj-demo

# Optional: Stop Minikube from any directory
minikube stop
```

