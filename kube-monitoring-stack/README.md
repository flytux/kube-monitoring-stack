### Install monitoring chart 


```
# need default storage class for sts

helm upgrade -i k8s-monitoring . -n monitoring --create-namespace
```
