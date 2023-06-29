# Get the pod name
$podName = kubectl get pods -o jsonpath='{.items[0].metadata.name}'

# Run port forwarding
kubectl port-forward $podName 80:8888

@echo off
powershell.exe -ExecutionPolicy Bypass -File "C:\path\to\port_forward.ps1"

