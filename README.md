# Introduction

This is cut and paste from official Flink docs for a Standalone Session cluster that you can spin up in Kind/Minikube/microk8s.

Office docs [here](https://ci.apache.org/projects/flink/flink-docs-release-1.12/deployment/resource-providers/standalone/kubernetes.html#starting-a-kubernetes-cluster-session-mode).

# Deploy 

Set your kube context, then:

```
kubectl apply -f k8s/
```

# Destroy

Set your kube context, then:

```
kubectl delete -f k8s/
```

