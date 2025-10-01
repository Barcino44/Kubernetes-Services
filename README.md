## Description:

This repository aims to explain the implementation of 4 deployments (each with different images) using different types of Kubernetes services (ClusterIP, NodePort, LoadBalancer, and Ingress).

## Solution:

The following steps were used to complete the lab.

## 1. Minikube Configuration

Minikube initialization was performed using the command:
```bash
minikube start --driver=docker
```
<p align="center">
  <img width="892" height="328" alt="image" src="https://github.com/user-attachments/assets/fd0c7db5-fd80-419c-83df-a0dfa2ccd2ac" />
</p>

Subsequently, the cluster status was viewed using:
```bash
minikube status
```
<p align="center">
  <img width="466" height="164" alt="image" src="https://github.com/user-attachments/assets/06e411d9-fd4e-43ff-bcca-eb7cfe13441d" />
</p>

Finally, the cluster IP address was verified.

<p align="center">
  <img width="366" height="72" alt="image" src="https://github.com/user-attachments/assets/c3492c55-83a4-4225-a36c-b05b123464f3" />
</p>

The list of services was also displayed using:
```bash
minikube service list
```

Additionally, the cluster dashboard was visualized.

<p align="center">
  <img width="798" height="399" alt="image" src="https://github.com/user-attachments/assets/5b8dd84c-4e3f-4314-a6ad-9eada7373225" />
</p>

```bash
minikube dashboard
```
<p align="center">
  <img width="1906" height="616" alt="image" src="https://github.com/user-attachments/assets/f6c06f33-804a-45ea-8c1d-b0c0d005d964" />
</p>

***Dashboard after completing the lab***

On the other hand, the nodes that consume the most memory were discovered using:
```bash
kubectl top nodes
```
<p align="center">
  <img width="567" height="97" alt="image" src="https://github.com/user-attachments/assets/ab0682f1-74c3-49b8-b983-06fbea087424" />
</p>

## 2. Creating Deployments and Services

**Definitions:**

- **port**: Port exposed by the service within the cluster (where other pods/services connect to).
- **targetPort**: Port where the application inside the container/pod actually listens.
- **NodePort**: Port exposed on the node, accessible externally (range: 30000-32767).

### 2.1 ClusterIP (nginx)

For the ClusterIP service type, the nginx image was used.

First, the deployment was created with 2 replicas using:
```bash
kubectl create deployment app-clusterip --image=nginx --replicas=2
```

Subsequently, a service was created to expose the deployment, specifying its type as ClusterIP:
```bash
kubectl expose deployment app-clusterip --port=80 --target-port=80 --type=ClusterIP --name=svc-clusterip
```

ClusterIP services can only be accessed by pods inside the cluster. Therefore, an ephemeral pod was created inside the cluster to access the service through the target-port (80):
```bash
kubectl run curl --image=curlimages/curl -it --rm --restart=Never -- \
  curl http://svc-clusterip:80
```

After executing it, we can see the nginx web page view.

<p align="center">
  <img width="926" height="571" alt="image" src="https://github.com/user-attachments/assets/11502629-e409-4ea9-8c3e-44afe4fff33e" />
</p>

### 2.2 LoadBalancer (tomcat)

For the LoadBalancer case, the tomcat image was used.

In this case, the deployment was again created with 2 replicas using:
```bash
kubectl create deployment app-loadbalancer --image=tomcat --replicas=2
```

When exposing the deployment, the service type (LoadBalancer) is specified:
```bash
kubectl expose deployment app-loadbalancer --port=80 --target-port=8080 --type=LoadBalancer --name=svc-loadbalancer
```

Since there is no cloud provider, a tunnel was created to access the service through localhost.

<p align="center">
  <img width="1059" height="216" alt="image" src="https://github.com/user-attachments/assets/4da38549-f51f-4455-9de3-e5336130766d" />
</p>

<p align="center">
  <img width="827" height="141" alt="image" src="https://github.com/user-attachments/assets/ca12ef3b-3341-4b37-8783-1a9cda5ce576" />
</p>

<p align="center">
  <img width="1002" height="311" alt="image" src="https://github.com/user-attachments/assets/41de8304-278b-4b3d-bdbe-a03d1697d783" />
</p>

***The unmodified tomcat image returns 404***

### 2.3 NodePort (httpd)

For the NodePort case, the httpd image was used.

The deployment was created with 2 replicas using:
```bash
kubectl create deployment app-nodeport --image=httpd --replicas=2
```

Then, the deployment was exposed and the service type (NodePort) was specified:
```bash
kubectl expose deployment app-nodeport --port=80 --target-port=80 --type=NodePort --name=svc-nodeport 
```

We can see that the port available for access by other nodes was auto-generated (30150).

<p align="center">
  <img width="833" height="129" alt="image" src="https://github.com/user-attachments/assets/4cb7c091-63ed-4d53-a62d-01f39a3f661a" />
</p>

The IP address provided by minikube (minikube ip) + that port will be used to access the service.

<p align="center">
  <img width="494" height="65" alt="image" src="https://github.com/user-attachments/assets/09196755-34a2-4fd5-8f0f-18f71a662a18" />
</p>

### 2.4 Ingress (Caddy)

For the Ingress case, the Caddy image was used.

In this case, a similar process is followed. The deployment was created with 2 replicas using:
```bash
kubectl create deployment app-ingress --image=caddy --replicas=2
```

The deployment was exposed. However, the service type specified was ClusterIP. As seen earlier, it can normally only be accessed by pods inside the cluster, but this will change with the help of Ingress:
```bash
kubectl expose deployment app-ingress --port=80 --target-port=80 --type=ClusterIP --name=svc-ingress
```

After that, a `myapp-ingress.yaml` file was created to define the Ingress. It is shown below:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-http-echo
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-ingress
            port:
              number: 80
```

In this YAML, the following is defined:

- The Ingress name: `ingress-http-echo`
- The Ingress class name: `nginx` (refers to the NGINX Ingress Controller)
- The service the Ingress points to: `svc-ingress` (previously exposed).
- The Service port to which Ingress routes traffic: `port: number 80` (port of the svc-ingress service).

Subsequently, the configurations from the YAML are applied using:
```bash
kubectl apply -f myapp-ingress.yaml
```

After this, the Ingresses present in the cluster can be viewed using:
```bash
kubectl get ingress
```

The Ingress IP address takes around a minute to be assigned. In this case, the one corresponding to the cluster IP (minikube ip) was assigned.

<p align="center">
  <img width="626" height="67" alt="image" src="https://github.com/user-attachments/assets/ce60ef0d-a247-4540-a452-b076bb7ed72e" />
</p>

Finally, this IP is used to access the Caddy service.

<p align="center">
  <img width="802" height="308" alt="image" src="https://github.com/user-attachments/assets/beb7a43e-8bd3-4c7c-a979-71b9963dd26e" />
</p>

We can see that it can be accessed even though Caddy is running on a ClusterIP service type.

## 3. Visualization

We can see the description of the services as well as the endpoints created during the lab.

**Service descriptions:**

***svc-clusterip***

<p align="center">
  <img width="629" height="371" alt="image" src="https://github.com/user-attachments/assets/4a997ebf-9447-4022-80a7-02a539ca04ab" />
</p>

***svc-loadbalancer***

<p align="center">
  <img width="687" height="441" alt="image" src="https://github.com/user-attachments/assets/0e0abc43-e6df-40ec-a301-1dd629be20bb" />
</p>

***svc-nodeport***

<p align="center">
  <img width="613" height="420" alt="image" src="https://github.com/user-attachments/assets/e1ca4f00-450e-4f78-885e-b54c4e9349cf" />
</p>

***svc-ingress***

<p align="center">
  <img width="615" height="377" alt="image" src="https://github.com/user-attachments/assets/36dbe161-c237-46b2-b683-9d3d71655711" />
</p>

**Endpoints:**

<p align="center">
  <img width="820" height="193" alt="image" src="https://github.com/user-attachments/assets/6888792b-d747-4511-b34e-00e7a0fd1e5d" />
</p>

We can also see a general status of the deployments and services from the web interface.

**Deployments:**

<p align="center">
  <img width="1627" height="671" alt="image" src="https://github.com/user-attachments/assets/6402708c-b19d-4c82-bd69-e3c6e0421545" />
</p>

**Services:**

<p align="center">
  <img width="1629" height="360" alt="image" src="https://github.com/user-attachments/assets/73a2f65e-bc76-4767-b141-75754a29e912" />
</p>
