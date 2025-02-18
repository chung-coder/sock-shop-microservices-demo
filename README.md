# Sock Shop : A Microservice Demo Application

The application is the user-facing part of an online shop that sells socks. It is intended to aid the demonstration and testing of microservice and cloud native technologies.

It is built using [Spring Boot](http://projects.spring.io/spring-boot/), [Go kit](http://gokit.io) and [Node.js](https://nodejs.org/) and is packaged in Docker containers.

You can read more about the [application design](./internal-docs/design.md).

## Deployment Platforms

The [deploy folder](./deploy/) contains scripts and instructions to provision the application onto your favourite platform. 

Please let us know if there is a platform that you would like to see supported.

## AWS EKS Deployment
This project demonstrates the deployment of the **Sock Shop Microservices Application** on **AWS EKS** (Elastic Kubernetes Service). The application is deployed using Kubernetes manifests and leverages AWS cloud-native services.

### 1. Set Up EKS Cluster
```sh
aws eks create-cluster --name sock-shop-cluster --region us-east-1 \
  --kubernetes-version 1.24 \
  --role-arn arn:aws:iam::YOUR_ACCOUNT_ID:role/EKS-Cluster-Role \
  --resources-vpc-config subnetIds=subnet-XXXXX,subnet-XXXXX,securityGroupIds=sg-XXXXX
```

### 2. Deploy Sock Shop Microservices
```sh
git clone https://github.com/microservices-demo/microservices-demo.git
cd microservices-demo/deploy/kubernetes
kubectl apply -f complete-demo.yaml
```

### 3. Deploy Monitoring (Prometheus & Grafana)
```sh
kubectl create namespace monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
```

### 4. Configure Horizontal Pod Autoscaling (HPA)
Configured Horizontal Pod Autoscaling (HPA) for each deployment, specifying the following parameters for all services:
- minReplicas: 1
- maxReplicas: 10
- targetCPUUtilizationPercentage: 50%
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: front-end
  namespace: sock-shop
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: front-end
    
minReplicas: 1
maxReplicas: 10
targetCPUUtilizationPercentage: 50
```
Apply it using:
```sh
kubectl apply -f hpa.yaml
```

### 5. Load Testing with Locust
Deploy a load testing application using Locust:
```sh
kubectl create namespace loadtest
kubectl apply -f load-test.yaml
```

Monitor HPA behavior using:
```sh
kubectl get hpa -w
```

## Screenshot

![Sock Shop frontend](https://github.com/microservices-demo/microservices-demo.github.io/raw/master/assets/sockshop-frontend.png)

## Visualizing the application

Use [Weave Scope](http://weave.works/products/weave-scope/) or [Weave Cloud](http://cloud.weave.works/) to visualize the application once it's running in the selected [target platform](./deploy/).

![Sock Shop in Weave Scope](https://github.com/microservices-demo/microservices-demo.github.io/raw/master/assets/sockshop-scope.png)

## Results
- **Successfully Deployed on AWS EKS**: Set up and configured an EKS cluster to run the Sock Shop microservices.
- **Implemented Monitoring**: Integrated Prometheus and Grafana to track key performance metrics and resource utilization.
- **Configured Auto-Scaling**: Used Kubernetes Horizontal Pod Autoscaler (HPA) to dynamically adjust pod counts based on CPU utilization.
- **Load Testing & Optimization**: Conducted stress testing with Locust to evaluate application performance and resource scaling.
- **Identified Resource-Intensive Services**: Determined that 'front-end', 'cart', and 'order' services consumed the most resources, allowing for better optimization strategies.
- **Scaling Observations**: The HPA successfully adjusted the number of pods based on real-time CPU utilization.
- **Performance**: The system maintained stability under increased load, and unnecessary resource consumption was avoided during low-traffic periods.
- **Resilience**: Liveness and readiness probes ensured fault tolerance and prevented requests from being sent to unhealthy pods.

## Demo Video
[Watch Deployment and Scaling in Action](https://drive.google.com/file/d/1m1Z0s1c5Nf_IU6GKFUXZa5LNjT8PpAiQ/view)

## Conclusion
This project demonstrates a **scalable, reliable, and cloud-native** deployment of a microservices-based e-commerce application on **AWS EKS**. By integrating monitoring, autoscaling, and load testing, we ensure **high availability and efficient resource utilization**.
