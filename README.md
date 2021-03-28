# Demo GRPC kubernetes local implementation
  The documentation is in line with the local minkube cluster implementation of a demo grpc employee-service.
  GRPC employee service with a rest api client @ [GRPC-Employee-Service](https://github.com/hitesh-sureify/grpc-demo)

### Pre-Requisite
- minikube must be installed
    - check version : minikube version
- kubectl must be installed
    - check version : kubectl version --short
- minikube version, kubectl client version and kubectl server version should be compatible to each other.
  if any command is not running/showing configuration as expected, please check and upgrade version of eithe kubectl or minikube before looking for solution.

### Step 1 : Creation of .yaml files
- employee-service.yaml, employee-api-service.yaml, mysql-depl.yaml and php-admin.yaml have 2 parts : Deployment component configuration and Service component configuration.
- employee-config-map.yaml : This file stores configuration for config map component.
- mysql-pod.yaml : This file stores configuration for pod component to run mysql application within its container. Need not to be used as mysql-depl.yaml already creates mysql pods.
- mysql-secret.yaml : This file stores configutation for secret component.

### Step 2 : Start minikube
- **minikube start [--vm-driver=<driver>]**
- docker can be used as a driver. Also docker can be set as default driver if other hypervisors are installed in machine : **minikube config set driver docker**

### Step 3 : Create components, in below order
- To create config map component : run command **kubectl create -f employee-config-map.yaml**
- To create secret component : run command **kubectl create -f mysql-secret.yaml**
- To create employee-service components : deployment, pod and service -> run command **kubectl create -f employee-service.yaml**
- To create employee-api-service components : deployment, pod and service -> run command **kubectl create -f employee-api-service.yaml**
- To create mysql components : deployment, pod and service -> run command **kubectl create -f mysql-depl.yaml**
**Note:** order needs to be maintained because some resources/components are dependent on availability of some other resources. Hence will fail to start if those resources are not found.

### employee-service.yaml
- This file contains two components' configuration : deployment and service. The configurations are separted by a '---'.
- The deployment configuration spins up pods that run application within them based on the image : **golang-grpc**.
- **golang-grpc** image is build from docker file : https://github.com/hitesh-sureify/grpc-demo/blob/main/Dockerfile 
- The image **golang-grpc** needs to be built in minikubes' docker service. Follow below steps to achieve this:
    - Set the environment variable with eval command **eval $(minikube docker-env)** in terminal.
    - build the docker file to create an image
    - set pod's **ImagePullPolicy** attribute to **Never**
- Service component masks the 'emp-srv-dpl' pod(s)' ips and exposes them to the cluster and also acts as a load balancer.
- The pods are exposed on 50052 port and service component redirects requests to the same port.
- This service component here is an internal service and cannot be accessed from outside the cluster.

### employee-api-service.yaml
- This file contains two components' configuration : deployment and service. The configurations are separted by a '---'.
- The deployment configuration spins up pods that run application within them based on the image : **employee-api**.
- **employee-api** image is build from docker file : https://github.com/hitesh-sureify/grpc-demo/blob/main/client/Dockerfile 
- The image **employee-api** needs to be built in minikubes' docker service. Follow below steps to achieve this:
    - Set the environment variable with eval command **eval $(minikube docker-env)** in terminal.
    - build the docker file to create an image
    - set pod's **ImagePullPolicy** attribute to **Never**
- Service component masks the 'emp-api-depl' pod(s)' ips and exposes them to the cluster and also acts as a load balancer.
- The pods are exposed on 8000 port and service component redirects requests to the same port.
- This service component here is an external service and is used to receive requests from outside the cluster.
- The external service is achieved by explicitly mentioning 'type' tag as 'NodePort' [for k8 cluster this would be 'loadBalancer'].
- For example, a GET request to this component will look like : http://<cluster-ip>:30080/api/employees/<employee-id>

### mysql-depl.yaml
- This file contains two components' configuration : deployment and service. The configurations are separted by a '---'.
- The deployment configuration spins up pods that run application within them based on the image : **mysql**.
- **mysql** image is pulled from dockerhub or use one if exists locally.
- Service component masks the 'emp-srv-dpl' pod(s)' ips and exposes them to the cluster and also acts as a load balancer.
- The pods are exposed on 50052 port and service component redirects requests to the same port.
- This service component here is an internal service and cannot be accessed from outside the cluster.

### employee-config-map.yaml
- This file contains configuration for configmap resource.
- This resource is responsible for containing environment variables.
- They are accessed in other component configuration like in employee-service.yaml. Then they are configured under 'env' tag in conjuction with 'configMapKeyRef' to be exposed as env vars.
- The pods created from employee-service.yaml access pods created from mysql-depl.yaml by using DSNString parts configured in configMap component's yaml under key 'host', 'driver' and 'dbName'.

### mysql-secret.yaml
- This file contains configuration for secret resource.
- This resource is responsible for containing credentials/certificate or any such data in base64 encoded format.
- They are accessed in other component configuration like in employee-service.yaml. Then they are configured under 'env' tag in conjuction with 'secretKeyRef' to be exposed as env vars with encoded base64 values.
- The pods created from employee-service.yaml access pods created from mysql-depl.yaml by using credentials configured in secret component's yaml under key 'password' and 'username'.

