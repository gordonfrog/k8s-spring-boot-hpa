# Autoscaling Spring Boot with the Horizontal Pod Autoscaler and custom metrics on Kubernetes



## Run on Local

brew install activemq
activemq start
Login to http://localhost:8161/admin/topics.jsp with admin/admin
Use the following settings, then run spring boot app:
queueName = "mainQueue"
workerName = "worker1"
workerEnabled = true
brokerUrl = "tcp://localhost:61616"

## Prerequisites

You should have minikube installed.

You should start minikube with at least 4GB of RAM:

```bash
minikube start \
  --memory 4096 \
  --extra-config=controller-manager.horizontal-pod-autoscaler-upscale-delay=1m \
  --extra-config=controller-manager.horizontal-pod-autoscaler-downscale-delay=2m \
  --extra-config=controller-manager.horizontal-pod-autoscaler-sync-period=10s
```

## Package the application for K8s

You package the application as a container with:

```bash
eval $(minikube docker-env)
docker build -t spring-boot-hpa .
```

## Deploying the application to K8s

docker push iamwill/spring-k8s-hpa:v1

Deploy ActiveMQ:

Create activemq-deployment.yaml

Create activemq-service.yaml

kubectl create -f activemq-deployment.yaml

kubectl create -f activemq-service.yaml

kubectl get pods -l=app=queue

Create Frontend:

Create fe-deployment.yaml

Create fe-service.yaml

kubectl create -f fe-deployment.yaml

kubectl create -f fe-service.yaml

kubectl get pods -l=app=frontend

Create Backend:

Create backend-deployment.yaml

Create backend-service.yaml

kubectl create -f backend-deployment.yaml

kubectl create -f backend-service.yaml

kubectl get pods -l=app=backend

minikube service frontend


## Autoscaling workers

You can scale the application in proportion to the number of messages in the queue with the Horizontal Pod Autoscaler. You can deploy the HPA with:

```bash
kubectl create -f kube/hpa.yaml
```

You can send more traffic to the application with:

```bash
while true; do sleep 0.5; curl -s http://<minikube ip>:32000/submit; done
```

When the application can't cope with the number of icoming messages, the autoscaler increases the number of pods only every 3 minutes.

You may need to wait three minutes before you can see more pods joining the deployment with:

```bash
kubectl get pods
```

The autoscaler will remove pods from the deployment every 5 minutes.

You can inspect the event and triggers in the HPA with:

```bash
kubectl get hpa spring-boot-hpa
```

