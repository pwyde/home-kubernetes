# prometheus-operator-crds

Prior to applying Prometheus CRDs, delete the existing CRDs.

```sh
kubectl delete crd podmonitors.monitoring.coreos.com \
               scrapeconfigs.monitoring.coreos.com \
               prometheusrules.monitoring.coreos.com \
               servicemonitors.monitoring.coreos.com
```
This is a dirty hack so the CRDs receive the proper metadata and labels.
