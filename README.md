# Deploy Mosquitto Message Broker with ConfigMap and Secret Volume Types

## Project Overview
This project demonstrates deploying the Mosquitto message broker using Kubernetes. It involves configuring the Mosquitto broker with a ConfigMap for its configuration settings and a Secret for sensitive data, leveraging Kubernetes' volume types. The deployment is set-up and tested using a Minikube Kubernetes cluster.

---

## Key Features
- **Mosquitto Message Broker**: A lightweight MQTT broker to facilitate message passing.
- **ConfigMap**: Stores and provides non-sensitive configuration data to the Mosquitto container.
- **Secret**: Stores sensitive information such as passwords and provides it securely to the container.
- **Volume Mounts**: Links ConfigMap and Secret data into the Mosquitto container filesystem.

---

## Technologies Used
- **Kubernetes**
- **Docker**
- **Mosquitto**

---

## Project Structure
This project includes the following Kubernetes manifest files:

1. **`mosquitto.yaml`**: Defines the Deployment for the Mosquitto broker and mounts ConfigMap and Secret volumes.
2. **`config-file.yaml`**: ConfigMap for Mosquitto's configuration.
3. **`secret-file.yaml`**: Secret containing sensitive data for the broker.

---

## Setup and Deployment

### Prerequisites
- Docker installed on your machine.
- Kubernetes cluster set up using Minikube.
- `kubectl` command-line tool installed and configured.

### Steps

#### 1. Define the ConfigMap
Create a file named `config-file.yaml` with the following content:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: mosquitto-config-file
data:
    mosquitto.conf: |
        log_dest stdout
        log_type all
        log_timestamp true
        listener 9001
```
Apply the ConfigMap:
```bash
kubectl apply -f config-file.yaml
```

#### 2. Define the Secret
Create a file named `secret-file.yaml` with the following content:
```yaml
apiVersion: v1
kind: Secret
metadata:
    name: mosquitto-secret-file
type: Opaque
data:
    secret.file: |
        VGVjaFdvcmxkMjAyMyEgLW4=
```
Apply the Secret:
```bash
kubectl apply -f secret-file.yaml
```

#### 3. Define the Deployment
Create a file named `mosquitto.yaml` with the following content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mosquitto
  labels:
    app: mosquitto
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mosquitto
  template:
    metadata:
      labels:
        app: mosquitto
    spec:
        containers:
          - name: mosquitto
            image: eclipse-mosquitto:2.0
            ports:
              - containerPort: 1883
            volumeMounts: 
              - name: mosquitto-config
                mountPath: /mosquitto/config
              - name: mosquitto-secret
                mountPath: /mosquitto/secret
                readOnly: true
        volumes:
          - name: mosquitto-config
            configMap:
              name: mosquitto-config-file
          - name: mosquitto-secret
            secret:
              secretName: mosquitto-secret-file
```
Apply the Deployment:
```bash
kubectl apply -f mosquitto.yaml
```

---

## Verify Deployment

1. Check the pod status:
   ```bash
   kubectl get pods
   ```
   Ensure the Mosquitto pod is in the `Running` state.

2. Access the Mosquitto container:
   ```bash
   kubectl exec -it <mosquitto-pod-name> -- /bin/sh
   ```

3. Verify the mounted files:
   - Configuration file:
     ```bash
     cat /mosquitto/config/mosquitto.conf
     ```
   - Secret file:
     ```bash
     cat /mosquitto/secret/secret.file
     ```

---

## Cleaning Up
To delete the resources created during the deployment, run:
```bash
kubectl delete -f mosquitto.yaml
kubectl delete -f config-file.yaml
kubectl delete -f secret-file.yaml
```

---

## Conclusion
This project demonstrates the use of Kubernetes ConfigMaps and Secrets to manage and secure application configurations. By following this README, you can deploy a Mosquitto message broker with these features on a Minikube cluster. This approach can be adapted for other applications needing secure and dynamic configuration management.
