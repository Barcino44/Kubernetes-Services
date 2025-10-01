# Kubernetes-Services

## Descripción:

El presente repositorio tiene como objetivo explicar la implementación de 4 deployments (Cada uno con imagenes diferentes) usando diferentes tipos de servicios de Kubernetes (ClusterIp, Nodeport, LoadBalancer e Ingress).

## Solución:

Para la realización del laboratorio, se emplearon los siguientes pasos:

### 1. Configuración de minikube.

La inicialización de minikube se realizó con ayuda del comando.
````
minikube start --driver=docker
````
Posteriormente, se visualizó el estado del cluster con ayuda de.
````
minikube status
````
<img width="466" height="164" alt="image" src="https://github.com/user-attachments/assets/06e411d9-fd4e-43ff-bcca-eb7cfe13441d" />

Finalmente, se verificó la dirección ip del cluster.

<img width="366" height="72" alt="image" src="https://github.com/user-attachments/assets/c3492c55-83a4-4225-a36c-b05b123464f3" />

También, se mostraron la lista de servicios. Lo anterior con ayuda de.

````
minikube service list
````
Además de la visualización del dashboard del cluster.

<img width="798" height="399" alt="image" src="https://github.com/user-attachments/assets/5b8dd84c-4e3f-4314-a6ad-9eada7373225" />

````
minikube dashboard
````
<img width="1906" height="616" alt="image" src="https://github.com/user-attachments/assets/f6c06f33-804a-45ea-8c1d-b0c0d005d964" />

***Dashboard tras finalizar el laboratorio***

Por otro lado, los nodos que más consumen memoria se decubrieron gracias a.

````
kubectl top nodes
````







