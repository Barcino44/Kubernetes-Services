# Kubernetes-Services

## Descripción:

El presente repositorio tiene como objetivo explicar la implementación de 4 deployments (Cada uno con imagenes diferentes) usando diferentes tipos de servicios de Kubernetes (ClusterIp, Nodeport, LoadBalancer e Ingress).

## Solución:

Para la realización del laboratorio, se emplearon los siguientes pasos:

## 1. Configuración de minikube

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
<img width="567" height="97" alt="image" src="https://github.com/user-attachments/assets/ab0682f1-74c3-49b8-b983-06fbea087424" />

## 2. Creación de deployments y services.

**Definiciones:**

•	port: Se refiere al puerto donde escucha la aplicación (Ngnix).
•	targetPort: Se refiere al puerto del service, por aquí se establece comunicación con otros pods de manera interna en el cluster.
•	NodePort: Puerto por donde escucha el nodo, accesible por otros nodos.

### 2.1 ClusterIP (ngnix)

Para el caso del tipo de servicio ClusterIP, se empleó la imagen de ngnix. 

En primera instancia, se creo el deployment con 2 replicas con ayuda de.

````
kubectl create deployment app-clusterip --image=nginx --replicas=2
````
Posteriormente, se creo un servicio para exponer dicho deployment especificando su tipo como ClusterIP.

````
kubectl expose deployment app-clusterip --port=80 --target-port=80 --type=ClusterIP --name=svc-clusterip
````
Los servicios de tipo ClusterIP unicamente pueden ser accedidos por pods al interior del cluster. Por tanto, fue creado un pod éfimero al interior del cluster para acceder al servicio a traves del target-port (80).

````
kubectl run curl --image=curlimages/curl -it --rm --restart=Never -- \
  curl http://svc-clusterip:80
````
Tras ejecutarlo, podemos apreciar la vista de la página web de ngnix.

<img width="926" height="571" alt="image" src="https://github.com/user-attachments/assets/11502629-e409-4ea9-8c3e-44afe4fff33e" />

### 2.2 Load Balancer (tomcat)

Ahora para el caso de LoadBalancer, se uso la imagen de tomcat.

En este caso, nuevamente el deployment fue creado con 2 replicas con ayuda de.

````
kubectl create deployment app-loadbalancer --image=tomcat --replicas=2
````
En la exposición del deployment, se especifica el tipo de servicio (LoadBalancer).
````
kubectl expose deployment app-loadbalancer --port=80 --target-port=8080 --type=LoadBalancer --name=svc-loadbalancer
````
Al no existir un proveedor de nube, fue creado un tunel para acceder al servicio a traves de mi localhost.

<img width="1059" height="216" alt="image" src="https://github.com/user-attachments/assets/4da38549-f51f-4455-9de3-e5336130766d" />

<img width="1002" height="311" alt="image" src="https://github.com/user-attachments/assets/41de8304-278b-4b3d-bdbe-a03d1697d783" />

***La imagen de tomcat sin modificaciones arroja 404***

### 2.3 Nodeport (httpd)

Para el caso de Nodeport, se uso la imagen de httpd.

Fue creado el deployment con 2 replicas con ayuda de.
````
kubectl create deployment app-nodeport --image=httpd --replicas=2
````
Luego, fue expuesto el deployment, y se especificó el tipo de servicio (Nodeport).

````
kubectl expose deployment app-nodeport --port=80 --target-port=80 --type=NodePort --name=svc-nodeport 
````
Vemos que el puerto disponible para el acceso de otros nodos fue auto generado (30150). 

<img width="833" height="129" alt="image" src="https://github.com/user-attachments/assets/4cb7c091-63ed-4d53-a62d-01f39a3f661a" />

Será usada la dirección ip del nodo maestro (minikube ip) + dicho puerto para acceder al servicio.

<img width="494" height="65" alt="image" src="https://github.com/user-attachments/assets/09196755-34a2-4fd5-8f0f-18f71a662a18" />

### 2.4 Ingress (Caddy)



