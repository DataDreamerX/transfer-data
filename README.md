#!/bin/bash

# Get the pod name
pod_name=$(kubectl get pods -o jsonpath='{.items[0].metadata.name}')

# Run port forwarding
kubectl port-forward $pod_name 80:8888
