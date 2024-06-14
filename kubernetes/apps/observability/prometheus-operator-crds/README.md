# prometheus-operator-crds

This is a dirty hack so the CRDs receive the proper metadata and labels.

Prior to applying Prometheus CRDs, delete all the existing CRDs.

```sh
kubectl delete crd podmonitors.monitoring.coreos.com \
               scrapeconfigs.monitoring.coreos.com \
               prometheusrules.monitoring.coreos.com \
               servicemonitors.monitoring.coreos.com \
               probes.monitoring.coreos.com \
               prometheuses.monitoring.coreos.com \
               thanosrulers.monitoring.coreos.com \
               alertmanagers.monitoring.coreos.com \
               prometheusagents.monitoring.coreos.com \
               alertmanagerconfigs.monitoring.coreos.com
```

In certain circumstances the `HelmRelease` must also be deleted., for example during a re-deploy.

```sh
kubectl delete -n observability helmreleases.helm.toolkit.fluxcd.io prometheus-operator-crds
```
