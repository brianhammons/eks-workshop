---
title: "Module 2: Deploy Spark Operator to Launch Spark Application"
weight: 20
draft: false
---

In this section, we will use helm to deploy a Spark operator and leverage this to deploy a sample Spark application.

#### Install Spark Operator

```
helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
kubectl create namespace spark-operator
helm install incubator/sparkoperator --generate-name --namespace spark-operator --set sparkJobNamespace=default --set enableWebhook=true --set operatorVersion=v1beta2-1.1.2-3.0.0
```

##### Output
```
NAME: sparkoperator-1604378070
LAST DEPLOYED: Mon Nov  2 20:34:31 2020
NAMESPACE: spark-operator
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

#### Install Spark Kubernetes RBAC

Create Spark service account and cluster role binding:
```
kubectl create serviceaccount spark
kubectl create clusterrolebinding spark-role --clusterrole='edit' --serviceaccount=default:spark --namespace=default
```

#### Configure Operator YAML for Spark Application

Deploy the following spark application using the spark operator:

```
cat <<EoF > environment/spark_application/spark-pi.yaml
apiVersion: "sparkoperator.k8s.io/v1beta2"
kind: SparkApplication
metadata:
  name: spark-pi
  namespace: default
spec:
  type: Scala
  mode: cluster
  image: "gcr.io/spark-operator/spark-operator:v1beta2-1.1.2-3.0.0"
  imagePullPolicy: Always
  mainClass: org.apache.spark.examples.SparkPi
  mainApplicationFile: "local:///opt/spark/examples/jars/spark-examples_2.12-3.0.0.jar"
  sparkConf:
    "spark.kubernetes.local.dirs.tmpfs": "true"
    "spark.kubernetes.authenticate.driver.serviceAccountName": spark
  sparkVersion: "3.0.0"
  restartPolicy:
    type: Never
  driver:
    cores: 1
    memory: "512m"
    serviceAccount: spark
  executor:
    cores: 1
    instances: 2
    memory: "512m"
EoF
```

#### Launch Spark Application

Deploy the Spark application:
```
kubectl apply -f ~/environment/spark_application/spark-pi.yaml
```

#### Check Status of Spark Application

Retrieve name of Spark application:
```
kubectl get SparkApplication
```

Check status:
```
kubectl describe SparkApplication spark-pi
```

Review Spark application logs:
```
kubectl logs spark-pi-driver

Accessing Driver UI
```
kubectl port-forward spark-pi-driver 4040:4040
```