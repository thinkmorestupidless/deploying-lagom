# Deploying Lagom on Minikube

## Start Minikube
```
# start minikube
minikube start --cpus 4 --memory 8192 --kubernetes-version=1.15.6

# point docker to minikube's internal registry
eval $(minikube docker-env)
```

## Build the Lagom Services as Docker images

### With SBT
```
cd lagom
sbt docker:publishLocal
```

### With Maven
```
cd lagom
mvn package docker:build
```

## Deploy Strimzi
Follow instructions at https://strimzi.io/quickstarts/minikube/

```
# create a namespace for kafka
kubectl create namespace kafka

# install strimzi
curl -L https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.16.0/strimzi-cluster-operator-0.16.0.yaml \
  | sed 's/namespace: .*/namespace: kafka/' \
  | kubectl apply -f - -n kafka 

# create a single-broker kafka cluster
kubectl apply -f https://raw.githubusercontent.com/strimzi/strimzi-kafka-operator/0.16.0/examples/kafka/kafka-persistent-single.yaml -n kafka 

```

## Deploy Cassandra
There's no operator for Cassandra so use StatefulSets (this can take a while to bring up all 3 nodes)
```
kubectl apply -f deploy/cassandra
```

## Deploy the Lagom Services
```
# create the rbac rules that allow akka pods to find each other and cluster
# creates the deployment for the 'hello' service and a kubernetes service to expose it
kubectl apply -f deploy/lagom
```