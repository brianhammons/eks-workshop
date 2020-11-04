---
title: "Cleanup"
weight: 40
draft: false
---

#### Cleanup
```
kubectl delete -f environment/spark_application/spark-pi.yaml
kubectl delete -f environment/spark_application/spark-rbac.yaml 
helm uninstall -n=spark-operator $(helm list -n spark-operator | sed 's/\ / /' | awk 'NR==2{print $1}')
```