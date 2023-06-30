kubectl get pods -n istio-system -l app=istiod -o jsonpath='{.items[0].metadata.labels.istio\.io/rev}'
