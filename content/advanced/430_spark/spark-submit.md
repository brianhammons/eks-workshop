---
title: "Deploy Spark-submit Job"
weight: 10
draft: false
---

In this section, we will deploy a schedule a series of Spark Driver and Executor pods to perform the Spark operations via spark-submit. By using spark-submit CLI, you can submit Spark jobs with various configuration options supported by Kubernetes.

#### Create folder for spark application resources

```
mkdir  ~/environment/spark_application/
```

#### Install Spark Kubernetes RBAC

Create Spark service account and cluster role binding:
```
kubectl create serviceaccount spark
kubectl create clusterrolebinding spark-role --clusterrole='edit' --serviceaccount=default:spark --namespace=default
```

#### Create pod template for Spark Drivers
```
cat <<EoF > ~/environment/spark_application/spark-driver-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels: 
    spark-role: driver
  namespace: default
spec:
  serviceAccountName: spark
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: lifecycle
            operator: In
            values:
            - OnDemand
EoF
```

#### Create pod template for Spark Executors
```
cat <<EoF > ~/environment/spark_application/spark-executor-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels: 
    spark-role: executor
  namespace: default
spec:
  serviceAccountName: spark
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: lifecycle
            operator: In
            values:
            - Ec2Spot
EoF
```

#### Launch Spark Application
Download [Spark](https://spark.apache.org/downloads.html) 

Download hadoop-aws JAR and aws-java-sdk-bundle jars from [Maven](https://mvnrepository.com/) repository. Copy them under the jars folder of the spark download.

Store Environment Variables and Create S3 Bucket:
```
export S3_LOGS_BUCKET=spark-logs-$(cat /dev/urandom | LC_ALL=C tr -dc "[:alpha:]" | tr '[:upper:]' '[:lower:]' | head -c 32)
aws s3 mb s3://$S3_LOGS_BUCKET

kubectl cluster-info
export KUBEAPI=<CONTROL PLANE ENDPOINT>
```

Deploy the Spark application:
```
bin/spark-submit \
--master $KUBEAPI \
--deploy-mode cluster \
--name 'Job Name' \
--conf spark.eventLog.dir=$S3_LOGS_BUCKET \
--conf spark.eventLog.enabled=true \
--conf spark.history.fs.inProgressOptimization.enabled=true \
--conf spark.history.fs.update.interval=5s \
--conf spark.kubernetes.container.image=<<Spark Docker Image>> \
--conf spark.kubernetes.container.image.pullPolicy=IfNotAvailable \
--conf spark.kubernetes.driver.podTemplateFile='~/environment/spark_application/driver_pod_template.yaml' \
--conf spark.kubernetes.executor.podTemplateFile='~/environment/spark_application/executor_pod_template.yaml' \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
--conf spark.dynamicAllocation.enabled=true \
--conf spark.dynamicAllocation.shuffleTracking.enabled=true \
--conf spark.dynamicAllocation.maxExecutors=100 \
--conf spark.dynamicAllocation.executorAllocationRatio=0.33 \
--conf spark.dynamicAllocation.sustainedSchedulerBacklogTimeout=30 \
--conf spark.dynamicAllocation.executorIdleTimeout=60s
--conf spark.driver.memory=8g \
--conf spark.kubernetes.driver.request.cores=2 \
--conf spark.kubernetes.driver.limit.cores=4 \
--conf spark.executor.memory=8g \
--conf spark.kubernetes.executor.request.cores=2 \
--conf spark.kubernetes.executor.limit.cores=4 \
--conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
--conf spark.hadoop.fs.s3a.connection.ssl.enabled=false \
--conf spark.hadoop.fs.s3a.fast.upload=true \
s3a://$S3_LOGS_BUCKET/script.py \
s3a://$S3_LOGS_BUCKEToutput
```

#### Check Status of Spark Application

Spark Event logs are redirected to S3, but you can use the Spark History Server to analyze the logs or can leverage services like CloudWatch Container Insights or Prometheus for deeper analysis.
