# Canary Deployment for Safer Kubernetes Updates

# Problem Statement:

An organization running a cloud-native application on Kubernetes is looking to improve its deployment process to reduce the risk of downtime and application errors during updates. The current approach uses full, immediate rollouts, which can lead to issues like service outages, bugs in the new version impacting all users, and a lack of quick rollback options. The company needs a more reliable deployment strategy that allows them to gradually introduce changes, monitor the new version's performance, and minimize the impact of potential issues on end-users.

The organization seeks to implement a Canary Deployment strategy on Kubernetes to incrementally release new application versions to a small subset of users. This approach will allow the team to gather feedback on the new version's stability and performance while ensuring the majority of users continue to interact with the stable version. The goal is to significantly reduce downtime and improve the ability to quickly roll back or fix issues if they arise during deployment.

This project aims to design and deploy a Canary Deployment pipeline within the organization’s Kubernetes cluster, leveraging advanced traffic routing to ensure a smooth and controlled application update process.


## So what is Canary Deployment? 
A canary deployment strategy is a strategy that rolls out a new version of your application to a small group of users, while the majority of users continue the old version of the application. If the new version works as expected, it gradually replaces the old version.


# Steps for Implementing Canary Deployment

# 1. Set Up a Kubernetes Cluster
You can set up your cluster using the following;
- Minikube
- Docker Desktop
- Kind (Kubernetes IN Docker)
- EKS

# 2. Create your main deployment and service (V1)
This is the main deployment of your application with the service that will be used to route to it.

```
echo "
---
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production
  labels:
    app: production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: production
  template:
    metadata:
      labels:
        app: production
    spec:
      containers:
      - name: production
        image: registry.k8s.io/ingress-nginx/e2e-test-echo@sha256:6fc5aa2994c86575975bb20a5203651207029a0d28e3f491d8a127d08baadab4
        ports:
        - containerPort: 80
      
---
# Service
apiVersion: v1
kind: Service
metadata:
  name: production
  labels:
    app: production
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: production
" | kubectl apply -f -

```


# 3. Create the canary deployment and service (V2)
This canary deployment will take a weighted amount of requests instead of the main deployment.

```
echo "
---
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: canary
  labels:
    app: canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: canary
  template:
    metadata:
      labels:
        app: canary
    spec:
      containers:
      - name: canary
        image: registry.k8s.io/ingress-nginx/e2e-test-echo@sha256:6fc5aa2994c86575975bb20a5203651207029a0d28e3f491d8a127d08baadab4
        ports:
        - containerPort: 80
        
---
# Service
apiVersion: v1
kind: Service
metadata:
  name: canary
  labels:
    app: canary
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: canary
" | kubectl apply -f -


```


# 4. Create Ingress Pointing To Your Main Deployment
Next, you will need to expose your main deployment with an ingress resource, note there are no canary-specific annotations on this ingress

```
echo "
---
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: production
  annotations:
spec:
  ingressClassName: nginx
  rules:
  - host: echo.prod.mydomain.com
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: production
            port:
              number: 80
" | kubectl apply -f -


```


# 5. Create Ingress Pointing To Your Canary Deployment
You will then create an Ingress that has the canary-specific configuration, please pay special notice of the following:
- The hostname is identical to the main ingress hostname
- The nginx.ingress.kubernetes.io/canary: 'true' annotation is required and defines this as a canary annotation (if you do not have this the Ingresses will clash)
- The nginx.ingress.kubernetes.io/canary-weight: '10' annotation dictates the weight of the routing, in this case, there is a "10%" chance a request will hit the canary deployment over the main deployment

```
echo "
---
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: canary
  annotations:
    nginx.ingress.kubernetes.io/canary: \'true\'
    nginx.ingress.kubernetes.io/canary-weight: \'10\'
spec:
  ingressClassName: nginx
  rules:
  - host: echo.prod.mydomain.com
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: canary
            port:
              number: 80
" | kubectl apply -f -
```



# 6. Test your setup
You can use the following command to test your setup (replacing INGRESS_CONTROLLER_IP with your ingress controller IP Address)

![image](https://github.com/user-attachments/assets/f0c5c49b-0781-4da9-97ab-47f2c1d1cebc)


As you can see above, 9 requests went to production and 1 request went to canary, this shows that the canary setup is working as expected.













