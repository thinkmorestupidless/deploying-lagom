# Deploying Lagom on Minikube

## Start Minikube
```
# start minikube
minikube start --cpus 4 --memory 8192 --kubernetes-version=1.15.6

# point docker to minikube's internal registry
eval $(minikube docker-env)
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
There's no operator for Cassandra so use StatefulSets
```
kubectl apply -f deploy/cassandra
```

## Build the Lagom Services
```
cd lagom
sbt runAll
```

## Deploy the Lagom Services
```
kubectl apply -f deploy/lagom
```