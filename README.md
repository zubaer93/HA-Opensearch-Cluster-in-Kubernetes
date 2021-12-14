# Opensearch Cluster in Kubernetes

Creating a 3 node(1 master, 2 data) highly-available Opensearch cluster in Kubernetes. We will create the cluster manually without using Helm.

## Prerequisite

1. Have to have access to a kubernets cluster (version < 1.22)
2. Kubectl installed
3. Docker installed

## Installation

### Cluster creation 

1. Run ```kubectl apply -f 1. master-node-deployment.yml```. This will create the master node. 
2. Run ```kubectl apply -f 2. service.yml``` to create the required services.
3. Now run ```kubectl apply -f 3. data-node-deployment.yml```. This will create 2 data nodes and now our 3 node cluster in up and running. To check this we can run ```kubectl port-forward service/opensearch 9200:9200``` to access the cluster from our browser. Visit [http://localhost:9200/](http://localhost:9200/) from browser 
  
  
      ![](/snapshots/cluster.png)
  
  
      We can also visit [http://localhost:9200/_cluster/health](http://localhost:9200/_cluster/health) to check the health of the cluster
  
  
      ![](/snapshots/cluster-health.png)
      
      
      Here we can see we have 3 nodes available and 2 of those are data nodes. We can always change the number of master or data nodes by changing the replica count from deployment file.
  
  
      **In our cluster we are not using tls certificates and the security modules are disabled. Therefore there is no https communication between nodes and also no authorization. Also there is no data persistence in the cluster. So this setup is not ideal for production usage**

### Dashboard creation

Now to visualize our data we need to install opensearch dashboard. Opensearch does not provide image for disabled security dashboard. So we have to create our own docker image for that.
      
1. Run ```cd no-security-dashboard-dockerfile && docker build --tag=opensearch-dashboards-no-security .``` to build our own opensearch dashboard docker image
2. Now run  ```cd .. && kubectl apply -f 4. dashboard.yml``` and our dashboard should be ready to use. To check this we can run ```kubectl port-forward service/os-dashboard 8080:80``` to access the dashboard from browser. Visit [http://localhost:8080/](http://localhost:8080/) from browser.
    
    
    ![](/snapshots/dashboard-port-forwarding.png)
    

### Ingress and Traefik

To properly route outside traffic to our cluster we have to implement Ingress. We will be using Traefik as an Ingress controller

1. Run ```kubectl apply -f 5. traefik-crd.yml``` to apply all the necessary CustomResourceDefinition for traefik
2. Now run ```kubectl apply -f 6. traefik.yml```, This will create a traefik pod in the cluster.
3. Finally run ```kubectl apply -f 7. ingress.yml``` to create the ingress. This will create a LoadBalancer type service which will provide us an EXTERNAL-IP.


    ![](/snapshots/services.png)
    
    
    We will use this external ip to access our cluster
    
    
    ![](/snapshots/dashboard.png)
    
## TODO

1. Enable data persistence by introducing StatefulSets
2. Add secured communication between nodes by enabling tls ecryption 
3. Add authorization by enabling security plugins
