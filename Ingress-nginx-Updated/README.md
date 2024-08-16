# Custom Nginx Ingress Deployment

This guide outlines the steps taken to deploy a custom `nginx-ingress` using Helm in a local Kubernetes environment on Ubuntu. The deployment was named `infra-nginx-ingress-trial` and configured to expose ingresses with annotation and without annotation.

## Prerequisites

- Kubernetes cluster running locally (I used Minikube)
- Helm installed
- Kubernetes CLI (`kubectl`) installed

![Screenshot 2024-08-16 144100](https://github.com/user-attachments/assets/16a7357f-3985-4027-bacb-dd35be1abcd2)


## Step-by-Step Deployment

### Step 1: Clone the Repository

Ensure that you have the repository cloned locally.

~~~
git clone https://github.com/SomeshRao007/Helm-nginx-ingress.git
cd ingress-nginx
~~~

### Step 2: Deploy Custom Nginx Ingress with Helm

Deploy the custom nginx-ingress with custom values:

~~~
helm install my-nginx-ingress . -f custom-values.yaml
~~~

### Step 3: Create a Deployment and Service for Testing

Create a basic deployment and service for a simple web server.

Create `deployment.yaml` 

~~~
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
  labels:
    app: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx:latest
        ports:
        - containerPort: 80
~~~

Create `service.yaml`

~~~
apiVersion: v1
kind: Service
metadata:
  name: web-server
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
~~~

Apply the Deployment and Service

~~~
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
~~~

### Step 4: Create Ingress Resources

Create two ingress resources: one with a specific annotation and another without.

Create `ingress-with-annotation.yaml`

~~~
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress-with-annotation
  annotations:
    custom-annotation: "true"
spec:
  ingressClassName: nginx
  rules:
  - host: web-with-annotation.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-server
            port:
              number: 80
~~~

Create `ingress-without-annotation.yaml`

~~~
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress-without-annotation
spec:
  ingressClassName: nginx
  rules:
  - host: web-without-annotation.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-server
            port:
              number: 80
~~~


Create `ingress-class.yaml`

~~~
---
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  labels:
    app.kubernetes.io/component: controller
  name: nginx
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: k8s.io/ingress-nginx
~~~


Apply the Ingress Resources

~~~
kubectl apply -f ingress-with-annotation.yaml
kubectl apply -f ingress-without-annotation.yaml
kubectl apply -f ingress-class.yaml
~~~

### Step 5: Verify Ingress Functionality


I have made a few modifications in `/etc/hosts` File

Edit the /etc/hosts file to map the hostnames to your cluster's IP:

~~~
sudo nano /etc/hosts
~~~


YOUR_CLUSTER_IP web-with-annotation.local

YOUR_CLUSTER_IP web-without-annotation.local

~~~
127.0.0.1       localhost
127.0.1.1       somesh-virtual-machine

192.168.49.2 web-with-annotation.local
192.168.49.2 web-without-annotation.local

10.244.0.7.default.pod.web-server.local
~~~

VERY IMP CHECK [LINK](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pods)

### Access the Ingress Resources

Try accessing the ingress resources using curl:

- curl http://web-with-annotation.local
- curl http://web-without-annotation.local


## Error Encountered

During the deployment and verification process, several issues were encountered and resolved as described below:


#### 1. Missing IngressClass Error

The logs indicated that the ingress resources did not contain a valid IngressClass: `ingress does not contain a valid IngressClass`

Solution:

Created an IngressClass resource as shown above and added spec in both the yaml files.

#### 2. Connection Refused Error

When trying to connect to the ingress resources, the following error was encountered:

`curl: (7) Failed to connect to web-with-annotation.local port 80 after 0 ms: Connection refused`

[link](https://stackoverflow.com/questions/65725124/connection-refused-between-kubernetes-pods-in-the-same-cluster)

[link2](https://discuss.kubernetes.io/t/unable-to-access-the-clusterip-service-or-internet-within-a-pod-by-using-servicename-or-domainname/19781)

![image](https://github.com/user-attachments/assets/5a3cc211-58a8-4c65-aee0-dc01b9304fef)


Solution:

- Verified that the ingress controller pods were running and ready.
- Checked the service configuration of the ingress controller to ensure it was properly exposed with a NodePort or LoadBalancer.
- Modified the /etc/hosts file to map the hostnames correctly.
- Ensured there were no network policies blocking traffic.
- Restarted the ingress controller pods to apply changes.


#### 3. Default Backend - 404 Error

When accessing the web server service directly within a pod, the following error was encountered: `default backend - 404`

Solution:

- Verified the IP address and port of the web server pod using kubectl get pods -o wide.
- Ensured that the service and ingress resources were correctly configured.
- Checked the logs of the ingress controller pods for any error messages.
- Restarted the ingress controller pods to force reloading of the configuration.


## OUTPUT IMAGES:

![Screenshot 2024-08-16 144625](https://github.com/user-attachments/assets/51416e70-b256-444f-8f18-b4a12b332648)

![Screenshot 2024-08-16 145020](https://github.com/user-attachments/assets/e4016eb1-9a08-4c99-8b7c-9792a81cf1cf)

![Screenshot 2024-08-16 145537](https://github.com/user-attachments/assets/d8a72b4e-8a55-45ba-ac3d-380ea26311b8)

![Screenshot 2024-08-16 171657](https://github.com/user-attachments/assets/4cda9d7c-9bc3-444a-8e3c-1c5095448a8d)

### Complete system

![Screenshot 2024-08-16 173647](https://github.com/user-attachments/assets/aa405725-1360-4fa0-8f04-be610090978c)

![Screenshot 2024-08-16 173721](https://github.com/user-attachments/assets/de1e3dc2-00ac-4730-af9f-23807d543b93)

![Screenshot 2024-08-16 173729](https://github.com/user-attachments/assets/4dc985ef-d0db-4f97-a119-dbe7510be26e)

![Screenshot 2024-08-16 173837](https://github.com/user-attachments/assets/680ae328-89e0-4ddb-b2a3-48ae383641d8)

![Screenshot 2024-08-16 173907](https://github.com/user-attachments/assets/2a1114ea-3f27-4d28-a7c3-695b903c4b31)







