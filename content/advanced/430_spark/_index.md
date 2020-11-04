---
title: "Spark on Amazon EKS"
weight: 430
draft: false
tags:
  - advanced
  - spark
  - spark-operator
---

Apache Spark is an open-source, distributed processing system for large-scale computing tasks such as big data processing, machine learning, real-time streaming, and ETL workloads. The Kubernetes ability to handle workloads at scale, and complex scheduling requirements makes it a highly desireable method of orchestrating Spark jobs, in comparison to YARN. 

Using Amazon EKS for running Spark jobs provides benefits for the following type of use cases:

* Workloads with high availability requirements. The Amazon EKS control plane is deployed in multiple availability zones, as can be the Amazon EKS worker nodes.
* Multi-tenant environments providing isolation between workloads and optimizing resource consumption. Amazon EKS supports Kubernetes Namespaces, Network Policies, Quotas, Pods Priority, and Preemption.
* Development environment using Docker and existing workloads in Kubernetes. Spark on Kubernetes provides a unified approach between big data and already containerized workloads.
* Focus on application development without worrying about sizing and configuring clusters. Amazon EKS is fully managed and simplifies the maintenance of clusters including Kubernetes version upgrades and security patches.
* Spiky workloads with fast autoscaling response time. Amazon EKS supports Kubernetes Cluster Autoscaler and can provides additional compute capacity in minutes.

In this chapter, we will walk through the process of deploying a spark application on EKS leveraging spark-submit and Spark operator capabilities.

### AWS Blogs on Spark

[Deploying Spark jobs on Amazon EKS](https://aws.amazon.com/blogs/opensource/deploying-spark-jobs-on-amazon-eks/)

[Optimizing Spark Performance on Kubernetes](https://aws.amazon.com/blogs/containers/optimizing-spark-performance-on-kubernetes/)
